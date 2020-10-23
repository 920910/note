# Git

- 分布式版本控制系统
- ![](https://img.mukewang.com/59c31e4400013bc911720340.png)

```txt
Workspace：工作区
Index / Stage：暂存区
Repository：仓库区（或本地仓库）
Remote：远程仓库
```



## 安装git命令行工具

- [下载](https://npm.taobao.org/mirrors/git-for-windows)

- 安装运行

- 配置用户名邮箱

  ```bash
  git config --global user.name tortoise
  git config --global user.email 12345678911@qq.com
  ```

- 解决中文乱码

  ```bash
  git config --global core.quotepath false
  ```

## 可视化工具

- [Sourcetree](https://www.sourcetreeapp.com/)

## Git命令使用

- 初始化本地仓库

  ```bash
  git init
  ```

- 添加文件到暂存区

  ```bash
  git add test.txt
  ```

- 提交文件到本地仓库

  ```bash
  git commit -m "提交说明"
  ```

- 查看当前状态

  ```bash
  git status
  ```

- 查看历史记录

  ```bash
  git log
  # 显示简信息
  git log --pretty=oneline
  ```

- 查看版本

  ```bash
  git reflog
  ```

- 回退版本

  ```bash
  # 回退到上一版本
  git reset --hard HEAD^
  # 回退到上上一个版本
  git reset --hard HEAD^^
  # 回退n个版本
  git reset --hard HEAD~n
  # 回退到指定版本
  git reset --hard 版本号
  ```

- 撤销,丢弃工作区修改的内容

  ```bash
  # 撤销修改,撤销删除
  git restore 文件
  ```

##  远程仓库

- 在用户目录的.ssh目录下生成密钥

  ```bash
  ssh-keygen -t rsa -C email
  ```

- 将生成的id_rsa.pub文件的内容放到码云或github等服务器上

- 新建仓库

  ```bash
  创建 git 仓库:
  
  mkdir git-test
  cd git-test
  git init
  touch README.md
  git add README.md
  git commit -m "first commit"
  git remote add origin https://gitee.com/coderwangbin/git-test.git
  git push -u origin master
  
  已有仓库?
  
  cd existing_git_repo
  git remote add origin https://gitee.com/coderwangbin/git-test.git
  git push -u origin master
  
  ```

- 分支创建,查看

  ```bash
  # 创建并切换dev分支
  # git branch dev 创建分支dev
  # git checkout dev切换到分支dev
  git checkout -b dev
  # 查看分支
  git branch
  ```

- 合并分支

  ```bash
  # 将dev分支合并到当前分支上
  git merge dev
  ```

- 删除分支

  ```bash
  # 删除dev分支
  git branch –d dev
  ```

- 分支关联

  ```bash
  # 本地新建分支远程没有,使用命令在远程新建
  git push --set-upstream origin branch_name
  # 如果远程新建了一个分支，本地没有该分支
  git checkout --track origin/branch_name 
  ```



## 分支说明

- Master: 主分支；主要是稳定的版本分支，正式发布的版本都从Master拉。
- Develop: 开发分支；更新和变动最频繁的分支，正常情况下开发都是在Develop分支上进行的。
- Release：预发行分支；一般来说，代表一个版本的功能全部开发完成后递交测试，测试出Bug后进行修复的分支。
- Features: 功能分支； 其实Features不是一个分支，而是一个分支文件夹。里面包含了每个程序员开发的功能点。Feature开发完成后合入Develop分支。
- HotFix: 最希望不会被创建的分支；这个分支的存在是在已经正式上线的版本中，发现了重大Bug进行修复的分支。