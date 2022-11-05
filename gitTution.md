# git基本使用
## git上传本地项目
1.　远程操作：在远程建立一个仓库

2.　本地操作：

> 1. 进入项目文件夹，执行`git init`命令;
> 2. 执行`git add .`命令，添加文件；
> 3. 执行`git commit -m "explaination"`命令，注明此次添加的注释信息；
> 4. 执行`git remote add origin(名字可以起其他的) 远程仓库的ssh地址`命令，与远程仓库建立链接；
> 5. 执行`git push origin branch_name`命令，将项目上传到远程仓库。

## git常用命令
1. 查看分支：`git branch`
2. 切换分支：`git checkout branch_name`
3. 新建分支并切换：`git checkout -b new_branch_name`
