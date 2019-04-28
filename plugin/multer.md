[TOC]





Multer

> Multer is a node.js middleware for handling `multipart/form-data`, which is primarily used for uploading files. It is written on top of [busboy](https://github.com/mscdex/busboy) for maximum efficiency.

Multer是一个用于处理`` multipart / form-data`` 的node.js中间件，主要用于上传文件。它基于 ``busboy  ``之上，以实现最高效率。

**注意**: Multer 只会处理 enctype 为 multipart/form-data 的表单

## 安装

```javascript
npm install --save multer
```

## 使用

Melter 会把上传字段添加到 req.body 对象中，如果是单个文件存储在 req.file 中，多文件的话会存储在 req.files 中

## 基本使用

```html
<!--单文件-->
<form action="/upload/file" method="post" enctype="multipart/form-data">
  <input type="file" name="doc" />
</form>


```

> **注意：** 不要忘记给表单添加   enctype="multipart/form-data"

### 单文件上传	

```javascript
const express = require('express');
const multer  = require('multer');
const upload = multer({ dest: 'uploads/' });  // 上传文件保存的目录 - multer 自动保存

const app = express();

// 单文件上传
app.post('/upload/file', upload.single('doc'), function (req, res, next) {
  // req.file is the `doc` file
  // req.body will hold the text fields, if there were any
});

```

**注意**： 在不作任何处理情况下，文件默认保存文件是没有后缀的

这里显然不是我们希望得到的，需要手动去重名了上述代码更改如下

```javascript
// 单文件上传
app.post('/upload/file', upload.single('doc'), function (req, res, next) {
  // req.file is the `doc` file
  // req.body will hold the text fields, if there were any
   	const file = req.file;
 	const prifix = path.resolve(__dirname, 'uploads');
    try{
        fs.renameSync(path.resolve(baseUrl, file.filename), path.resolve(baseUrl, file.originalname));
    	res.json(file);
    }catch(err){
       next(err); 
    }
});


```

上述用到 ``file`` 模块 和 ``path`` 模块。

### 多文件上传

```html
<!--多文件-->
<form action="/upload/files" method="POST" enctype="multipart/form-data">
    <input type="file" name="files" multiple="multiple" >
    <input type="submit">提交</input>
</form>
```

```javascript

app.post('/upload/files', upload.array('files', 5), function (req, res, next) {
  // req.files is array of `files` files
  // req.body will contain the text fields, if there were any
    const files = req.files;
    const baseUrl = path.resolve(__dirname, 'uploads');
    for(let file of files) {
        fs.renameSync(path.resolve(baseUrl, file.filename), path.resolve(baseUrl, file.originalname));
    }
    res.json(files);
})
```

### 组合上传

```html
<!--单文件-->
<form action="/upload/combination" method="post" enctype="multipart/form-data">
  <input type="file" name="doc" />
  <input type="file" name="files" multiple="multiple" >
  <input type="submit">提交</input>
</form>
```

```javascript
var cpUpload = upload.fields([{ name: 'doc', maxCount: 1 }, { name: 'files', maxCount: 5 }]);

app.post('/upload/combination', cpUpload, function (req, res, next) {
  // req.files 是一个对象， 其中 fieldname 是对象中Key， 值为文件数组
  //  { 
  //	 "doc":[],
  //	 "files": []
  //  }
  // req.body will contain the text fields, if there were any
    
  // .... 处理文件重命名问题 
})
```
如果你需要处理 纯文本 multipart 表单提交,  你需要使用 ``.none()`` 方法
```javascript
var express = require('express')
var app = express()
var multer  = require('multer')
var upload = multer()

app.post('/text', upload.none(), function (req, res, next) {
  // req.body contains the text fields
    
})；
```
## API	
### 文件对象信息

------

每个文件对象包含以下信息:

| 字段         | 描述                               | Note          |
| ------------ | ---------------------------------- | ------------- |
| fieldname    | 表单指定的上传的字段名             |               |
| originalname | 用户系统上文件名称                 |               |
| encoding     | 文件编码                           |               |
| mimetype     | 文件的MIME类型  例如("image/jpeg") |               |
| size         | 上传大小(单位字节数)               |               |
| destination  | 保存上传文件的路径                 | DiskStorage   |
| filename     | 保存在 `destination` 中的文件名    | DiskStorage   |
| path         | 已上传文件的完整路径               | DiskStorage   |
| buffer       | 一个存放整个文件的``Buffer``       | MemoryStorage |

### 实例方法

#### single(fieldname)

接收名字为``fieldname``指定的单个文件， 单个文件将存储在 ``req.file``

#### array(fieldname[, maxCount])

接收一组文件，全部使用名称为 ``filedname``. maxCount 限制上传文件数，如果超过将报错 .  这些文件存取在 ``req.files``

#### fields(fields)

接收混合文件，通过``fields``指定， 这些文件保存在 ``req.files``

``fileds`` 应该是具有名称和maxCount（可选的）的对象数组。如下

```javascript
[
  { name: 'avatar', maxCount: 1 },
  { name: 'gallery', maxCount: 8 }
]
```

#### none()

只接收文本字段，如果上传了任何文件，就会出现代码“limit_ted_file”错误。

#### any()

接收传输的所有文件，文件将保存在 ``req.files``

**警告：**确保始终处理用户上传的文件。不要将multer作为全局中间件添加，因为恶意用户可能会将文件上载到您没有预料到的路由上。仅在处理上传文件的路由上使用此函数。

### multer(opts)

------

Multer 接受一个 options 对象，其中最基本属性是 ``dest`` 属性，这将告诉 Multer上传文件的位置。如果你省略 options 对象， 这些文件将保存在内存中永远不会写入磁盘。

默认，Multer 将重命名上传文件避免命名冲突，重命名方式可以根据你的需要进行定制。

下面是可以传递给Multer的选项。

| 属性            | 名称                                   |
| --------------- | -------------------------------------- |
| dest or storage | 文件存储的位置                         |
| fileFilter      | 控制允许上传的文件                     |
| limits          | 上传数据大小限制                       |
| preservePath    | 保留文件的完整路径，而不仅仅是基本名称 |

一般应用程序中，可能只需要 ``dest`` 并且配置显示如下所示

```javascript
var upload = multer({ dest: 'uploads/' })
```

如果您想要更好地控制上传，则需要使用``storage``选项而不是``dest ``。 Multer附带存储引擎DiskStorage和MemoryStorage; 第三方提供更多引擎。

#### storage 

##### DiskStorage

------

disk storage 引擎可以让您完全控制将文件存储到磁盘。

```javascript
var storage = multer.diskStorage({
  destination: function (req, file, cb) {
    cb(null, '/tmp/my-uploads')
  },
  filename: function (req, file, cb) {
    cb(null, file.fieldname + '-' + Date.now())
  }
})
 
var upload = multer({ storage: storage })
```

有两个可用的选项， ``destination``  和 ``filename`` . 两者都确定文件存储的位置的函数。``destination``   确定上传文件存储的位置。如果没有设置  ``destination``  则使用操作系统的临时文件的默认目录。

​	**注意：** `` destination`` 给定是一个函数的时候需要自己手动创建文件目录。当给定是字符串时候, Multer 将为您创建目录.

``filename`` 用于确定文件在文件夹中名字。如果没有指定 ````filename`` `` , 将为每个文件给定一个随机名称，以及不包含文件扩展名。

​	**注意：**Multer 生成文件不会附加文件扩展名，应该通过函数返回带了文件拓展名的文件名。



##### MemoryStorage

------

``memory storage`` 引擎将文件作为Buffer对象存储在内存中。它没有 ``options``选项

```javascript
var storage = multer.memoryStorage()
var upload = multer({ storage: storage })
```

当使用内存存取时，文件信息将包含一个名为buffer的字段，其中包含整个文件。

**警告：**当使用内存存储时，上传非常大的文件或大量的相对较小的文件会导致应用程序耗尽内存。

#### limits

------

一个对象，指定一些数据大小的限制。Multer 通过这个对象使用 busboy，详细的特性可以在 [busboy's page](https://github.com/mscdex/busboy#busboy-methods) 找到。

可以使用下面这些选项：-

|      Key      | Description                                              | Default   |
| :-----------: | -------------------------------------------------------- | --------- |
| fieldNameSize | field 名字最大长度                                       | 100 bytes |
|   fieldSize   | field 值的最大长度                                       | 1MB       |
|    fields     | 非文件 field 的最大数量                                  | 无限      |
|   fileSize    | 在 multipart 表单中，文件最大长度 (字节单位)             | 无限      |
|     files     | 在 multipart 表单中，文件最大数量                        | 无限      |
|     parts     | 在 multipart 表单中，part 传输的最大数量(fields + files) | 无限      |
|  headerPairs  | 在 multipart 表单中，键值对最大组数                      | 2000      |

设置 limits 可以帮助保护你的站点抵御拒绝服务 (DoS) 攻击。



#### fileFilter 

------

设置为一个函数，以控制哪些文件应该上传，哪些文件应该跳过。

函数应该像这样：

```javascript
function fileFilter (req, file, cb) {
 
  // 拒绝文件通过，像下面这样设置
  cb(null, false)
 
  // 如果允许文件通过，像下面这样设置
  cb(null, true)
 
  // 如果出现错误，可以抛出错误
  cb(new Error('I don\'t have a clue!'))
 
}
```

实例： 比如过滤上传的图片

```javascript
function fileFilter(req, file,cb){
    // 只允许上传后缀为 .png gif jpg bmp
    const ext = /\.(png|gif|jpg|bmp)$/i;
    const filename = file.originalname;
    if(ext.test(filename)) {
        cb(null, true)
    }else {
        cb(null, false);  // 文件没上传成功，程序不会中断
        // 或
        cb(new Error("请上传图片格式文件"));
    }
}
```

### 错误处理

------

当遇到错误， Multer将把错误委托给 Express。想展示漂亮的错误页面请使用 [the standard express way](http://expressjs.com/guide/error-handling.html)

如果想捕获Multer中的错误，可以调用这中间件函数。如果只想捕获Multer错误，可以使用附加到multer对象本身的MulterError类（例如 err instanceof multer.MulterError）

```javascript
var multer = require('multer')
var upload = multer().single('avatar')
 
app.post('/profile', function (req, res) {
  upload(req, res, function (err) {
    if (err instanceof multer.MulterError) {
     
    } else if (err) {
     
    }
 
    // Everything went fine.
  })
})
```





### 文件重命名几种方式



方式一：利用node.js 内置的 fs模块 renameSync 或 rename方法

```javascript
const dest = 'public/uploads/';   //文件存储目录
fs.renameSync(path.resolve(dest, file.filename), path.resolve(dest, file.originalname));
```

方式二：使用 Multer 提供 DiskStorage 引擎 

```javascript
const storage = multer.diskStorage({
	destination: 'public/uploads',
	filename(req, file, cb){
        // 取到拓展名
        const {originalname} = file;
        const ext = originalname.
        substring(originalname.lastIndexOf('.'));
        cb(null, `${file.fieldname}-${Date.now()}${ext}`);
	}    
})
```

