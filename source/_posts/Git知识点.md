---
title: git笔记
---

## Git

### 1.版本回退

`git reset`

```js
git reset --hard HEAD^		回到上一个版本
git reset --hard ca1020   回到指定版本
```

HEAD^  表示上一个版本
HEAD^^   上上一个版本
HEAD~3   往上三个版本

ca1020表示某个commit的id，怎么找到想恢复版本的commit id，使用`git reflog`：

```js
znh@99-12-199-38 webpack-vue-demo % git reflog           
4a48ea1 (HEAD -> master) HEAD@{0}: reset: moving to 4a48
d2770f4 HEAD@{1}: reset: moving to d277
4a48ea1 (HEAD -> master) HEAD@{2}: commit: 配置字体文件打包2
d2770f4 HEAD@{3}: commit: 配置字体文件打包
e852570 HEAD@{4}: commit: 加载图片用自带的AssetsModules
aedbeea HEAD@{5}: commit: 手写plugin
```

### 2.撤销修改

git checkout -- filenname 

- 撤销还没添加到暂存区的修改，让filenname 文件回到最近一次git commit或git add时的状态

- 假设修改已经添加到暂存区了，怎么撤销？git reset：

```js
znh@99-12-199-38 webpack-vue-demo % git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        modified:   src/pages/Home/index.vue
【此时修改在暂存区】
znh@99-12-199-38 webpack-vue-demo % git reset HEAD src/pages/Home/index.vue 
Unstaged changes after reset:
M       src/pages/Home/index.vue
znh@99-12-199-38 webpack-vue-demo % git status
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

        modified:   src/pages/Home/index.vue
【执行git reset 后，此时修改回到工作区】
no changes added to commit (use "git add" and/or "git commit -a")
znh@99-12-199-38 webpack-vue-demo % git checkout src/pages/Home/index.vue
【此时执行git checkout -- filename即可撤销修改】
```

- 撤销提交到版本库的修改，使用git reset，参考版本回退

删除文件，在文件管理器删除了某个文件：

确实要删除：

```shell
git rm filename 
git commit -m "删除文件filename"
```

删错了，可以从版本库中恢复，前提是该文件之前已经提交到版本库了：

```shell
git checkout -- filename
```



### 3.git代理设置

```shell
#设置
git config --global https.proxy http://127.0.0.1:1080
git config --global https.proxy https://127.0.0.1:1080
#取消代理
git config --global --unset http.proxy
git config --global --unset https.proxy
```

