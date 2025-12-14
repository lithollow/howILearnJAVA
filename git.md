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



## 二、稀疏检出 (Git Sparse Checkout)

只需要使用远程仓库的某些目录，可以只检出仓库的特定部分，避免拉取不需要的文件

1. 克隆仓库但不检出文件

​	`git clone --no-checkout <远程仓库URL> <本地目录>`

​	`cd <本地目录>`

2. 启用稀疏检出

​	`git config core.sparseCheckout true`

3. 设置要检出的文件夹（例如只检出src目录）

​	`echo "src/" >> .git/info/sparse-checkout`

4. 检出指定分支

​	`git checkout <分支名>`

5. 如果之后需要添加更多文件夹

​	`echo "docs/" >> .git/info/sparse-checkout`

​	`git checkout`



临时禁用稀疏检出：

```
# 暂时获取所有文件
git config core.sparseCheckout false
git checkout main
# 操作完成后重新启用
git config core.sparseCheckout true
git checkout main
```

查看稀疏检出设置：

```
# 检查是否启用了稀疏检出
git config core.sparseCheckout
# 查看当前稀疏检出规则
cat .git/info/sparse-checkout
```