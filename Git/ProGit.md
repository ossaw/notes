# ProGit读书笔记

--- 

### git 配置命令

命令来列出所有Git当时能找到的配置
git config --list

检查Git的某一项配置
git config <key>
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
