# EditorConfig 配置

在VSCode 中配置 editorconfig

- 在当前项目根目录下添加.editorconfig文件
- 安装[EditorConfig](https://marketplace.visualstudio.com/items?itemName=EditorConfig.EditorConfig)扩展
- 全局安装或局部安装editorconfig依赖包(npm install -g editorconfig | npm install -D editorconfig)
- 打开需要格式化的文件并手动格式化代码（shift+alt+f）。

简单说明下每一步的作用：

- 第一步的editorconfig文件是定义一些格式化规则（此规则并不会被vscode直接解析）
- 第二步EditorConfig扩展的作用是读取第一步创建的editorconfig文件中定义的规则，并覆盖user/workspace settings中的对应配置（从这我们也可以看出vscode本身其实是并不直接支持editorconfig的）
- 第三步安装editorconfig依赖包主要是因为EditorConfig依赖于editorconfig包，不安装的可能会导致EditorConfig无法正常解析我们在第一步定义的editorconfig文件
- 第四步的作用就是让经过EditorConfig扩展覆盖后的user/workspace settings生效。

常见的问题：

当 user/workspace setting 中的 files.trimTrailingWhitespace=true时trim_trailing_whitespace = false 就不会生效了

end_of_line属性貌似不被支持（可直接修改user/workspace setting中的files.eol配置，直接配置成"files.eol": "\n"即可）

关于上述两个设置： 在VSCODE中   打开 文件菜单 ->首选项-> 配置 - > 在搜索栏中输入  files.trimTrailingWhitespace 即可