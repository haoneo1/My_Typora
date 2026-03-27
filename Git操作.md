# Git 常用命令笔记

### 配置Git：

```
git config --global user.name "Hao Ji"
git config --global user.email "hao1neo@gmail.com"
```

## 第一阶段：单机基本功（本地仓库）

### 0. Git 三层模型

**HEAD**：上一次提交（历史快照）

- **Staging / Index（暂存区）**：这次准备提交的快照
- **Working tree（工作区）**：你正在编辑的真实文件

对应关系：

- `git diff`：工作区 ↔ 暂存区（未暂存的改动）
- `git diff --staged`：暂存区 ↔ HEAD（将要提交的改动）
- `git commit`：暂存区 → 变成新的 HEAD

------

### 1) `git init`

**作用：** 在当前目录创建/初始化 Git 仓库（生成 `.git/`）。
 **什么时候用：** 项目第一次纳入 Git 管理。
 **注意：** 不会自动跟踪任何文件，只是“开仓”。

------

### 2) `git status -sb`

**作用：** 查看仓库状态（短格式 + 显示分支）。

- `-s`：短格式
- `-b`：显示分支信息

**什么时候用：** 提交前必看：改了啥、哪些已暂存、哪些没跟踪。

**输出标记（左列/右列）：**

- `?? file`：未跟踪（untracked）
- ` M file`：已修改但**未暂存**
- `M  file`：已**暂存**等待提交
- `MM file`：既有暂存改动，又有未暂存改动

------

### 3) `git add <file>` / `git add -p`

`git add <file>`

**作用：** 把某个文件的改动加入**暂存区**。
 **什么时候用：** 这个文件的改动都要进下一次 commit。

`git add -p`

**作用：** 交互式按块（hunk）暂存，只提交部分修改。
 **什么时候用：** 一个文件改了多件事，想拆成多个 commit；或只想提交部分行。

常用选项：

- `y` 暂存该块
- `n` 不暂存该块
- `s` 切成更小块
- `e` 手动编辑暂存内容
- `q` 退出

------

### 4) `git commit -m "msg"`

**作用：** 把暂存区内容提交成一个新的 commit。
 **关键点：** commit 只包含**已暂存**的内容。
 **什么时候用：** `git add` / `git add -p` 选好本次提交内容之后。

------

### 5) `git log --oneline --graph --decorate --all`

**作用：** 以紧凑 + 分支图的形式查看提交历史与分支指向。

- `--oneline`：一条 commit 一行（短 hash + 标题）
- `--graph`：画分支/合并线
- `--decorate`：显示 HEAD / 分支 / tag 指向
- `--all`：显示所有分支历史

**什么时候用：** 找提交点、看分叉合并、找 hash 做回滚/对比。

------

### 6) `git diff`

**作用：** 看**未暂存**的改动（工作区相对暂存区）。
 一句话：**看“还没 add 的改动”。**

------

### 7) `git diff --staged`

**作用：** 看**已暂存**的改动（暂存区相对 HEAD）。
 一句话：**看“下一次 commit 会提交什么”。**

## 第二阶段：连接远程仓库

**Git 远端协作最核心的理解**

Git 里要盯住 **两条线**：

- `<branch>`：你的**本地分支**
- `origin/<branch>`：你本地保存的**远端分支快照**

比如你在 `main` 分支上：

- `main` = 你本地代码的位置
- `origin/main` = 你上次 `fetch` 后记录的远端位置

注意：**`origin/main` 不是实时远端**，它只有在你执行 `git fetch` 后才会更新。

#### 1. `git fetch origin --prune`

作用：

- 更新远端信息到本地
- 更新 `origin/<branch>`
- **不改你的本地代码**

`--prune` 的作用：

- 清理本地那些已经被远端删除的分支记录

一句话理解：

> **先看看远端变成什么样了，但先不动我自己的代码。**

#### 2. `git pull --rebase`

作用：

- 先 `fetch`
- 再把你本地提交接到远端最新提交后面

一句话理解：

> **先拿远端最新内容，再把我的改动放到最新版本后面。**

为什么推荐 `--rebase`：

- 历史更直
- 更干净
- 少 merge 记录

可以设置成默认：

```
git config --global pull.rebase true
```

------

#### 3. `git push -u origin <branch>`

作用：

- 把本地分支推到远端
- `-u` 建立本地和远端的跟踪关系

例如：

```
git push -u origin main
```

以后通常就可以直接写：

```
git push
git pull
```

------

**最推荐的协作顺序**

```
git fetch origin --prune
git log --oneline HEAD..origin/<branch>
git pull --rebase
git push
```

含义是：

1. 先更新远端信息
2. 看远端比我多了什么
3. 再拉取并整理本地历史
4. 最后把自己的提交推上去

# 

Git 协作时最重要的是分清：

- **本地分支** `<branch>`
- **远端快照** `origin/<branch>`

`fetch` 是**更新远端信息但不改本地代码**，
 `pull --rebase` 是**拉远端更新并把本地提交接到后面**，
 `push` 是**把本地提交传到远端**。









# git常见命令

## 提交新文件

**一、环境准备（只需一次）**

1️⃣ 安装 GitHub CLI

```
sudo apt update
sudo apt install -y gh
```

2️⃣ 终端登录 GitHub（SSH）

```
gh auth login
```

推荐选择：

- GitHub.com
- SSH
- 使用浏览器完成一次性授权（这是 gh 的标准流程）

验证：

```
gh auth status
```

------

**二、在本地初始化 Git 仓库**

进入你的项目目录：

```
cd ~/Documents/SBIR/LVM_test_text_head
```

初始化 git：

```bash
git init
git branch -M main

#git branch：分支操作命令

#-M：--move --force，即 重命名分支，如果目标分支名 main 已经存在，也会强制覆盖/替换（比 -m 更“硬”）

#main：新的分支名
```

创建或准备要提交的文件：

```
touch LVM_text_head
```

提交本地内容：

```
git add .
git commit -m "Initial commit"
```

------

三、完全通过终端创建 GitHub Repo（核心步骤）

✅ 使用 gh 创建远端仓库

```
gh repo create LVM_text_head --private --source=.
```

说明：

- `LVM_text_head`：GitHub 仓库名
- `--private`：私有仓库（可改成 `--public`）
- `--source=.`：使用当前目录作为仓库内容
- **不加 `--push` 也没关系**（你刚才的实践证明了这一点）

⚠️ 如果提示：

```
Unable to add remote "origin"
```

**不用慌，这是因为本地已经存在 origin，不影响仓库创建。**

------

**四、手动绑定 SSH remote（推荐显式做**）

为避免 HTTPS / SSH 混乱，**统一使用 SSH**：

```
git remote remove origin 2>/dev/null || true
git remote add origin git@github.com:haoneo1/LVM_text_head.git
```

检查：

```
git remote -v
```

期望看到：

```
origin  git@github.com:haoneo1/LVM_text_head.git (fetch)
origin  git@github.com:haoneo1/LVM_text_head.git (push)
```

------

**五、推送到 GitHub（你已成功的步骤）**

```
git push -u origin main
```

成功标志：

```
[new branch]      main -> main
Branch 'main' set up to track remote branch 'main' from 'origin'.
```

🎉 至此：
 **Repo 创建 + 本地代码上传，全部通过终端完成**

------



## 分支错误

**1. 现象**

- GitLab 网页左上角选择了分支 `tg2.0_lite`，网页里能看到完整文件。

- 本地执行：

  ```bash
  git clone git@git.x-humanoid-cloud.com:motion-intelligence-group/mig-tg2/tk_lab.git
  ```

下载下来只有 `README`，内容与 `main` 分支一致。

**2. 原因（关键点）**

- `git clone` **默认只会检出远端的“默认分支”**（GitLab 页面标注为“默认”的分支，一般是 `main`）。
- GitLab 网页切换分支 **只影响网页显示**，不会影响你本地仓库状态。

------

**3. 正确做法 A：直接 clone 指定分支（推荐）**

```
git clone -b tg2.0_lite \
  git@git.x-humanoid-cloud.com:motion-intelligence-group/mig-tg2/tk_lab.git
```

------

**4. 正确做法 B：已 clone 的情况下切换到 `tg2.0_lite`**

进入仓库目录：

```
cd tk_lab
```

拉取远端分支信息：

```
git fetch origin
```

查看远端分支：

```
git branch -a
```

切换分支（推荐用 `switch`）：

```
git switch tg2.0_lite
```

如果本地没有该分支，从远端创建本地分支：

```
git switch -c tg2.0_lite origin/tg2.0_lite
```

------

**5. 验证是否成功（必须）**

```
git branch
```

应看到：

```
* tg2.0_lite
```

检查最近提交：

```
git log --oneline --max-count=3
```

提交记录应与 GitLab 网页 `tg2.0_lite` 分支一致。

------



## Git 提交并同步到远端

**1) `git add -A`**

- 作用：把当前目录里**所有改动**加入“暂存区（staging area）”
- 包括：**新增文件、修改文件、删除文件**
- 含义：告诉 Git「这些变化我准备提交」

**2) `git commit -m "update notes"`**

- 作用：把暂存区里的内容**打包成一次提交（commit）**，生成一条提交记录
- `-m` 后面是提交说明（建议写清楚改了什么）
- 含义：在本地仓库里保存一个“版本快照”

**3) `git push`**

- 作用：把本地的 commit **推送到远端仓库（GitHub/GitLab）**
- 远端网页只有在你 push 了新的 commit 后才会更新
- 含义：同步到服务器，让别人也能看到你的更新

总结：

```bash
git status        # 看改了什么
git add -A        # 全部加入暂存区
git commit -m "your message"
git push

```



## 不推送保存本地

想长期保留本地改动，但不想每次 stash/pop（更“永久”的本地状态）
建一个本地分支存着改动，但不推送

```bash
你可以提交到本地分支，然后不 push：

git switch -c my_local_wip
git add -A
git commit -m "local wip (do not push)"


之后切到你想去的分支：

git switch tg_pro_tklab
```


这时候改动已经安全地存在你本地分支 my_local_wip，远端完全不知道，除非你手动 git push.

如果担心误 push，可以把这个分支设置成永不 push，或者干脆不设置 upstream。

## 现有仓库开一个新分支

```
git switch -c FG_ZS_SBIR
git push -u origin FG_ZS_SBIR
```

它的效果是：

1. 在**当前仓库**里新建一个本地分支 `FG_ZS_SBIR`
2. 把这个分支推到**当前远端 origin**
3. 建立本地分支和远端分支的跟踪关系
