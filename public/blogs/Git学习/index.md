![](/blogs/Git学习/5756a38574e91279.webp)

本文整理了 Git 的基础命令、Fork 仓库同步更新流程，以及日常开发中的高频命令。

---

## 一、Git 基础命令

### 1.1 初始化与配置

```bash
# 初始化仓库
git init
# 配置用户信息（首次使用必须设置）
git config --global user.name "你的名字"
git config --global user.email "你的邮箱"
# 查看配置
git config --list

```
### 1.2 基本工作流

```bash
# 查看状态
git status
# 添加文件到暂存区
git add <文件名>        # 添加单个文件
git add .              # 添加所有修改
# 提交到本地仓库
git commit -m "提交信息"
# 查看提交历史
git log
git log --oneline      # 简洁版

```
### 1.3 远程仓库操作

```bash
# 克隆远程仓库
git clone <仓库地址>
# 添加远程仓库
git remote add origin <仓库地址>
# 查看远程仓库
git remote -v
# 推送到远程
git push origin <分支名>
# 拉取远程更新
git pull origin <分支名>

```

## 二、 常用命令速查表

### 2.1 分支操作

```bash
# 查看分支
git branch           # 本地分支
git branch -a        # 所有分支（含远程）
# 创建分支
git branch <分支名>
# 切换分支
git checkout <分支名>
git switch <分支名>   # 新语法
# 创建并切换分支
git checkout -b <分支名>
# 删除分支
git branch -d <分支名>        # 安全删除
git branch -D <分支名>        # 强制删除
# 合并分支
git merge <分支名>

```

### 2.2 撤销与回退

```bash
# 撤销工作区修改（未 add）
git checkout -- <文件名>
git restore <文件名>   # 新语法
# 撤销暂存区（已 add，未 commit）
git reset HEAD <文件名>
git restore --staged <文件名>  # 新语法
# 回退提交
git reset --soft HEAD^    # 回退commit，保留修改
git reset --hard HEAD^    # 回退commit，丢弃修改（危险！）
# 回退到指定版本
git reset --hard <commit-id>

```

### 2.3 暂存工作区

```bash
# 暂存当前修改（切分支时很有用）
git stash
# 查看暂存列表
git stash list
# 恢复暂存
git stash pop          # 恢复并删除暂存
git stash apply        # 恢复但保留暂存

```

### 2.4 查看与对比

```bash
# 查看修改内容
git diff                      # 工作区 vs 暂存区
git diff --staged             # 暂存区 vs 最新提交

# 查看某次提交的内容
git show <commit-id>

# 查看文件历史
git log -p <文件名>

```

### 2.5 标签操作
```bash
# 创建标签
git tag v1.0.0
git tag -a v1.0.0 -m "版本说明"

# 推送标签
git push origin v1.0.0
git push origin --tags    # 推送所有标签

```

## 三、Fork 仓库后同步更新

> 当你 Fork 了别人的仓库，原仓库有更新时，如何同步？

### 3.1 初始设置

```bash
# 1. 添加上游仓库（原仓库）
git remote add upstream <原仓库地址>

# 2. 验证是否添加成功
git remote -v
# 应该看到：
# origin    https://github.com/你的用户名/仓库名.git (fetch)
# origin    https://github.com/你的用户名/仓库名.git (push)
# upstream  https://github.com/原作者/仓库名.git (fetch)
# upstream  https://github.com/原作者/仓库名.git (push)

```

### 3.2 同步更新流程

```bash
# 1. 获取上游仓库的更新
git fetch upstream
# 2. 切换到主分支
git checkout main  # 或 master
# 3. 合并上游的更新
git merge upstream/main #大概率会出现冲突，解决措施见第四部分
# 4. 推送到自己的远程仓库
git push origin main

```

### 3.3一键同步脚本

```bash
# 可以创建一个 alias 简化操作
git fetch upstream && git merge upstream/main && git push origin main

```

## 四、常见问题

### 4.1 提交信息写错了

```bash
git commit --amend -m "新的提交信息"

```

### 4.2 忘记切分支就开始写代码了

```bash
# 先暂存当前修改
git stash
# 切换到正确的分支
git checkout <正确分支>
# 恢复修改
git stash pop

```

### 4.3 合并冲突怎么办

>为什么会出现冲突？
> >当两个分支**修改了同一个文件的同一行**时，Git 无法自动判断保留哪个版本，就会产生冲突。常见场景：
> > >- 同步 Fork 仓库时，你和原仓库修改了同一文件
> > >- 合并分支时，多人修改了同一处代码

```bash
# 1. 获取上游更新
git fetch upstream
# 2. 尝试合并
git merge upstream/main
# 输出：CONFLICT (content): Merge conflict in README.md
# 3. 查看冲突文件
git status
# 4. 打开文件手动解决（或用 VS Code）
code README.md
# 5. 解决后，添加文件
git add README.md
# 6. 完成合并
git commit -m "Merge upstream/main, resolve README conflict"
# 7. 推送
git push origin main

```

**解决冲突流程**

```text
merge 冲突
    │
    ▼
git status 查看冲突文件
    │
    ▼
打开文件，看 <<<<<<< ======= >>>>>>>
    │
    ▼
决定保留哪部分 → 删除冲突标记 → 保存
    │
    ▼
git add <文件>
    │
    ▼
git commit
    │
    ▼
git push
    │
    ▼
完成 ✅

```

## 总结

| 场景 | 命令 |
|------|------|
| 日常提交 | `add` → `commit` → `push` |
| 同步更新 | `pull` 或 `fetch` + `merge` |
| 分支开发 | `checkout -b` → 开发 → `merge` |
| Fork 同步 | `fetch upstream` → `merge` → `push` |
| 紧急切分支 | `stash` → `checkout` → `stash pop` |