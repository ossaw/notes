# ProGit读书笔记

--- 

### git 配置命令

命令来列出所有Git当时能找到的配置
git config --list

检查Git的某一项配置
git config key
eg: git config user.name

配置用户名
git config --global user.name "test"

配置邮箱
git config --global user.email test@example.com

配置默认编辑器(默认vim)
git config --global core.editor emacs

### git基础

---

### 获取 Git 仓库

有两种取得 Git 项目仓库的方法。 第一种是在现有项目或目录下导入所有文件到 Git 中； 第二种是从一个服务器克隆一个现有的 Git 仓库。

#### 现有目录中初始化仓库

**git init**

* 该命令将创建一个名为 .git 的子目录，这个子目录含有你初始化的 Git 仓库中所有的必须文件，这些文件是
Git 仓库的骨干。

#### 克隆现有的仓库

**git clone [url]**

* eg: git clone https://github.com/libgit2/libgit2

自定义本地仓库的名字
* eg: git clone https://github.com/libgit2/libgit2 mylibgit

Git 支持多种数据传输协议。 上面的例子使用的是 https:// 协议，不过你也可以使用 git:// 协议或者使用 SSH 传输协议，比如 user@server:path/to/repo.git 。

### 记录每次更新到仓库

![1-1](https://github.com/ossaw/notes/blob/master/Pictures/git/pro-git-01.png)

#### 检查当前文件状态

**git status**

#### 状态简览

**git status -s**

 M README
MM Rakefile
A lib/git.rb
M lib/simplegit.rb
?? LICENSE.txt

新添加的未跟踪文件前面有 ?? 标记，新添加到暂存区中的文件前面有 A 标记，修改过的文件前面有 M 标记。 你 可能注意到了 M 有两个可以出现的位置，出现在右边的 M 表示该文件被修改了但是还没放入暂存区，出现在靠左 边的 M 表示该文件被修改了并放入了暂存区。

#### 跟踪新文件

**git add filename**

* eg: git add README.md

#### 忽略文件

文件 .gitignore 的格式规范如下：

* 所有空行或者以 ＃ 开头的行都会被 Git 忽略。
* 可以使用标准的 glob 模式匹配。
* 匹配模式可以以（/）开头防止递归。
* 匹配模式可以以（/）结尾指定目录。
* 要忽略指定模式以外的文件或目录，可以在模式前加上惊叹号（!）取反。

glob 模式是指 shell 所使用的简化了的正则表达式。 星号（\*）匹配零个或多个任意字符；[abc] 匹配 25 任何一个列在方括号中的字符（这个例子要么匹配一个 a，要么匹配一个 b，要么匹配一个 c）；问号（?  ）只匹配一个任意字符；如果在方括号中使用短划线分隔两个字符，表示所有在这两个字符范围内的都可以匹配 （比如 [0-9] 表示匹配所有 0 到 9 的数字）。使用两个星号（\*）表示匹配任意中间目录，比如`a/**/z` 可以匹配 a/z, a/b/z 或 `a/b/c/z`等。

详见: https://github.com/github/gitignore

#### 查看已暂存和未暂存的修改

**git diff**

若要查看已暂存的将要添加到下次提交里的内容，可以用 git diff --cached 命令。（Git 1.6.1 及更高版本还允许使用 git diff --staged，效果是相同的，但更好记些。）

#### 提交更新

**git commit**

#### 跳过使用暂存区域

**git commit -a**

#### 移除文件

**git rm**

**git rm -f**

* 如果删除之前修改过并且已经放到暂存区域的话，则必须要用强制删除选项 -f

**git rm --cache**

* 把文件从 Git 仓库中删除（亦即从暂存区域移除），但仍然希望保留在当前工作目录中。

#### 移动文件

**git mv**

* eg :git mv file_from file_to

### 查看提交历史

**git log**

* 默认不用任何参数的话，git log 会按提交时间列出所有的更新，最近的更新排在最上面。

**git log -p -2**

* 显示每次提交的内容差异。 你也可以加上 -2 来仅显示最近两次提交

**git log --stat**

* 看到每次提交的简略的统计信息

**git log --pretty=oneline**

ca82a6dff817ec66f44342007202690a93763949 changed the version number
085bb3bcb608e1e8451d4b2432f8ecbe6306e7e7 removed unnecessary test
a11bef06a3f659402fe7563abf99ad00de2209e6 first commit

**git log --pretty=format:"%h - %an, %ar : %s"**

ca82a6d - Scott Chacon, 6 years ago : changed the version number
085bb3b - Scott Chacon, 6 years ago : removed unnecessary test
a11bef0 - Scott Chacon, 6 years ago : first commit

git log --pretty=format 常用的选项

 选项 | 说明 
:----: | :----: 
%H | 提交对象（commit）的完整哈希字串
%h | 提交对象的简短哈希字串
%T | 树对象（tree）的完整哈希字串
%t | 树对象的简短哈希字串
%P | 父对象（parent）的完整哈希字串
%p | 父对象的简短哈希字串
%an | 作者（author）的名字
%ae | 作者的电子邮件地址
%ad | 作者修订日期（可以用 --date= 选项定制格式）
%ar | 作者修订日期，按多久以前的方式显示
%cn | 提交者（committer）的名字
%ce | 提交者的电子邮件地址
%cd | 提交日期
%cr | 提交日期，按多久以前的方式显示
%s | 提交说明

**git log --pretty=format:"%h %s" --graph**

* 这个选项添加了一些ASCII字符串来形象地展示你的分支、合并历史

**git log 的常用选项**

 选项 | 说明
 :----: | :----:
-p | 按补丁格式显示每个更新之间的差异。
--stat | 显示每次更新的文件修改统计信息。
--shortstat | 只显示 --stat 中最后的行数修改添加移除统计。
--name-only | 仅在提交信息后显示已修改的文件清单。
--name-status | 显示新增、修改、删除的文件清单。
--abbrev-commit | 仅显示 SHA-1 的前几个字符，而非所有的 40 个字符。
--relative-date | 使用较短的相对时间显示（比如，“2 weeks ago”）。
--graph | 显示 ASCII 图形表示的分支合并历史。
--pretty | 使用其他格式显示历史提交信息。可用的选项包括 oneline，short，full，fuller 和 format（后跟指定格式）。

### 撤消操作

**git commit --amend**

* 提交完了才发现漏掉了几个文件没有添加，或者提交信息写错了。 可以运行带有 --amend 选项的提交命令尝试重新提交

#### 取消暂存的文件

**git reset HEAD file**

#### 撤消对文件的修改

**git checkout -- file**

* git checkout -- [file] 是一个危险的命令，这很重要。 你对那个文件做的任何修改都会消失 - 你只是拷贝了另一个文件来覆盖它。 除非你确实清楚不想要那个文件了，否则不要使用这个命令。

### 远程仓库的使用

#### 查看远程仓库

**git remote**

* 查看你已经配置的远程仓库服务器, origin - 这是 Git 给你克隆的仓库服务器的默认名字

**git remote -v**

* 显示需要读写远程仓库使用的 Git 保存的简写与其对应的 URL

#### 添加远程仓库

**git remote add shortname url**

* eg: git remote add origin https://github.com/ossaw/git-practice.git

#### 从远程仓库中抓取与拉取

**git fetch [remote-name]**

* 访问远程仓库，从中拉取所有你还没有的数据。 执行完成后，你将会拥有那个远程仓库中所有分支的引用，可以随时合并或查看。 必须注意 git fetch 命令会将数据拉取到你的本地仓库 - 它并不会自动合并或修改你当前的工作。 当准备好时你必须手动将其合并入你的工作。

#### 推送到远程仓库

**git push [remote-name] [branch-name]**

* eg: git push origin master

#### 查看远程仓库

**git remote show [remote-name]**

* eg: git remote show origin

#### 远程仓库的移除与重命名

**git remote rename oldname newname**

* eg: git remote rename origin origin1 (将 origin 重命名为 origin1)

**git remote rm name**

* eg: git remote rm origin

### 打标签

#### 列出标签

**git tag**

* git tag -l 'v1.8.5*' (查找 v1.8.5系列的标签)

#### 创建标签

Git 使用两种主要类型的标签：轻量标签（lightweight）与附注标签（annotated）

#### 附注标签

**git tag -a tagname**

* git tag -a v1.4 -m 'my version 1.4', -m 选项指定了一条将会存储在标签中的信息

#### 轻量标签

**git tag tagname**

* git tag v1.5

#### 后期打标签

对过去的提交打标签

**git tag -a v1.2 commitid

#### 共享标签

**git push origin tagname**

* git push 命令并不会传送标签到远程仓库服务器上。 在创建完标签后你必须显式地推送标签到共享服务器上。 这个过程就像共享远程分支一样 - 你可以运行 git push origin [tagname]

**git push origin --tags**

* 一次性推送很多标签，也可以使用带有 --tags 选项的 git push 命令。 这将会把所有不在远程仓库服务器上的标签全部传送到那里。

#### 删除标签

**git tag -d tagname**

* 上述命令并不会从任何远程仓库中移除这个标签

**git push <remote> :refs/tags/tagname**

* git push origin :refs/tags/v1.4 (删除远程标签)

#### 检出标签

略

### Git别名

**git config --global alias.aliascmd command**

* eg: git config --global alias.co checkout
