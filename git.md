## 一、本地项目传到远程仓库

1. 进入本地项目文件夹

2. **`git init`**，生成一个隐藏的 .git 文件夹

3. **`git remote add origin https://gitee.com/用户名/仓库名.git`**

4. **`git add .`**, **`git commit -m "提交说明"`**

5. 同步远程仓库与本地仓库**`git pull origin master --allow-unrelated-histories`**

   若直接执行 `git pull origin master` 报错「fatal: refusing to merge unrelated histories」，需添加 `--allow-unrelated-histories` 参数，解决本地与远程仓库无相关性的问题

6. **`git push origin master`**

   **`git push -u origin master`**: 把本地的`master`分支和远程的`master`分支关联起来，简化以后的推送或者拉取的命令

7. 解除本地和远程的绑定**`git remote rm origin`**

   查看远程库信息: **`git remote -v`**