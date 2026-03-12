## git的创建

### 创建git仓库

[github](https://github.com/)中新建repository
输入仓库名即可创建git

需要.gitignore文件

```c++
Binaries
DerivedDataCache
Intermediate
Saved
Build

.vscode
.vs
*.VC.db
*.opensdf
*.opendb
*.sdf
*.sln
*.suo
*.xcodeproj
*.xcworkspace
```

### 建立连接

在git中新建git后需要在对于文件中打开GitBash

```c++
git init
git status
git add .
git status
git commit -m "这是一个提交的名字"
git branch -M main
复制你网站上的链接
如:
git remote add origin git@github.com:orangedanger/Aura.git
git push -u origin main
```

### 后期修改

如果在修改后进行提交

```c++
git add .
git status
git commit -m "这是一个提交的名字"
git push -u origin main
```

### github SSH pull/push 连接超时/连接失败解决方案

[]: https://blog.csdn.net/qq_39522310/article/details/135923081

