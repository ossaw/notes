## Git 分支

---

#### 分支创建

**git branch branchname**

* eg: git branch testing

![两个指向相同提交历史的分支](https://github.com/ossaw/notes/blob/master/Pictures/git/pro-git-02.png)
![HEAD 指向当前所在的分支](https://github.com/ossaw/notes/blob/master/Pictures/git/pro-git-03.png)

#### 分支切换

**git checkout branchname**

* eg: git checkout testing

![HEAD 指向当前所在的分支](https://github.com/ossaw/notes/blob/master/Pictures/git/pro-git-04.png)

再提交一次

![HEAD 分支随着提交操作自动向前移动](https://github.com/ossaw/notes/blob/master/Pictures/git/pro-git-05.png)

切换回 master 分支

![检出时 HEAD 随之移动](https://github.com/ossaw/notes/blob/master/Pictures/git/pro-git-06.png)

在master分支做些修改并提交

![项目分叉历史](https://github.com/ossaw/notes/blob/master/Pictures/git/pro-git-07.png)

### 分支的新建与合并

#### 新建分支

新建一个分支并同时切换到那个分支

**git checkout -b branchname**

* eg: git checkout -b iss53

![创建一个新分支指针](https://github.com/ossaw/notes/blob/master/Pictures/git/pro-git-08.png)

在iss53上做了修改并提交

![iss53 分支随着工作的进展向前推进](https://github.com/ossaw/notes/blob/master/Pictures/git/pro-git-09.png)

有个紧急问题等待你来解决。 有了 Git 的帮助，你不必把这个紧急问题和 iss53 的修改混在一起，你也不需要花大力气来还原关于 53# 问题的修改，然后再添加关于这个紧急问题的修改，最后将这个修改提交到线上分支。 你所要做的仅仅是切换回 master 分支, 但是，在你这么做之前，要留意你的工作目录和暂存区里那些还没有被提交的修改，它可能会和你即将检出的分支产生冲突从而阻止 Git 切换到该分支。 最好的方法是，在你切换分支之前，保持好一个干净的状态。 有一些方法可以绕过这个问题（即，保存进度（stashing） 和 修补提交（commit amending）），我们会在储藏与清理 中看到关于这两个命令的介绍。 现在，我们假设你已经把你的修改全部提交了，这时你可以切换回master 分支了

![基于 master 分支的紧急问题分支 hotfix branch](https://github.com/ossaw/notes/blob/master/Pictures/git/pro-git-10.png)

你可以运行你的测试，确保你的修改是正确的，然后将其合并回你的 master 分支来部署到线上。 你可以使用 git merge 命令来达到上述目的


![master 被快进到 hotfix](https://github.com/ossaw/notes/blob/master/Pictures/git/pro-git-11.png)

这个紧急问题的解决方案发布之后，你准备回到被打断之前时的工作中。 然而，你应该先删除 hotfix 分支，因为你已经不再需要它了 —— master 分支已经指向了同一个位置。 你可以使用带 -d 选项的 git branch 命令来删除分支

![继续在 iss53 分支上的工作](https://github.com/ossaw/notes/blob/master/Pictures/git/pro-git-12.png)

你在 hotfix 分支上所做的工作并没有包含到 iss53 分支中。 如果你需要拉取 hotfix 所做的修改，你可以使用 git merge master 命令将 master 分支合并入 iss53 分支，或者你也可以等到 iss53 分支完成其使命，再将其合并回 master 分支。

#### 分支的合并

假设你已经修正了 #53 问题，并且打算将你的工作合并入 master 分支。master 分支所在提交并不是 iss53 分支所在提交的直接祖先，Git 不得不做一些额外的工作。 出现这种情况的时候，Git 会使用两个分支的末端所指的快照（C4 和 C5）以及这两个分支的工作祖先（C2），做一个简单的三方合并。

![一次典型合并中所用到的三个快照](https://github.com/ossaw/notes/blob/master/Pictures/git/pro-git-13.png)

和之前将分支指针向前推进所不同的是，Git 将此次三方合并的结果做了一个新的快照并且自动创建一个新的提交指向它。 这个被称作一次合并提交，它的特别之处在于他有不止一个父提交。

![一个合并提交](https://github.com/ossaw/notes/blob/master/Pictures/git/pro-git-13.png)

删除iss53分支

git branch -d iss53

#### 遇到冲突时的分支合并

在你解决了所有文件里的冲突之后，对每个文件使用 git add 命令来将其标记为冲突已解决。 一旦暂存这些原本有冲突的文件，Git 就会将它们标记为冲突已解决。如果你对结果感到满意，并且确定之前有冲突的的文件都已经暂存了，这时你可以输入 git commit 来完成合并提交。
