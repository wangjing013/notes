[TOC]

# [qs](https://www.npmjs.com/package/qs)

> 一个 查询字符串解析和 对象序列化的库，增加一些安全处理措施；

区别 `nodeJS` 自带 [querystring](https://github.com/tj/node-querystring) ，`qs` 可以理解为是其增强版本；起初 `qs` 作者也是 `querystring` 同一个，现在由 [Jordan Harband](https://github.com/ljharb) 为主要维护者；

## Install

------

```
npm install qs -S
```
## Usage
------
```javascript
var qs = require('qs');
var assert = require('assert');  // node 自带断言库
 
var obj = qs.parse('a=c');
// {a: 'c'}
assert.deepEqual(obj, { a: 'c' });
// ok

var str = qs.stringify(obj);
// 'a=c'
assert.equal(str, 'a=c');
// ok
```

### Parsing Objects

```javascript
qs.parse(string, [options]);
```

**qs**  允许你的查询字符串中创建嵌套层级的对象，其中对象 key 通过方括号进行包围 `[]`； 例如： 字符串 'foo[bar]=baz' 转换为：

```javascript
assert.deepEqual(qs.parse('foo[bar]=baz'), {
    foo: {
        bar: 'baz'
    }
});
```

**qs** 提供可选`options` 对象， 当使用时添加 `plainObjects` 选项，解析的值作为空对象返回 .  

***空对象*** : 这里不是返回 {} ，而是返回的对象 o.__ proto __ = null;

等价通过Object.create(null)创建，因此您应该意识到原型方法将不存在于其上，用户可以将这些名称设置为他们喜欢的任何值:

```javascript
var o = qs.parse('a[hasOwnProperty]=b');
console.log(o); // {}
console.log('toString' in o);  // true
console.log(o.hasOwnProperty); // [Function: hasOwnProperty]
console.log(o.__proto__.isPrototypeOf(Object)); // true


var nullObject = qs.parse('a[hasOwnProperty]=b', { plainObjects: true });

console.log(nullObject); // { a: { hasOwnProperty: 'b' } }
console.log('toString' in nullObject);  // false

assert.deepEqual(nullObject, { a: { hasOwnProperty: 'b' } });
// ok
```

默认情况下，会忽略将覆盖对象原型上的属性的参数；   如果需要保留提交的字段数据，请使用  `plainObjects`  选项；  或者设置 `allowPrototypes`  为 true；这样允许用户输入字段覆盖其原型上的属性；

```javascript

var protoObject = qs.parse('a[hasOwnProperty]=b', { allowPrototypes: true });

// { a: { hasOwnProperty: 'b' } }

assert.deepEqual(protoObject, { a: { hasOwnProperty: 'b' } });
```

>  **警告**： 一般推荐启动该选项，它可能引起一些未知的错误；



处理经过 URL 编码的字符串：

```javascript
assert.deepEqual(qs.parse('a%5Bb%5D=c'), {
    a: { b: 'c' }
});
// ok
```

也可以嵌套对象，像这样： 'foo[bar][baz]=foobarbaz' ：

```javascript
assert.deepEqual(qs.parse('foo[bar][baz]=foobarbaz'), {
    foo: {
        bar: {
            baz: 'foobarbaz'
        }
    }
});
```

默认情况下， **qs**  只会解析嵌套对象层级为 5级，  这意味着如果你解析的字符串像这样： `'a[b][c][d][e][f][g][h][i]=j'`  ，解析后的结果将为：

```javascript
var expected = {
    a: {
        b: {
            c: {
                d: {
                    e: {
                        f: {
                            '[g][h][i]': 'j'
                        }
                    }
                }
            }
        }
    }
};

var string = 'a[b][c][d][e][f][g][h][i]=j';
assert.deepEqual(qs.parse(string), expected);

```

**qs** 提供 **depth**  选项，可以用来覆盖默认解析字符串的层级，如下：

```javascript
var deep = qs.parse('a[b][c][d][e][f][g][h][i]=j', { depth: 1 });

assert.deepEqual(deep, { a: { b: { '[c][d][e][f][g][h][i]': 'j' } } });
```

当 **qs** 用于解析用户输入时，深度限制有助于减轻滥用，建议将其保持在相当小的数量. 



出于类似的原因， 默认情况下， **qs**  最多只能解析1000个参数，如果想覆盖默认行为，通过在解析的时候传入 `parameterLimit`  选项：

```javascript
var limited = qs.parse('a=b&c=d', { parameterLimit: 1 });
// { a: 'b' }
assert.deepEqual(limited, { a: 'b' });
```

假如解析字符串中像这样： `?a=b&c=d` ； 需要绕过前置的 `？`（也就是解析内容中不包含?），使用 `ignoreQueryPrefix` 选项 ：

```javascript
var prefixed = qs.parse('?a=b&c=d');
// { '?a': 'b', c: 'd' }

var prefixed = qs.parse('?a=b&c=d', { ignoreQueryPrefix: true });
//{ a: 'b', c: 'd' }

assert.deepEqual(prefixed, { a: 'b', c: 'd' });
```

还可以传递可选的**分隔符**：

```javascript
var delimited = qs.parse('a=b;c=d', { delimiter: ';' });
// { a: 'b', c: 'd' }
assert.deepEqual(delimited, { a: 'b', c: 'd' });
```

分隔符也可以是正则表达式：

```javascript
var regexed = qs.parse('a=b;c=d,e=f', { delimiter: /[;,]/ });
// { a: 'b', c: 'd', e: 'f' }
assert.deepEqual(regexed, { a: 'b', c: 'd', e: 'f' });
```

如果希望使用习惯  `.` ， 那么可以通过设置 `allowDots` 选项:

```javascript
var withDots = qs.parse('a.b.d.f.e=c', { allowDots: true });
// {"a":{"b":{"d":{"f":{"e":"c"}}}}}
assert.deepEqual(withDots, {"a":{"b":{"d":{"f":{"e":"c"}}}}});
```

如果需要处理旧版本浏览器或服务， 还支持将 [percent-encode](https://www.cnblogs.com/DaoMuRen/p/5695030.html)  的八位字节解码为iso-8859-1：

```javascript
var oldCharset = qs.parse('a=%A7', { charset: 'iso-8859-1' });

assert.deepEqual(oldCharset, { a: '§' });
```



### Parsing Arrays

------

**qs** 可以解析数组，使用类似的 `[]` 表示：

```javascript
var withArray = qs.parse('a[]=b&a[]=c');

assert.deepEqual(withArray, { a: ['b', 'c'] });
```

也可以指定索引：

```javascript
var withIndexes = qs.parse('a[1]=c&a[0]=b');
assert.deepEqual(withIndexes, { a: ['b', 'c'] });
```

**注意**： 数组中的索引和对象中的键之间的惟一区别是，方括号之间的值必须是创建数组的数字。在创建具有**特定索引**的数组时，**qs**  将稀疏数组压缩为仅保留现有值顺序的稀疏数组:

```javascript
var noSparse = qs.parse('a[1]=b&a[15]=');
// {a: [b, c]}
// noSparse.a.length  === 2
assert.deepEqual(noSparse, { a: ['b', 'c'] });

```

注意：这里处理与我们定义数组区别：

```javascript
var a = [];
a[15] = 1;
a.length // => 16
```

请注意：如果提供解析参数中，其参数值为空字符串，其也会保留下来：

```javascript
var withEmptyString = qs.parse('a[]=&a[]=b');
// { a : ['', 'b'] }
assert.deepEqual(withEmptyString, { a: ['', 'b'] });

var withIndexedEmptyString = qs.parse('a[0]=b&a[1]=&a[2]=c');
// { a : ['b', '', 'c'] }
assert.deepEqual(withIndexedEmptyString, { a: ['b', '', 'c'] });
```

**qs** 还将数组中的索引限制为最大索引 **20**,  任何索引大于20的数组成员都将转换为索引为键的对象:

```javascript
var withMaxIndex = qs.parse('a[0]=1&a[100]=b');
// { a: { '0': '1', '100': 'b' } }

var withMaxIndex = qs.parse('a[100]=b');
// { a : {'100', 'b'}}
assert.deepEqual(withMaxIndex, { a: { '100': 'b' } });

```

同解析对象一样， **qs**  允许通过指定 ***arrayLimit***  选项去覆盖默认的限定：

```javascript

var withArrayLimit = qs.parse('a[1]=b', { arrayLimit: 0 });
assert.deepEqual(withArrayLimit, { a: { '1': 'b' } });
```

如果完全禁用数组解析，可以设置 ***parseArrays***   为 **false**

```javascript
var noParsingArrays = qs.parse('a[]=b', { parseArrays: false });
assert.deepEqual(noParsingArrays, { a: { '0': 'b' } });
```

上述默认转成对象形式；

如果使用混合符号， **qs** 将把两项合并为一个对象：

```javascript
var mixedNotation = qs.parse('a[0]=b&a[b]=c');
// { a: { '0': 'b', b: 'c' } }
assert.deepEqual(mixedNotation, { a: { '0': 'b', b: 'c' } });
```

**记住： 如果解析数组的化，其提供索引值必须是数字**

你也可以创建对象数组：

```javascript
var arraysOfObjects = qs.parse('a[][b]=c');
assert.deepEqual(arraysOfObjects, { a: [{ b: 'c' }] });
```

解析方式基本上讲完，下面来看看序列化;



### Stringifying  对象序列化

------

```javascript
qs.stringify(object, [options]);
```

当使用序列化时， **qs**  默认情况下使用 URI 编码后输出；如下：

```javascript

assert.equal(qs.stringify({ a: 'b' }), 'a=b');
assert.equal(qs.stringify({ a: { b: 'c' } }), 'a%5Bb%5D=c');

```

如果不想对其进行编码，通过设置 `encode`  选项为 false,  禁用其编码：

```javascript
var unencoded = qs.stringify({ a: { b: 'c' } }, { encode: false });
assert.equal(unencoded, 'a[b]=c');
```

通过设置`encodeValuesOnly`为 true，来禁用对 key 进行编码：

```javascript
var encodedValues = qs.stringify(
    { a: 'b', c: ['d', 'e=f'], f: [['g'], ['h']] }
);
//=>  a=b&c%5B0%5D=d&c%5B1%5D=%27e%3Df%27&f%5B0%5D%5B0%5D=%27g%27&f%5B1%5D%5B0%5D=%27h%27

var encodedValues = qs.stringify(
    { a: 'b', c: ['d', 'e=f'], f: [['g'], ['h']] },
    { encodeValuesOnly: true }
);
assert.equal(encodedValues,'a=b&c[0]=d&c[1]=e%3Df&f[0][0]=g&f[1][0]=h');

```

也可以自定义编码方式， 通过设置 **encoder**选项：

```javascript
var encoded = qs.stringify({ a: { b: 'c' } }, { encoder: function (str) {
  	// Passed in values a[b]，c
    
    return // Return encoded string
}})
```

从传入 `str` 看出，先把输入序列化为 a[b] = c , 再传入key，value 值；

**注意**： 如果 ***encode*** 设置 false,  ***encoder*** 选项则无效；

**与 *decoder*  类似，parse 还有一个 *decoder* 选项，用于覆盖属性和值的解码:**

```javascript
var decoded = qs.parse('x=z', { decoder: function (str) {
    // Passed in values `x`, `z`
    return // Return decoded string
}})
```

**当数组被序列化时，默认情况下返回值包含精准索引值：**

```javascript
qs.stringify({ a: ['b', 'c', 'd'] }）；
// a[0]=b&a[1]=c&a[2]=d
```

你可以通过把 ***indices*** 选项设置为 false ，覆盖它：

```javascript
qs.stringify({ a: ['b', 'c', 'd'] }, { indices: false });
// 'a=b&a=c&a=d'
```

你可以使用 ***arrayFormat*** 选项去指定输出数组的格式：

```javascript
// indices 索引  brackets 方括号  repeat 重复
qs.stringify({ a: ['b', 'c'] }, { arrayFormat: 'indices' })
// 'a[0]=b&a[1]=c'
qs.stringify({ a: ['b', 'c'] }, { arrayFormat: 'brackets' })
// 'a[]=b&a[]=c'
qs.stringify({ a: ['b', 'c'] }, { arrayFormat: 'repeat' })
// 'a=b&a=c'
```

当对象被序列化时，默认使用 ***brackets*** （方括号）符号：

```javascript
qs.stringify({ a: { b: { c: 'd', e: 'f' } } });
// 'a[b][c]=d&a[b][e]=f'
```

您可以通过将  ***allowDots***  选项设置为true，来覆盖此选项以使用 '.' 方式表示：

```javascript
qs.stringify({ a: { b: { c: 'd', e: 'f' } } }, { allowDots: true });
// 'a.b.c=d&a.b.e=f'
```

**如果属性值为空字符串和 null 时将忽略， 但是等号（=）仍会存在：**

```javascript
assert.equal(qs.stringify({ a: '' }), 'a=');
```

**key 没有对应值(如： 空对象或空数组)，将返回 "":**

```javascript
assert.equal(qs.stringify({ a: [] }), '');
assert.equal(qs.stringify({ a: {} }), '');
assert.equal(qs.stringify({ a: [{}] }), '');
assert.equal(qs.stringify({ a: { b: []} }), '');
assert.equal(qs.stringify({ a: { b: {}} }), '');
```

因为上述其实都是没有任何实际意义的，所以返回空都是正确的；



**当属性值设置为 `undefined`  , 将会完全忽略 ； 等价设置 null 或  ''** 

```javascript
assert.equal(qs.stringify({ a: null, b: undefined }), 'a=');
```

既然在解析时候可以忽略查询字符串前 `?`,  那么通过指定 ***addQueryPrefix*** 为 **true**， 序列化后的查询字符串前面添加 `？` :

```javascript
var str = qs.stringify({ a: 'b', c: 'd' }, { addQueryPrefix: true }; 
// '?a=b&c=d'
assert.equal(str, '?a=b&c=d');
// ok
```

还可以通过指定序列号的分隔符  **delimiter**：

```javascript
var str = qs.stringify({ a: 'b', c: 'd' }, { delimiter: ';' });
// 'a=b;c=d'
assert.equal(str, 'a=b;c=d');

```

如果只想覆盖Date对象的序列化，你可以提供 **serializeDate** 选项：

```javascript
var date = new Date(7);
var str = qs.stringify({ a: date });
// a=1970-01-01T00%3A00%3A00.007Z

assert.equal(str, 'a=1970-01-01T00:00:00.007Z'.replace(/:/g, '%3A'));


assert.equal(
    qs.stringify({a: date}, {
        serializeDate: function (d) {
            return d.getTime();
        }
    }),
    'a=7'
);
```

你可以使用 **sort** 选项去影响参数key的排序：

```javascript
function alphabeticalSort(a, b) {
    return a.localeCompare(b);
}

assert.equal(qs.stringify({ a: 'c', z: 'y', b : 'f' }, { sort: alphabeticalSort }), 'a=c&b=f&z=y');
```

最后，你可以通过 **filter** 选项去限制序列化输出中只包含哪些 key . 如果传递是的函数， 序列化每个 key 时它都会被调用来获取替换的值；除此之外， 如果传递的时数组， 它将用于为字符串化选择属性和数组索引:

看代码理解：

```javascript
function filterFunc(prefix, value) {
    if (prefix == 'b') {

        // 如果需要忽略某个属性，可以返回一个undefined
        // Return an `undefined` value to omit a property.
        return;
    }
    if (prefix == 'e[f]') {
        return value.getTime();
    }
    if (prefix == 'e[g][0]') {
        return value * 2;
    }
    return value;
}

qs.stringify({a: 'b', c: 'd', e: {f: new Date(123), g: [2]}}, {filter: filterFunc});
// 'a=b&c=d&e[f]=123&e[g][0]=4'

qs.stringify({a: 'b', c: 'd', e: 'f'}, {filter: ['a', 'e']});
// 'a=b&e=f'

qs.stringify({a: ['b', 'c', 'd'], e: 'f'}, {filter: ['a', 0, 2]});

// 'a[0]=b&a[2]=d'
```

上述可以看出，如果是序列化是数组，那么当且只有其索引在 filter 指定的范围内； 如果序列化的为对象，只有其属性名在 filter 数组指定的范围内；

这里是有点抽象，可以通过分步来理解：

```javascript
var o = {
    a : ['b', 'c', 'd'],
    e: 'f'
};
var filterArray = ['a', 0, 2];
console.log(qs.stringify(o ,
    {
        filter: filterArray,
        encode: false  // 这添加禁止对输出进行编码，方法查看
    }
));


//1 我们可以先忽略 filter 选项
qs.stringify(o)  // => 'a[0]=b&a[1]=c&a[2]=d&e=f'

//2 先来看看上述存在哪些 key 值： 
keys = ['a', 0, 1, 2 , 'e'];

//3 在根据提供 key 与 filterArray 取交集，即为需要序列化的输出的内容
intersection = ['a', 0, 2]

//4 最后输出内容 
'a[0]=b&a[2]=d'

```

​	



### 处理空值 - null

------

默认情况下， **null** 值处理方式像空字符串：

```javascript
var withNull = qs.stringify({ a: null, b: '' });
assert.equal(withNull, 'a=&b=');
```

解析不区分带等号和不带等号的参数. 两者都转化为空字符串：

```javascript

var equalsInsensitive = qs.parse('a&b=');
assert.deepEqual(equalsInsensitive, { a: '', b: '' });
```

要区分 **null** 值 和 **空字符串**，请使用 **strictNullHandling** 标志。这样在结果字符串中值为 null 的属性不包含  `=` 符号；

```javascript
var strictNull = qs.stringify({ a: null, b: '' }, { strictNullHandling: true });
// 'a&b='
assert.equal(strictNull, 'a&b=');
```

同样解析的时，也可以使用 ***strictNullHandling*** 标识来处理 ：

```javascript
var parsedStrictNull = qs.parse('a&b=', { strictNullHandling: true });

assert.deepEqual(parsedStrictNull, { a: null, b: '' });
```

完全跳过值为 null 的 key，请使用  **skipNulls** 标志:

```javascript
var nullsSkipped = qs.stringify({ a: 'b', c: null}, { skipNulls: true });
//=> 'a=b'
assert.equal(nullsSkipped, 'a=b');
```



### 处理特殊字符集

------

默认情况下，字符的编码和解码是使用  utf-8 完成的， 并且 iso-8859-1支持也可以通过指定 `charset`  参数；

如果需要编码查询字符串中存在不同的字符集. 你需要使用 [qs-iconv](https://github.com/martinheidegger/qs-iconv) 库:

```javascript
var encoder = require('qs-iconv/encoder')('shift_jis');
var shiftJISEncoded = qs.stringify({ a: 'こんにちは！' }, { encoder: encoder });

assert.equal(shiftJISEncoded, 'a=%82%B1%82%F1%82%C9%82%BF%82%CD%81I');
```

同样也适用于解码查询字符串：

```javascript
var decoder = require('qs-iconv/decoder')('shift_jis');

var obj = qs.parse('a=%82%B1%82%F1%82%C9%82%BF%82%CD%81I', { decoder: decoder });

assert.deepEqual(obj, { a: 'こんにちは！' });
```

### RFC 3986 and RFC 1738 空格编码

------

**[RFC3986**](https://tools.ietf.org/html/rfc3986) 作为默认选项，将 '' 编码为向后兼容的 %20 .  同时，输出可以按照[**RFC1738**](https://tools.ietf.org/html/rfc1738) 进行字符串化，' '  === '+'。

```javascript
assert.equal(qs.stringify({ a: 'b c' }), 'a=b%20c');
assert.equal(qs.stringify({ a: 'b c' }, { format : 'RFC3986' }), 'a=b%20c');
assert.equal(qs.stringify({ a: 'b c' }, { format : 'RFC1738' }), 'a=b+c');
```

RFC3986 与 RFC1738 为两中 URL的规范；



### API 

------

#### 解析对象：

| 选项              | 类型          | 说明                                                         |
| :---------------- | ------------- | ------------------------------------------------------------ |
| plainObjects      | boolean       | 默认是 false； 返回对象原型Object.prototype，如果指定为 true 等价于	Object.create(null) 创建的对象； |
| allowPrototypes   | boolean       | 默认是 false, 允许输入属性覆盖其原型上属性名                 |
| depth             | number        | 指定解析层级，qs 默认层级 5                                  |
| parameterLimit    | number        | 解析参数个数限定，默认最大为 1000                            |
| ignoreQueryPrefix | boolean       | 如果查询字符串前缀包含? ,指定为true则不会包含在解析内容中  默认: false |
| delimiter         | string\|Regex | 指定参数解析分隔符； 默认是： '&'                            |
| allowDots         | boolean       | 允许解析字符串已 . 方式表示, 例如  a.b.c = 1; 默认表示方式为 `a[b][c]=1`  默认值：false |
| charset           | string        | 指定解码的编码 默认 utf-8                                    |
| decoder           | string        | 自定义解码方式                                               |

#### 解析数组：

| 选项        | 类型    | 说明                                                         |
| ----------- | ------- | ------------------------------------------------------------ |
| arrayLimit  | number  | 限定数组下标，默认最大值 20                                  |
| parseArrays | boolean | 如果查询字符串总包含数组，默认会把数组转为对象，数组索引作为对象的key  ；  默认值:true |

#### 序列化:

| 选项             | 类型            | 说明                                                         |
| ---------------- | --------------- | ------------------------------------------------------------ |
| encode           | boolean         | 对需要序列化对象的内容进行(key-value)编码，默认：true  其编码格式：utf-8 |
| encodeValuesOnly | boolean         | qs 默认编码会对 key 和 value 编码；如果只想对值进行编码指定为 true ； 默认值： false |
| encoder          | function        | 自定义编码方式； 注意：如果encode:true前提下，该选项才会生效 |
| indices          | boolean         | 数组序列化时，默认生成索引值； 如果指定为false，则不会生成索引值；  默认值：true |
| arrayFormat      | string          | 可选值：indices ，brackets， repeat                          |
| allowDots        | string          | 允许序列化对象时使用 . 表示对象和属性的关系，默认用方括号 [ ]；     默认值 false |
| addQueryPrefix   | boolean         | 是否在序列化生成查询字符串前添加 ？前缀， 默认是false        |
| delimiter        | string          | 序列化时，指定分隔符 ； 默认 &                               |
| serializeDate    | function        | 自定义日期对象序列化方式                                     |
| sort             | function        | 参数排序方式                                                 |
| filter           | function\|array | 只允许序列化生成查询字符串包含哪些参数                       |
|                  |                 |                                                              |



#### 处理空值：

| 选项               | 类型    | 说明                                  |
| ------------------ | ------- | ------------------------------------- |
| strictNullHandling | boolean | 用来 null 值 和 空字符串  默认：false |
| skipNulls          | boolean | 跳过值为 null 属性                    |
|                    |         |                                       |

