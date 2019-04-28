# properties 与 attibutes 区别

当编写HTML代码时，可以在HTML元素上定义属性. 当浏览解析代码时，将创建相应的DOM节点.  这个节点是一个对象，因此它具有属性。

例如，这个HTML元素：

```html
<input type="text" value="Name:">
```

包含2个属性 （`type` ，`value`）.

一旦浏览器解析这段代码，将创建一个[HTMLInputElement](https://developer.mozilla.org/en-US/docs/Web/API/HTMLInputElement)对象，该对象将包含许多属性，如:accept、accessKey、align、alt、attributes、autofocus、baseURI、checked、childElementCount、childNodes、children、classList、className、clientHeight等。

对于给定的DOM节点对象， `properties`   表示该对象的属性 ；`attributes`  为 `properties`   的属性；

```javascript
<input type="text" value="Name:">
    
var input = document.querySelector('input');
// input 为 HTMLInputElement 实例
input  // input properties表述对象中包含属性  
input instanceof HTMLInputElement  // true
input.attributes // attributes
```

当为给定的HTML Element 创建DOM节点时，节点的很多属性与 `attributes` 具有相同的名称或关联；但这不是一对一的关系； 例如, 对于这个HTML元素：

```html
<input id="the-input" type="text" value="Name:">
```

对应的DOM节点将具有id、type和value属性(以及其他属性):

`properties.id` 属性与 attribute 对象中 id 是映射关系： 获取该属性将读取该 attribute中对应的属性并且设置该属性的值等于改变 attribute中对应属性值.   `id`  是一个纯粹映射属性，它不限制其赋值的内容；

`properties.type` 属性与 attribute 对象中 type 是映射关系:  获取该属性将读取该 attribute中对应的属性并且设置该属性的值等于改变 attribute中对应属性值. `type` 不是一个纯粹的映射属性，因为`type`限定了已知的值（有效的 input 类型  text password checkbox radio等）；如果有一个  `<input type="foo">`, 然后 `theInput.getAttribute("type")` 返回的是 `"foo"` 但是`theInput.type` 返回的是 `"text"`.

相反，`properties.value` 属性与 attirbute 中`value`属性不是映射关系。  它表示当前输入框的值。 当用户手动更改input中修改值时，`properties.value` 相应的被改变。因此，如果用户在输入框中输入“John”，则：

```javascript
theInput.value // returns "John"
```

而:

```javascript
theInput.getAttribute('value') // returns "Name:"
```

这 `properties.value` 属性反映文本框当前的文本内容； 而 `attribute.value`  表示 HTML  代码指定的 value 的初始值；







