常用命令：

- 初始化创建并连接版本库

```bash
# 创建目录
$ mkdir dir

# 在 github 上创建项目库

# 将本地库与远程库进行关联
$ git remote add origin https://github.com/Darenfy/[file].git

# 本地库的所有内容推送到远程库并关联分支
$ git push -u origin master

```

- 操作类
```bash
# 添加到暂存区
$ git add [file]

# 提交到版本库
$ git commit -m "message"

# 回退到上一个版本/特定版本
$ git reset --hard HEAD^/[version]

# 删除文件
$ git rm [file]

# 推送修改至 github
$ git push origin master

# 删除远程库
$ git remote rm origin

```

- 功能类
```bash
# 查看版本状态
$ git status

# 查看修改内容
$ git diff [file]

# 查看项目日志
$ git log [--pretty=oneline] 

# 查看命令记录
$ git reflog
```

---
#### 参考链接
https://www.liaoxuefeng.com/wiki/896043488029600

https://training.github.com/downloads/zh_CN/github-git-cheat-sheet/

