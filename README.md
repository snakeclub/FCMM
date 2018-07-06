# FCMM - 流程配置管理模型
## LICENSE

**@黎慧剑 CC-BY-NC-ND**



## 简介

**current Version：V1.0.0**

流程配置管理模型（Flow-based Configuration Management Model）是基于软件开发标准流程所定义的一套配置管理模型，为软件开发团队提供配置管理规范制定、流程执行等方面的参考，在保障软件开发过程中对必要过程资产保存的前提下，降低版本控制风险，减少开发团队在配置管理工作上的学习成本和操作成本的投入。

根据开发模式的不同，使用者可根据实际的开发流程、人员分工等不同，自行对模型及流程进行裁剪，以更好地适应自身的需要。

该模型可以在各类配置管理工具上落地实现，但主要概念是参考Git的配置管理观念，同时本项目也提供了一个简单的python工具fcmm4git来简化开发人员在自己电脑上按照FCMM配置和管理项目的工作效率。

**注：git是分布式的配置管理，每一个仓库（不管本地还是服务端）都可以是主版本，但FCMM的原则是以服务器端版本作为主库管理，版本应以服务器为准，这点需在使用时注意。**

## 1. 标准词汇参考

| **词汇**         | **缩写** | **说明**                                           |
| ---------------- | -------- | -------------------------------------------------- |
| long-branch      | lb       | 长期版本分支，该分支建立后会一直持续更新，不会删除 |
| temp-branch      | tb       | 临时分支，针对临时任务建立，任务最终完成后即可删除 |
| package          | pkg      | 打包，主要指程序定版                               |
| configure        | cfg      | 参数配置                                           |
| develop          | dev      | 开发                                               |
| sit              | sit      | SIT测试                                            |
| uat              | uat      | UAT测试                                            |
| quasi-production | qprd     | Quasi production environment，准生产环境           |
| test             | test     | 测试                                               |
| production       | prd      | 生产，例如production environment：生产环境         |
| requirement      | req      | 需求                                               |
| hotfix           | fix      | 补丁                                               |
| feature          | feat     | 特性                                               |
| version          | ver      | 版本                                               |
| publish          | pub      | 发布                                               |
| backup           | bak      | 备份                                               |



## 2.  配置管理分支模型

![](/docs/media/README-pic001.png)

**模型中的各类配置管理分支的定义如下：**

- master : 主版本，代码版本保持与已上线的生产版本一致
- 定版（package，长期分支）：为通过测试后，已完成定版，待投产的代码版本
- 配置（Configure，长期分支）：用于保存不同环境（如SIT、UAT、准生产、生产等）配置信息的分支，主要用于根据不同的提交要求，将环境配置信息合并到版本中
- 需求/故障/特性（requirement/hotfix/feature，临时分支）：针对不同开发需求拆分为独立分支进行开发版本管理，需求投产后可删除
- 测试（Test，临时分支）：用于生成测试版本（合并环境配置和开发分支）的临时分支，测试任务完成后可删除
- 发布（Publish，临时分支）：用于合并主程序代码及环境配置信息，发布到特定环境执行



## 3. 分支命名规范及版本号规范

### （1） 分支命名规范

- 长期分支以lb开头，临时分支以tb开头，词与词之间用’-’代替例如tb-merge-；
- 定版分支：固定命名为lb-pkg （long branch package）；
- 配置分支：规范为lb-cfg-环境标识（long branch configure），例如lb-cfg-sit（SIT测试）、lb-cfg-uat（UAT测试）、lb-cfg-prd（生产）、lb-cfg-dev（开发）；
- 开发分支：分为多类，tb-req-需求标识（例如需求编号）、tb-fix-缺陷标识（例如故障编号）、tb-feat-特性标识
- 测试分支：tb-test-环境标识，例如tb-test-sit、tb-test-uat
- 发布分支：tb-pub-环境标识，例如tb-pub-sit、tb-pub-uat
- 备份分支：tb-bak-备份版本标识，版本标识只要能让人区分清除就好，例如tb-bak-**20180706-by-lhj**
- 开发者临时分支：tb-dev-开发者标识，该分支用于开发者上传临时代码服务器中自己的临时分支（例如临时切换其他开发任务，或者用于多台电脑的开发代码传输）

### （2）版本号Tag规范

- 基于产品的版本号：格式为标准的以v开头的3段式编号，第1段为大版本，当有不兼容改造（breaking）时，大版本应加1；第2段为中版本，当有新功能（feature）改造时，中版本应加1；第3段为小版本，当有bug修复时（fix），小版本应加1；例如v0.0.1
- 基于投产日期的版本号：以d（date）开头，然后为8位日期，最后部分为“-”加序号，用于标识版本的上线日期，例如d20180620-1；
- 基于投产时间的版本号：以dt（datetime）开头，然后后8位日期+“-"+6位时间，用于标识版本的上线时间,例如dt20180620-010338；



## 4. 标准处理流程

![](/docs/media/README-pic002.png)

### 4.1. 主流程说明

​	（1）当收到各类改造需求时，从lb-pkg（定版分支）中获取指定版本的代码，通过checkout创建对应的开发分支；
​	（2）各开发分支开发完成，根据版本计划，基于lb-pkg（版本分支）的指定版本创建tb-test-sit（SIT测试分支），并将各开发分支内容合并（merge）进来；
​	（3）通过tb-pub-sit（SIT发布分支）将SIT的版本和环境信息合并起来，供一并编译并部署到SIT服务器上进行测试；
	（4）基于lb-pkg（版本分支）的指定版本创建tb-test-uat（UAT测试分支），对于通过SIT的版本，从tb-test-sit发布到tb-test-uat（UAT测试分支）； 
​	（5）通过tb-pub-uat（UAT发布分支）将UAT的版本和环境信息合并起来，供一并编译并部署到UAT服务器上进行测试；
	（6）通过UAT测试的版本，可以从tb-test-uat合并到lb-pkg（定版分支），标注版本号为上线日期，等待最终上线；
​	（7）通过tb-pub-prd（生产发布分支）将定版的版本和环境信息合并起来，供一并编译并部署到生产环境（上线）；
​	（8）对于已上线的代码，从lb-pkg（定版分支）合并到master版本，完成生产版本的同步更新。

### 4.2. 流程基本原则

​	流程最基本的原则是要保证所设立分支的最长提交流程，不允许跨流程节点跳跃提交的做法，基于该原则，大致有以下的流程要求：

​	（1）存在测试分支的情况下（tb-test-），开发分支（tb-req-、tb-feat-、tb-fix-）只允许合并到测试分支，不允许合并到版本分支（lb-pkg）和主版本（master）；
	（2）测试分支（tb-test-）应按实际的测试环节进行合并，非第一个节点的测试分支不允许从开发分支合并版本（tb-req-、tb-feat-、tb-fix-）；
	（3）版本分支（lb-pkg）只能从最后一个测试分支（tb-test-）合并版本；
	（4）主版本（master）只能从版本分支（lb-pkg）合并版本；
	（5）与环境相关的代码不允许放到非配置分支（lb-cfg-）上。

### 4.3. 流程裁剪

可根据实际项目的需要，对配置管理分支和流程进行裁剪，来调节风险控制与执行效率的平衡。可进行的裁剪情景说明如下：

（1）单人开发（最精简场景）：单人开发的情况一般没有并行代码提交的问题，可以只保留master和开发分支，直接在开发分支上进行测试，测试通过的代码直接从开发分支同步到master；

（2）无待投产的场景：如果开发模式不存在定版待投产的状态，可以取消版本分支（lb-pkg），直接使用master作为版本分支进行处理；

（3）无发布分支的场景：建议与环境相关的信息开发为外部可替换的配置文件方式，不要写入代码中，这样可以编译程序可以直接分别获取测试分支（tb-test-）或版本分支（lb-pkg）的代码部分直接编译，然后再获取配置分支（lb-cfg-）的配置文件替换到服务器部署上，无需增加发布分支（tb-pub-）；

（4）按需设置测试分支：测试分支数量可按实际需要设置，不一定要区分sit、uat等多个环境。



## 5. 提交说明规范

每次代码提交必须填写提交说明，以保证过程信息能完整、准确地体现代码变更过程，同时通过规范提交说明内容，也可在最终版本打包时能自动生成变更说明内容。FCMM要求的提交说明规范遵循[*Angular*JS项目Git提交日志规范](https://github.com/angular/angular.js/blob/f3377da6a748007c11fde090890ee58fae4cefa5/CONTRIBUTING.md#commit_)。对于使用GIT进行配置管理的情况，可以使用典型的git工作流程或通过使用CLI向导Commitizen来添加提交说明消息格式，让使用人员参考并生成符合AngularJS规范的提交说明 。

可参考以下文档在Windows上安装使用Commitizen：[《Commitizen在Windows的安装使用手册》](https://github.com/snakeclub/DevStandards/blob/master/docs/git/commitizen-windows.md)。

以下对AngularJS提交日志规范进行说明：

### 提交消息格式

每次提交，Commit message 都包括三个部分：header，body 和 footer。其中，header 是必需的，body 和 footer 可以省略。不管是哪一个部分，任何一行都不得超过72个字符。这是为了避免自动换行影响美观。格式如下（<*BLANK LINE*>代表空行）：

```
<type>(<scope>): <subject>
<BLANK LINE>
<body>
<BLANK LINE>
<footer>
```



#### Header

Header部分只有一行，包括三个字段：**type**（必需）、**scope**（可选）和**subject**（必需） 。



**Type** 用于说明 commit 的类别，只允许使用下面7个标识：

- feat：新功能（feature）

- fix：修补bug

- docs：文档（documentation）

- style： 格式（不影响代码运行的变动）

- refactor：重构（即不是新增功能，也不是修改bug的代码变动）

- test：增加测试

- chore：构建过程或辅助工具的变动

​        如果type为feat和fix，则该 commit 将肯定出现在 Change log 之中。其他情况（docs、chore、style、refactor、test）由你决定，要不要放入 Change log，建议是不要。



**Scope**：用于说明 commit 影响的范围，比如数据层、控制层、视图层等等，视项目不同而不同。例如在Angular，可以是$location, $browser, $compile, $rootScope, ngHref, ngClick, ngView等。如果你的修改影响了不止一个scope，你可以使用\*代替，例如root\*。



**Subject**是 commit 目的的简短描述，不超过50个字符，需要注意的事项有：

- 以动词开头，使用第一人称现在时，比如change，而不是changed或changes
- 第一个字母小写
- 结尾不加句号（.）



#### Body

Body 部分是对本次 commit 的详细描述，可以分成多行，下面是一个范例：

  ```
  More detailed explanatory text, if necessary.  Wrap it to 
  about 72 characters or so. 
  
  Further paragraphs come after blank lines.
  
  - Bullet points are okay, too
  - Use a hanging indent
  ```
body部分有3个注意点:

- 使用第一人称现在时，比如使用change而不是changed或changes
- 永远别忘了第2行是空行
- 应该说明代码变动的动机，以及与以前行为的对比



#### Footer

Footer 部分只用于两种情况：不兼容变动（**BREAKING CHANGE**）和关闭问题（**Issue**）。



**不兼容变动**

如果当前代码与上一个版本不兼容，则 Footer 部分以**BREAKING CHANGE**开头，后面是对变动的描述、以及变动理由和迁移方法，例如以下示例：

```
BREAKING CHANGE: isolate scope bindings definition has changed.

    To migrate the code follow the example below:

    Before:

    scope: {
      myAttr: 'attribute',
    }

    After:

    scope: {
      myAttr: '@',
    }

    The removed `inject` wasn't generaly useful for directives so there should be no code using it.
```



**关闭问题**

如果当前 commit 针对某个issue，那么可以在 Footer 部分关闭这个 issue 。例如：

```
Closes #234
```

关闭其他仓库的Issue：

```
Closes example_user/example_repo#76
```

关闭多个Issue：

```
This closes #34, closes #23, and closes example_user/example_repo#42
```



#### Revert

还有一种特殊情况，如果当前 commit 用于撤销以前的 commit，则必须以**revert:**开头，后面跟着被撤销 Commit 的 Header。例如：

```
revert: feat(pencil): add 'graphiteWidth' option

This reverts commit 667ecc1654a317a13331b17617d973392f415f02.
```

​	**Body** 部分的格式是固定的，必须写成 **This reverts commit &lt;hash>**.，其中的hash是被撤销 commit 的 SHA 标识符。
	如果当前 commit 与被撤销的 commit，在同一个发布（release）里面，那么它们都不会出现在 Change log 里面。如果两者在不同的发布，那么当前 commit，会出现在 Change log 的Reverts小标题下面。



## 6. 文件规范

FCMM对项目仓库的文件结构有一定的规范要求，以下为一个标准的文件结构：

```
ProjectName
  |__src/
  |__test/
  |__data/
  |__docs/
  |__package.json
  |__README.md
  |__CHANGELOG.md
  |__LICENSE
```

### package.json

package.json是每个项目根目录必须包含的文件，用于登记项目名称、版本、依赖等一些关键信息。package.json的属性定义可以参考阮一峰老师的开源项目《JavaScript标准参考教程》中的[package.json文件](https://github.com/ruanyf/jstutorial/blob/gh-pages/nodejs/packagejson.md)章节。以下为针对FCMM的主要属性定义说明：

#### name

必须，用于说明项目名称。

#### version

必须，用于说明项目当前版本，遵守“大版本.次要版本.小版本”的格式。

#### repository

必须，用于说明项目版本管理仓库信息，如果url为空代表是本地仓库，无远程仓库。示例：

```
"repository": {
    "type": "git",
    "url": "git+https://github.com/snakeclub/FCMM.git"
  }
```

#### scripts

非必须，指定了运行脚本命令的npm命令行缩写，比如start指定了运行`npm run start`时，所要执行的命令。例如项目中使用了standard-version进行版本生成，则应有以下配置：

```
"scripts": {
    "release": "standard-version"
  }
```

#### config

非必须，用于添加命令行的环境变量。例如项目中使用了commitizen，可通过该配置指定执行文件路径：

```
"config": {
    "commitizen": {
      "path": "C:/Users/hi.li/AppData/Roaming/nvm/npm/node_modules/cz-conventional-changelog"
    }
  }
```



### README.md

README.md文件用于说明项目内容，包括介绍、使用手册等信息，是每个项目必要的文档。README.md文件遵循Markdown标签文档的规范，可以用Markdown编辑器编写和浏览。推荐一款免费的Markdown编辑器[typora](https://www.typora.io/)。



### CHANGELOG.md

非必须，CHANGELOG.md文件用于说明每个版本的修改内容。建议通过conventional-changelog自动生成，相关按照及使用手册参考[《Commitizen在Windows的安装使用手册》](https://github.com/snakeclub/DevStandards/blob/master/docs/git/commitizen-windows.md)。



### LICENSE

非必须，说明本项目采用的许可协议。



### 路径规范

非必须，本处列出部分标准的路径说明：

**src** : 源代码路径

**test** : 测试代码路径

**data** : 装载数据路径

**docs** : 文档路径



## 7. fcmm4git

**fcmm4git**（FCMM for Git）是基于FCMM模型，针对git简化进行代码托管的命令行小工具，大家可在Github的[fcmm4git开源项目](https://github.com/snakeclub/fcmm4git/blob/master/README.md)中了解并获取最新版本。



## 8. 项目反馈

对于FCMM存在的问题或建议，可进入Github的[Issues](https://github.com/snakeclub/FCMM/issues)进行反馈，我将尽快修复和答复。

同时欢迎大家积极开发和发布基于FCMM的版本管理小工具（例如fcmm4svn等），以提高版本管理的效率。