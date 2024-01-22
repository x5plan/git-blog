# 每日知识分享
博客开发完成前的临时解决方案


## 文件命名格式
请自行维护以下文件：
- 文章 `<alias>/<year>/<month>/<date>/<title>.md`  
  例：`Antares/2024/01/22/测试内容.md`
- 插图 `<alias>/<year>/<month>/<date>/images/<name>`  
  例：`Antares/2024/01/22/images/图例.jpg`
- 目录 `<alias>/index.md`  
  例：`Antares/index.md`  

## 目录书写说明
使用 Markdown 如下编写：
```md
# 目录
## 2022
- [01/20 文章1](2022/01/20/文章1.md)
- [01/20 文章2](2022/01/20/文章2.md)
- [01/22 文章3](2022/01/22/文章3.md)

## 2024
- [01/20 文章4](2024/01/20/文章4.md)
- [01/20 文章5](2024/01/20/文章5.md)
- [01/22 文章6](2024/01/22/文章6.md)
```

## 提交内容

第一次提交请完整走完所有流程，之后请直接从 Step 2 做。

### Step 1 clone项目

安装 [Git](https://git-scm.com/)，自己找教程安装。

Clone项目，可选用 HTTPS 或 SSH （推荐）

```sh
git clone https://github.com/x5plan/git-blog.git x5plan-git-blog  # 使用 https
git clone git@github.com:x5plan/git-blog.git x5plan-git-blog      # 使用 ssh

# 注意：字符 # 后半部分是注释，仅仅用于说明，没有必要去粘贴到终端中。
```

如果选用SSH，请自行生成 SSH key pair。

### Step 2 拉取最新的 main 并创建自己的分支

进入 git repo 文件夹，切换到main分支并拉取最新的远程分支

```sh
cd x5plan-git-blog
git checkout main
git pull
```

创建并进入用户自己的分支

```sh
git checkout -b user/<alias>/<branch name>
```

例： `git checkout -b user/antares/first-commit`

注意，全**小写**，包括你的用户名，全部**小写**。

### Step 3 写作并提交代码

以创建 `Antares/2024/01/22/测试内容.md` 为例

首先创建对应的文件，并书写完成内容。可自由选择编辑器，推荐 VS Code 或 Typora。

编辑完成后，Stage 你的文件并commit。

```sh
git add Antares/2024/01/22/测试内容.md
git commit -m "update 测试内容.md" 
# -m 后为提交注释，随意填写但必须得有，几个简单的词描述一下做了啥即可
```

### Step 4 推送并创建 PR

将本地分支推送至远程：

```sh
git push
```

打开 [x5plan/git-blog](https://github.com/x5plan/git-blog) ，选择 Pull requests，点击按钮 New pull request.

