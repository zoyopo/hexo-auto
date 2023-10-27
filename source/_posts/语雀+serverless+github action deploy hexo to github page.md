tags: [serverless,github action,yuque,hexo]
categories: [前端]

---

## 前言
![yuque.jpeg](https://cdn.nlark.com/yuque/0/2020/jpeg/1512483/1599910009298-154d2a4f-db41-4e29-91fe-858ef7718868.jpeg#height=445&id=KVOk0&originHeight=445&originWidth=720&originalType=binary&ratio=1&rotation=0&showTitle=false&size=28716&status=done&style=none&title=&width=720)
语雀作为一个阿里提供的免费平台，在写作这方面用户体验没的说。但是我们想写的东西能够美美的展示出来，那就需要一个自己的网站。所以我们想要达到的效果就是，在语雀里写好了一篇文章，也会同样发布到我的网站上。


## 领一个免费的静态网站
## ![ghpage.jpg](https://cdn.nlark.com/yuque/0/2020/jpeg/1512483/1599910100268-41bcb526-a77d-49e3-a12a-18c745061add.jpeg#height=300&id=kHQYg&originHeight=300&originWidth=582&originalType=binary&ratio=1&rotation=0&showTitle=false&size=11399&status=done&style=none&title=&width=582)
那就不得不提大名鼎鼎的 `github page` 。顾名思义，它是github帮你生成的网站，怎么生成呢？很简单，建一个仓库存放你的博客项目，项目名 **有约定** ， **必须是** _username_**.github.io。然后往里面加一个index.html，就能作为你的博客主页进行访问啦。**


## 快速搭建一个优雅的静态博客
![hexo.jpeg](https://cdn.nlark.com/yuque/0/2020/jpeg/1512483/1599910161121-60c996da-df9f-4bbe-8b3c-e91db11e43cc.jpeg#height=282&id=zV2R1&originHeight=282&originWidth=540&originalType=binary&ratio=1&rotation=0&showTitle=false&size=129419&status=done&style=none&title=&width=540)
我使用的是一个名为 `hexo` 的搭建博客框架。应该算是比较流行的，部署简单，页面生成快，有很多大佬贡献的主题。
原理也很简单 在node环境下 将各种静态模板 通过模板引擎 转成页面文件。
可以通过 `yaml` 文件对网站进行各种配置。
这里就不多说，感兴趣的可以自行探索。（笔者也只是拿来主义）
官网地址 [hexo](https://hexo.io/)

我们先用官网的方式，生成一个hexo项目，命名为 `hexo-auto` 然后在github里创建此项目仓库，把本地的push上去。
## 搞一个云函数

说到这个，不得不提一下时下很火的 `serveless` 
> **The Serverless Framework** – Build applications comprised of microservices that run in response to events, auto-scale for you, and only charge you when they run. This lowers the total cost of maintaining your apps, enabling you to build more logic, faster.
> 构建由微服务组成的应用程序，这些服务可以响应事件运行，为您自动伸缩运行空间，并且只在它们运行时向您收费。这降低了维护应用程序的总成本，使您能够更快地构建更多逻辑。


这个框架，感觉就像流量日租卡，用时收费，也不要办啥额外套餐。所以它很适合一些专题活动场景，比如说各类节日活动对服务器的需求。
扯远了。
我用了一个名为 `SCF(serveless cloud function)` 的东东，中文叫做“云函数”，它干了一个什么事情呢？
![云函数.png](https://cdn.nlark.com/yuque/0/2020/png/1512483/1599905824664-2c42f834-9ab9-43e6-baf4-7f14d5e9a1c8.png#height=1184&id=U1biU&originHeight=1184&originWidth=2062&originalType=binary&ratio=1&rotation=0&showTitle=false&size=308926&status=done&style=none&title=&width=2062)
在服务端执行一段脚本，作为一个触发器触发github的action。触发我们先前就在 `hexo-auto` 项目中配置的自动部署脚本。
![github workflow.png](https://cdn.nlark.com/yuque/0/2020/png/1512483/1599907167448-8488de3c-dd7f-49cb-89c3-f5667c1ac150.png#height=732&id=zM9Zh&originHeight=732&originWidth=1040&originalType=binary&ratio=1&rotation=0&showTitle=false&size=79894&status=done&style=none&title=&width=1040)
```yaml
# workflow name name: Deploy To Github Pages # 当有 push到仓库和外部触发的时候就运行 
on: [push, repository_dispatch] 
# YUQUE_TOKEN # GIT_SSH_PRIVATE_KEY 
jobs: 
  deploy: 
    name: Deploy Hexo Public To Pages 
    runs-on: ubuntu-latest 
    env: 
      TZ: Asia/Shanghai 



    steps: 
      # check it to your workflow can access it # from: https://github.com/actions/checkout 
      - name: Checkout Repository master branch 
        uses: actions/checkout@master 
      # from: https://github.com/actions/setup-node 
      - name: Setup Node.js 10.x 
        uses: actions/setup-node@master 
        with: 
          node-version: "10.x" 
      # from https://github.com/x-cold/yuque-hexo  
      # npm 获取的hexo-cli或yuque-hexo有问题 generate不了 故用cnpm
      # 主动clone获取next主题 防止部署缺失出错
      - name: Setup Hexo Dependencies      
        run: |
          npm install -g cnpm --registry=https://registry.npm.taobao.org
          cnpm install hexo-cli -g 
          cnpm install yuque-hexo -g 
          cnpm install 
      - name: Setup Deploy Private Key 
        env: 
          HEXO_DEPLOY_PRIVATE_KEY: ${{ secrets.GIT_SSH_PRIVATE_KEY }} 
        run: |
          mkdir -p ~/.ssh/ 
          echo "$HEXO_DEPLOY_PRIVATE_KEY" > ~/.ssh/id_rsa 
          chmod 600 ~/.ssh/id_rsa 
          ssh-keyscan github.com >> ~/.ssh/known_hosts 
      - name: Setup Git Infomation 
        run: |
          git config --global user.name '694534942@qq.com' 
          git config --global user.email '694534942@qq.com'
      - name: Deploy Hexo 
        env: 
          YUQUE_TOKEN: ${{ secrets.YUQUE_TOKEN }} 
        run: |
          git clone https://github.com/theme-next/hexo-theme-next themes/next
          yuque-hexo clean 
          yuque-hexo sync
          hexo clean 
          hexo generate
          hexo d
```

里面的两个变量是从github项目配置里读取的。
![github yaml config.png](https://cdn.nlark.com/yuque/0/2020/png/1512483/1599906565502-36509499-0f0a-4414-b1b7-1d609f6a7540.png#height=846&id=CiDJS&originHeight=846&originWidth=2428&originalType=binary&ratio=1&rotation=0&showTitle=false&size=222968&status=done&style=none&title=&width=2428)
github这种类似自动部署的功能叫 `github action` ,上述过程我们只要触发此 `action` ,github会为我们分配一个linux环境去执行 `yaml` 中的脚本。

脚本里会执行hexo的部署命令，部署命令会执行 `hexo-auto`  `_config.yaml` 中的一段部署脚本
```yaml
# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git
  repo: git@github.com:godlikedeveloper/godlikedeveloper.github.io
  branch: master
```


简单来说就是部署到github的哪个库中，所以为了要能顺利部署,我们需要 `GIT_SSH_PRIVATE_KEY` 权限验证。
私钥是你本地之前生成的。
![私钥.png](https://cdn.nlark.com/yuque/0/2020/png/1512483/1599908264228-48e88894-4fc5-49fb-bd3e-c65ae0c3f370.png#height=226&id=rrxAY&originHeight=226&originWidth=1112&originalType=binary&ratio=1&rotation=0&showTitle=false&size=30710&status=done&style=none&title=&width=1112)
.pub上面那个。

## 语雀

让我们再回到语雀，因为我们漏掉了最关键的一点，那就是 **同步文章** 到hexo-auto项目中。 **只有将语雀里的文章同步到项目中，再部署才会生成我们想要看到的最新的博客** 。
所以我们需要用一个库 `hexo-sync` 来进行同步文章。
同样的，我们也需要语雀的授权，所以要去语雀设置中，生成一个 `token` 

![yuque.png](https://cdn.nlark.com/yuque/0/2020/png/1512483/1599908500527-8a0527eb-0796-424d-a8c8-4abd7c8531b4.png#height=848&id=v2lLJ&originHeight=848&originWidth=2152&originalType=binary&ratio=1&rotation=0&showTitle=false&size=104308&status=done&style=none&title=&width=2152)

还有一个需要解决的问题。
我们在语雀里增删改文章，怎么去通知到云函数呢？
值得庆幸的是，语雀中提供了 `webhook` 来进行调用
![webhook yuque.png](https://cdn.nlark.com/yuque/0/2020/png/1512483/1599908978401-93334343-c86a-40ba-9a9f-60bce529662e.png#height=1118&id=rSERE&originHeight=1118&originWidth=1862&originalType=binary&ratio=1&rotation=0&showTitle=false&size=175067&status=done&style=none&title=&width=1862)

url就是我们云函数暴露给外界调用的url
![scf url.png](https://cdn.nlark.com/yuque/0/2020/png/1512483/1599909254184-d0812f7a-6ec4-498c-a289-2dbf275beb6a.png#height=1078&id=fj1Ui&originHeight=1078&originWidth=1794&originalType=binary&ratio=1&rotation=0&showTitle=false&size=121408&status=done&style=none&title=&width=1794)


## 总结
主要流程：
![yuque_diagram (1).png](https://cdn.nlark.com/yuque/0/2020/png/1512483/1590979698769-6ecdc30d-4146-455b-9ddd-715702b17f31.png#height=506&id=rOOsT&originHeight=506&originWidth=1120&originalType=binary&ratio=1&rotation=0&showTitle=false&size=76219&status=done&style=none&title=&width=1120)
            

## 
## 后记
### 部署到自己服务器
由于一些显而易见的问题，github访问比较慢，所以笔者还是在自己服务器上也搞了一个。
### 思路

1. 在自己的服务器上搭建一个git服务 建个文章的空仓库，在 `post-recieve` 钩子里去将文件的将内容同步到服务器的某个目录下，再放入`nginx` 的静态服务器，在hexo项目的主 `_config.yaml` 下的 `deploy` 的  `repo` 里填自己服务器的git地址

**问题** ：github action的模拟环境 连接我的git服务器 无法使用公钥 和`ssh-copy-id` cmd进行免登陆校验 设置

2. 在自己服务器上写一个 `node` 脚本,开启一个定时任务进行 `git pull`  因为博客的实时性要求并不是很高 所以可以设置一个比较长的间隔 几个消失到一天感觉都行


笔者使用了第二套方案，定时拉github.io库里的代码，再用nginx托管下就万事大吉了。

