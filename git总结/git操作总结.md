## 基础命令：

```
git checkout dev 切换到dev分支
git branch -r 查看远程分支
git branch -a 查看所有分支，包括远程和本地
git branch -d dev 删除分支dev
git push origin --delete Chapater6   可以删除远程分支Chapater6
git push -u origin dev:dev 将本地dev分支推送到远程仓库中
git pull master 将远程仓库master分支的代码下载到本地
git commit -m "注释" 提交代码到本地仓库
git branch --set-upstream-to=origin/master master
git log --oneline /*打印日志*/ --graph /*打印分支日志*/
git reflog 提交点ID/版本号tag 查询所有提交点
git tag -a v1.0(版本号) cdeef67（每个文件的ID）
git  reset --hard 提交点/版本号 （回退到某提交点）
git branch -d gateway 删除本地gateway分支
git push origin --delete gateway 删除远端gateway分支
```
---
## 实际操作：
### 一、将git分支合并到master

（1）在dev环境下开发：

	git  add .
	git  commit -m ‘dev'
	git push -u origin dev

（2）切换到master分支：

	git checkout master
	git pull origin master
	git merge dev

### 二、创建本地分支，并推送到远端
  ```
  git branch -a
  git checkout -b dev  //创建本地分支dev
  git push -u origin dev:dev 将本地dev分支推送到远程仓库中
  ```
### 三、如果出现git====please tell me who you are
	需要运行
	
	git config --global user.email "邮箱"
	git config --global user.name "名字"

---
### 四、git pull出现代码冲突：
```
直接git pull 拉取代码，出现冲突
使用git status -uno 查看当前状态

其中出现 unmerged paths：
both modified：XXX.txt (这个.txt就是需要手动合并的)

cat XXX.txt
<<<<<<<<<<    代码   =========== 之间是自己的内容
==========    代码   >>>>>>>>>>> 之间是其他人提交的内容
修改代码，然后
git add XXX.txt
git commit -m 'fix conflict commit'
git push origin 分支
查看日志完成冲突解决
git log --graph
```

---
### 五、避免代码覆盖

	1、提交前先把本地修改存入栈 git stash save
	
	2、从服务端更新代码 git pull

  3、弹出代码git pop

  4、若有冲突解决冲突

  5、添加代码 git add.

  6、提交代码 git commit -m 'msg'

  7、推送远程 git <!--push-->
