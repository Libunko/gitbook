# git 实用指南

##  配置 Git

### 配置全局用户

```
git config --global user.name "用户名" 
git config --global user.email "git账号"
```

### 删除全局配置
```
git config --global --unset alias.xxx
git config --global --unset user.xxx
```

### 配置别名
```
git config --global alias.co checkout
git config --global alias.ss status
git config --global alias.cm commit
git config --global alias.br branch
git config --global alias.rg reflog
```

### 配置 filemode

```
git config --add core.filemode false
```

**注：**设置为 false 时，文件权限变化是 git diff 将不会显示改变

### 设置git代理
git config --global https.proxy http://127.0.0.1:10809
git config --global http.proxy http://127.0.0.1:10809

### 取消git代理
git config --global --unset http.proxy
git config --global --unset https.proxy

## 修改默认编辑器
git config –-global core.editor vim

## 添加文件
`git add .`
`git add filename`

## 提交

-   单行commit log：`git commit -m "本功能全部完成"`

-   进入vim编辑器，可输入多行 commit 注释：`git commit` 

## 撤回 commit
`git reset --soft HEAD^`
**注：**

- HEAD^的意思是上一个版本，也可以写成`HEAD~1`，如果你进行了2次commit，想都撤回，可以使用`HEAD~2`
- 注意，仅仅是撤回commit操作，代码改动仍然保留

### reset 参数：
- --mixed
意思是：不删除工作空间改动代码，撤销commit，并且撤销git add . 操作
这个为默认参数，git reset --mixed HEAD^ 和 git reset HEAD^ 效果是一样的。

- --soft 
不删除工作空间改动代码，撤销commit，不撤销git add . 

- --hard
删除工作空间改动代码，撤销commit，撤销git add . 
注意完成这个操作后，就恢复到了上一次的commit状态。

注：
通过 git checkout -- 文件名 命令可以撤销文件在工作区的修改。
通过 git reset 文件名 命令可以撤销指定文件的 git add 操作，即这个文件在暂存区的修改。
通过 git reset 命令可以撤销之前的所有 git add 操作，即在暂存区的修改。

## 修改commit注释
- git commit --amend
此时会进入默认vim编辑器，修改注释完毕后保存就好了。

## 删除远程分支
- git push origin --delete Chapater6   可以删除远程分支Chapater6

## 推送改动到远程分支

- git push origin feature-branch:feature-branch 推送本地的feature-branch(冒号前面的)分支到远程origin的feature-branch(冒号后面的)分支(没有会自动创建)

## 暂存改动的代码

git 如何不 commit 当前分支的修改而切换到其它分支呢？且从其他分支切回该分支时能继续当前工作。

答案就是：`git stash`

`git stash save a`	将当前改动（包括 git add）保存起来并命名为 a
`git stash pop a`      恢复 a 并删掉 stash list a
`git stash apply a` 恢复 a 但不删掉 stash list a

参考资料：
https://blog.csdn.net/nrsc272420199/article/details/85227095
https://www.liaoxuefeng.com/wiki/896043488029600/900394246995648

## 不小心 drop 了 stash 代码，如何找回？

`git stash drop a` 时，会打印 drop 的 hash 可以通过 `git stash apply hash` 找回 drop 的代码。

## 工作流开发步骤

1. 新建并切换到本dev新分支
```shell
git checkout -b dev
```
2. 修改代码
3. 提交代码
```shell
git commit -m xxxx
```
4. 切换到主分支master
```shell
git checkout master
```
5. 合并
```shell
git merge dev
```
6. 推送本地master到远程master
```shell
git push origin master:master
```

## gitlab 或 github 下 fork 后如何同步源的新更新内容？
https://www.zhihu.com/question/28676261/answer/44606041

## 合并某个 commit 到 master

```undefined
# 合并 62ecee 到 master 分支
git checkout master
git cherry-pick 62ecee
```

## 批量删除文件尾 ^M 字符

^M 字符通常来源于 \r，当文件在 windows 编辑后，换行会带有 \r\n，而当该文件拷贝至 linux 后，linux 不识别 \r，导致出现 ^M 字符，解决方法只需 dos2unix 转化即可：dos2unix ./file

```
# 当 git diff 时发现 ^M 时，可批量转换
git status | grep modified | awk '{print $2}' | xargs dos2unix
```

## 刷新 git 缓存

cached 其实就是暂存区，然后一个是工作的目录，你的工作目录的东西做出修改时，会和缓存区进行对比，因此你 git status 时，会显示出来这个差异，因此为了使 .gitignore中 的内容生效，那么就删除掉暂存区，然后将所有本地文件追踪一下，就得到最新的暂存区文件。

```
git rm -r --cached .
git add .
git restore --staged .	# 撤回暂存
```

