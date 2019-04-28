# 表单

## 为什么需要表单验证？

​	通过验证用户输入的准确性和完整性，来增强整体数据质量.

## 验证的两种方式:

- 模板驱动表单
- 响应式表单



[数据的校验基于H5表单校验.](https://developer.mozilla.org/zh-CN/docs/Web/Guide/HTML/HTML5/Constraint_validation) 

常用的包括	:  [pattern](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/input#attr-pattern) ，[min](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/input#attr-min) ，[max](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/input#attr-max) ，[required](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/input#attr-required) ，[step](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/input#attr-step) ，[maxlength](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/input#attr-maxlength) .

校验状态	：valid，invalid，untouched，dirty（肮脏的-值被修改）， pristine(纯净的) 

