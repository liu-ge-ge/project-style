<h1 align="center">代码提交规范</h1>

|   type   |                       Value                        |
| :------: | :------------------------------------------------: |
|   feat   |                    新增一个功能                    |
|   fix    |                    修复一个BUG                     |
|   docs   |                      文档变更                      |
|  style   |  代码格式（不影响功能，例入空格、分好等格式修正）  |
| refactor |                      代码重构                      |
|   perf   |                      改善性能                      |
|   test   |                        测试                        |
|  build   |               变更项目构建或外部依赖               |
|    ci    | 更改持续集成软件的配置文件和package中的scripts命令 |
|  chore   |               变更构建流程或辅助工具               |
|  revert  |                      代码回退                      |

## commitlint

## Husky

husky是常见的git hook工具 ，使用它可以挂载Git钩子，当我们本地进行git commit 或 git push 操作前，能够执行一些操作，比如进行eslint检查，如果不通过，就不允许commit / push

- 什么是git hook

> 是在**Git** 中触发重要动作时触发的自定义脚本，客户端的钩子用的比较多，包括提交工作流钩子，电子邮件工作流钩子等等。这些钩子通常存储在项目的.git / hooks 下，需要关注的主要的钩子是提交工作流钩子，主要包括一下几种

- **pre-commit** : 该钩子在键入提交信息前运行。它用于检查即将提交的快照。如果该钩子以非零退出，Git将放弃此次提交，可以利用该钩子，来检查代码风格是否一致。
- **prepare-commit-msg** : 该钩子在启动提交信息编辑器之前，默认信息被创建之后运行，允许编辑提交者所看到的默认信息。 commit-msg : 该钩子接受一个参数,此参数存有当前提交信息的临时文件的路径。如果该钩子脚本以非零退出，Git将放弃提交，因此可以用来在提交通过前验证项目状态或提交信息
- **post-commit** : 该钩子一般用于通知之类的事情

```shell
# 安装husky
npm i husky -s

# 手动初始化
> package.json
 "scripts": {
    "prepare": "husky install"
  }

# 运行
# 确保 git init 之后
npm run prepare

# 然后再根目录下就有一个.husky的文件夹

```

##添加 commit-msg hook
commit-msg 文件中可以配置在 git commit 时对commit注释的校验指令

- 执行 npx husky add .husky/commit-msg `npx --no -- commitlint --edit "$1"`
  > 会在.husky 下创建一个commit-msg 的文件

```shell
 #!/usr/bin/env sh
. "$(dirname -- "$0")/_/husky.sh"

#表示在git commit 前先执行一下 npx --no -- commitlint --edit "$1" 指令，对commit的注释进行校验
```

## 添加 pre-commit hook

> pre-commit 文件中可以配置在git commit 前需要执行的操作

```shell
npx husky add .husky/pre-commit "npm run lint-staged"

# 在 commit 之前运行 npm run lint-staged ，意思就是对所有代码进行eslint 校验和stylelint校验不符合规则就终止commit

```

## eslint & stylelint

- eslint

```shell
1. npm i eslint # 安装

2. npx eslint --init  初始化文件


```

- stylelint

```shell
npm i stylelint
# "postcss-less": "^6.0.0", //用于解析less
# "stylelint": "^16.2.0",
# "stylelint-config-recommended-less": "^3.0.1", //less规则
# "stylelint-config-standard": "^36.0.0", //推荐规则
# "stylelint-order": "^6.0.4" //用于规范css编写顺序

```

> .stylelintrc.js

```js
module.exports = {
    root:true,
    //集成其它规则，用来扩展配置
    extends:[
        "stylelint-config-standard",
        'stylelint-config-recommended-less'
    ],
    //引入插件，用于扩展stylelint的原生rules
    plugins:[
        'stylelint-order'
    ],
    //为不同格式的文件分别配置
    overrides:[
        {
            files:["**/*.{less | vue}"],
            customSyntax:'postcss-less'
        }
    ],
    rules:{
        'order/properties-order': [
            'position',
            ...
          ]
    }
}

```

## Lint-staged

- 什么是Lint-staged

> 就是当我们运行ESlint 或 Stylelint 命令时，可以通过设置指定只检查我们通过git add 添加到缓存区的文件,可以避免我们每次检查都把整个项目的代码都检查一遍，从而提高效率.

```shell
npm i lint-staged

```

> package.json 添加

```json
  "lint-staged": {
    "*.{js,jsx,ts,tsx}": [
      "eslint --fix",
      "prettier --write"
    ],
    "*.json": [
      "prettier --write"
    ],
    "*.vue": [
      "eslint --fix",
      "prettier --write",
      "stylelint --fix"
    ],
    "*.{scss,less,html}": [
      "stylelint --fix",
      "prettier --write"
    ]
  }
```

> 当我们执行git commit 时 ，就会触发husky 的pre-commit 钩子,调用lint-staged命令，而lint-staged包含了对各种文件类型的操作
>
> 若前面的指令都执行通过，那么将通过 git add 命令将文件重新加入到本地的 git commit 中，如果没有执行通过，那么将不能 commit
