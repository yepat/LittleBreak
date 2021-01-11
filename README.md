# LittleBreak

Lerna有很多强大的CLI命令，我这里用的有:
lerna init: 初始化项目
lerna bootstrap: 自动构建项目
lerna ls: 列出当前项目所有包
lerna clean: 清理node_modules文件夹
lerna add: 添加依赖（类似npm install)
lerna publish: 发版

yarn run commit 代码提交


lerna管理前端模块最佳实践
2.1 安装
推荐全局安装，因为会经常用到 lerna 命令
npm i -g lerna

2.2 初始化项目
lerna init

2.3 创建npm包
增加两个 packages
lerna create @mo-demo/cli
lerna create @mo-demo/cli-shared-utils

2.4 增加模块依赖
分别给相应的 package 增加依赖模块
lerna add chalk // 为所有 package 增加 chalk 模块 
lerna add semver --scope @mo-demo/cli-shared-utils // 为 @mo-demo/cli-shared-utils 增加 semver 模块 
lerna add @mo-demo/cli-shared-utils --scope @mo-demo/cli // 增加内部模块之间的依赖

2.5 发布
lerna publish

2.6 依赖包管理
上述1-5步已经包含了 Lerna 整个生命周期的过程了，但当我们维护这个项目时，新拉下来仓库的代码后，需要为各个 package 安装依赖包。

我们在第4步 lerna add 时也发现了，为某个 package 安装的包被放到了这个 package 目录下的 node_modules 目录下。这样对于多个 package 都依赖的包，会被多个 package 安装多次，并且每个 package 下都维护 node_modules ，也不清爽。于是我们使用 --hoist 来把每个 package 下的依赖包都提升到工程根目录，来降低安装以及管理的成本。

lerna bootstrap --hoist

为了省去每次都输入 --hoist 参数的麻烦，可以在 lerna.json 配置：

{
"packages": [
"packages/*"
],
"command": {
"bootstrap": {
"hoist": true
}
},
"version": "0.0.1-alpha.0"
}

配置好后，对于之前依赖包已经被安装到各个 package 下的情况，我们只需要清理一下安装的依赖即可：

lerna clean


3.2 优雅的提交
3.2.1 commitizen && cz-lerna-changelog
commitizen 是用来格式化 git commit message 的工具，它提供了一种问询式的方式去获取所需的提交信息。
cz-lerna-changelog 是专门为 Lerna 项目量身定制的提交规范，在问询的过程，会有类似影响哪些 package 的选择。

安装完成后，在 package.json 中增加 config 字段，把 cz-lerna-changelog 配置给 commitizen。同时因为commitizen不是全局安全的，所以需要添加 scripts 脚本来执行 git-cz

{
"name": "root",
"private": true,
"scripts": {
"commit": "git-cz"
},
"config": {
"commitizen": {
"path": "./node_modules/cz-lerna-changelog"
}
},
"devDependencies": {
"commitizen": "^3.1.1",
"cz-lerna-changelog": "^2.0.2",
"lerna": "^3.15.0"
}

之后在常规的开发中就可以使用 yarn run commit 来根据提示一步一步输入，来完成代码的提交。

3.2.2 commitlint && husky
上面我们使用了 commitizen 来规范提交，但这个要靠开发自觉使用yarn run commit 。万一忘记了，或者直接使用 git commit 提交怎么办？答案就是在提交时对提交信息进行校验，如果不符合要求就不让提交，并提示。校验的工作由 commitlint 来完成，校验的时机则由 husky 来指定。husky 继承了 Git 下所有的钩子，在触发钩子的时候，husky 可以阻止不合法的 commit,push 等等。

安装 commitlint 以及要遵守的规范

yarn add -D @commitlint/cli @commitlint/config-conventional

在工程根目录为 commitlint 增加配置文件 commitlint.config.js 为commitlint 指定相应的规范

module.exports = { 
extends: ['@commitlint/config-conventional'] 
}

安装 husky
yarn add -D husky

在 package.json 中增加如下配置
"husky": { 
"hooks": { 
"commit-msg": "commitlint -E HUSKY_GIT_PARAMS" 
}
}

"commit-msg"是git提交时校验提交信息的钩子，当触发时便会使用 commitlit 来校验。安装配置完成后，想通过 git commit 或者其它第三方工具提交时，只要提交信息不符合规范就无法提交。从而约束开发者使用 yarn run commit 来提交。


feat:     ✨  A new feature (note: this will indicate a release)  一个新特性（注意：这表示发布）
fix:      🛠  A bug fix (note: this will indicate a release)  错误修复（注意：这表示发布）
docs:     Documentation only changes  仅文档更改
style:    Changes that do not affect the meaning of the code 不影响代码含义的更改(空白、格式、缺少分号等)
(white-space, formatting, missing semi-colons, etc) 
refactor: A code change that neither fixes a bug nor adds a feature  既不修复bug也不添加特性的代码更改
perf:     A code change that improves performance  提高性能的代码更改
test:     Adding missing tests  添加缺少的测试
chore:    Changes to the build process or auxiliary tools  对生成过程或辅助工具的更改,例如生成库和文档
and libraries such as documentation generation 
revert:   Revert to a commit  恢复为提交
❯ WIP:      Work in progress  正在进行的工作


? Denote the scope of this change:  表示此更改的范围
? Write a short, imperative tense description of the change: 写一个简短的，祈使时态的变化描述
文档修改
? Provide a longer description of the change (optional). Use "|" to break new line: 提供更改的详细说明（可选）。使用“|”换行

? List any BREAKING CHANGES (if none, leave blank): 列出任何中断更改（如果没有，留空）

? List any ISSUES CLOSED by this change (optional). E.g.: #31, #34: 列出此更改关闭的所有问题（可选）。E、 g.：#31，#34

? The packages that this commit has affected (0 detected) 此提交已影响的包（检测到0）
(Press <space> to select, <a> to toggle all, <i> to invert selection) 按<space>选择，<a>切换全部，<i>反转选择
