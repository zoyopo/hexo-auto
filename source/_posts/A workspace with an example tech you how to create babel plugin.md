tags: [babel plugin,optional chain,tutorial]
categories: [babel]

---

# [how-to-implement-babel-optional-chain](https://github.com/zoyopo/how-to-implement-babel-optional-chain)
This is an example about how to implement `babel-optional-chain` plugin.Also,you can use it as simple babel plugin workspace.

## Quick start

- step 1

```bash
yarn or npm i
```

- step2 entry config
config your entry in `babel.config.js` like:

```javascript
module.exports = {
  plugins: [["./plugin-with-option-loose.js", { loose: true }]],
};
```

- step3

```bash
npm run watch
```

- step4

modify code in input.js and save,you will see genrated code in output.js

**If you wanna konw about option-chain implement progress,keep page moving up**

## The simplest implementation

![c1.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1633772642217-dfd9f0f8-638c-459a-a7ea-d51bab6b8bfc.png#clientId=ubb1d753a-e999-4&from=ui&id=u6de8c905&margin=%5Bobject%20Object%5D&name=c1.png&originHeight=466&originWidth=1062&originalType=binary&ratio=1&size=30343&status=done&style=none&taskId=u57b77761-4245-498f-8acc-460556cc3b2#id=XNLIC&originHeight=466&originWidth=1062&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

![c2.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1633772811427-f06aeee2-4ef4-456d-ba21-66836fbc50fc.png#clientId=ubb1d753a-e999-4&from=ui&id=uc6fc7e07&margin=%5Bobject%20Object%5D&name=c2.png&originHeight=380&originWidth=1085&originalType=binary&ratio=1&size=22211&status=done&style=none&taskId=u3df0642b-342e-4a8c-9d8d-4d0087184b3#id=pEBw6&originHeight=380&originWidth=1085&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

![c3.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1633772842675-2b777cb7-8ab4-46b5-8c0b-a35732e628c8.png#clientId=ubb1d753a-e999-4&from=ui&id=u3a872809&margin=%5Bobject%20Object%5D&name=c3.png&originHeight=423&originWidth=977&originalType=binary&ratio=1&size=24498&status=done&style=none&taskId=uf8cc1459-9c38-481e-9e6c-4852e682241#id=zhBMx&originHeight=423&originWidth=977&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)
Although simple, it achieves the simplest effect,But add another attribute and kneel immediately
![c4.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1633773096772-eeef896d-3571-4428-9483-3fd008d851a7.png#clientId=ubb1d753a-e999-4&from=ui&id=ub0b29d71&margin=%5Bobject%20Object%5D&name=c4.png&originHeight=529&originWidth=1094&originalType=binary&ratio=1&size=33335&status=done&style=none&taskId=uc4060f2f-66ac-4234-b92f-a3192af3fbe#id=Py9uH&originHeight=529&originWidth=1094&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

![c5.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1633773114901-667cbe3f-7fc1-4325-9aed-991a42ab3c19.png#clientId=ubb1d753a-e999-4&from=ui&id=udfeb8f7e&margin=%5Bobject%20Object%5D&name=c5.png&originHeight=470&originWidth=1315&originalType=binary&ratio=1&size=28247&status=done&style=none&taskId=u76935c61-6da0-42cd-b57e-9a442c052fc#id=RWctg&originHeight=470&originWidth=1315&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

## Repair normal '.' symbol also triggers "?." Conversion problem

Obviously, b.c does not need to judge b, so we continue to improve it

![c9.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1633917368341-42d91c27-e387-4829-8d9d-93a157a5c388.png#clientId=ucbc57d5c-bffa-4&from=ui&id=u5874a0aa&margin=%5Bobject%20Object%5D&name=c9.png&originHeight=510&originWidth=1021&originalType=binary&ratio=1&size=32878&status=done&style=none&taskId=u1cdb31f0-d5c8-44bc-9ee2-b06e8f4b480#id=M292P&originHeight=510&originWidth=1021&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)
Try again, it 's not bad

![c10.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1633917469210-b120f3e7-0f29-49f2-8798-e222c23d6636.png#clientId=ucbc57d5c-bffa-4&from=ui&id=uc129eec2&margin=%5Bobject%20Object%5D&name=c10.png&originHeight=239&originWidth=940&originalType=binary&ratio=1&size=14024&status=done&style=none&taskId=u1877a779-e435-452d-ac16-b83d1572af1#id=coTjR&originHeight=239&originWidth=940&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

But if you add an optional value
![c11.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1633917642510-d5f34feb-f2a6-4475-bae9-258bab849a0b.png#clientId=ucbc57d5c-bffa-4&from=ui&id=u4480e249&margin=%5Bobject%20Object%5D&name=c11.png&originHeight=253&originWidth=714&originalType=binary&ratio=1&size=14303&status=done&style=none&taskId=uf26e58ba-1b01-4231-8bdf-6da00b10a12#id=btKAo&originHeight=253&originWidth=714&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)
There will be problems
![c12.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1633917651958-c216df1a-71c7-4409-ac34-5e51fb081fcb.png#clientId=ucbc57d5c-bffa-4&from=ui&id=u4ed40422&margin=%5Bobject%20Object%5D&name=c12.png&originHeight=244&originWidth=1272&originalType=binary&ratio=1&size=15539&status=done&style=none&taskId=ude0be574-ee36-4238-a0c7-ba86be76ddd#id=OuUSP&originHeight=244&originWidth=1272&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

## Store calculation results

`a==null?undefined:a.b`It was repeated twice，**So let's optimize it again, as shown below**
![c13.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1633918062184-b40e1537-49b8-43ac-939e-fc8edabb2eb1.png#clientId=ucbc57d5c-bffa-4&from=ui&id=u475a33cc&margin=%5Bobject%20Object%5D&name=c13.png&originHeight=508&originWidth=1102&originalType=binary&ratio=1&size=34171&status=done&style=none&taskId=ua5d81bd7-bb35-445d-b2c5-d2f193099f6#id=ij6a6&originHeight=508&originWidth=1102&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)
First,we create a unique variable in the current scope and injects it into the scope. Then, while performing the ternary operation, we assign the result and store the operation result

![c14.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1633918200394-b294d27f-246c-4cb0-bd5b-873f20216c35.png#clientId=ucbc57d5c-bffa-4&from=ui&id=ud9023fe3&margin=%5Bobject%20Object%5D&name=c14.png&originHeight=323&originWidth=1257&originalType=binary&ratio=1&size=22233&status=done&style=none&taskId=ua328f2a2-e30c-4199-9d98-40d48d37551#id=yv9eX&originHeight=323&originWidth=1257&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

You think it's over, but there's another question
![c15.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1633919298883-89684e57-e47b-49bd-b23c-2e8c72b94321.png#clientId=ucbc57d5c-bffa-4&from=ui&id=uf3890796&margin=%5Bobject%20Object%5D&name=c15.png&originHeight=326&originWidth=1223&originalType=binary&ratio=1&size=25973&status=done&style=none&taskId=ua34e985f-9e3c-4d46-b117-36c2a1bef97#id=VX0h7&originHeight=326&originWidth=1223&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

## avoid 'undefined' being declared,like 'let undefined = 2'

In order to prevent a big brother from assigning an undefined value, we need to use Babel's API to build the undefined value in the scope
![c17.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1633919467079-b0b9d41d-f649-4ef4-867d-8405b98274bd.png#clientId=ucbc57d5c-bffa-4&from=ui&id=uaf640400&margin=%5Bobject%20Object%5D&name=c17.png&originHeight=533&originWidth=1070&originalType=binary&ratio=1&size=37508&status=done&style=none&taskId=uc31e2784-1278-4ad5-a45a-dd640d02d1c#id=z9YdJ&originHeight=533&originWidth=1070&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)
here is the result below
![c18.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1633919667172-f129df16-ee57-4720-882a-b2e9b4723aa7.png#clientId=ucbc57d5c-bffa-4&from=ui&id=u98a13f37&margin=%5Bobject%20Object%5D&name=c18.png&originHeight=303&originWidth=1223&originalType=binary&ratio=1&size=25557&status=done&style=none&taskId=uf563f335-3487-4c89-9822-75aff77f31b#id=DuYra&originHeight=303&originWidth=1223&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

## add '&&' form

Before we finish, let's play again. Babel's afferent option constructs another form
![c20.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1633919945726-7cbba642-44d5-4f9c-a4f2-2c585a5fb5d4.png#clientId=ucbc57d5c-bffa-4&from=ui&id=uda9a4340&margin=%5Bobject%20Object%5D&name=c20.png&originHeight=814&originWidth=1614&originalType=binary&ratio=1&size=136523&status=done&style=shadow&taskId=u5fd6ff88-7edc-46ea-9f97-a6e6076b7b7#id=mUAmH&originHeight=814&originWidth=1614&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

Parameter incoming
![c21.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1633920121928-635582b3-f57b-4996-bce1-97dc9658f8cd.png#clientId=ucbc57d5c-bffa-4&from=ui&id=uc7ca890e&margin=%5Bobject%20Object%5D&name=c21.png&originHeight=351&originWidth=1037&originalType=binary&ratio=1&size=25299&status=done&style=none&taskId=u0bccb021-b66f-4567-a4ed-4220435abe2#id=HqY2O&originHeight=351&originWidth=1037&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)
result outing
![c22.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1633920135249-0c257e41-268c-4d1f-af30-8cf52817c578.png#clientId=ucbc57d5c-bffa-4&from=ui&id=u39265a60&margin=%5Bobject%20Object%5D&name=c22.png&originHeight=350&originWidth=929&originalType=binary&ratio=1&size=20120&status=done&style=none&taskId=u96194ffd-4e63-4c87-92e3-f036216ef3c#id=FMhPk&originHeight=350&originWidth=929&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

![c23.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1633920135308-1e4218ca-229e-4903-92cf-db5eb470d31f.png#clientId=ucbc57d5c-bffa-4&from=ui&id=u3bd91012&margin=%5Bobject%20Object%5D&name=c23.png&originHeight=270&originWidth=922&originalType=binary&ratio=1&size=16373&status=done&style=none&taskId=u4ff86db8-9188-4b3f-bd04-cc1ff39759a#id=RQcTc&originHeight=270&originWidth=922&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)
Just a simple plug-in.

## Complete code

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
        let { object, property } = path.node;
        if (path.node.optional) {
          let tmp = path.scope.generateUidIdentifier("obj");
          path.scope.push({ id: tmp });
          let undef = path.scope.buildUndefinedNode();
          if (loose) {
            path.replaceWith(
              buildLoose({
                tmp,
                obj: object,
                expr: template.expression.ast`${tmp}.${property}`,
              })
            );
            return;
          }
          path.replaceWith(
            template.expression
              .ast`(${tmp} =${object})== null?${undef}:${tmp}.${property}`
          );
        } else {
          path.replaceWith(template.expression.ast`${object}.${property}`);
        }
      },
    },
  };
};
```

## Another implementation

This is the original author's implementation, but it is more complex to understand

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
        let tmp = path.scope.generateUidIdentifier("obj");
        path.scope.push({ id: tmp });

        while (!path.node.optional) {
          path = path.get("object");
        }

        let { object } = path.node;

        let memberExpr = tmp;
        do {
          memberExpr = t.memberExpression(
            memberExpr,
            path.node.property,
            path.node.computed
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
          );
        } else {
          let undef = path.scope.buildUndefinedNode();

          originalPath.replaceWith(
            template.expression.ast`
                    (${tmp} = ${object}) == null
                      ? ${undef}
                      : ${memberExpr}
                  `
          );
        }
      },
      //#endregion
    },
  };
};
```

[Github repo address](https://github.com/zoyopo/how-to-implement-babel-optional-chain)

Welcome to star.

## reference resources

[video address](https://www.youtube.com/embed/UeVq_U5obnE)

[origin author's github](https://github.com/nicolo-ribaudo/conf-holyjs-moscow-2019)
