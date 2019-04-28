# NPM(Node Package Manger) 模块管理器

## 简介

> 是Node默认的模块管理器，是一个命令行下的软件，用来安装和管理Node模块

`npm`不需要独立安装。当安装nodejs后，npm 自动安装的。因为npm比node更新频率高些，确保安装Npm最新版本可以通过一下命令来操作
`		npm -v 查看npm版本`	
如果您看到的版本与最新版本不匹配，查看最新版本[https://docs.npmjs.com/getting-started/installing-node] 请运行: 
`npm install npm@latest -g`

**@latest **表示最新版本，**-g** 表示全局安装  所以，命令的主干是**npm install npm**，也就是使用**npm**安装自己。之所以可以这样，是因为**npm**本身与Node的其他模块没有区别。

## 常用命令

| 命令      | 介绍              |
| -------------- | :---------------- |
| npm  help | 查看 npm 命令列表 |
| npm  -l   | 查看各个命令的简单用法 |
| npm  -v   | 查看 npm 的版本 |
| npm  config list -l | 查看 npm 的配置 |
| npm  init | 用来初始化生成一个新的package.json文件 |
| npm  set | 用来设置环境变量 |
| npm  info | 命令可以查看每个模块的具体信息 |
| npm  list | 以树型结构列出当前项目安装的所有模块，以及它们依赖的模块。加上-global 会列出全局安装的模块 也可以查看单个模块 npm list XX模块 |

**重点介绍使用最多几个命令**

### **1 npm install**   

### 简介

> 用来安装Node模块

每个模块可以“全局安装”，也可以“本地安装”。“全局安装”指的是将一个模块安装到系统目录中，各个项目都可以调用。一般来说，全局安装只适用于工具模块，比如`grunt`和`gulp`。“本地安装”指的是将一个模块下载到当前项目的`node_modules`子目录，然后只有在项目目录之中，才能调用这个模块。

**语法**

```markdown
npm install (with no args, in package dir)
npm install [<@scope>/]<pkg>
npm install [<@scope>/]<pkg>@<tag>
npm install [<@scope>/]<pkg>@<version>
npm install [<@scope>/]<pkg>@<version range>
npm install <folder>
npm install <tarball file>
npm install <tarball url>
npm install <git:// url>
npm install <github username>/<github project>

aliases: i
add common options: [--save-prod|--save-dev|--save-optional] [--save-exact] [--no-save]

```
默认不给定参数时候，找当前命令执行路径下 package.json文件进行依赖安装。不存在则报错。

**执行 npm install 命令大概的步骤**

1. 安装之前，`npm install`会先检查，`node_modules`目录之中是否已经存在指定模块。
2. 如果存在，就不再重新安装了，即使远程仓库已经有了一个新版本，也是如此。

如果你希望，一个模块不管是否安装过，npm 都要强制重新安装，可以使用`-f`或`--force`参数。

```	markdown
npm install <packageName> --force
```
如果你希望，所有模块都要强制重新安装，那就删除`node_modules`目录，重新执行`npm install`。
```markdown
window 下可以通过 安装`rimraf`的模块进行删除

npm install rimraf - g
rimraf node_modules

UNIX 或 linux 下可以通过

rm -rf node_modules  
npm install
```

**安装不同版本**

install 命令总是安装模块的最新版本,如果要安装模块的特定版本，可以模块后面加上@和版本号
```markdown
npm install [<@scope>/]<pkg>@<tag>
npm install [<@scope>/]<pkg>@<version>
npm install [<@scope>/]<pkg>@<version range>

npm install vue@latest
npm install vue@2.5
npm install vue@">=2.2 < 2.5"
```
如果使用`--save-exact`参数，会在package.json文件指定安装模块的确切版本。
```markdown
 npm install readable-stream --save 
 
 "dependencies": {
    "readable-stream": "^3.0.6"
  }
```
```markdown
 npm install readable-stream --save --save-exact
 
 "dependencies": {
    "readable-stream": "3.0.6"
  }
```
install命令可以使用不同参数，指定所安装的模块属于哪一种性质的依赖关系，即出现在packages.json文件的哪一项中。

> –save：模块名将被添加到  **dependencies**，可以简化为参数`-S`。

> –save-dev: 模块名将被添加到  **devDependencies**，可以简化为参数`-D`

```markdown
$ npm install vue --save
$ npm install babel-polifill --save-dev
# 或者
$ npm install vue -S
$ npm install babel-polifill -D

```
`npm install`默认会安装`dependencies`字段和`devDependencies`字段中的所有模块，如果使用`--production`参数，可以只安装`dependencies`字段的模块。

```markdown
$ npm install --production
# 或者
$ NODE_ENV=production npm install
```

### **2 npm update，npm uninstall**

`npm update`命令可以更新本地安装的模块

```markdown
# 升级当前项目的指定模块
npm update [package name]

# 升级全局安装的模块
npm update -global [package name]
```

**执行 npm update 命令大概步骤**：

1. 它会先到远程仓库查询最新版本，然后查询本地版本。
2. 如果本地版本不存在，或者远程版本较新，就会安装。

使用`-S`或`--save`参数，可以在安装的时候更新`package.json`里面模块的版本号。

```markdown
 # 更新前
 "dependencies": {
    "readable-stream": "3.0.6",
    "vue": "^2.4.4"
  }
 # 更新后
 "dependencies": {
    "readable-stream": "3.0.6",
    "vue": "^2.5.17"
  }
```

**注意**，从**npm v2.6.1** 开始，`npm update`只更新顶层模块，而不更新依赖的依赖，以前版本是递归更新的。如果想取到老版本的效果，要使用下面的命令。

~~~markdown
npm --depth 9999 update
~~~

`npm uninstall` 命令，卸载已安装的模块。

~~~markdown
npm uninstall [package name]

# 卸载全局模块
npm uninstall [package name] -global
~~~

### **3 npm run** 

`npm`不仅可以用于模块管理，还可以用于执行脚本。`package.json`文件有一个`scripts`字段，可以用于指定脚本命令，供`npm`直接调用。

~~~
{
  "name": "node-basic",
  "version": "1.0.0",
  "scripts": {
    "lint": "eslint ./main.js"
  },
  "devDependencies": {
    "eslint": "^5.8.0"
  }
}

~~~
上面代码中，`scripts`字段指定了命令`lint`。命令行输入`npm run-script lint`或者`npm run lint`，就会执行`eslint main.js` . `npm run`是`npm run-script`的缩写，一般都使用前者，但是后者可以更好地反应这个命令的本质。

`npm run`命令会自动在环境变量`$PATH`添加`node_modules/.bin`目录，所以`scripts`字段里面调用命令时不用加上路径，这就避免了全局安装NPM模块。

`npm run`如果不加任何参数，直接运行，会列出`package.json`里面所有可以执行的脚本命令。

```markdown
npm run
available via `npm run-script`:
  lint
    eslint ./main.js
```

`npm内置了两个命令简写，`npm test`等同于执行`npm run test`，`npm start`等同于执行`npm run start`。













## 设置淘宝镜像

------

```
npm install -g cnpm --registry = https://registry.npm.taobao.org
```

或者

​	直接更改npm镜像 ： 

​		npm set registry https://registry.npm.taobao.org

​		npm get registry  查看镜像