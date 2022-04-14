# git 使用笔记



官方中文教程网址 

>https://git-scm.com/book/zh/v2/%E8%B5%B7%E6%AD%A5-%E5%85%B3%E4%BA%8E%E7%89%88%E6%9C%AC%E6%8E%A7%E5%88%B6



## 一些名词

**暂存区**：git中有暂存的概念，如果我们跟踪（`add`）了一个文件，那么它就会被添加到暂存区，当我们提交`commmit`它时它就会被保存为一个版本。而在非暂存区中的文件不会被存到新的版本中。





## 分支相关操作

```bash
git checkout branch2       #转到 branch2 这个分支。
git checkout -b branch3     #创建 branch3 这个分支并转到这个分支。
#上面的 -b 我觉得就是 branch 的缩写
git branch                  #查看当前有哪些分支，并且当前 checkout 哪个分支
git branch branch4          #创建一个 branch4 分支
git merge branch3		   #将 branch3  分支和当前分支合并。
git branch -d branch3       #将 branch3 分支删除。
```



## 文件相关操作

```bash
git add filename.cpp #将当前 文件夹 中的 filename.cpp 加入跟踪，当然必须是基于当前目录的完整路径，比如本例 filename.cpp 就和 .git 在同一目录。
git add *            #跟踪当前 文件夹 中的所有文件
git status           #查看当前 repository 中有改动文件的状态。 一般有改动但为跟踪会标记为红色，有改动跟踪了会是绿色
git commit			#使用该命令，会提交所有跟踪中的内容，不过会提前弹出一个文件，编辑该文件可以备注这次 commit 的内容。编辑完那个文件直接关掉就会完成 commit。 值得注意的是，该命令也会提交之前创建的本地分支
git commit -m "啥也没改"  #使用该命令可以跳过备注本次更改的步骤，本次更改的内容备注为： 啥也没改
git commit -a         #给 git commit 加上 -a 选项，Git 就会自动把所有已经跟踪过的文件暂存起来一并提交，从而跳过 git add 步骤
git rm filename.o     #将filename.o文件从暂存区中删除
```



## 查看历史

```bash
git log                #可以查看提交的历史和备注 按 q 键可以推出
git log -p             #加入 -p 后会显示每次提交的差异。
git log -p -2          #加入 -2 就只显示最近两次的提交了

```





## git 的一些设置

```bash
cat .gitignore
#忽略所有以 .o 或 .a 结尾的文件。一般这类对象文件和存档文件都是编译过程中出现的。
*.[oa]
#忽略所有名字以波浪符（~）结尾的文件，许多文本编辑软件（比如 Emacs）都用这样的文件名保存副本。 此外，你可能还需要忽略 log，tmp 或者 pid 目录，以及自动生成的文档等等。 要养成一开始就为你的新仓库设置好 .gitignore 文件的习惯，以免将来误提交这类无用的文件。
*/~
# 忽略所有的 .a 文件
*.a

# 但跟踪所有的 lib.a，即便你在前面忽略了 .a 文件
!lib.a

# 只忽略当前目录下的 TODO 文件，而不忽略 subdir/TODO
/TODO

# 忽略任何目录下名为 build 的文件夹
build/

# 忽略 doc/notes.txt，但不忽略 doc/server/arch.txt
doc/*.txt

# 忽略 doc/ 目录及其所有子目录下的 .pdf 文件
doc/**/*.pdf

#所有空行或者以 # 开头的行都会被 Git 忽略。
#可以使用标准的 glob 模式匹配，它会递归地应用在整个工作区中。
#匹配模式可以以（/）开头防止递归。
#匹配模式可以以（/）结尾指定目录。
#要忽略指定模式以外的文件或目录，可以在模式前加上叹号（!）取反。
```



## 其他问题

```bash
ybw@ybw:~/LinuxC/PracticeCode$ git branch
* (HEAD detached from refs/heads/vmUbuntu20.04)

HEAD detached from refs/heads/
```

这种情况是因为现在指向branch的指针处于**游离状态**，导致这种情况出现的原因是之前切换到了一个历史版本中（可能是你利用`git checkout < commit id>`切换到了某一次指定的提交），若此时你进行了`commit`操作，那么就会导致一个**匿名的分支**被创建，当你要进行`checkout`操作时，会提醒你，将匿名分支进行保存，而保存的方法就是给这个匿名分支新建一个分支。

### 关于token

使用以下命令可以避免每次提交都输入token



```bash
#也可以 把token直接添加远程仓库链接中，这样就可以避免同一个仓库每次提交代码都要输入token了：
git remote set-url origin https://<your_token>@github.com/<USERNAME>/<REPO>.git
#your_token：换成你自己得到的token
#USERNAME：是你自己github的用户名
#REPO：是你的仓库名称
例如
git remote set-url origin https://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@github.com/shliang0603/Yolov4_DeepSocial.git/
```



