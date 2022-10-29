1. #### Git的安装全局设置

   配置

   ```bash
   git config --global user.name "名字"
   git config --global user.email "邮箱地址"
   ```

2. #### 初始化本地Git仓库

   ```bash
   git init
   ```

3. #### 本地仓库常用指令

   ```bash
   git status 查看状态
   git add 把文件添加到Git工作区
   git reset 把暂存区的文件取消暂存或切换到指定版本（版本回滚）(需要结合git log使用)
   git commit 把暂存区文件提交到版本库
   git log 查看日志(结合git reset使用)
   ```

   1.1	```git add```例子

   ```bash
   $ git add * # 添加当前目录下所有文件到git工作区
   $ git add hello.txt # 添加hello.txt到工作区
   ```

   1.2	```git status```

   ```bash
   $ git status
   On branch main
   Your branch is up to date with 'origin/main'.
   
   Changes to be committed:
     (use "git restore --staged <file>..." to unstage)
           modified:   .gitignore #modified 表示已修改的状态
           new file:   pom.xml
           ······
   ```

   1.3	```git commit```

   ``` bash
   $ git commit -m '提示信息' hello.txt # 把hello.txt提交到版本库(保存到内存中)
   $ git commit -m '提示信息' * # 把所有文件提交
   ```

   1.4	```git log```

   ```bash
   $ git log
   commit 5584ab58f732233bca3e93138bfe145333b5dff8 (HEAD -> main, origin/main, origin/HEAD, dev)
   Author: githubUser <89060516+githubUser@users.noreply.github.com>
   Date:   Sat Oct 22 13:36:55 2022 +0800
   
   	Update README.md
   commit:······
   ```

   1.5	```git reset```

   ```bash
   $ git reset hello.txt # 取消hello.txt的暂存
   $ git reset --hard 5584ab58f732233bca3e93138bfe145333b5dff8 # 回滚到git log时获取到的commit指定的版本
   ```

4. #### 远程仓库指令

   ```bash
   git remote # 查看远程仓库
   git remote add <shotName> <url> # 添加远程仓库
   git clone # 从远程仓库克隆(下载)
   git pull <short-name> <branch-name> # 从远程仓库拉取(更新)
   git push <remote-name> <branch-name># 推送到远程仓库(上传),remote-name一般是git remote后出现的那个别名
   ```

   1.1	```git remote```

   ```bash
   $ git remote # 没有出现任何信息表示只是本地仓库，没有关联远程仓库
   # 关联远程仓库后会出现origin（远程仓库简称,一般不修改）
   $ git remote
   origin
   # 添加-v会出现关联的远程仓库地址的信息
   $ git remote -v
   origin  https://github.com/githubUser/test.git (fetch)
   origin  https://github.com/githubUser/test.git (push)
   ```

   1.2	```git remote add```

   ```bash
   $ git remote add origin https://github.com/githubUser/test2.git # 添加远程仓库test2（前提:远程仓库必须存在）
   ```

   1.3	```git clone```

   ```bash
   $ git clone https://github.com/githubUser/test.git # 克隆test到本地,挂代理时https的url不能克隆
   ```

   1.4	```git pull```

   ```bash
# 在git clone之后远程仓库有更新时就能更新
   $ git pull origin dev
   ```
   
   当本地仓库有文件时更新远程仓库会出错，此时在克隆命令后添加```--allow-unrelated-histories```

   1.5	```git push```

   ```bash
$ git push origin master # 把git commit后的文件推送到origin(远程仓库简称)为master的分支
   ```

   第一次提交时会要求输入```github```的用户名和密码

5. #### 分支指令

   ```bash
   git branch # 查看分支
   git branch [name] # 添加新分支
   git checkout [name] # 切换分支
   git push [shortName] [name] # 推送到远程仓库的分支
   git merge [name] # 合并分支
   ```

   1.1	```git branch```

   ```bash
   $ git branch # 列出所有本地分支
   * main  # 当前使用的本地分支
   $ git branch -r # 列出所有远程分支
     origin/HEAD -> origin/main # 指针指向使用的远程仓库分支
     origin/dev
     origin/main
   $ git branch -a # 列出所有本地分支和远程分支
   $ git branch -a
   * main # 当前使用的本地分支
     remotes/origin/HEAD -> origin/main
     remotes/origin/dev
     remotes/origin/main
   ```
   
   1.2	```git branch```
   
   ```bash
   $ git branch dev
   ```
   
   1.3	```git checkout```

   ```bash
   $ git checkout dev
   M       .gitignore
   A       pom.xml
   ······
   MINGW64 /e/Java Projects/test (dev) # 已经切换到dev分支
   ```
   
   1.4	```git push```
   
   ```bash
   $ git push origin dev # 推送到dev分支
   ```
   
   1.5	```git merge```
   
   ```bash
   MINGW64 /e/Java Projects/test (master) # 当前在master分支，把dev分支合并过来
   $ git merge -m "描述信息" dev
   ```
   
   