- [引子](#%e5%bc%95%e5%ad%90)
- [Git工程化实践](#git%e5%b7%a5%e7%a8%8b%e5%8c%96%e5%ae%9e%e8%b7%b5)
  - [信息配置](#%e4%bf%a1%e6%81%af%e9%85%8d%e7%bd%ae)
    - [用户信息配置](#%e7%94%a8%e6%88%b7%e4%bf%a1%e6%81%af%e9%85%8d%e7%bd%ae)
    - [远程信息配置](#%e8%bf%9c%e7%a8%8b%e4%bf%a1%e6%81%af%e9%85%8d%e7%bd%ae)
  - [.gitignore](#gitignore)
  - [GitHook配置](#githook%e9%85%8d%e7%bd%ae)
- [细则讲解](#%e7%bb%86%e5%88%99%e8%ae%b2%e8%a7%a3)
  - [branch 命名规范](#branch-%e5%91%bd%e5%90%8d%e8%a7%84%e8%8c%83)
  - [branch 操作规范](#branch-%e6%93%8d%e4%bd%9c%e8%a7%84%e8%8c%83)
    - [合并流程](#%e5%90%88%e5%b9%b6%e6%b5%81%e7%a8%8b)
  - [commit 规范](#commit-%e8%a7%84%e8%8c%83)
    - [commit msg规范](#commit-msg%e8%a7%84%e8%8c%83)
    - [commit 粒度](#commit-%e7%b2%92%e5%ba%a6)
- [Git Alias](#git-alias)
  - [开源的 alias 规范](#%e5%bc%80%e6%ba%90%e7%9a%84-alias-%e8%a7%84%e8%8c%83)
    - [zsh](#zsh)
    - [GitBash](#gitbash)
    - [GitAlias](#gitalias)
  - [辅助命令封装](#%e8%be%85%e5%8a%a9%e5%91%bd%e4%bb%a4%e5%b0%81%e8%a3%85)


# 引子
本repo基于 [git-standardize](https://github.com/vimerzhao/git-standardize) 修改，旨在更好地在实践中使用Git
* 操作直接看 [Git工程化实践](#git%e5%b7%a5%e7%a8%8b%e5%8c%96%e5%ae%9e%e8%b7%b5) 章节
* 原理可看 [细则讲解](#%e7%bb%86%e5%88%99%e8%ae%b2%e8%a7%a3) 章节，再辅以参考资料

其他参考资料：
* [Git对象介绍](https://github.com/vimerzhao/vimerzhao.github.io/blob/master/dev-tools/2019-11-26-git-internals.adoc)
* [工作流管理](https://nvie.com/posts/a-successful-git-branching-model)
* [commit规范](https://github.com/angular/angular.js/blob/master/DEVELOPERS.md#-git-commit-guidelines)


# Git工程化实践
## 信息配置
使用 `git config` 配置必须要的用户信息
* `--local`: 本repo设置，级别最高  (默认)
* `--global`: 用户通用，级别中等
* `--system`: 系统通用，级别最低

> 使用`git config --<level> -l`查看配置
>
> 使用`git config --<level> -e`编辑配置文件

### 用户信息配置
一般是配置`--global`级别，用户通用：
```shell
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
```
若本repo不通用，可通过`--local`单独配置

### 远程信息配置
* 查看： `git remote`
* 添加： `git remote add origin <server>`
* 密钥： https://www.cnblogs.com/SUNSHINEC/p/8617029.html

## .gitignore
[Git - gitignore Documentation](https://git-scm.com/docs/gitignore): 忽略无需Git管理的文件
* [github-gitignore](https://github.com/github/gitignore): 提供了各类型项目的 `.gitignore` 模版，基本可以直接使用。
* 一般而言，本地的配置、编译生成的中间文件是需要在项目建立之初就添加 `.gitignore` 的。

## GitHook配置
GitHook应该在仓库创建之后尽早设置

```shell
# working dir
# after `git init`
git clone https://github.com/mifenfeifen/git-standardize.git
cp -R ./git-standardize/.githooks ./
chmod +x .githooks/*[^rule]
git config core.hooksPath .githooks
rm -rf git-standardize
```

如果管理员配置好了hook并完成了服务器端的push，那么对于其他开发者，只需要

```shell
git pull --rebase
git config core.hooksPath .githooks
```

# 细则讲解
## branch 命名规范
个人开发时，主要使用[Git-Workflow](https://nvie.com/posts/a-successful-git-branching-model)
* 以 `master` 和 `develop` 为主要分支
* 完成任务的分支，要及时删除
* （[不同工作流的差别](https://blog.csdn.net/qq_32452623/article/details/78905181)）

![git-workflow-pic](https://nvie.com/img/git-model@2x.png)

只允许使用以下分支命名项目

* `master`
  - 主分支（用于部署生产环境的分支）：确保 `master` 分支稳定性，任何时间都**不能直接在该分支修改代码**
  - 利用tag进行标示：三段式 `A.B.C` ： `A` 为大版本号，不随意改动； `B` 为小版本号，用于功能迭代； `C` 为线上问题修复版本号
* `develop`/`dev`
  - 日常开发分支
* `refactor`
  - 试验性重构代码
  - 一般都是 breaking change
* `release`
  - `release` 为预上线分支，发布提测阶段，会以 `release` 分支代码为基准提测
  - 分支命名细则: `release-version`
  - 当有一组 `feature` 开发完成，首先会合并到 `develop` 分支。进入提测时，会创建 `release` 分支
  - **如果测试过程中若存在bug需要修复，则直接由开发者在 `release` 分支修复并提交**
  - 当测试完成，经过一个周期的灰度之后，合并 `release` 分支到 `master` 分支
  - `release` 分支的最终状态是每个线上版本的归档镜像
* `hotfix`
  - 线上出现紧急问题时，需要及时修复，以 `master` 分支为基线，创建 `hotfix` 分支，修复完成后，需要合并到 `master` 分支和 `develop` 分支
  - 分支命名细则: `hotfix-creator-description`
* `feature`/`topic`
  - 开发新功能时，以 `develop` 为基础创建 `feature` 分支
  - 分支命名细则: `feature-creator-description`

## branch 操作规范
git 中合并不同分支主要有两种方法：
* `merge`
  - fast-forward: （默认）直接移动`HEAD`指针到最新的提交，从commit history无法体现merge
  - recursive strategy: 非快进式合并：`git merge --no-ff`
* `rebase`: 修改commit历史，变为**线性提交历史**，无分叉
  - 会改变分支 branch out 节点
  - 线性，但不严格按时间线
  - **不要对多人开发的分支进行rebase**，极易引发冲突

> **不同分支之间，要勤于反复合并，避免以后的冲突**
>
> 注意`cherry-pick`的使用，可以减少反复合并，避免污染 commit history

### 合并流程
1. 更新`develop`到最新（从remote中`pull`最新的代码）
2. 将本地`feature`分支通过`git rebase dev`合并dev并整理成线性 commit history
3. 切换到`dev`分支，`git merge --no-ff feature`合并

> PS: 主要是为了梳理feature里的各种merge —— 如果feature没有merge，可以跳过rebase

## commit 规范
### commit msg规范
参考 [angular.js commit规范](https://github.com/angular/angular.js/blob/master/DEVELOPERS.md#-git-commit-guidelines)
* 在 commit 信息中写明操作内容，方便后续管理和查看

> 键入`git commit`后enter，即可通过editor编辑msg（默认是`vi`）

```shell
<type>(<scope>):<subject>  # <1> <2> <3>
<BLANK LINE>
<body>  # <4>
<BLANK LINE>
<footer>  # <5>
```
1. `type`: 本次改动的类型
   * `feat`: 添加新特性
   * `fix`: 修复bug
   * `refactor`: 代码重构，没有加新功能或者修复bug
   * `docs`: 仅仅修改了文档
   * `style`: 仅仅修改了空格、格式缩进、都好等等，不改变代码逻辑
   * `chore`: 改变构建流程、或者增加依赖库、工具等
   * `perf`: 增加代码进行性能测试
   * `test`: 增加测试用例
2. `scope`:本次改动影响的范围，建议每个工程划分好自己的模块，方便填写
   * `$MODULE` : 模块
   * `.` : 当前文件
   * `*` : 多个文件
3. `subject`: 本次改动的简要描述，一般写这个就够了
4. `body`: 更详细的改动说明
5. `footer`: 描述下与之关联的 issue 或 break change，一般不使用

### commit 粒度
commit 粒度要合适，方便 review
* `git commit --amend -C head`: 提交到上次提交中（不生成新的commit对象）
  - `-C head`指复用head指向的commit中的msg，不添加可重写
* `git rebase -i`: 通过交互式rebase，事后整理 commit history
  - https://git-scm.com/docs/git-rebase


# Git Alias
别名在类Unix系统中普遍存在，即将我们最常用的命令设置一个更短更容易记住的别名，以提高使用效率。
* 省力省心，开源社区长期优化的配置大概率比我们自己折腾的更加科学和健壮
* 通用性强，任何一个使用 zsh 的用户和你的别名习惯都是一致的，便于交流

## 开源的 alias 规范
### zsh
zsh的 `git` 插件提供了一套别名，推荐使用，避免重复造轮子
* [文档 git-cheatsheet](https://github.com/robbyrussell/oh-my-zsh/wiki/Cheatsheet#git)
* [配置 git.plugin.zsh](https://github.com/robbyrussell/oh-my-zsh/blob/master/plugins/git/git.plugin.zsh)


### GitBash
如果开发环境是 Windows，目前看来还是使用 GitBash 最佳，推荐将 `git.plugin.zsh` 的别名规则平移到 GitBash 下，这样也符合上文提到的通用性原则。

移植方法很简单，如下
```shell
move path/to/oh-my-zsh/plugins/git/git.plugin.zsh ~/.git.plugin.zsh
echo `source ~/.git.plugin.zsh >> .bash_profile`
```

### GitAlias
GitBash 使用 *Linux别名* 时有一个不太友好的问题，就是不支持自动补全，对于像 `git checkout` 这样的命令，非常需要自动补全，否则很容易拼错分支名称。

oh-my-zsh 的别名通过`shell alias`实现，即：
```shell
alias gbr='git branch --remote'
alias gbs='git bisect'
alias gbsb='git bisect bad'
alias gbsg='git bisect good'
```

GitAlias 均以 `git` 开头，所以对应的Git别名配置可以是
```shell
git config --global alias.br    'branch --remote'
git config --global alias.bs    'bisect'
git config --global alias.bsb   'bisect bad'
git config --global alias.bsg   'bisect good'
```

> Linux别名： 通过bash命令 `alias` 设置。
> 
> Git别名：通过 `git config --global alias` 设置。

具体操作如下：

```shell
# 提取所有关于别名的配置，并暂存
grep -E "alias g[a-zA-Z]+=.*" path/to/git.plugin.zsh > temp1.sh
# 改成GitAlias风格的别名
sed "s/^alias g/git config --global alias./g" temp1.sh > temp2.sh
# 去掉命令里面的 `git`，符合 GitAlias 语法
sed "s/git //g" temp2.sh > git.alias.config.sh
# 进行配置，运行后会注册到 .gitconfig
source git.alias.config.sh
# 删除中间文件
rm temp*
```

## 辅助命令封装
所有的辅助命令均以 `gs_` 开头，这样的好处是可以利用 `Tab` 键的补全机制自动选择命令，避免冗长难记的输入
* `gs_clear_local_barnch` ：清理本地存在但是服务器端不存在的分支 +
* `gs_branch_last_commit` ：查看分支最后提交人和存活周期，辅助删除过期分支
* `gs_past_commit_statistic` ：统计过去一段时间内的代码提交数量，参数为时间段或者起始时间，如 `7.days` 、`2019-10-10`

