## 引言
在开发中台过程中 我们的原型中有很多表单，antd有表单组件，但是粒度比较细，就单纯组件而言，无可厚非，但是在开发过程中，可能会造成代码不够聚合，有些表单公共逻辑无法提取，copy paste比较多，所以可以加以封装，搞一个兼容性和扩展性都契合项目本身的组件。

## 主要思路

我简单查阅了一番资料，参考了一下别人的封装方式，决定以`Col`和`FormItem`作为基本单元，进行表单的搭建，主要原因

- 可以将`FormItem`的信息和`Col`的信息以对象方式传入，这样我们可以更加专注于组件的包含的信息 减少CV代码可能导致的bug
- `Col`可以进行布局的调整，可以调整表单单元位置和所占宽度
- 表单的组件形式是不定的，可能是input也有可能是select,所以可以进行外部传入，通用属性可以内部修改
- 复用性高，可以用它来实现一个纯提交表单 和列表组件结合成可搜索表单  或者其他任何项目里需要自定义的一个表单

## 实现细节

### 抽象的`FormItemInfo`

```typescript
export interface FormItemInfo {
  name: string, //label
  id: string, // 属性
  colLayOut?: object,// 列布局
  formItemLayout?: object,// 表单项布局
  Comp?: any, // 传入的组件 不传默认input
  ruleArr?: RuleObj[],// 验证规则
  initialValue?: string, // 初始值
  required?: boolean, // 是否必填
  CompExtraProps?: object // 传入组件额外属性
  isDetail?: boolean //是否需要被getFieldDecorator托管
}
```

### baseForm组件

```typescript
 * @param name 表单项label
 * @param id 属性字段名
 * @param colLayOut 列布局
 * @param formItemLayout 表单项布局
 * @param Comp 传入的组件 不传默认input
 * @param ruleArr 验证规则 长度只需传入｛max:xxx｝验证信息是统一的
 * @param initialValue 初始值 编辑时推荐使用antd的mapPropsToFields 不要手动设置回显值
 * @param required 是否必填 默认否 因为要统一写验证提示
 * @param CompExtraProps 组件额外属性
  const mapToFormItem = (val: FormItemInfo) => {
 ......
 // 是否传入组件
    const hasCompType = Comp && Comp.type
    // 根据组件类型 给出提示文字
    const msg = getMsgByCompType(hasCompType && hasCompType.name, name)
    // 判断是不是select组件 是的话 调整宽度样式
    const mixStyle = fixStyleByCompType(hasCompType && hasCompType.name)
    // 生成验证规则
    const rules = getFormRules(ruleArr || [], required || false, name)

  return (
      <Col {...(colLayOut || defaultColSpan)} key={id}>
        {!isDetail ?
          <FormItem label={name} key={id} {...(formItemLayout || {})}>
            {/*用cloneElement方法给传入的组件加新属性*/}
            {
              getFieldDecorator(id, {
                initialValue: initialValue || '',
                rules,
              })(Comp ? React.cloneElement(Comp, { placeholder: msg, ...mixStyle, ...CompExtraProps }) : <Input type="text" placeholder={msg}/>)
            }
          </FormItem> : <FormItem label={name} key={id} {...(formItemLayout || {})}> {Comp}</FormItem>}
      </Col>
    )
｝
```

### 统一方法

- 根据组件类型 给出提示文字

```typescript
  /**根据组件类型 给出提示文字
   * @param type 组件类型 根据传入组件的函数名判断 在map里维护
   * @param name label名称
   * @returns 提示文字
   * */
  const getMsgByCompType = (name: string, type?: string): string => {
    if (!type) {
      return `请输入${name}`
    }
    const map: { [props: string]: string } = {
      Select: '请选择',
      Input: '请输入',
      default: '请输入',
    }
    return `${map[type] || map.default}${name}`
  }
```

- 生成验证规则

```
  // 生成验证规则
 * @param ruleArr 验证规则 长度只需传入｛max:xxx｝验证信息是统一的
 * @param name 表单项label
 * @param required 是否必填 默认否 因为要统一写验证提示
  const getFormRules = (val: RuleObj[], required: boolean, name: string) => {
    // 根据max生成最长输入提示
    const ruleArr: object[] = []
    val.forEach(item => {
      if ('max' in item) {
        ruleArr.push({ ...item, message: `输入信息不能超过${item.max}字` })
      } else {
        ruleArr.push(item)
      }
    })
    // 根据name生成报错提示
    return [{ required: required, message: `${name}不能为空` }, ...ruleArr]
```

- 根据组件类型 给出统一样式修改

```typescript
  /**根据组件类型 给出统一样式修改
   * @param type 组件类型 根据传入组件的函数名判断 在map里维护
   * @returns 样式对象
   * */
  const fixStyleByCompType = (type?: string): object => {
    if (!type) {
      return {}
    }
    const map: { [props: string]: object } = {
      Select: { style: { width: '100%' } },
      default: {},
    }
    return map[type] || map.default
  }
```

注意：**当出现表单显示一个详情文字或者一个按钮 此时需要用`isDetail`干掉`getFieldDecorator托管`**

### 实现提交表单SubmitForm

```typescript
 * @param form -`antd`的form
 * @param title - 主标题
 * @param subTitle - 分组标题
 * @param FormItemInfos - 二维数组 顺序和数量和分组标题对应 存放表单项信息
 * @param isLoading - `dva-loading` 执行effects过程中的loading
 * @param onSubmit - 传入的submit方法
 * @param buttonArea - 可选 不传默认数提交和取消 传入覆盖
 * @param children - 在表单下面 按钮区域上面的传入内容
 * @param formLayOutType - 布局方式 详情见`getLayOutByType`方法
 * @returns ReactNode
......
 <Form onSubmit={onSubmit}>

        <SubmitLayOut subTitle={subTitle} title={title} renderFormArea={renderFormArea}>
          {children}
          {buttonArea?buttonArea: <Row type="flex" gutter={24} justify="center">
            <Col>
              <Button type="primary" htmlType="submit" loading={isLoading}>提交</Button>
            </Col>
            <Col>
              <Button type="default" onClick={() => router.go(-1)}>取消</Button>
            </Col>
          </Row>}
        </SubmitLayOut>

    </Form>
```

对`BaseForm`的调用在`renderFormArea`方法里

```typescript
// 水平和垂直布局
export enum FormLayOutType {
  normal = 'normal',
  vertical = 'vertical'
}

const renderFormArea = (idx: number) => {
    // 在这边加上normal表单的layout
    const FormItemInfo = FormItemInfos[idx]
  // 根据传入参数 返回布局类型
    const _FormItemInfo = getLayOutByType(formLayOutType||FormLayOutType.normal, FormItemInfo)
    return <BaseForm formItems={_FormItemInfo} form={form}/>
  }
```

`FormItemInfos`这里是二维数组，二维分别承载分组信息和表单项信息 而且在这里用`getLayOutByType`封装常用的水平 垂直布局方式

这里结合了我们业务里的表单布局 大标题 和小标题对表单区域进行分组，我将布局单独搞了个`SubmitLayOut`组件 将渲染逻辑和样式组件在其中实现 这样也方便更换成其他`LayOut`

```typescript
// SubmitLayOut
 * @param title - 主标题
 * @param subTitle - 分组标题
 * @param renderFormArea - 根据分组的顺序 渲染表单区域
 * @param children - 传入内容
......
  <Card title={title} bordered={false} className={styles["special-card"]}>
      <List itemLayout="vertical" className="special-list">
        {subTitle.map((item,idx) => {
          return (
            <>
              <Meta title={item}/>
              <List.Item>
                {renderFormArea(idx)}
              </List.Item>
            </>
          )
        })}
      </List>
      {children}
    </Card>
```


### SubmitForm 完整代码

```typescript
/**
 * 提交表单
 * @param form -`antd`的form
 * @param title - 主标题
 * @param subTitle - 分组标题
 * @param FormItemInfos - 二维数组 顺序和数量和分组标题对应 存放表单项信息
 * @param isLoading - `dva-loading` 执行effects过程中的loading
 * @param onSubmit - 传入的submit方法
 * @param buttonArea - 可选 不传默认数提交和取消 传入覆盖
 * @param children - 在表单下面 按钮区域上面的传入内容
 * @param formLayOutType - 布局方式 详情见`getLayOutByType`方法
 * @param model - 编辑回显传入的值
 * @returns ReactNode
 */
import React, { ReactNode, useState } from 'react';
import BaseForm, { FormItemInfo } from '../BaseForm/BaseFormArea';
import { Button, Col, Form, message, Row } from 'antd';
import { If } from 'jsx-control-statements';
import SubmitLayOut from './SubmitLayOut';
import router from 'umi/router';
import { unShowSearchToken } from '../../../constants/index';
import moment from 'moment';
import { isDeepEmptyObj } from '@/utils/common';
import addOperLog from '@/utils/operLog';
import { flatten } from 'lodash';
import { useDispatch } from 'react-redux';
import styles from '@/pages/policy/index.less';

interface IProps {
  form: any;
  title?: string | ReactNode; // 新增页 主标题
  subTitle?: string[]; // 新增页 分组标题
  FormItemInfos: FormItemInfo[][]; // 二维数组 应对项目里面表单内容分组
  isLoading?: boolean; // dva的loading
  onSubmit: Function;
  buttonArea?: ReactNode; // 重写按钮区域
  children?: ReactNode;
  formLayOutType?: string;
  model?: { [props: string]: string };
  onCancel?: Function;
}

// 水平和垂直布局
export enum FormLayOutType {
  normal = 'normal',
  vertical = 'vertical',
}
enum CompType {
  Select = 'Select',
  PickerWrapper = 'PickerWrapper',
  Cascader = 'Cascader',
  Money = 'Money',
}

enum initVal {
  Zero = '0',
}
const detailStyle = {
  width: '250px',
  overflow: 'hidden',
  textOverflow: 'ellipsis',
  display: 'block',
  whiteSpace: 'nowrap',
};
const SubmitForm = (props: IProps) => {
  const dispatch = useDispatch();
  // const [submitButtonisDisabled,setSubmitButtonisDisabled]=useState(false)
  const {
    title,
    subTitle,
    FormItemInfos,
    form,
    isLoading,
    buttonArea,
    children,
    formLayOutType,
    model,
    onCancel,
  } = props;

  // 根据分组的顺序 渲染表单区域
  const renderFormArea = (idx: number) => {
    console.log('FormItemInfos===>', FormItemInfos);
    console.log('subTitle====>', subTitle);
    const FormItemInfo = FormItemInfos[idx];
    // 在这边加上normal表单的layout
    const _FormItemInfo = setFeatureToFormItemInfo(FormItemInfo);
    return <BaseForm formItems={_FormItemInfo} form={form} />;
  };

  // 鼠标移入 如果文本溢出 。。。 显示完整的
  const onDetailMouseEnter = (e: any) => {
    const target = e.target;
    const containerLength = target.offsetWidth;
    const textLength = target.scrollWidth;
    const text = target.innerHTML;
    if (textLength > containerLength) {
      target.setAttribute('title', text);
    } else {
      target.removeAttribute('title');
    }
  };

  /**
   * 根据传入参数 返回布局类型
   * @param type - 表单布局类型 可能为从左到右 也可能为自上而下 默认前者
   * @param FormItemInfo - 表单项信息
   * @returns FormItemInfo
   */
  const getLayOutByType = (type: string, FormItemInfo: FormItemInfo[]): FormItemInfo[] => {
    const map: { [props: string]: Function } = {
      // 从左至右的布局
      [FormLayOutType.normal]: () => {
        return FormItemInfo.filter(item => [undefined, true].includes(item.visible)).map(
          (item, idx: number) => {
            if (idx % 3 === 0) {
              return {
                colLayOut: { span: 6 },
                ...item,
              };
            } else {
              return {
                colLayOut: { span: 6, offset: 2 },
                ...item,
              };
            }
          },
        );
      },
      // 自上而下的布局
      [FormLayOutType.vertical]: () => {
        return FormItemInfo.map(item => ({
          colLayOut: { span: 16, offset: 4 },
          formItemLayout: {
            labelCol: { span: 6 },
            wrapperCol: onCancel ? { span: 18 } : { span: 8 },
            ...item.formItemLayout,
          },
          ...item,
        }));
      },
    };
    return map[type]();
  };

  /**
   * 根据组件类型加上额外属性
   *    *
   * @param FormItemInfo - 表单项信息
   * @returns FormItemInfo
   */
  const setExtrProps = (FormItemInfos: FormItemInfo[]): FormItemInfo[] => {
    return FormItemInfos.map(item => {
      // 是否传入组件
      // const hasCompType = item.Comp && item.Comp.type
      // console.log('CompType',hasCompType,hasCompType&&hasCompType.name )
      const type = item.type;
      // 允许清除
      const commonFeature = { allowClear: !item.required };
      // 如果是‘Select’组件且name不在配置之中
      if (type && type === CompType.Select) {
        if (!unShowSearchToken.includes(item.id)) {
          return {
            CompExtraProps: { ...commonFeature, showSearch: true, optionFilterProp: 'children' },
            ...item,
          };
        } else {
          return { CompExtraProps: { ...commonFeature }, ...item };
        }
      } else if (type && type === CompType.Cascader && item.id === 'provincesRegions') {
        return {
          CompExtraProps: {
            ...commonFeature,
            fieldNames: { label: 'parValue', value: 'parKey', children: 'childParameter' },
          },
          ...item,
        };
      } else if (type && type === CompType.Money) {
        return {
          CompExtraProps: {
            min: 0,
            max: 9999999999.99,
            precision: 2,
          },
          ruleArr: [
            { type: 'number', max: 9999999999.99 },
            { type: 'number', min: 0 },
          ],
          ...item,
        };
      } else {
        return item;
      }
    });
  };
  // 编辑的时候加上回显值
  const setInitValueWhenEdit = (FormItemInfos: FormItemInfo[]): FormItemInfo[] => {
    // console.log(isDeepEmptyObj(model))
    if (model && !isDeepEmptyObj(model)) {
      return FormItemInfos.map((item, idx: number) => {
        const hasCompType = item.type;
        // 下拉框 后端给的枚举值是字符串 回显是number 在这里统一
        let detailValue: string | number | moment.Moment | object[] = '';
        // 如果是多层级的数据 在这里转换
        if (item.id.includes('.')) {
          const wrapperArr = item.id.split('.');
          // @ts-ignore
          detailValue = wrapperArr.reduce(
            (previous, current) => previous && previous[current],
            model,
          );
          // 回显自动加给uploads组件加规定格式的值
        } else if (item.id === 'fileIdEn') {
          detailValue = model['fileIdEn']
            ? [
                {
                  fileIdEn: model['fileIdEn'],
                  fileName: model['fileName'],
                  fileUrl: model['fileUrl']
                    ? `/api${model['fileUrl']}`
                    : `/api/file/downloadFile?fileIdEn=${model['fileIdEn']}`,
                },
              ]
            : [];
        } else {
          detailValue = model[item.id];
        }
        if (hasCompType && hasCompType === CompType.Select) {
          // 判断是不是null null不转 /判断是不是0 是0转换
          return {
            ...item,
            initialValue:
              detailValue || `${detailValue}` === initVal.Zero ? `${detailValue}` : undefined,
          };
          // 如果是日期组件 回显值就用moment处理
        } else if (hasCompType && hasCompType === CompType.PickerWrapper) {
          return { ...item, initialValue: detailValue ? moment(detailValue) : undefined };
        } else if (
          hasCompType &&
          hasCompType === CompType.Cascader &&
          item.id === 'provincesRegions'
        ) {
          return {
            ...item,
            initialValue: model['province']
              ? [model['province'], model['city'], model['county']]
              : undefined,
          };
        } else if (item.isDetail) {
          // // 表单中存在 显示内容
          // if (detailValue||(detailValue===0)) {

          //   return { Comp: item.id.includes('fileIdEn') ? <a target="_blank" href={`/api/file/downloadFile?fileIdEn=${model['fileIdEn']}`}>{model['fileName']}</a> : <span style={{ whiteSpace: 'nowrap' }}>{detailValue}</span>, isDetail: true, ...item }
          // } else {

          return {
            ...item,
            CompExtraProps: { style: detailStyle },
            Comp: item.initialValue ? (
              <span onMouseEnter={onDetailMouseEnter}>{item.initialValue}</span>
            ) : (
              <span
                onMouseEnter={onDetailMouseEnter}
                dangerouslySetInnerHTML={{ __html: '&nbsp;' }}
              ></span>
            ),
          };
          // }
        }
        // 判断是0的情况 显示0
        else
          return {
            ...item,
            initialValue:
              item.initialValue ||
              (`${detailValue}` === initVal.Zero ? `${detailValue}` : detailValue),
          };
      });
    } else {
      // 新增 填入需要显示的值
      return FormItemInfos.map(item => {
        if (item.isDetail) {
          return {
            ...item,
            CompExtraProps: { style: detailStyle },
            Comp:
              item.initialValue || `${item.initialValue}` === initVal.Zero ? (
                <span onMouseEnter={onDetailMouseEnter}>{item.initialValue}</span>
              ) : (
                <span
                  onMouseEnter={onDetailMouseEnter}
                  dangerouslySetInnerHTML={{ __html: '&nbsp;' }}
                ></span>
              ),
          };
        } else {
          return item;
        }
      });
    }
  };

  const setStyleToFormItem = (FormItemInfos: FormItemInfo[]): FormItemInfo[] => {
    return FormItemInfos.map(item => {
      if (item.id?.includes('fileIdEn')) {
        return { ...item, className: styles.policyUpload };
      } else {
        return item;
      }
    });
  };
  const setFeatureToFormItemInfo = (FormItemInfos: FormItemInfo[]): FormItemInfo[] => {
    let result: FormItemInfo[];
    result = getLayOutByType(formLayOutType || FormLayOutType.normal, FormItemInfos);

    result = setExtrProps(result);

    result = setInitValueWhenEdit(result);

    result = setStyleToFormItem(result);

    return result;
  };
  // 预处理提交的值
  const onSubmitedValuesPreHandle = (values: { [props: string]: string }) => {
    const result: { [props: string]: string } = {};
    for (let key in values) {
      if (typeof values[key] === 'string') {
        result[key] = values[key].trim();
      } else {
        result[key] = values[key];
      }
    }
    return result;
  };

  /**
   * @summary 调用传入的`onSubmit`函数
   * @returns void
   */
  const onSubmit = (e: any): void => {
    e.preventDefault();
    const {
      form: { validateFieldsAndScroll },
      onSubmit,
    } = props;

    validateFieldsAndScroll((err: boolean, values: { [props: string]: string }) => {
      if (!err) {
        console.log('submiting values:', values);
        let submitValues = { ...values };
        // 前置处理DatePicker值
        const formFilterArr = flatten(FormItemInfos).filter(
          item => item.type === CompType.PickerWrapper,
        );
        formFilterArr.forEach(
          item => (submitValues[item.id] = moment(values[item.id]).format('YYYY-MM-DD')),
        );
        // 前置处理Uploads的值
        if (values.hasOwnProperty('fileIdEn')) {
          submitValues = {
            ...submitValues,
            fileIdEn: values.fileIdEn[0] && values.fileIdEn[0].fileIdEn,
            fileName:
              values.fileIdEn[0] && (values.fileIdEn[0].fileName || values.fileIdEn[0].name),
          };
        }
        const cb = (path: string, data: boolean | string) => {
          console.log('rst', data);
          let character: string;
          if (path.includes('?')) {
            character = '&';
          } else {
            character = '?';
          }
          if (data) {
            message.success('提交成功');

            setTimeout(() => {
              dispatch({
                type: 'app/updateState',
                payload: { isAuthorized: true },
              });
              router.push(path + `${character}detailIdEn=${data}`);
            });
          } else {
            dispatch({
              type: 'app/updateState',
              payload: { isAuthorized: true },
            });
          }
        };
        // 新增编辑操作加上对应操作日志
        // operType add/update
        // moduleName 模块汉字
        // overrideValues 外部传入的当前values 防止有不在表单里的外部参数
        const onOperLogSaveCallback = (
          operType: string,
          moduleName: string,
          operParams: { targetType: string; targetIdEn: string },
          overrideValues: {},
        ) => {
          // 编辑
          if (model && !isDeepEmptyObj(model)) {
            addOperLog(
              operType,
              moduleName,
              operParams,
              model,
              overrideValues || values,
              FormItemInfos,
            );
          } else {
            addOperLog(operType, moduleName, operParams);
          }
        };
        const handledValues = onSubmitedValuesPreHandle(submitValues);
        // 搞出loading
        dispatch({
          type: 'app/updateState',
          payload: { isAuthorized: false },
        });
        onSubmit(handledValues, cb, onOperLogSaveCallback);
      }
    });
  };

  const onBtnCancel = () => {
    if (onCancel) {
      onCancel();
    } else {
      router.go(-1);
    }
  };

  return (
    <Form onSubmit={onSubmit} labelAlign="right">
      <SubmitLayOut subTitle={subTitle} title={title} renderFormArea={renderFormArea}>
        {children}
        {buttonArea ? (
          buttonArea
        ) : (
          <Row type="flex" gutter={24} justify="center">
            <Col>
              <Button type="primary" htmlType="submit" loading={isLoading}>
                提交
              </Button>
            </Col>
            <If condition={(model && !isDeepEmptyObj(model)) || onCancel}>
              <Col>
                <Button loading={isLoading} type="default" onClick={onBtnCancel}>
                  取消
                </Button>
              </Col>
            </If>
          </Row>
        )}
      </SubmitLayOut>
    </Form>
  );
};

export default SubmitForm;

```

## FormDetail--表单详情页
```typescript
/**
 * 表单详情
 * @param form -`antd`的form
 * @param title - 主标题
 * @param subTitle - 分组标题
 * @param FormItemInfos - 二维数组 顺序和数量和分组标题对应 存放表单项信息
 * @param isLoading - `dva-loading` 执行effects过程中的loading
 * @param onSubmit - 传入的submit方法
 * @param buttonArea - 可选 不传默认数提交和取消 传入覆盖
 * @param children - 在表单下面 按钮区域上面的传入内容
 * @param formLayOutType - 布局方式 详情见`getLayOutByType`方法
 * @param model - 详情回显传入的值
 * @returns ReactNode
 */
import React, { ReactNode } from 'react';
import BaseForm from '../BaseForm/BaseFormArea';
import { Form, Row, Col, Button, message } from 'antd';
import SubmitLayOut from './SubmitLayOut';
import { FormItemInfo } from '../BaseForm/BaseFormArea';
import styles from './index.less';

interface IProps {
  title?: string; // 新增页 主标题
  subTitle?: string[]; // 新增页 分组标题
  FormItemInfos: FormItemInfo[][]; // 二维数组 应对项目里面表单内容分组
  buttonArea?: ReactNode; // 重写按钮区域
  children?: ReactNode;
  formLayOutType?: FormLayOutType;
  model: { [props: string]: string };
}

// 水平和垂直布局
export enum FormLayOutType {
  normal = 'normal',
  vertical = 'vertical',
}

enum CompType {
  Select = 'Select',
  PickerWrapper = 'PickerWrapper',
}

const detailStyle = {
  width: '250px',
  overflow: 'hidden',
  textOverflow: 'ellipsis',
  display: 'block',
  whiteSpace: 'nowrap',
};

const FormDetail = (props: IProps) => {
  const { title, subTitle, FormItemInfos, buttonArea, children, formLayOutType, model } = props;
  // 根据分组的顺序 渲染表单区域
  const renderFormArea = (idx: number) => {
    const FormItemInfo = FormItemInfos[idx];
    // 在这边加上normal表单的layout
    const _FormItemInfo = setFeatureToFormItemInfo(FormItemInfo);
    return <BaseForm isDetail formItems={_FormItemInfo} />;
  };
  // 鼠标移入 如果文本溢出 。。。 显示完整的
  const onDetailMouseEnter = (e: any) => {
    const target = e.target;
    const containerLength = target.offsetWidth;
    const textLength = target.scrollWidth;
    const text = target.innerHTML;
    if (textLength > containerLength) {
      target.setAttribute('title', text);
    } else {
      target.removeAttribute('title');
    }
  };

  /**
   * 根据传入参数 返回布局类型
   * @param type - 表单布局类型 可能为从左到右 也可能为自上而下 默认前者
   * @param FormItemInfo - 表单项信息
   * @returns FormItemInfo
   */
  const getLayOutByType = (type: string, FormItemInfo: FormItemInfo[]): FormItemInfo[] => {
    const map: { [props: string]: Function } = {
      // 从左至右的布局
      [FormLayOutType.normal]: () => {
        return FormItemInfo.filter(item => [undefined, true].includes(item.visible)).map(
          (item, idx: number) => {
            if (idx % 3 === 0) {
              return {
                colLayOut: { span: 6 },
                formItemLayout: { labelCol: { span: 5 }, wrapperCol: { span: 5 } },
                ...item,
              };
            } else {
              return {
                colLayOut: { span: 6, offset: 2 },
                formItemLayout: { labelCol: { span: 5 }, wrapperCol: { span: 5 } },
                ...item,
              };
            }
          },
        );
      },
      // 自上而下的布局
      [FormLayOutType.vertical]: () => {
        return FormItemInfo.map(item => ({
          colLayOut: { span: 16, offset: 6 },
          formItemLayout: {
            labelCol: { span: 5 },
            wrapperCol: { span: 8 },
            ...item.formItemLayout,
          },
          ...item,
        }));
      },
    };
    return map[type]();
  };

  /**
   * 给表单加上详情属性
   *    *
   * @param FormItemInfo - 表单项信息
   * @returns FormItemInfo
   */
  const setDetailStyle = (FormItemInfos: FormItemInfo[]): FormItemInfo[] => {
    return FormItemInfos.map(item => {
      let detailValue;
      if (item.id.includes('.')) {
        const wrapperArr = item.id.split('.');
        // @ts-ignore
        detailValue = wrapperArr.reduce(
          (previous, current) => previous && previous[current],
          model,
        );
      } else {
        detailValue = model[item.id];
      }
      if (item.render) {
        const renderValueArr =
          item.renderParams &&
          item.renderParams.map(item => {
            const _wrapperArr = item.split('.');
            // @ts-ignore
           const _detailValue = _wrapperArr.reduce(
              (previous, current) => previous && previous[current],
              model,
            );
            return _detailValue;
          });
        return {
          Comp: item.render(detailValue, renderValueArr),
          isDetail: true,
          ...item,
          CompExtraProps: { style: { whiteSpace: 'nowrap' } },
        };
      }
      if (item.Comp) {
        return { isDetail: true, ...item, CompExtraProps: { style: { whiteSpace: 'nowrap' } } };
      }
      if (!detailValue && detailValue !== 0) {
        return {
          Comp: (
            <span
              dangerouslySetInnerHTML={{ __html: '&nbsp;' }}
              style={{ whiteSpace: 'nowrap' }}
            ></span>
          ),
          isDetail: true,
          ...item,
        };
      }
      // 对 fileIdEn特殊处理
      if (item.id.includes('fileIdEn')) {
        // 如果是数组里的
        const fileIdEnIdArr = item.id.split('.');
        if (fileIdEnIdArr.length > 1) {
          fileIdEnIdArr.splice(fileIdEnIdArr.length - 1, 1, 'fileName');
          const fileNameValue = fileIdEnIdArr.reduce(
            (previous, current) => previous && previous[current],
            model,
          );
          return {
            Comp: (
              <a
                onMouseEnter={onDetailMouseEnter}
                style={detailStyle}
                target="_blank"
                href={`/api/file/downloadFile?fileIdEn=${detailValue}`}
              >
                {fileNameValue}
              </a>
            ),
            isDetail: true,
            ...item,
          };
        }
        return {
          Comp: (
            <a
              onMouseEnter={onDetailMouseEnter}
              style={detailStyle}
              target="_blank"
              href={`/api/file/downloadFile?fileIdEn=${model['fileIdEn']}`}
            >
              {model['fileName']}
            </a>
          ),
          isDetail: true,
          ...item,
        };
      }
      return {
        Comp: item.id.includes('fileIdEn') ? (
          <a
            onMouseEnter={onDetailMouseEnter}
            style={detailStyle}
            target="_blank"
            href={`/api/file/downloadFile?fileIdEn=${model['fileIdEn']}`}
          >
            {model['fileName']}
          </a>
        ) : (
          <span onMouseEnter={onDetailMouseEnter} style={detailStyle}>
            {detailValue}
          </span>
        ),
        isDetail: true,
        ...item,
      };
    });
  };

  const setFeatureToFormItemInfo = (FormItemInfos: FormItemInfo[]): FormItemInfo[] => {
    let result: FormItemInfo[];
    result = getLayOutByType(formLayOutType || FormLayOutType.normal, FormItemInfos);
    // result = setShowSearch(result)
    result = setDetailStyle(result);
    return result;
  };

  return (
    <Form labelAlign="left" className={styles.detailForm}>
      <SubmitLayOut subTitle={subTitle} title={title} renderFormArea={renderFormArea}>
        {children}
        {buttonArea}
      </SubmitLayOut>
    </Form>
  );
};

export default FormDetail;

```
## searchForm 搜索列表

```typescript
/**
 * @Author ZhangYunpeng0126
 * @Description 搜索表单
 * @param FormItemInfos - 一维数组 存放表单项内容对象
 * @param loading  - `dva-loading` 执行effects过程中的loading
 * @param history 路由信息
 * @param title - 主标题
 */
import React, { ReactNode, useState } from 'react';
import SearchLayOut from './SearchLayOut';
import BaseForm, { FormItemInfo } from '../BaseForm/BaseFormArea';
import { Row, Col, Button, Icon } from 'antd';
import router from 'umi/router';
import moment from 'moment';
// @ts-ignore
import queryString from 'query-string';
import styles from '@/components/XyzForm/SearchForm/index.less';
import { unShowSearchToken } from '../../../constants/index';
import { PaginationConfig } from 'antd/lib/pagination';
import Item from 'antd/lib/list/Item';

// 搜索表单 默认的formItemLayout
const formItemLayout = {
  labelCol: { span: 8 },
  wrapperCol: { span: 16 },
};

enum initValue {
  Empty = '',
  All = '-1',
}

enum matchKeyName {
  Time = 'Time',
  Date = 'Date',
}

const dateFormatSt = 'YYYY-MM-DD 00:00:00';
const dateFormatEd = 'YYYY-MM-DD 23:59:59';
const dateFormat = 'YYYY-MM-DD';
interface TableProps {
  loading: boolean;
  columns: [];
  dataSource: [];
}

interface IProps {
  FormItemInfos: FormItemInfo[];
  form: { getFieldsValue: Function; resetFields: Function };
  loading?: boolean;
  history: { location: { pathname: string; query: { [props: string]: string } } };
  title?: string | ReactNode;
  children: any;
  tableProps?: TableProps;
  extraButtonArea?: ReactNode;
  handleSearch?: Function;
}
enum CompType {
  Select = 'Select',
  PickerWrapper = 'PickerWrapper',
  Cascader = 'Cascader',
  RangePicker = 'RangePicker',
  MonthPicker = 'MonthPicker',
  RangeInputNumber = 'RangeInputNumber',
}

enum initVal {
  Zero = '0',
}
function SearchForm(props: IProps) {
  const { FormItemInfos, form, loading, history, title, children, extraButtonArea, handleSearch } = props;
  const [expand, setExpand] = useState(true);

  const OnpaginationChange = (
    pagination: PaginationConfig,
    filtersArg: object,
    sorter: { order: string; field: string },
  ) => {
    const { pathname, query } = history.location;
    const params: { [propName: string]: string | number | undefined } = {
      page: pagination.current === 1 ? undefined : pagination.current,
      limit: pagination.pageSize === 10 ? undefined : pagination.pageSize,
    };
    if (sorter && sorter.field) {
      const { field, order } = sorter;
      params.orderType = order === 'ascend' ? 'ASC' : 'DESC';
      params.orderBy = field;
    }
    router.push({
      pathname,
      search: queryString.stringify({
        ...query,
        ...params,
      }),
    });
  };
  /**
   * @Author ZhangYunpeng0126
   * @Description 搜索方法 拼上路由方法 dva监听到去请求
   * @Date 11:53 2019/12/5
   */

  const onSearch = (): void => {
    const { pathname } = history.location;
    const params = getParams();
    // debugger
    router.push({
      pathname,
      search: queryString.stringify({
        ...params,
      }),
    });
  };

  const toggle = () => {
    setExpand((prev: boolean) => !prev);
  };
  /**
   * @Author ZhangYunpeng0126
   * @Description 清除搜索内容 并清除url上拼的参数
   * @Date 11:37 2019/12/5
   */
  const onClearSearch = (): void => {
    const { pathname } = history.location;
    form.resetFields();
    router.push({
      pathname,
    });
  };
  /**
   * @Author ZhangYunpeng0126
   * @Description 获取搜索键值对象
   * @Date 15:00 2019/12/9
   * @return object 搜索键值对象
   */
  const getParams = (): { [propName: string]: string } => {
    const { getFieldsValue } = form;
    const fields = getFieldsValue();
    console.log(fields);

    for (const key in fields) {
      const formItemInfo = FormItemInfos.find(item => item.id === key);
      let type;
      if (formItemInfo) {
        type = formItemInfo.type;
      }
      if ([initValue.All, initValue.Empty].includes(fields[key])) {
        delete fields[key];
      } else if (type === CompType.RangePicker) {
        fields[key + 'Start'] = fields[key][0]?.format(dateFormatSt);
        fields[key + 'End'] = fields[key][1]?.format(dateFormatEd);
        delete fields[key];
      } else if (type === CompType.PickerWrapper) {
        fields[key] = fields[key].format(dateFormat);
      } else if (type === CompType.MonthPicker) {
        fields[key] = fields[key].format('YYYY-MM');
      } else if (type === CompType.RangeInputNumber) {
        console.log('RangeInputNumber', fields[key].start);
        // debugger
        fields[key + 'Start'] = fields[key]?.start;
        fields[key + 'End'] = fields[key]?.end;
        console.log('RangeInputNumber', fields[key + 'Start']);
        delete fields[key];
      } else if (typeof fields[key] === 'string') {
        fields[key] = fields[key].trim();
      }
    }
    return fields;
  };
  const getLayOut = (FormItemInfos: FormItemInfo[]): FormItemInfo[] => {
    return FormItemInfos.map(item => ({
      colLayOut: { span: 8 },
      formItemLayout: formItemLayout,
      ...item,
    }));
  };

  const setInitValue = (FormItemInfos: FormItemInfo[]): FormItemInfo[] => {
    // initialValue
    const { query } = history.location;

    return FormItemInfos.map(item => {
      if (item.type === CompType.RangePicker) {
        return {
          initialValue: query[`${item.id}Start`] &&
            query[`${item.id}End`] && [
              moment(query[`${item.id}Start`]),
              moment(query[`${item.id}End`]),
            ],
          ...item,
        };
      } else if (item.type === CompType.PickerWrapper) {
        return { initialValue: query[item.id] && moment(query[item.id]), ...item };
      } else if (item.type === CompType.Select) {
        console.log({
          initialValue: `${query[item.id]}` === initVal.Zero ? `${query[item.id]}` : '',
          ...item,
        });
        return {
          initialValue: `${query[item.id]}` === initVal.Zero ? `${query[item.id]}` : '',
          ...item,
        };
      } else if (item.type === CompType.RangeInputNumber) {
        return {
          initialValue: { start: query[`${item.id}Start`], end: query[`${item.id}End`] },
          ...item,
        };
      } else {
        return { initialValue: query[item.id], ...item };
      }
    });
  };

  /*
   * @Author: zhangyunpeng
   * @Date: 2020-01-13 09:11:01
   * @Last Modified by:   zhangyunpeng
   * @Last Modified time: 2020-01-13 09:11:01
   * @Summary: 根据Select类型塞入是否可搜索、可清空feture
   * @param: FormItemInfo[]
   * @returns: FormItemInfo[] */
  const setShowSearch = (FormItemInfos: FormItemInfo[]): FormItemInfo[] => {
    return FormItemInfos.map(item => {
      // 是否传入组件
      const type = item.type;
      // 加入可清空属性
      const commonFeature = { allowClear: true };
      // 如果是‘Select’组件且name不在配置之中
      if (type && type === CompType.Select) {
        if (!unShowSearchToken.includes(item.id)) {
          return {
            CompExtraProps: { ...commonFeature, showSearch: true, optionFilterProp: 'children' },
            ...item,
          };
        } else {
          return { CompExtraProps: { ...commonFeature }, ...item };
        }
      } else {
        return item;
      }
    });
  };
  const setExpandFormItems = (FormItemInfos: FormItemInfo[]): FormItemInfo[] => {
    return expand ? FormItemInfos.slice(0, 6) : FormItemInfos;
  };

  const setFeatureToFormItems = (FormItemInfos: FormItemInfo[]): FormItemInfo[] => {
    return setInitValue(setShowSearch(getLayOut(setExpandFormItems(FormItemInfos))));
  };

  return (
    <SearchLayOut title={title}>
      <div className={styles.searchAreaContent}>
        <BaseForm form={form} formItems={setFeatureToFormItems(FormItemInfos)} />
        <Row type="flex" gutter={24} justify="end">
          <Col>
            <Button type="primary" onClick={ handleSearch ? handleSearch : onSearch } loading={loading}>
              查询
            </Button>
          </Col>
          <Col>
            <Button type="default" onClick={onClearSearch} loading={loading}>
              重置
            </Button>
          </Col>
          <If condition={FormItemInfos.length > 6}>
            <Col style={{ display: 'flex', alignItems: 'center' }}>
              <a style={{ marginLeft: 8, fontSize: 12 }} onClick={toggle}>
                {expand ? '展开' : '收起'} <Icon type={expand ? 'down' : 'up'} />
              </a>
            </Col>
          </If>
        </Row>
        <Row>{extraButtonArea}</Row>
      </div>
      {React.cloneElement(children, { onChange: OnpaginationChange })}
    </SearchLayOut>
  );
}

export default SearchForm;

```

## ModalForm 弹窗表单

```typescript
import { Modal, Form } from 'antd';
import SubmitForm from '@/components/XyzForm/SubmitForm';
import React, { ReactNode, ReactElement, useState,useRef } from 'react';
import { FormItemInfo } from '@/components/XyzForm/BaseForm/BaseFormArea';
import BaseResponse from '@/requests/common/BaseResponse';
import { ModalProps } from 'antd/lib/modal';
import { useDispatch } from 'react-redux';

interface Iprops {
  modalProps?: ModalProps; // 弹框属性
  children: ReactElement; // 按钮
  extraFormProps: IFormProps; //表单属性
  request: (params: any) => Promise<BaseResponse>; // 请求
  extraParams?: {}; // 表单以外额外参数
  callBack?: Function; // 保存完回调,
  onParamsPreHandle?: Function; // 前置处理参数
  formRef?:typeof useRef; // 拿表单form
}
interface IFormProps {
  title?: string | ReactNode; // 新增页 主标题
  subTitle?: string[]; // 新增页 分组标题
  FormItemInfos: FormItemInfo[][]; // 二维数组 应对项目里面表单内容分组
  isLoading?: boolean; // dva的loading
  buttonArea?: ReactNode; // 重写按钮区域
  children?: ReactNode;
  formLayOutType?: string;
  model?: { [props: string]: string };
}
const FormModal = (props: Iprops) => {
  const dispatch = useDispatch();
  const {
    formRef,
    onParamsPreHandle,
    children,
    extraFormProps,
    request,
    modalProps,
    extraParams,
    callBack,
  } = props;
  const [visible, setVisible] = useState(false);
  const [loading, setLoading] = useState(false);
  console.log('children', children);
  const onButtonClick = () => {
    if (children.props.onClick && children.props.onClick()) {
      return;
    }

    setVisible(true);
  };
  const onSubmit = (values: { fileIdEn: string }) => {
    //  debugger
    onParamsPreHandle ? (values = onParamsPreHandle(values)) : null;
    //   debugger
    setLoading(true);
    request({ ...values, ...(extraParams || {}) })
      .then(res => {
        if (res.isSuccess()) {
          setVisible(false);
          setLoading(false);
          callBack && callBack();
          dispatch({
            type: 'app/updateState',
            payload: { isAuthorized: true },
          });
        } else {
          setLoading(false);
          dispatch({
            type: 'app/updateState',
            payload: { isAuthorized: true },
          });
        }
      })
      .catch(err => {
        setLoading(false);
        console.log(err);
      });
  };

  const onModalCancel = () => {
    setVisible(false);
  };
  const InnerForm =Form.create()(({ form }) =>{ 
    formRef? formRef.current=form:null
  return (<SubmitForm
      form={form}
      onSubmit={onSubmit}
      onCancel={onModalCancel}
      {...extraFormProps}
      isLoading={loading}
    />
  )});
  // tslint:disable-next-line:jsx-wrap-multiline
  return (
    <>
      {React.cloneElement(children, { onClick: onButtonClick })}
      <Modal width={800} visible={visible} footer={null} onCancel={onModalCancel} {...modalProps}>
        {InnerForm && <InnerForm />}
      </Modal>
    </>
  );
};

export default FormModal;

```

## 结语
个人觉得封装的思路主要是

- 底层组件越纯粹越好
- 中层可以实现一些适用性比较高的具体业务组件和通用方法
- 高层就具体要页面的细节的方方面面了
