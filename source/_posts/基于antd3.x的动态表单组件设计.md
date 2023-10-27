
---

tags: [ant-design,dynamicField,metadata]
categories: [组件设计]

---

## 效果
![动态表单编辑](https://cdn.nlark.com/yuque/0/2021/png/1512483/1637571605647-e0480666-b6df-46d4-88b3-c5492c1542c0.png#clientId=uc5b91357-e4b8-4&from=ui&height=384&id=u1fff1e99&originHeight=699&originWidth=1540&originalType=binary&ratio=1&rotation=0&showTitle=true&size=54152&status=done&style=none&taskId=u2ca96edc-dde4-4699-ae02-d4ebd9a6421&title=%E5%8A%A8%E6%80%81%E8%A1%A8%E5%8D%95%E7%BC%96%E8%BE%91&width=845 "动态表单编辑")

![后台回显](https://cdn.nlark.com/yuque/0/2021/png/1512483/1637575150309-ac6e2a12-008f-4abb-a108-ccb17525cf54.png#clientId=uc5b91357-e4b8-4&from=ui&id=ua09e2c86&originHeight=179&originWidth=1591&originalType=binary&ratio=1&rotation=0&showTitle=true&size=15836&status=done&style=none&taskId=ue15123cb-a082-4689-8dce-e9a68030f0a&title=%E5%90%8E%E5%8F%B0%E5%9B%9E%E6%98%BE "后台回显")
## 业务关系
![业务关系](https://cdn.nlark.com/yuque/0/2021/png/1512483/1637635229415-a1b8ab3e-9e0d-441a-b3d0-919767f7e806.png#clientId=u53703c6e-da83-4&from=ui&id=uf43d9719&originHeight=614&originWidth=1416&originalType=binary&ratio=1&rotation=0&showTitle=true&size=22626&status=done&style=none&taskId=ub91bc74c-22a0-4b9d-a085-725c126541d&title=%E4%B8%9A%E5%8A%A1%E5%85%B3%E7%B3%BB "业务关系")
![业务形态和组件对应关系](https://cdn.nlark.com/yuque/0/2021/png/1512483/1637635257721-6de6987f-a94b-4f50-bf9c-2538acc26098.png#clientId=u53703c6e-da83-4&from=ui&id=ude34f806&originHeight=504&originWidth=1439&originalType=binary&ratio=1&rotation=0&showTitle=true&size=22443&status=done&style=none&taskId=u2fcbe5e7-17d3-4955-812b-68e3fc82448&title=%E4%B8%9A%E5%8A%A1%E5%BD%A2%E6%80%81%E5%92%8C%E7%BB%84%E4%BB%B6%E5%AF%B9%E5%BA%94%E5%85%B3%E7%B3%BB "业务形态和组件对应关系")
## 代码设计

![表单域元数据设计](https://cdn.nlark.com/yuque/0/2021/png/1512483/1637635312207-46a8341a-f420-4cc8-a681-1ddf9a0bfb0c.png#clientId=u53703c6e-da83-4&from=ui&id=uc0d9ce2b&originHeight=547&originWidth=1404&originalType=binary&ratio=1&rotation=0&showTitle=true&size=45982&status=done&style=none&taskId=ub23dbbe1-ae88-45db-ab16-10fe1553c24&title=%E8%A1%A8%E5%8D%95%E5%9F%9F%E5%85%83%E6%95%B0%E6%8D%AE%E8%AE%BE%E8%AE%A1 "表单域元数据设计")
![模块设计](https://cdn.nlark.com/yuque/0/2021/png/1512483/1637635349609-2e427be9-0b74-4931-8a2c-efea104e3e52.png#clientId=u53703c6e-da83-4&from=ui&id=u3d1177c7&originHeight=664&originWidth=1069&originalType=binary&ratio=1&rotation=0&showTitle=true&size=43236&status=done&style=none&taskId=u015a158d-41b6-407b-85f6-0fac4987446&title=%E6%A8%A1%E5%9D%97%E8%AE%BE%E8%AE%A1 "模块设计")
