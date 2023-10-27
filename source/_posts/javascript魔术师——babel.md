
![2.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1632906697335-d530ae4b-ddff-4aba-88cb-c204c2bb855c.png#clientId=u75c3defa-69f0-4&from=ui&id=uaf20ceb8&originHeight=462&originWidth=1091&originalType=binary&ratio=1&size=36717&status=done&style=none&taskId=u443fdfaa-533e-45a4-9578-9552ba431c0)
## 什么是babel
babel是一个编译器,负责将一些未纳入标准的js语法转换成符合当前浏览器（或node）的标准
**可以毫不避讳的说，babel的出现让前端开发进入了新时期，新阶段** 

### 为了理解其独一无二的作用，下面我们看个例子
![5.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1632906865927-89c58dde-a523-4fb4-98b2-6a1e00667456.png#clientId=u75c3defa-69f0-4&from=ui&id=u83e224c9&originHeight=381&originWidth=1086&originalType=binary&ratio=1&size=30440&status=done&style=none&taskId=u3f3263c6-206b-40b3-b859-dc8c5ab3264)
一个简单的方法将`n=>n**2` 转换成`Math.pow(n,2)`  —— **正则匹配法**
![6.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1632907100078-b1295b21-5985-4125-8ac4-c46a52cf2258.png#clientId=u75c3defa-69f0-4&from=ui&id=u7a3fdbdb&originHeight=510&originWidth=1063&originalType=binary&ratio=1&size=49770&status=done&style=none&taskId=u156693fb-7f47-485b-846c-181bdc00176)

![7.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1632907131842-f512b280-729b-49d5-a3dd-e307bab6ab0d.png#clientId=u75c3defa-69f0-4&from=ui&id=ub5064fd8&originHeight=526&originWidth=1066&originalType=binary&ratio=1&size=58402&status=done&style=none&taskId=u2e63265e-97b3-49d6-b3bd-bea3cac60eb)

![8.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1632907191698-4d530f95-7f14-4cf2-9e31-8a8ae3367ba7.png#clientId=u75c3defa-69f0-4&from=ui&id=u70f1dd54&originHeight=494&originWidth=1057&originalType=binary&ratio=1&size=68206&status=done&style=none&taskId=ube894eab-dde6-4564-aa32-e86a1a2633e)
但是对于下面这种格式的情形，正则表达式就出问题了，虽然它可以在字符串的世界里可以无敌，却识别不了**词法**和**语法**
![9.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1632907214643-da395cde-3a36-466b-a26e-0c30bc003309.png#clientId=u75c3defa-69f0-4&from=ui&id=ue8e2a744&originHeight=463&originWidth=1038&originalType=binary&ratio=1&size=64816&status=done&style=none&taskId=u9ee74b13-595e-4b9a-9f06-fe936830510)
#### 独门秘技抽象语法树
![4.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1632907479260-173e1212-dad6-4d2f-b9ea-ee9f2389c433.png#clientId=u75c3defa-69f0-4&from=ui&id=uaa60176c&originHeight=451&originWidth=1058&originalType=binary&ratio=1&size=41377&status=done&style=none&taskId=ua7e5a0a8-559f-483e-8d85-8a1e3214e46)
##### 语法树的构建
![11.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1632907524981-7776d80b-8c89-44ad-a01f-1242b1e3bfec.png#clientId=u75c3defa-69f0-4&from=ui&id=u69560891&originHeight=537&originWidth=1074&originalType=binary&ratio=1&size=53789&status=done&style=none&taskId=u41e43082-fbb5-44f1-9a67-1e14cc194cb)
![12.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1632907546015-26d92e48-06f1-4739-8b05-944cccec8a32.png#clientId=u75c3defa-69f0-4&from=ui&id=u18b80de2&originHeight=558&originWidth=1083&originalType=binary&ratio=1&size=57651&status=done&style=none&taskId=ua4187f73-adfb-4bcd-b7e1-b64386eced1)
![13.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1632907571752-7860d7c7-4d06-4a41-89d2-4f6d2e61af9d.png#clientId=u75c3defa-69f0-4&from=ui&id=u1d708b76&originHeight=559&originWidth=1076&originalType=binary&ratio=1&size=57843&status=done&style=none&taskId=ub4bbb3eb-4d60-4c1c-96d8-826856875a4)
![14.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1632907590777-fdbe149f-d691-441a-b838-e7bc06926e20.png#clientId=u75c3defa-69f0-4&from=ui&id=u7318fb3c&originHeight=542&originWidth=1078&originalType=binary&ratio=1&size=61109&status=done&style=none&taskId=u67d72f10-347b-4083-abc2-9ca39a18d6c)
![15.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1632907603113-20af3473-df63-4973-a509-ff0d359321b8.png#clientId=u75c3defa-69f0-4&from=ui&id=u1f01c3b2&originHeight=555&originWidth=1075&originalType=binary&ratio=1&size=61156&status=done&style=none&taskId=u60bf6ff4-f940-449d-b2cf-aa2dcd5fa65)
##### 结点替换
![16.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1632907627287-d2a6384a-d179-4940-b5f6-8ca1f741c06f.png#clientId=u75c3defa-69f0-4&from=ui&id=ub00f18d5&originHeight=528&originWidth=1039&originalType=binary&ratio=1&size=60016&status=done&style=none&taskId=u08b55fd7-651d-4aef-91f8-f5b03d4557b)
![17.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1632907640890-9107964b-aae7-4f49-b578-6a2c9fb874ef.png#clientId=u75c3defa-69f0-4&from=ui&id=u0ae035a9&originHeight=559&originWidth=1091&originalType=binary&ratio=1&size=71941&status=done&style=none&taskId=u8e27994a-d281-4e33-8831-fa7468ba500)

![18.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1632907731876-4af293fc-1da8-43aa-a51a-fae77788e203.png#clientId=u75c3defa-69f0-4&from=ui&id=uee85baff&originHeight=402&originWidth=1039&originalType=binary&ratio=1&size=35846&status=done&style=none&taskId=u068c6c1e-dbcd-4e6a-ab56-888dd85c7cb)
![19.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1632907749410-9d532438-9e5c-4c3d-8fa9-2be5ab312683.png#clientId=u75c3defa-69f0-4&from=ui&id=u2acf091a&originHeight=480&originWidth=1067&originalType=binary&ratio=1&size=67200&status=done&style=none&taskId=u486fc811-f986-437a-94e4-8d3389f1309)

 复杂的使用完全可以使用语法树进行拆解，字符串类型直接被识别到了 不进行转换，nice!!

![20.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1632907988611-127c58c2-62a1-45d8-be2b-a3741368a251.png#clientId=u75c3defa-69f0-4&from=ui&id=u741e4e12&originHeight=539&originWidth=1071&originalType=binary&ratio=1&size=72573&status=done&style=none&taskId=uca5f1fd4-a77b-4b38-8d18-a2a77f3a594)

将`BinaryExpression`的结点树替换成`CallExpression`,再把树通过`babel/generator`生成为代码，nice!!
## plugins
![74.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1632918784539-d6d3c82e-73cc-4fb6-a38b-d3ba8a5bdd37.png#clientId=u90e17270-27f7-4&from=ui&id=BJlrp&originHeight=377&originWidth=804&originalType=binary&ratio=1&size=8471&status=done&style=none&taskId=u2407c1d8-b91f-4a39-91dc-a9560e4dd2f)![75.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1632918784731-02795815-9a01-4f8e-a922-e768ba145bba.png#clientId=u90e17270-27f7-4&from=ui&id=nMBBb&originHeight=482&originWidth=1033&originalType=binary&ratio=1&size=75263&status=done&style=none&taskId=ubaa57a37-5efa-49ce-8c31-bd1e04fba78)![76.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1632918785084-ca13c305-317f-4b2a-8301-722ba1d6063d.png#clientId=u90e17270-27f7-4&from=ui&id=a1Hr0&originHeight=539&originWidth=1039&originalType=binary&ratio=1&size=67184&status=done&style=none&taskId=uc5637524-3fd0-4af2-a0d8-590d9919feb)
### 前置知识
![66.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1632918783395-8b08fb10-fd0c-48a7-b1c0-b29cce066867.png#clientId=u90e17270-27f7-4&from=ui&id=uTA0W&originHeight=499&originWidth=1024&originalType=binary&ratio=1&size=67350&status=done&style=none&taskId=u204fd46b-342a-4663-a3a9-f51d449f82c)
**入口文件及对应的接收者**


### 

![67.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1632918783395-e5517036-c424-455a-b243-7a59f6129049.png#clientId=u90e17270-27f7-4&from=ui&id=XieHf&originHeight=454&originWidth=1063&originalType=binary&ratio=1&size=29861&status=done&style=none&taskId=u56284827-c3b9-40c5-bdd7-3e323d81ee6)
#### 结点校验
![68.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1632918783637-edc560bf-e604-4993-bc8f-a92704d88922.png#clientId=u90e17270-27f7-4&from=ui&id=yG8KJ&originHeight=517&originWidth=1051&originalType=binary&ratio=1&size=65463&status=done&style=none&taskId=u1a3f139a-4b70-47ec-9df5-d26e1991133)
#### 结点构建
![69.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1632918783860-70fd83fc-d6b8-40b8-9779-56e63298130a.png#clientId=u90e17270-27f7-4&from=ui&id=PSUKn&originHeight=550&originWidth=1061&originalType=binary&ratio=1&size=67492&status=done&style=none&taskId=ueff2bb03-4ca4-47cc-b8b1-028f642fda6)
#### template

用types来构建复杂树非常反人类，所以template出现了
![70.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1632918783645-75571f1c-1a89-4a30-986a-bbf5e4f74301.png#clientId=u90e17270-27f7-4&from=ui&id=dkmiT&originHeight=395&originWidth=953&originalType=binary&ratio=1&size=29225&status=done&style=none&taskId=u1547a0b0-55e1-4058-a93d-6be97164845)
![71.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1632918784157-c21d83e9-0757-4af9-bc76-e44db0ce6cdf.png#clientId=u90e17270-27f7-4&from=ui&id=lXahN&originHeight=518&originWidth=1074&originalType=binary&ratio=1&size=81403&status=done&style=none&taskId=u86cf7180-a654-46ab-8db3-d435c3452c3)![72.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1632918784398-432776ff-cd75-416c-9048-c033b3808663.png#clientId=u90e17270-27f7-4&from=ui&id=QgEX0&originHeight=541&originWidth=1078&originalType=binary&ratio=1&size=54557&status=done&style=none&taskId=u13002d95-6fd4-47b5-81e0-777947d6364)![73.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1632918784531-3cc5010f-68e6-4cde-b72e-92d433574298.png#clientId=u90e17270-27f7-4&from=ui&id=cgVSb&originHeight=516&originWidth=1018&originalType=binary&ratio=1&size=63300&status=done&style=none&taskId=u7c6b6fb1-4296-495f-a65a-dfc8fa437d4)
#### nodePath
![84.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1632918827690-30d3965c-92b8-445d-a044-59d66af02240.png#clientId=u90e17270-27f7-4&from=ui&id=P34ZN&originHeight=459&originWidth=1045&originalType=binary&ratio=1&size=28125&status=done&style=none&taskId=ucdeba7b0-2f7b-486a-b4fa-f3b0a2c906a)![85.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1632918827735-faac6c47-11d9-4d4b-9d20-bac9cf3bf7a2.png#clientId=u90e17270-27f7-4&from=ui&id=zWjZG&originHeight=552&originWidth=1027&originalType=binary&ratio=1&size=88301&status=done&style=none&taskId=ub91e1f7b-f082-43db-a745-d5d3e33a578)![86.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1632918827778-8725c804-2877-46f2-8611-86a93e1d2c27.png#clientId=u90e17270-27f7-4&from=ui&id=DMPZj&originHeight=533&originWidth=1077&originalType=binary&ratio=1&size=91343&status=done&style=none&taskId=u008454dc-2d45-4919-8083-a14c0a7cf6b)
#### Scope	 ![88.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1632918828240-51894232-5d1d-4c05-a08d-b544ef8a5c4f.png#clientId=u90e17270-27f7-4&from=ui&id=pT3G0&originHeight=539&originWidth=1062&originalType=binary&ratio=1&size=91540&status=done&style=none&taskId=u2a8c63a9-50c5-43cc-ac2e-7e705705b31)
### 如何创建插件
![77.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1632918785189-1c9f29e9-be2f-4221-b694-c132454c7eb7.png#clientId=u90e17270-27f7-4&from=ui&id=HytTA&originHeight=292&originWidth=1011&originalType=binary&ratio=1&size=17067&status=done&style=none&taskId=u69bb25d9-845b-432e-ac39-99b4eee72c7)
![78.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1632918826791-ea4f3add-f442-49c1-8c57-6bb673ca5aca.png#clientId=u90e17270-27f7-4&from=ui&id=tWwXB&originHeight=546&originWidth=1073&originalType=binary&ratio=1&size=75820&status=done&style=none&taskId=u82a4466e-46a9-4898-a120-ac5aabcf0dd)![79.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1632918826811-4f44a9ea-a10f-42f9-b8fc-40e88b95bf60.png#clientId=u90e17270-27f7-4&from=ui&id=gVAq0&originHeight=536&originWidth=1071&originalType=binary&ratio=1&size=70933&status=done&style=none&taskId=u7f55fee6-cc44-4a88-a73f-e5b04c107b9)![80.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1632918826791-0434a8c2-62df-4f3f-a986-6e377a34b8bf.png#clientId=u90e17270-27f7-4&from=ui&id=Wo6J6&originHeight=536&originWidth=1093&originalType=binary&ratio=1&size=74600&status=done&style=none&taskId=u2a2303a6-34fe-483c-8cd7-709fe8d9d97)![82.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1632918826796-aa63a46a-e477-4449-bb37-306bab303152.png#clientId=u90e17270-27f7-4&from=ui&id=HV2zZ&originHeight=546&originWidth=1068&originalType=binary&ratio=1&size=93320&status=done&style=none&taskId=u247ce501-b8ed-4cb2-97fb-5f6f1362227)![83.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1632918826802-07ab1229-8fc6-4db7-87ff-090ca2ad63f2.png#clientId=u90e17270-27f7-4&from=ui&id=zPirr&originHeight=539&originWidth=1093&originalType=binary&ratio=1&size=64697&status=done&style=none&taskId=ucc6e9bec-dc8f-4bc7-9185-07e37f7885a)


### 例子
####  最简陋的实现
以我们代码中常用到的`可选链`插件为例，我们来重新实现一下这个插件

![c1.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1633772642217-dfd9f0f8-638c-459a-a7ea-d51bab6b8bfc.png#clientId=ubb1d753a-e999-4&from=ui&id=u6de8c905&originHeight=466&originWidth=1062&originalType=binary&ratio=1&size=30343&status=done&style=none&taskId=u57b77761-4245-498f-8acc-460556cc3b2)

![c2.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1633772811427-f06aeee2-4ef4-456d-ba21-66836fbc50fc.png#clientId=ubb1d753a-e999-4&from=ui&id=uc6fc7e07&originHeight=380&originWidth=1085&originalType=binary&ratio=1&size=22211&status=done&style=none&taskId=u3df0642b-342e-4a8c-9d8d-4d0087184b3)

![c3.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1633772842675-2b777cb7-8ab4-46b5-8c0b-a35732e628c8.png#clientId=ubb1d753a-e999-4&from=ui&id=u3a872809&originHeight=423&originWidth=977&originalType=binary&ratio=1&size=24498&status=done&style=none&taskId=uf8cc1459-9c38-481e-9e6c-4852e682241)
虽然简陋但是实现了最简单的效果
但是再加一个属性，就立刻跪了
![c4.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1633773096772-eeef896d-3571-4428-9483-3fd008d851a7.png#clientId=ubb1d753a-e999-4&from=ui&id=ub0b29d71&originHeight=529&originWidth=1094&originalType=binary&ratio=1&size=33335&status=done&style=none&taskId=uc4060f2f-66ac-4234-b92f-a3192af3fbe)

![c5.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1633773114901-667cbe3f-7fc1-4325-9aed-991a42ab3c19.png#clientId=ubb1d753a-e999-4&from=ui&id=udfeb8f7e&originHeight=470&originWidth=1315&originalType=binary&ratio=1&size=28247&status=done&style=none&taskId=u76935c61-6da0-42cd-b57e-9a442c052fc)
#### 修复正常.符号也会触发?.转换问题
很显然，b.c不需要判断b,所以我们继续完善

![c9.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1633917368341-42d91c27-e387-4829-8d9d-93a157a5c388.png#clientId=ucbc57d5c-bffa-4&from=ui&id=u5874a0aa&originHeight=510&originWidth=1021&originalType=binary&ratio=1&size=32878&status=done&style=none&taskId=u1cdb31f0-d5c8-44bc-9ee2-b06e8f4b480)
再试运行一次，差强人意

![c10.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1633917469210-b120f3e7-0f29-49f2-8798-e222c23d6636.png#clientId=ucbc57d5c-bffa-4&from=ui&id=uc129eec2&originHeight=239&originWidth=940&originalType=binary&ratio=1&size=14024&status=done&style=none&taskId=u1877a779-e435-452d-ac16-b83d1572af1)

但是如果再加个可选值
![c11.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1633917642510-d5f34feb-f2a6-4475-bae9-258bab849a0b.png#clientId=ucbc57d5c-bffa-4&from=ui&id=u4480e249&originHeight=253&originWidth=714&originalType=binary&ratio=1&size=14303&status=done&style=none&taskId=uf26e58ba-1b01-4231-8bdf-6da00b10a12)
就会出现问题
![c12.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1633917651958-c216df1a-71c7-4409-ac34-5e51fb081fcb.png#clientId=ucbc57d5c-bffa-4&from=ui&id=u4ed40422&originHeight=244&originWidth=1272&originalType=binary&ratio=1&size=15539&status=done&style=none&taskId=ude0be574-ee36-4238-a0c7-ba86be76ddd)
#### 存储计算结果
`a==null?undefined:a.b`被重复声明了两次，**这样会造成一个重复执行三目运算，显然是不合理的**
**所以我们再加以优化，如下所示**
![c13.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1633918062184-b40e1537-49b8-43ac-939e-fc8edabb2eb1.png#clientId=ucbc57d5c-bffa-4&from=ui&id=u475a33cc&originHeight=508&originWidth=1102&originalType=binary&ratio=1&size=34171&status=done&style=none&taskId=ua5d81bd7-bb35-445d-b2c5-d2f193099f6)
我们先生成一个当前作用域里独一无二的变量并注入到作用域，再边进行三目运算，边进行结果赋值，对运算结果进行存储

![c14.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1633918200394-b294d27f-246c-4cb0-bd5b-873f20216c35.png#clientId=ucbc57d5c-bffa-4&from=ui&id=ud9023fe3&originHeight=323&originWidth=1257&originalType=binary&ratio=1&size=22233&status=done&style=none&taskId=ua328f2a2-e30c-4199-9d98-40d48d37551)

你以为这就完事大吉了，可还有个问题呢
![c15.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1633919298883-89684e57-e47b-49bd-b23c-2e8c72b94321.png#clientId=ucbc57d5c-bffa-4&from=ui&id=uf3890796&originHeight=326&originWidth=1223&originalType=binary&ratio=1&size=25973&status=done&style=none&taskId=ua34e985f-9e3c-4d46-b117-36c2a1bef97)
#### 防止undefined被声明 如let undefined = 2
为了防止真的有大哥对undefined进行赋值，我们要用babel的api在作用域中对undefined值进行build
![c17.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1633919467079-b0b9d41d-f649-4ef4-867d-8405b98274bd.png#clientId=ucbc57d5c-bffa-4&from=ui&id=uaf640400&originHeight=533&originWidth=1070&originalType=binary&ratio=1&size=37508&status=done&style=none&taskId=uc31e2784-1278-4ad5-a45a-dd640d02d1c)
结果
![c18.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1633919667172-f129df16-ee57-4720-882a-b2e9b4723aa7.png#clientId=ucbc57d5c-bffa-4&from=ui&id=u98a13f37&originHeight=303&originWidth=1223&originalType=binary&ratio=1&size=25557&status=done&style=none&taskId=uf563f335-3487-4c89-9822-75aff77f31b)
#### 添加&&的实现方式
还没完 我们再玩一下，babel的可传入option,构造另一种形式
![c20.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1633919945726-7cbba642-44d5-4f9c-a4f2-2c585a5fb5d4.png#clientId=ucbc57d5c-bffa-4&from=ui&id=uda9a4340&originHeight=814&originWidth=1614&originalType=binary&ratio=1&size=136523&status=done&style=shadow&taskId=u5fd6ff88-7edc-46ea-9f97-a6e6076b7b7)

参数传入
![c21.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1633920121928-635582b3-f57b-4996-bce1-97dc9658f8cd.png#clientId=ucbc57d5c-bffa-4&from=ui&id=uc7ca890e&originHeight=351&originWidth=1037&originalType=binary&ratio=1&size=25299&status=done&style=none&taskId=u0bccb021-b66f-4567-a4ed-4220435abe2)
运行结果
![c22.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1633920135249-0c257e41-268c-4d1f-af30-8cf52817c578.png#clientId=ucbc57d5c-bffa-4&from=ui&id=u39265a60&originHeight=350&originWidth=929&originalType=binary&ratio=1&size=20120&status=done&style=none&taskId=u96194ffd-4e63-4c87-92e3-f036216ef3c)

![c23.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1633920135308-1e4218ca-229e-4903-92cf-db5eb470d31f.png#clientId=ucbc57d5c-bffa-4&from=ui&id=u3bd91012&originHeight=270&originWidth=922&originalType=binary&ratio=1&size=16373&status=done&style=none&taskId=u4ff86db8-9188-4b3f-bd04-cc1ff39759a)
一个简单的插件就完成了。
#### 完整代码
```javascript

module.exports = function plugin(
    { types: t, template },
    { loose = false } = {}
) {
    const buildLoose = template.expression(`
    (%%tmp%% = %%obj%%) && %%expr%%
  `);

    return {
        name: "optional-chaining",
        visitor: {
            OptionalMemberExpression(path) {
                let {object, property} = path.node
                if (path.node.optional) {
                    let tmp = path.scope.generateUidIdentifier("obj");
                    path.scope.push({id: tmp});
                    let undef = path.scope
                        .buildUndefinedNode();
                    if(loose){
                        path.replaceWith(buildLoose({tmp,obj:object,expr:template.expression.ast`${tmp}.${property}`}))
                        return
                    }
                    path.replaceWith(
                        template.expression.ast`(${tmp} =${object})== null?${undef}:${tmp}.${property}`
                    )
                } else {
                    path.replaceWith(
                        template.expression.ast`${object}.${property}`
                    )
                }
            }
        }
    }
};

```
#### 另一种实现
这是原作者的实现，但是理解起来比较复杂
```javascript
module.exports = function plugin(
  { types: t, template },
  { loose = false } = {}
) {
  const buildLoose = template.expression(`
    (%%tmp%% = %%obj%%) && %%expr%%
  `);

  return {
    name: "optional-chaining",
    visitor: {
      OptionalMemberExpression(path) {
       //#region 另一种实现方式
            let originalPath = path;
            let tmp =
                path.scope.generateUidIdentifier("obj");
            path.scope.push({id: tmp});

            while (!path.node.optional) {
              path = path.get("object");
            }

            let {object} = path.node;

            let memberExpr = tmp;
            do {
              memberExpr = t.memberExpression(
                  memberExpr,
                  path.node.property,
                  path.node.computed,
              );

              if (path === originalPath) {
                break;
              }

              path = path.parentPath;
            } while (true);
            if (loose) {
              path.replaceWith(
                  buildLoose({
                    tmp,
                    obj: object,
                    expr: memberExpr,
                  })
              )
            } else {
              let undef =
                  path.scope
                      .buildUndefinedNode();

              originalPath.replaceWith(
                  template.expression.ast`
                    (${tmp} = ${object}) == null
                      ? ${undef}
                      : ${memberExpr}
                  `
              )
            }
          }
        //#endregion
    }
  }
};


```
[代码仓库地址](http://git.xyz.cn/zhangyunpeng0126/babel-pulgin-playground)
![21.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1632908051279-ec899ae0-f86c-4bc9-af1d-ef9ad9a88590.png#clientId=u75c3defa-69f0-4&from=ui&id=u048cd509&originHeight=480&originWidth=1096&originalType=binary&ratio=1&size=17657&status=done&style=none&taskId=ucf53303e-52a4-4d9b-94c0-1f49d607583)
## 转换过程
![22.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1632918433291-2c433c61-0a3e-4071-b890-088370036851.png#clientId=u90e17270-27f7-4&from=ui&id=ua5135b44&originHeight=502&originWidth=1094&originalType=binary&ratio=1&size=47779&status=done&style=none&taskId=u60195a13-0b1e-4b88-b5aa-53d65e7aadd)
## 所用模块
![23.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1632918482960-13cab926-0f93-47c1-bc06-5aead981a5f7.png#clientId=u90e17270-27f7-4&from=ui&id=u6f12411f&originHeight=503&originWidth=1081&originalType=binary&ratio=1&size=62676&status=done&style=none&taskId=uebf04ec8-4eaa-4dae-9c71-fcbb2afae97)
![24.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1632918524187-f4300412-3afb-4e04-95c2-15dd13a74964.png#clientId=u90e17270-27f7-4&from=ui&id=ufd39dd80&originHeight=548&originWidth=1085&originalType=binary&ratio=1&size=71393&status=done&style=none&taskId=u417407a4-0105-49cc-a71c-f9b65a96c19)
## js到语法树——解析

![25.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1632918554574-628095bf-fe64-4bdb-ac05-736bbb0da4fc.png#clientId=u90e17270-27f7-4&from=ui&id=uebb6c22c&originHeight=547&originWidth=1080&originalType=binary&ratio=1&size=48245&status=done&style=none&taskId=u5137f457-36f3-4e29-b202-1b1bb56f901)
### 词法分析

![26.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1632918570442-2537caeb-a615-49f4-99f1-f135d69e2934.png#clientId=u90e17270-27f7-4&from=ui&id=u3bbb5798&originHeight=470&originWidth=1064&originalType=binary&ratio=1&size=25952&status=done&style=none&taskId=udea293fd-cde5-4812-9bd9-c2ec30ffd11)
![27.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1632918619075-9481d5b9-1e9c-4529-8e44-cf446642f0f0.png#clientId=u90e17270-27f7-4&from=ui&id=u4e8e027f&originHeight=436&originWidth=1063&originalType=binary&ratio=1&size=30626&status=done&style=none&taskId=u3bde5224-931a-4856-94d0-0c376cab269)![28.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1632918619075-63fdd4be-bfae-4ab9-8077-f3a994de1db7.png#clientId=u90e17270-27f7-4&from=ui&id=u9d3141ad&originHeight=539&originWidth=1035&originalType=binary&ratio=1&size=44335&status=done&style=none&taskId=u61de6141-790d-4fb7-ad16-e3da9a88348)![29.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1632918619061-95ff8148-8f82-4c4b-b58b-0d3d2a7642a4.png#clientId=u90e17270-27f7-4&from=ui&id=ud6723984&originHeight=319&originWidth=970&originalType=binary&ratio=1&size=19312&status=done&style=none&taskId=uf683fa22-8ab7-469d-817a-029d0d85396)![30.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1632918619081-bbcacbb0-784b-4b69-a474-9bf6f6902367.png#clientId=u90e17270-27f7-4&from=ui&id=udfb878a5&originHeight=474&originWidth=1029&originalType=binary&ratio=1&size=50132&status=done&style=none&taskId=uce5c9249-6d3f-403f-8d2d-821131667f5)
### 语法分析
![31.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1632918619082-5571f980-9100-493b-8dc6-d623ec2a00b0.png#clientId=u90e17270-27f7-4&from=ui&id=u71611d3a&originHeight=549&originWidth=816&originalType=binary&ratio=1&size=50120&status=done&style=none&taskId=u676ae25d-84e5-467d-86fa-d801c928fa1)![32.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1632918619869-43f3e6b4-1f11-44bf-b8a3-ffd2b2b63bdb.png#clientId=u90e17270-27f7-4&from=ui&id=u8c3a58a1&originHeight=363&originWidth=752&originalType=binary&ratio=1&size=20732&status=done&style=none&taskId=u03a5d7d5-5509-447e-ad64-bb442960fa3)![33.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1632918619934-292d112d-c871-40e7-8178-6cc6d0efe258.png#clientId=u90e17270-27f7-4&from=ui&id=ua87a1eef&originHeight=370&originWidth=966&originalType=binary&ratio=1&size=36712&status=done&style=none&taskId=uccf85c0a-0676-4a0c-8c86-e4336198926)
![35.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1632918695572-66fcab78-6a5a-4e60-a7ce-8b95ec7dae7d.png#clientId=u90e17270-27f7-4&from=ui&id=u5085eace&originHeight=504&originWidth=946&originalType=binary&ratio=1&size=52421&status=done&style=none&taskId=u34da35ba-1f47-4fc3-aa83-06dca4e278a)
![36.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1632918742683-05e95add-6e09-49f3-be34-ecd85c54c1c6.png#clientId=u90e17270-27f7-4&from=ui&id=u3c180743&originHeight=452&originWidth=979&originalType=binary&ratio=1&size=20853&status=done&style=none&taskId=u92d2721e-1c88-4b63-b3c6-29de78d2fd8)![37.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1632918742710-e6625cbe-a53b-426e-8543-e278a5450a78.png#clientId=u90e17270-27f7-4&from=ui&id=uf2b72509&originHeight=475&originWidth=963&originalType=binary&ratio=1&size=42079&status=done&style=none&taskId=ue69145c2-7e8c-4d65-b124-d6f93ae3209)
### 语义分析
![38.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1632918742804-66389d64-0417-4146-b9c9-8caaf8b45621.png#clientId=u90e17270-27f7-4&from=ui&id=u1abc991d&originHeight=320&originWidth=1018&originalType=binary&ratio=1&size=26014&status=done&style=none&taskId=u2b3694dd-f13c-4950-8ed3-84ec3363406)![39.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1632918742866-74ccc874-fda8-4fa0-8213-58bcf21f06a2.png#clientId=u90e17270-27f7-4&from=ui&id=ufeb565bd&originHeight=521&originWidth=1029&originalType=binary&ratio=1&size=60415&status=done&style=none&taskId=u2953d958-52e3-4d56-934e-8f1f02853b1)![40.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1632918742802-c1ca79eb-946e-4e1c-8215-78535d84f0c9.png#clientId=u90e17270-27f7-4&from=ui&id=u3327dee1&originHeight=276&originWidth=974&originalType=binary&ratio=1&size=25470&status=done&style=none&taskId=ua4ef7a33-c23a-4ca6-b580-de1aff696fb)![41.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1632918743453-0f3c5876-c127-4568-be7e-f559d9d4b921.png#clientId=u90e17270-27f7-4&from=ui&id=ud8692e1d&originHeight=494&originWidth=1036&originalType=binary&ratio=1&size=62052&status=done&style=none&taskId=ub9603d3e-df59-4f74-9290-9bf5bd784e8)
## 语法树的遍历——深度优先遍历
![42.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1632918743519-8a6bbe63-a7da-4859-9f38-5ca34b276378.png#clientId=u90e17270-27f7-4&from=ui&id=u96bace98&originHeight=493&originWidth=955&originalType=binary&ratio=1&size=50630&status=done&style=none&taskId=uf81d4286-1711-46fb-a297-f31798ad3c2)![43.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1632918743590-32bdbdc6-3a28-4e63-95a6-0ac37f3ea214.png#clientId=u90e17270-27f7-4&from=ui&id=u736a4534&originHeight=520&originWidth=1056&originalType=binary&ratio=1&size=64811&status=done&style=none&taskId=ue13d37cf-4515-4f3f-bff0-d74315f4178)![44.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1632918743790-d6566bae-88a3-4f9a-b7ed-b3926ecafd5e.png#clientId=u90e17270-27f7-4&from=ui&id=ub2b28c14&originHeight=529&originWidth=1038&originalType=binary&ratio=1&size=61109&status=done&style=none&taskId=u37e39816-119a-4166-a2d4-deb8b69c34f)

### 一个例子fn(3)->[6,fn]

![45.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1632918743855-85c6b6ed-49a5-4a36-ad24-0e1707c5eb1b.png#clientId=u90e17270-27f7-4&from=ui&id=ud836df95&originHeight=420&originWidth=1055&originalType=binary&ratio=1&size=48752&status=done&style=none&taskId=u098ff621-b63e-4144-8819-7a16bf80f28)
#### 知识补充：
#### 二叉树的遍历一般使用循环或者递归进行。
#### 以递归为例
#### 深度优先遍历，会优先遍历left结点，没有left结点，再遍历right结点。
#### 递是函数的入栈，归是函数的出栈，从首节点往下递，从尾结点网上归。
![46.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1632918744381-c7af84a4-a4a1-4517-8f39-662e93becdb3.png#clientId=u90e17270-27f7-4&from=ui&id=u32b9a2b1&originHeight=547&originWidth=1045&originalType=binary&ratio=1&size=62546&status=done&style=none&taskId=u738bdfe1-98af-41a1-a89b-5bc89cb03fe)![47.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1632918744616-5160d215-e094-4cbc-be59-958a46a052cb.png#clientId=u90e17270-27f7-4&from=ui&id=uf76561fc&originHeight=532&originWidth=1094&originalType=binary&ratio=1&size=62836&status=done&style=none&taskId=u0ceefed1-2b59-4ca7-b450-64f4812de41)![48.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1632918744636-5e01e4e3-e673-4534-9983-f9eff6600b05.png#clientId=u90e17270-27f7-4&from=ui&id=u945f3536&originHeight=536&originWidth=1078&originalType=binary&ratio=1&size=63390&status=done&style=none&taskId=uecb7806b-19a1-4aa7-afa4-42b752e1fee)![49.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1632918745122-e16ab0bc-b8cb-4ddb-bdf2-755930643cf1.png#clientId=u90e17270-27f7-4&from=ui&id=u1637a168&originHeight=529&originWidth=1083&originalType=binary&ratio=1&size=62832&status=done&style=none&taskId=ubf494f25-30df-4bf6-9694-5ed8454cdcb)![50.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1632918745405-f64d1f2f-fb8a-4a05-af29-6e157c0b88c9.png#clientId=u90e17270-27f7-4&from=ui&id=u17526321&originHeight=529&originWidth=1073&originalType=binary&ratio=1&size=62902&status=done&style=none&taskId=u27bcffc5-359b-4534-bccf-1e2ed704241)![51.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1632918745442-758a5a14-281d-4d42-8709-880ad33c4eac.png#clientId=u90e17270-27f7-4&from=ui&id=uf499ed17&originHeight=521&originWidth=1080&originalType=binary&ratio=1&size=63460&status=done&style=none&taskId=u741cecab-e9e7-41d6-93a4-f6a78903b6d)![52.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1632918745825-6da8c084-250b-45e6-a8f4-f0e39e874b14.png#clientId=u90e17270-27f7-4&from=ui&id=u240a7912&originHeight=515&originWidth=1091&originalType=binary&ratio=1&size=62842&status=done&style=none&taskId=u3541a977-e012-4efc-9186-6ce2bc16cb4)![53.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1632918745783-83ed734e-651b-4c79-ab4c-4762a990a79b.png#clientId=u90e17270-27f7-4&from=ui&id=uc92488f1&originHeight=540&originWidth=1084&originalType=binary&ratio=1&size=63564&status=done&style=none&taskId=u533f1ee2-a524-471c-bf1b-4ca46ef03ee)
![54.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1632918746056-b4d3a41e-9b54-4a4f-8b72-46f603df9140.png#clientId=u90e17270-27f7-4&from=ui&id=u4b52aedf&originHeight=534&originWidth=1077&originalType=binary&ratio=1&size=63342&status=done&style=none&taskId=u8ca31d32-c833-4b94-b766-c611daaddf0)

**当CallExpression出栈之后，进行结点替换操作**
![55.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1632918746384-50485c14-aa9a-49d0-b002-f9fa138eb50f.png#clientId=u90e17270-27f7-4&from=ui&id=u642b8cb2&originHeight=512&originWidth=1037&originalType=binary&ratio=1&size=70508&status=done&style=none&taskId=udbbc170f-69b5-41f0-8df2-b9d5386786d)

![56.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1632918781493-87de38ad-7687-4648-a3bd-8fc5ce84f5db.png#clientId=u90e17270-27f7-4&from=ui&id=u66027ac1&originHeight=538&originWidth=1071&originalType=binary&ratio=1&size=60449&status=done&style=none&taskId=u72c6776f-f61d-4e97-998a-602c71aa14a)![57.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1632918781501-fcf0ac0c-5b05-4af6-b336-558ce6a5ed6b.png#clientId=u90e17270-27f7-4&from=ui&id=ucea8f685&originHeight=530&originWidth=1082&originalType=binary&ratio=1&size=63076&status=done&style=none&taskId=uf821e710-7532-42ce-b1cf-1281e17b6ad)![58.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1632918781501-16d2764d-e176-4a0f-95ac-7fb338226c2c.png#clientId=u90e17270-27f7-4&from=ui&id=u723f7c2d&originHeight=525&originWidth=1075&originalType=binary&ratio=1&size=63267&status=done&style=none&taskId=u14e6cf06-2810-4ba9-bbb9-40961d7282b)![59.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1632918781513-59b2700c-fbb1-4456-b1c8-53bdcc49c616.png#clientId=u90e17270-27f7-4&from=ui&id=u186e7e19&originHeight=512&originWidth=1084&originalType=binary&ratio=1&size=63747&status=done&style=none&taskId=ub7c81317-5c21-4fc9-a561-cf7fa6275e9)![60.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1632918781515-77617e28-e6d2-42b4-bc3f-24c83331f43b.png#clientId=u90e17270-27f7-4&from=ui&id=u53eb0336&originHeight=515&originWidth=1004&originalType=binary&ratio=1&size=62295&status=done&style=none&taskId=uf74c3919-6465-4a68-a64b-ea025b517a1)![61.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1632918782385-e3d29374-7e7c-468a-923b-bffd7d76dea5.png#clientId=u90e17270-27f7-4&from=ui&id=u90eecf03&originHeight=539&originWidth=1062&originalType=binary&ratio=1&size=63803&status=done&style=none&taskId=ude40e729-b588-425f-aaa0-d23b0bab6ce)![62.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1632918782429-32c49e02-05e3-4922-836c-7a7dc1db5cf4.png#clientId=u90e17270-27f7-4&from=ui&id=ub4387340&originHeight=526&originWidth=1059&originalType=binary&ratio=1&size=63448&status=done&style=none&taskId=u1529f5a5-011b-4e27-9c59-64c73f2894f)![63.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1632918782496-d751bc0d-f543-4138-a4fd-29816259638d.png#clientId=u90e17270-27f7-4&from=ui&id=u3f982ecc&originHeight=524&originWidth=1065&originalType=binary&ratio=1&size=63608&status=done&style=none&taskId=ube801537-d2be-4ec9-bcce-3661951acc0)

## 
## 
## 结语
一句话与大家共勉：认识是不断螺旋上升的过程，理论的形成也是基于当时的时空，在不断变化的环境中，不断实践才能出真知。


## 参考
[视频地址](https://www.youtube.com/embed/UeVq_U5obnE)
[原作者github](https://github.com/nicolo-ribaudo/conf-holyjs-moscow-2019)
