### git常用命令有哪些？

- ###### 基础命令

  初始化现有的git本地仓库

  ````bash
  git flow init
  ````

  创建feature分支,并切换到develop分支上

  ````bash
  git branch fea
  git checkout dev
  ````

  查看当前分支

  ````bash
  git branch 
  ````

  删除分支

  ```bash
  git branch -d fea
  ```

  合并分支

  ```bash
  git merge --no--ff fea 
  ```

  --no--ff 的作用是正常合并后，在master分支上生成一个新的节点

  暂存文件

  ````bash
  git add 1.txt
  ````

  提交文件

  ````bash
  git commit 2.txt
  ````

  