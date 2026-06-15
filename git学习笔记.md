# Git 学习笔记

> 汽车软件应用工程师 Git 学习指南
> 创建日期：2026-06-06
> 为便于演示 `diff`、冲突和 `blame`，示例文件统一使用文本文件 `model_v1.txt`。

---

## 第一阶段：核心概念

### 1.1 三区概念

Git 在本地有三个重要的"区域"，理解它们是学习 Git 的基础；远程仓库用于协作，不属于本地三区：

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   工作区     │───▶│   暂存区     │───▶│   本地仓库   │
│ (Working)   │    │  (Staging)  │    │  (Repository)│
└─────────────┘    └─────────────┘    └─────────────┘
      │                                      │
      │           ┌─────────────┐            │
      └──────────▶│   远程仓库   │◀───────────┘
                  │  (Remote)   │
                  └─────────────┘
```

- **工作区（Working Directory）**：你电脑上能看到的文件目录，就是你编辑文件的地方
- **暂存区（Staging Area）**：一个临时区域，存放你"准备提交"的修改
- **本地仓库（Local Repository）**：Git 存储提交历史的地方，在你电脑上
- **远程仓库（Remote Repository）**：服务器上的仓库（如 GitLab / GitHub），用于团队共享，不属于本地三区

### 1.2 文件的四种状态

| 状态 | 含义 | 对应命令 |
|------|------|----------|
| **Untracked** | 新文件，Git 还没开始跟踪 | 刚创建的 .txt 文件 |
| **Modified** | 已跟踪，但有修改 | 修改了模型但没提交 |
| **Staged** | 已放入暂存区，等待提交 | `git add` 之后 |
| **Committed** | 已提交到本地仓库 | `git commit` 之后 |

**记忆口诀**：新建→未跟踪，修改→已修改，add→暂存，commit→提交

---

## 第二阶段：基础命令

### 2.0 初始配置：`git config`

**三个作用域（优先级：local > global > system）：**

| 作用域 | 命令 | 生效范围 | 配置文件位置 |
|--------|------|---------|------------|
| `--system` | `git config --system` | 这台电脑上所有用户的所有仓库 | `C:\ProgramData\Git\config` |
| `--global` | `git config --global` | 当前用户的所有仓库（最常用） | `C:\Users\你的用户名\.gitconfig` |
| `--local` | `git config --local` | 仅当前仓库 | `.git/config` |

**必配项：**
- `user.name` — 提交时显示的名字
- `user.email` — 提交时显示的邮箱

输入命令：
```bash
git config --global user.name "你的名字"
git config --global user.email "你的邮箱"
```

反馈结果：（没有输出 = 成功）

**其他常用配置：**
- `core.autocrlf` — 换行符处理，Windows 设为 `true`
- `core.editor` — 默认编辑器，比如设成 VSCode：`git config --global core.editor "code --wait"`

**查看配置：**
- `git config --list` — 查看所有配置
- `git config user.name` — 查看某一项

> `--global` 配默认值，建仓前后都行；`--local` 配特殊情况，必须在仓库内执行。

> **注意：** Windows 命令行里写 commit 备注时，统一用双引号 `"备注"`。

### 2.1 创建仓库

输入命令：
```bash
git init
```

在当前文件夹创建一个 Git 仓库（生成 `.git` 隐藏文件夹）。这一步只是把目录变成 Git 仓库；文件要在 `git add` 并提交后才会被正式跟踪。

反馈结果：
```
Initialized empty Git repository in D:/xxx/practice/.git/
```

### 2.2 忽略文件：`.gitignore`

在仓库根目录创建 `.gitignore` 文件，告诉 Git 哪些文件不用跟踪。

**常用规则写法：**

| 写法 | 含义 |
|------|------|
| `*.tmp` | 忽略所有 .tmp 文件 |
| `*.log` | 忽略所有 .log 文件 |
| `build/` | 忽略 build 整个文件夹 |
| `!important.log` | 不忽略 important.log（感叹号 = 例外） |

规则文件示例（Simulink 项目常用）：
```bash
slprj/         # 忽略所有 slprj 编译缓存目录
*.slxc         # 忽略所有 Simulink 缓存文件
*.mexw64       # 忽略所有 S-Function 编译产物
```

输入命令：
```bash
# 创建 .gitignore 后，提交让团队共享
git add .gitignore
git commit -m "添加.gitignore"
```

#### 关键原则：`.gitignore` + `git rm --cached` 缺一不可

**如果文件已经被提交过，`.gitignore` 对它不生效。** 需要先从 Git 中移除：

- `.gitignore` — 管**将来**：以后 `git add` 自动跳过这些文件
- `git rm --cached` — 管**过去**：把已被跟踪的从索引中踢掉

只做其中一个是不够的：

| 只做哪个 | 后果 |
|---------|------|
| 只加 `.gitignore` | 已被跟踪的文件继续被跟踪，修改仍然显示 |
| 只用 `git rm --cached` | 下次 `git add -A` 又全部加回来了 |

#### 单个文件移除

输入命令：
```bash
git rm --cached <文件名>
git commit -m "移除xxx，交给.gitignore管理"
```

> `git rm --cached` 只从 Git 里移除跟踪，电脑上的文件不受影响。

#### 批量移除（实战中最常用）

实际项目中缓存文件往往成百上千，需要批量处理：

**按类型移除：**
```bash
# -r 参数支持通配符
git rm --cached -r '*.slxc' '*.mexw64'
```

**按目录移除（glob 失效时的替代方案）：**
```bash
# Windows Bash 下 glob 可能不生效，用 ls-files + xargs
git ls-files '**/slprj/*' | xargs git rm --cached
```

**查看还有哪些文件被跟踪（写 .gitignore 前的摸底）：**
```bash
git ls-files '*.slxc'            # 被跟踪的指定类型文件
git ls-files '**/slprj/'         # 被跟踪的指定目录
git ls-files '*.slxc' | wc -l   # 统计数量
```

#### 实战端到端流程

```bash
# 步骤1：创建 .gitignore，写入屏蔽规则（如 slprj/、*.slxc、*.mexw64）

# 步骤2：批量移除已被跟踪的缓存文件
git ls-files '*.slxc' '*.mexw64' '**/slprj/*' | xargs git rm --cached

# 步骤3：一次性暂存。此时 .gitignore 已生效，
#        缓存文件自动跳过，只留下真正的修改
git add -A

# 步骤4：提交
git commit -m "清理：添加 .gitignore，移除编译缓存文件跟踪"
```

#### `.gitignore` 的真正价值：让 `git add -A` 永远干净

没有 `.gitignore` 时，每次提交都要手动挑选文件：

```bash
git add VDC01_VDIP/VDC01_VDIP.slx    # 一个个挑着加
git add VDC02_VDSM/VDC02_VDSM.slx    # 漏了就丢，多加了又添垃圾
# ... 痛苦
```

有了 `.gitignore` 后：

```bash
git add -A        # 自动跳过缓存文件，放心全选
git commit -m "xxx"
```

> 本质：`.gitignore` 让你永远可以用 `git add -A` 放心全选，不用动脑子。

### 2.3 核心工作流（最常用）

日常使用 Git 90% 的时间都在重复这个流程：

```
改文件 → git diff → git add → git commit → git log
```

完整示例

输入命令：
```bash
# 1. 修改了文件后，先看看改了什么
git diff

# 2. 把修改放入暂存区（没有输出 = 成功）
git add model_v1.txt

# 3. 提交，写备注
git commit -m "v2: 添加刹车模块"

# 4. 查看提交历史
git log --oneline
```

反馈结果：#1

```
diff --git a/model_v1.txt b/model_v1.txt
@@ -1 +1,2 @@
 # 汽车项目模型 v2 - 添加了发动机模块
+# 添加了刹车模块
```

反馈结果：#2（没有输出 = 成功）

反馈结果：#3

```
[master 5e2c659] v2:添加刹车模块
 1 file changed, 1 insertion(+)
```

反馈结果：#4
```
5e2c659 v2:添加刹车模块
98dc266 v1:初始模型
```

### 2.4 查看状态：`git status`

输入命令：
```bash
git status
```

显示当前文件处于哪个状态。

| 输出内容                                | 含义                     |
| --------------------------------------- | ------------------------ |
| `Untracked files`                       | 新文件，Git 还没开始跟踪 |
| `Changes not staged for commit`         | 文件已修改，但还没 add   |
| `Changes to be committed`               | 文件已暂存，等待 commit  |
| `nothing to commit, working tree clean` | 干净，全部已提交         |

反馈结果：

**新文件（还没跟踪）：**

```
Untracked files:
  (use "git add <file>..." to include in what will be committed)
        model_v1.txt
```

**已修改（还没 add）：**

```
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
        modified:   model_v1.txt
```

**已暂存（还没 commit）：**

```
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        modified:   model_v1.txt
```

**全部已提交（干净）：**

```
On branch master
nothing to commit, working tree clean
```

#### `git status --short` 精简输出（两列格式）

文件多的时候，`git status` 完整输出太长，用 `-s`/`--short` 精简显示：

| 输出 | 含义 |
|------|------|
| `?? 文件名` | Untracked（未跟踪） |
| `M  文件名` | Modified — 已暂存（第一列有字母） |
| ` M 文件名` | Modified — 未暂存（第二列有字母，第一列空格） |
| `D  文件名` | Deleted — 已暂存删除 |
| ` D 文件名` | Deleted — 未暂存删除 |

> 第一列 = 暂存区状态，第二列 = 工作区状态。两列都有字母说明同一个文件在暂存区和工作区都有变化。

**快速分类统计：**
```bash
# 查看各状态文件数量
git status --short | awk '{print $1}' | sort | uniq -c | sort -rn

# 只看未被跟踪的文件
git status --short | grep "^?"

# 只看已修改未暂存的文件
git status --short | grep "^ M"
```

### 2.5 查看差异：`git diff`

| 命令 | 对比什么 | 使用场景 |
|------|---------|---------|
| `git diff` | 工作区 vs 暂存区 | 改了文件，看看改了啥 |
| `git diff --staged` | 暂存区 vs 上次提交 | add 之后，commit 之前确认一下 |
| `git diff 版本A 版本B` | 两次提交之间 | 回顾两个版本之间的所有变化 |

输出格式：
- `+` 开头 = 新增的内容
- `-` 开头 = 删除的内容

反馈结果：

**`git diff`（工作区 vs 暂存区）：**

```
diff --git a/model_v1.txt b/model_v1.txt
@@ -1 +1,2 @@
 # 汽车项目模型 v2 - 添加了发动机模块
+# 添加了刹车模块
```

**`git diff --staged`（暂存区 vs 上次提交）：**

```
diff --git a/model_v1.txt b/model_v1.txt
@@ -1 +1,2 @@
 # 汽车项目模型 v2 - 添加了发动机模块
+# 添加了刹车模块
```

> 两个命令输出格式一样，区别是对比的基准不同。

**`git diff 版本A 版本B`（两次提交之间）：**

示例（对比 v1 和 v3）：
输入命令：

```bash
git diff 98dc266 075333b
```

反馈结果：
```
diff --git a/model_v1.txt b/model_v1.txt
@@ -1 +1,3 @@
 # 汽车项目模型 v2 - 添加了发动机模块
+# 添加了刹车模块
+# 添加了转向模块
```

### 2.6 查看历史：`git log`

输入命令：
```bash
git log            # 完整日志（作者、日期、备注）
git log --oneline  # 精简日志（一行一条）
git log --graph    # 图形化显示分支合并历史
```

**常用选项：**

| 选项 | 作用 |
|------|------|
| `--oneline` | 精简显示（一行一条） |
| `--graph` | 图形化显示分支和合并 |
| `--stat` | 显示修改的文件和行数统计 |
| `--author="名字"` | 只显示某个作者的提交 |
| `--since="2024-01-01"` | 只显示指定日期之后的提交 |
| `--reverse` | 逆向显示（从最早开始） |
| `--no-merges` | 不显示合并提交 |

反馈结果：

完整日志输出：

```
commit 075333b (HEAD -> master)
Author: unknown <baoliang.lin@mxdrv.local>
Date:   Sat Jun 6 16:04:46 2026 +0800

    v3 添加转向
```

精简日志输出：
```
075333b (HEAD -> master) v3 添加转向
5e2c659 v2:添加刹车模块
98dc266 v1:初始模型
```

每次提交都有一个唯一 ID（如 `98dc266`），可以用在 `git diff` 和 `git reset` 中。

### 2.7 撤销操作

#### 2.7.1 撤销未提交的修改：`git restore`


| 命令 | 效果 | 修改是否保留 |
|------|------|------------|
| `git restore <file>` | 丢弃修改，恢复原样 | 不保留 |
| `git restore --staged <file>` | 取消暂存 | 保留 |

> 这两个命令成功时都没有输出，用 `git status` 确认效果。

**`git restore` 示例（丢弃修改）：**

操作前状态：
```
Changes not staged for commit:
        modified:   model_v1.txt
```

输入命令：
```bash
git restore model_v1.txt
git status
```

反馈结果：
```
nothing to commit, working tree clean
```

> 文件内容也恢复了，刚才的修改消失。

**`git restore --staged` 示例（撤回暂存）：**

操作前状态：
```
Changes to be committed:
        modified:   model_v1.txt
```

输入命令：
```bash
git restore --staged model_v1.txt
git status
```

反馈结果：
```
Changes not staged for commit:
        modified:   model_v1.txt
```

> 文件从"已暂存"变回"已修改"，但修改内容还在。

#### 2.7.2 撤销已提交：`git reset`

| 命令 | 作用 |
|------|------|
| `git reset --soft HEAD~1` | 撤销提交，修改保留在暂存区 |
| `git reset --mixed HEAD~1` | 撤销提交，修改保留在工作区（未暂存） |
| `git reset --hard HEAD~1` | 撤销提交，并丢弃当前未提交修改！ |

`HEAD~1` 表示"上一次提交"，`HEAD~2` 表示"上上次提交"。

| 模式 | 撤销commit | 撤销add | 丢弃修改 | 危险程度 |
|------|-----------|---------|---------|---------|
| `--soft` | ✅ | ❌ | ❌ | 最安全 |
| `--mixed` | ✅ | ✅ | ❌ | 中等 |
| `--hard` | ✅ | ✅ | ✅ | 危险！ |

**注意：** `--hard` 会永久丢弃当前未提交的工作区/暂存区修改；已经提交过但被挪走的提交，通常还能用 `reflog` 找回。

**记忆口诀**：soft 只撤提交，mixed 多撤暂存，hard 全部撤掉

**`git reset --soft` 示例：**

操作前状态（4次提交）：
```
9a0b79c v4 添加雨刷模块
075333b v3 添加转向
5e2c659 v2:添加刹车模块
98dc266 v1:初始模型
```

输入命令：
```bash
git reset --soft HEAD~1
```

反馈结果：
```
# git log --oneline：v4 消失了
075333b v3 添加转向
5e2c659 v2:添加刹车模块
98dc266 v1:初始模型

# git status：修改还在暂存区
Changes to be committed:
        modified:   model_v1.txt
```

**`git reset --mixed` 示例：**

输入命令：
```bash
git reset --mixed HEAD~1
```

反馈结果：
```
# git log --oneline：v4 消失了
# git status：修改回到工作区（未暂存）
Changes not staged for commit:
        modified:   model_v1.txt
```

**`git reset --hard` 示例：**

输入命令：
```bash
git reset --hard HEAD~1
```

反馈结果：
```
# git log --oneline：
HEAD is now at 075333b v3 添加转向

# git status：干净，修改全部丢弃
nothing to commit, working tree clean
```

> 文件内容也恢复到 v3 的状态，v4 的修改彻底消失。

### 2.8 查看仓库规模

接手一个仓库时，先快速了解规模和结构：

| 命令 | 作用 |
|------|------|
| `git rev-list --all --count` | 总共有多少个提交节点 |
| `git branch -a` | 列出所有分支（本地 + 远程） |
| `git log --all --oneline --graph` | 图形化展示所有分支的提交关系 |
| `git ls-tree -r --name-only <分支> \| wc -l` | 某个分支有多少个文件 |
| `git ls-files \| wc -l` | 当前版本有多少个被跟踪的文件 |

**实战示例 — 快速摸底一个陌生仓库：**

输入命令：
```bash
# 1. 有多少节点
git rev-list --all --count                # → 4

# 2. 有哪些分支
git branch -a                             # → master, develop, mil

# 3. 各节点的关系和内容量
git log --all --oneline --graph           # 看清谁合并了谁
git ls-tree -r --name-only master | wc -l     # → 4054 个文件
git ls-tree -r --name-only mil | wc -l        # → 20591 个文件

# 4. 各分支的提交历史
git log --oneline master
git log --oneline develop
git log --oneline mil
```

> `git ls-tree` 查看的是提交中记录的文件快照；`git ls-files` 查看的是当前索引（暂存区 + 工作区）的文件。

---

## 第三阶段：分支与合并

### 3.1 分支基础

| 命令 | 作用 |
|------|------|
| `git branch` | 查看所有分支（`*` 表示当前分支） |
| `git branch <名称>` | 创建新分支 |
| `git checkout <分支名>` | 切换到指定分支 |
| `git checkout -b <名称>` | 创建并切换分支 |
| `git branch -d <名称>` | 删除已合并的分支 |
| `git branch -D <名称>` | 强制删除未合并的分支 |

> `git switch` 是 `git checkout` 的替代命令，专门用于切换分支，更直观。

**完整示例：**

输入命令：
```bash
# 1. 创建并切换到新分支
git checkout -b feature-brake
```

反馈结果：
```
Switched to a new branch 'feature-brake'
```

输入命令：
```bash
# 2. 查看分支
git branch
```

反馈结果：
```
* feature-brake
  master
```

输入命令：
```bash
# 3. 在新分支上修改文件并提交
git add model_v1.txt
git commit -m "刹车模块优化分支中"
```

反馈结果：
```
[feature-brake 735f071] 刹车模块优化分支中
 1 file changed, 2 insertions(+), 1 deletion(-)
```

输入命令：
```bash
# 4. 切回 master
git checkout master
```

反馈结果：
```
Switched to branch 'master'
```

> 切回 master 后，新分支上的改动在 master 上看不到。

输入命令：
```bash
# 5. 先合并回 master
git merge feature-brake
```

反馈结果：
```
Updating 211fbcb..735f071
Fast-forward
 model_v1.txt | 3 ++-
 1 file changed, 2 insertions(+), 1 deletion(-)
```

输入命令：
```bash
# 6. 删除已合并的分支
git branch -d feature-brake
```

反馈结果：
```
Deleted branch feature-brake (was 735f071).
```

> 如果确定不要一个尚未合并的分支，可以用 `git branch -D feature-brake` 强制删除。

### 3.2 临时保存：`git stash`

| 命令 | 作用 |
|------|------|
| `git stash` | 临时保存当前修改，工作区变干净 |
| `git stash pop` | 恢复最近一次 stash，并删除 stash |
| `git stash list` | 查看所有 stash 记录 |
| `git stash apply` | 恢复 stash，但不删除（可恢复多次） |
| `git stash drop stash@{n}` | 删除指定的 stash |
| `git stash clear` | 清空所有 stash |

**场景：** 改了一半 → stash 存起来 → 切分支干别的事 → 切回来 → stash pop 恢复 → 继续干活

**什么时候会用到：**

输入命令：
```bash
git checkout feature-steering
```

反馈结果（切换失败）：
```
error: Your local changes to the following files would be overwritten by checkout:
        model_v1.txt
Please commit your changes or stash them before you switch branches.
Aborting
```

> Git 拒绝切换，因为当前有未提交的修改，且目标分支的同一个文件内容不同。

**用 stash 解决：**

输入命令：
```bash
# 1. 临时保存修改
git stash
```

反馈结果：
```
Saved working directory and index state WIP on master: eb0f9ae 发动机参数调试中
```

输入命令：
```bash
# 2. 确认工作区干净了
git status
```

反馈结果：
```
nothing to commit, working tree clean
```

输入命令：
```bash
# 3. 现在可以切分支了
git checkout feature-steering
# ... 在别的分支上干完活 ...

# 4. 切回来，恢复修改
git checkout master
git stash pop
```

反馈结果：
```
On branch master
Changes not staged for commit:
        modified:   model_v1.txt
Dropped refs/stash@{0}
```

> 修改恢复了，stash 自动删除。

### 3.3 合并分支

输入命令：
```bash
git merge <分支名>
```

将指定分支合并到当前分支。

**Fast-forward 合并：** 当前分支没有新提交，直接往前移指针，最简单的情况。

输入命令：
```bash
git checkout master
git merge feature-brake
```

反馈结果：
```
Updating 211fbcb..735f071
Fast-forward
 model_v1.txt | 3 ++-
 1 file changed, 2 insertions(+), 1 deletion(-)
```

### 3.4 合并冲突

**为什么会冲突：** 两个分支修改了同一个文件的同一处内容，Git 不知道该听谁的，让你来决定。

**冲突标记怎么看：**

```
<<<<<<< HEAD
发动机温度参数调整到85度（master 的内容）
=======
转向模块已改完（feature-steering的内容）
>>>>>>> feature-steering
```

- `<<<<<<< HEAD` 到 `=======` = 当前分支的内容
- `=======` 到 `>>>>>>> feature-steering` = 要合并的分支的内容

**解决步骤：**

输入命令：
```bash
# 1. 打开冲突文件，手动编辑
#    决定保留哪些内容，删掉所有冲突标记（<<< === >>>）

# 2. 标记冲突已解决
git add model_v1.txt

# 3. 提交合并结果
git commit -m "合并xxx，解决冲突"
```

> 也可以用 `git merge --abort` 放弃合并，回到合并前的状态。

---

## 第四阶段：分支管理规范

### 常见规范之一：`git flow`

Git Flow 是一种经典分支管理规范，由 Vincent Driessen 在 2010 年提出，帮助团队有序地管理开发、测试和发布。它不是所有团队的默认做法，很多团队也会使用 GitHub Flow 或 trunk-based workflow。

**五种分支：**

| 分支 | 用途 | 从哪来 | 合并到哪 | 生命周期 |
|------|------|--------|---------|---------|
| `master` | 生产环境代码，随时可发布 | — | — | 永远存在 |
| `develop` | 开发主分支，集成所有功能 | master | — | 永远存在 |
| `feature/xxx` | 开发某个功能 | develop | develop | 用完就删 |
| `release/xxx` | 准备发布，最后测试 | develop | master + develop | 发布完就删 |
| `hotfix/xxx` | 紧急修复线上 bug | master | master + develop | 修完就删 |

**工作流程：**

| 步骤 | 命令 | 说明 |
|------|------|------|
| 1 | `git checkout develop` | 切到开发分支 |
| 2 | `git checkout -b feature/brake` | 从 develop 切出功能分支 |
| 3 | 开发、提交... | 在 feature 分支上干活 |
| 4 | `git checkout develop` → `git merge feature/brake` | 功能完成，合并回 develop |
| 5 | `git branch -d feature/brake` | 删掉功能分支 |
| 6 | `git checkout -b release/v1.0 develop` | 要发布了，从 develop 切出 release |
| 7 | 测试、修复小问题 | 在 release 分支上操作 |
| 8 | `git checkout master` → `git merge release/v1.0` | 合并到 master |
| 9 | `git tag v1.0` | 打标签标记版本 |
| 10 | `git checkout develop` → `git merge release/v1.0` | 也要合并回 develop |
| 11 | `git branch -d release/v1.0` | 删掉 release 分支 |

**紧急修复（hotfix）：**

| 步骤 | 命令 | 说明 |
|------|------|------|
| 1 | `git checkout -b hotfix/bug001 master` | 从 master 切出 hotfix |
| 2 | 修复、提交 | 在 hotfix 上干活 |
| 3 | `git checkout master` → `git merge hotfix/bug001` | 合并到 master |
| 4 | `git checkout develop` → `git merge hotfix/bug001` | 也要合并到 develop |
| 5 | `git branch -d hotfix/bug001` | 删掉 hotfix |

**注意：** release 和 hotfix 合并后都要打 tag，develop 也要同步合并。

**git-flow 工具（可选）：**

有专门的 git-flow 插件，可以把多步操作合并成一条命令：

| git-flow 命令 | 等价的手动操作 |
|--------------|--------------|
| `git flow init` | 初始化 git flow 配置 |
| `git flow feature start xxx` | `git checkout -b feature/xxx develop` |
| `git flow feature finish xxx` | 合并到 develop + 删除分支 |
| `git flow release start v1.0` | `git checkout -b release/v1.0 develop` |
| `git flow release finish v1.0` | 合并到 master 和 develop + 打 tag + 删除分支 |
| `git flow hotfix start xxx` | `git checkout -b hotfix/xxx master` |
| `git flow hotfix finish xxx` | 合并到 master 和 develop + 打 tag + 删除分支 |

> 不装工具也能用 git flow，手动操作就是上面的普通 git 命令。

### 4.2 独立分支模式（不合并的分支）

并非所有分支都要合并回 master。有时两个分支是同一项目的不同变体（如开发版 vs 测试版），各自独立维护，永远不交汇。

**典型场景：**
- `master` / `develop`：开发主线，正常迭代
- `mil`：MIL 测试专用版本，文件更多，内容不同，独立维护

**判断方法：先摸底再决定**

```bash
# 1. 看各分支文件规模
git ls-tree -r --name-only develop | wc -l   # →  4054 个文件
git ls-tree -r --name-only mil | wc -l       # → 20591 个文件

# 2. 看分支关系图
git log --all --oneline --graph
# 如果两个分支从 Initial commit 就分叉，各自发展，没有交集
```

**操作模式：**

| 分支 | 操作 | 合并到 master? |
|------|------|---------------|
| `develop` | 日常修改 → 提交 → merge 到 master | ✅ 会合并 |
| `mil` | 独立修改 → 独立提交 | ❌ 不合并 |

```bash
# mil 的日常维护
git checkout mil
# ... 修改 ...
git add -A
git commit -m "xxx"
# 完事，不需要 merge 到 master
```

> 核心原则：不是所有分支都要合到一起。用 `git log --all --graph` 看清关系，用 `git ls-tree` 了解规模，再决定操作策略。

---

## 第五阶段：进阶操作

### 5.1 变基：`git rebase`

rebase 和 merge 都是合并分支的方式，区别在于提交历史的形状：

| 方式 | 提交历史 | 特点 |
|------|---------|------|
| `git merge` | 有分叉，有合并节点 | 保留真实历史 |
| `git rebase` | 线性，没有分叉 | 历史更干净 |

输入命令：
```bash
# 将当前分支变基到 master
git rebase master
```

**交互式变基（更强大）：**

输入命令：
```bash
git rebase -i HEAD~3
```

可以对最近 3 次提交进行编辑、合并、删除：

| 选项 | 作用 |
|------|------|
| `pick` | 保留提交 |
| `reword` | 修改提交信息 |
| `squash` | 合并到前一个提交，保留信息 |
| `fixup` | 合并到前一个提交，丢弃信息 |
| `drop` | 删除提交 |

**注意：不要 rebase 已经推送到远程的提交，会打乱其他人的历史。**

### 5.2 拣选提交：`git cherry-pick`

`git cherry-pick <commit-id>` — 把某个提交"搬"到当前分支。

场景：只想把某个 bugfix 搬过来，不要整个分支。

输入命令：
```bash
git checkout master
git cherry-pick abc123
```

如果拣选过程中出现冲突，解决后继续：

输入命令：
```bash
# 解决冲突后
git add <冲突文件>
git cherry-pick --continue
```

也可以放弃拣选：
```bash
git cherry-pick --abort
```

### 5.3 标签：`git tag`

标签用于标记发布版本（如 v1.0），方便以后找到对应的提交。

| 命令 | 作用 |
|------|------|
| `git tag v1.0` | 给当前提交打轻量标签 |
| `git tag -a v1.0 -m "备注"` | 给当前提交打附注标签（推荐） |
| `git tag -a v1.0 abc123` | 给指定提交打标签（追溯） |
| `git tag` | 查看所有标签 |
| `git show v1.0` | 查看标签详情 |
| `git tag -d v1.0` | 删除本地标签 |

**轻量标签 vs 附注标签：**

| 类型 | 命令 | 信息 | 适用场景 |
|------|------|------|---------|
| 轻量标签 | `git tag v1.0` | 只有标签名 | 临时标记 |
| 附注标签 | `git tag -a v1.0 -m "备注"` | 包含作者、日期、备注 | 正式发布（推荐） |

**推送到远程：**

输入命令：
```bash
# 推送单个标签
git push origin v1.0

# 推送所有标签
git push origin --tags
```

**删除远程标签：**

输入命令：
```bash
git push origin --delete v1.0
```

> 默认情况下 `git push` 不会推送标签，需要显式推送。

### 5.4 交互式暂存

一个文件改了多处，只想提交其中一部分？用 `git add -p`。

输入命令：
```bash
git add -p
```

Git 会逐块（hunk）显示更改，你选择是否暂存每个块：

| 选项 | 作用 |
|------|------|
| `y` | 暂存当前块 |
| `n` | 跳过当前块 |
| `s` | 拆分成更小的块 |
| `e` | 手动编辑当前块 |
| `q` | 退出暂存 |

场景：你改了一个文件的 3 个地方，但这次只想提交其中 2 个，用 `git add -p` 逐块选择。

### 5.5 逐行追溯：`git blame`

`git blame <文件>` — 查看文本文件每一行是谁在什么时候改的。

输入命令：
```bash
git blame model_v1.txt
```

反馈结果：
```
abc1234 (LinBL 2026-06-06) # 汽车项目模型 v2 - 添加了发动机模块
def5678 (LinBL 2026-06-06) # 添加了刹车模块
ghi9012 (LinBL 2026-06-06) # 添加了转向模块
```

场景：发现某行代码有问题，想知道是谁改的、什么时候改的。

**常用选项：**

| 选项 | 作用 |
|------|------|
| `git blame <文件>` | 显示每一行的作者和提交信息 |
| `git blame -L 10,20 <文件>` | 只看第 10-20 行 |

### 5.6 撤销提交：`git revert`

`git revert <commit-id>` — 创建一个**新提交**来撤销指定提交，**不改变历史**。

和 `git reset` 的区别：

| 命令 | 效果 | 历史是否改变 | 适用场景 |
|------|------|------------|---------|
| `git reset` | 删除提交 | 改变（回退） | 还没推送到远程的提交 |
| `git revert` | 新增一个"反向提交" | 不改变 | 已经推送到远程的提交 |

输入命令：
```bash
git revert abc123
```

反馈结果：
```
[master xxxxxx] Revert "xxx的提交信息"
 1 file changed, 1 deletion(-)
```

> `git revert` 安全，因为它不删历史，只是加了一个"撤销"提交。推送到远程后也能用。

### 5.7 后悔药：`git reflog`

`git reflog` — 查看所有 HEAD 的移动记录，常用于找回被 `reset` 挪走的提交。

输入命令：
```bash
git reflog
```

反馈结果：
```
abc1234 HEAD@{0} commit: 发动机温度参数调整
def5678 HEAD@{1} reset: moving to HEAD~1
ghi9012 HEAD@{2} commit: v4 添加雨刷模块
...
```

**场景：** 你不小心 `git reset --hard` 丢掉了一个提交，用 reflog 找回来：

输入命令：
```bash
# 找到丢失的提交 ID
git reflog

# 恢复到那个提交
git reset --hard ghi9012
```

> `reflog` 主要用于找回已经存在过的提交；未提交且被覆盖掉的工作区修改，通常找不回来。

---

## 第六阶段：远程仓库操作

### 6.1 远程认证：SSH Key 与 Token

如果远程仓库使用 SSH 地址（如 `git@github.com:用户名/仓库名.git`），需要配置 SSH 密钥进行身份验证。其中 `用户名` 是你的 GitHub 用户名，`仓库名` 是你的仓库名称，实际地址在 GitHub 仓库页面点 **Code** → **SSH** 查看。

输入命令（把 `你的邮箱@example.com` 换成你注册 GitHub 用的邮箱）：
```bash
ssh-keygen -t ed25519 -C "你的邮箱@example.com"
```

> 也可以用 RSA 格式：`ssh-keygen -t rsa -C "你的邮箱@example.com"`。两种都能用，GitHub 都支持。ed25519 更短更安全，rsa 兼容性更好。

执行后会依次提示：

| 提示 | 含义 | 输入 |
|------|------|------|
| `Enter file in which to save the key` | 钥匙保存路径 | 直接回车用默认路径 |
| `Enter passphrase` | 给钥匙设密码（可选） | 直接回车不设密码 |
| `Enter same passphrase again` | 确认密码 | 直接回车 |

> 如果没有特殊情况，以上三步都可以直接回车，使用默认配置即可。

成功后在用户目录下生成 `.ssh` 文件夹（Windows 路径：`C:\Users\你的用户名\.ssh\`），里面有：
- 私钥（不要泄露、不要发给别人、不要提交到 Git 仓库）
- 公钥（复制这个，贴到 GitHub 上）
- `known_hosts` — 记录连接过的远程服务器信息（不用管）

| 生成时用的类型 | 私钥文件名 | 公钥文件名 |
|--------------|----------|----------|
| ed25519 | `id_ed25519` | `id_ed25519.pub` |
| rsa | `id_rsa` | `id_rsa.pub` |

> 两种类型的文件名不同，一台电脑上可以同时存在，不会互相覆盖。

> 如果已有 SSH Key，用 PowerShell 查看公钥内容即可，不用重新生成：`Get-Content $env:USERPROFILE\.ssh\id_ed25519.pub` 或 `Get-Content $env:USERPROFILE\.ssh\id_rsa.pub`。同一个公钥可以同时配到 GitHub 和 GitLab，互不影响。

**操作步骤：**
1. 复制公钥的内容
2. 登录 GitHub → Settings → SSH and GPG keys → New SSH key
3. 粘贴公钥，保存
4. 验证连接：

输入命令：
```bash
ssh -T git@github.com
```

反馈结果：
```
Hi username! You've successfully authenticated.
```

**SSH Key 安全知识：**

- **私钥**（`id_rsa` 或 `id_ed25519`）存在本地电脑上，不要泄露、不要发给别人、不要提交到 Git 仓库
- **公钥**（`id_rsa.pub` 或 `id_ed25519.pub`）可以安全地贴到 GitHub/GitLab 等平台
- 同一个公钥可以同时配到多个平台（GitHub + GitLab），互不影响
- 如果保存到已有的同名文件路径，会**覆盖**旧钥匙；如果换文件名，可以和旧钥匙并存
- 生成时可以设密码（passphrase），每次用钥匙时要输入，更安全但更麻烦

**SSH vs HTTPS + Token：**

| | SSH | HTTPS + Token |
|--|-----|--------------|
| 配置 | 生成钥匙 + 贴公钥到 GitHub | 生成 Token |
| 推/拉代码 | 自动验证，不用输密码 | 首次或凭据失效时输入 Token |
| 安全性 | 私钥在本地，泄露了要重新生成 | Token 可以随时撤销、设过期时间、控制权限 |
| 推荐 | ✅ 日常使用 | 备用方案，或公司要求时使用 |

> GitHub 从 2021 年起不再支持用账号密码通过 HTTPS 做 Git 认证，HTTPS 方式需要用 Personal Access Token 代替密码。
> Token 本质上就是你的身份凭证，应该像密码一样保管，只用于“代表你自己”访问仓库，不要发给别人共用。

**Token 生成步骤：**
1. GitHub 右上角头像 → **Settings**
2. 左侧最下面 → **Developer settings**
3. **Personal access tokens** → 选择 **Fine-grained tokens（推荐）** 或 **Tokens (classic)**
4. 点 **Generate new token**
5. 按用途选择仓库范围和权限
6. 设置过期时间
7. 点 **Generate token**，**复制保存好**（只显示一次）

**Token 使用方法：**
用 HTTPS 地址推代码时，首次输入凭据或凭据失效时，在输密码的地方粘贴 Token：
```bash
git remote add origin https://github.com/用户名/仓库名.git
git push -u origin master
# 提示输入密码时，粘贴 Token
```

**Token 安全知识：**
- Token 只显示一次，生成后马上复制保存
- 可以随时在 GitHub 上撤销，撤销后立即失效
- 可以设过期时间，到期自动失效
- 可以控制权限（只读、只写、全部权限等）
- Token 代表的是你的身份，不要分享给别人使用
- 访问组织仓库时，组织策略可能会限制 Token 的可用范围

**SSH Key 的适用场景：**
- 日常自己使用，配好一次就不用管
- 不适合临时分享（私钥泄露风险高）

**Token 的适用场景：**
- 公司要求使用 HTTPS 的情况
- 需要控制权限或设过期时间
- 在你自己的电脑上临时使用 HTTPS，或者不方便配置 SSH 时

### 6.2 远程仓库与地址

> 远程仓库操作本身不区分“这是你自己的仓库”还是“别人的仓库”，命令形式基本一样；真正决定能不能成功的是你是否有对应权限，以及仓库本身有没有限制。
> 一般来说：`clone` / `fetch` / `pull` 需要读权限，`push` 需要写权限；如果目标分支受保护，可能只能提 Pull Request，不能直接推到 `master`。

| 命令 | 作用 |
|------|------|
| `git remote` | 查看远程仓库列表 |
| `git remote -v` | 查看远程仓库详细地址 |
| `git remote add origin <url>` | 添加远程仓库，起别名 origin |
| `git remote rm origin` | 删除远程仓库别名 |

**远程仓库的两种常见地址格式（同一个仓库，地址不同）：**

| 类型 | 地址格式 | 验证方式 |
|------|---------|---------|
| SSH | `git@github.com:用户名/仓库名.git` | SSH Key（不用输密码，推荐） |
| HTTPS | `https://github.com/用户名/仓库名.git` | Token（首次或凭据失效时输入，详见 6.1） |

> **GitHub CLI（gh）** 是 GitHub 官方出的命令行工具，功能比 git 更多（创建仓库、提 PR、看 issue 等）。比如 `gh repo clone 用户名/仓库名` 也能克隆仓库，但它是一条命令，不是远程仓库地址。日常用 SSH 或 HTTPS 就够了，不用专门装 gh。

> 在 GitHub 仓库页面点 **Code** 按钮，可以切换查看 SSH 和 HTTPS 两种地址。

输入命令：
```bash
git remote -v
```

反馈结果：
```
origin    git@github.com:user/repo.git (fetch)
origin    git@github.com:user/repo.git (push)
```

### 6.3 克隆仓库

从远程仓库复制一份到本地：

输入命令：
```bash
git clone git@github.com:user/repo.git
```

反馈结果：
```
Cloning into 'repo'...
remote: Enumerating objects: 39, done.
remote: Total 39 (delta 12), reused 33 (delta 10)
Receiving objects: 100% (39/39), done.
```

克隆后自动设置好远程仓库关联，可以直接 `git push` / `git pull`。

> 可以指定目录名：`git clone <url> 新目录名`

### 6.4 获取远程更新

**两种方式：**

| 命令 | 效果 | 安全性 |
|------|------|--------|
| `git fetch` | 只下载，不合并 | 安全，可以先看看再决定合不合并 |
| `git pull` | 默认配置下下载 + 自动合并 | 方便，但可能有意外冲突 |

**推荐方式（fetch + merge）：**

输入命令：
```bash
# 1. 先下载远程更新（不合并）
git fetch
```

反馈结果：
```
From github.com:user/repo
   4b9dca5..685fd3a  master     -> origin/master
```

输入命令：
```bash
# 2. 看看远程改了什么（当前分支是谁，就拿谁和 origin/master 比）
git diff HEAD origin/master
```

反馈结果：
```
+# fetch测试
```

输入命令：
```bash
# 3. 确认没问题再合并
git merge origin/master
```

反馈结果：
```
Updating 4b9dca5..685fd3a
Fast-forward
 model_v1.txt | 1 +
 1 file changed, 1 insertion(+)
```

> `git fetch` 只下载到 `.git` 文件夹，工作区文件不变。用 `git status` 会显示 `Your branch is behind 'origin/master'`。

**快捷方式：**

输入命令：
```bash
# 一条命令搞定（默认配置下等于 fetch + merge）
git pull
```

反馈结果：
```
From github.com:user/repo
   c369cb0..4b9dca5  master     -> origin/master
Updating c369cb0..4b9dca5
Fast-forward
 model_v1.txt | 2 +-
 1 file changed, 1 insertion(+), 1 deletion(-)
```

> 如果设置了 `pull.rebase=true`，那么 `git pull` 会变成 `fetch + rebase`。

### 6.5 推送到远程

输入命令：
```bash
# 首次推送，-u 设置上游分支（以后可以直接 git push）
git push -u origin master

# 之后推送
git push
```

> `-u` 只需要第一次加，之后 Git 记住了关联关系，直接 `git push` 就行。

反馈结果：
```
Enumerating objects: 33, done.
Writing objects: 100% (33/33), 3.25 KiB, done.
Total 33 (delta 10), reused 0 (delta 0)
To github.com:user/repo.git
 * [new branch]      master -> master
branch 'master' set up to track 'origin/master'.
```

> `* [new branch] master -> master` 表示本地 master 推到了远程 master。

> SSH 地址格式：`git@github.com:用户名/仓库名.git`，在 GitHub 仓库页面点 **Code** → **SSH** 可以找到。

### 6.6 推送标签

默认 `git push` 不会推送标签，需要显式推送：

输入命令：
```bash
git push origin v1.0       # 推送单个标签
git push origin --tags     # 推送所有标签
```

### 6.7 常见工况与处理流程

> 远程操作没有一条“放之四海而皆准”的完整流程，要先判断自己处在哪种工况，再选对应动作。下面把最常见的工况集中列出来。

**工况A：本地已有仓库，要第一次推到空远程**

> 这个工况默认远程仓库是空的；如果你在 GitHub 上先创建了 README、LICENSE 等文件，第一次 `git push` 可能会被拒绝，需要先 `fetch/pull` 再处理。

```bash
# 1. 本地创建提交
git init
git add .
git commit -m "初始提交"

# 2. 关联远程仓库
git remote add origin git@github.com:user/repo.git

# 3. 首次推送
git push -u origin master
```

**工况B：从远程克隆下来，正常开分支开发**

```bash
# 1. 克隆
git clone git@github.com:user/repo.git

# 2. 进入目录，创建分支开发
cd repo
git checkout -b feature-xxx

# 3. 修改文件并提交
git add .
git commit -m "feature: 完成某个功能"

# 4. 推送分支
git push -u origin feature-xxx

# 5. 在 GitHub 上创建 Pull Request，合并到 master
```

**工况C：远程有新提交，本地工作区干净，只想安全同步**

```bash
# 1. 先下载远程更新
git fetch origin

# 2. 看看差异
git diff HEAD origin/master

# 3. 确认后合并
git merge origin/master
```

> 想一步完成也可以用 `git pull`，但学习阶段更推荐 `fetch` 后先看再合并。

**工况D：本地有已跟踪但未提交的修改，远程也更新了**

```bash
# 1. 先看本地改了什么
git status
git diff

# 2. 二选一：先提交，或者先 stash
git add .
git commit -m "WIP: 暂存当前进度"

# 或者：
git stash

# 3. 再获取远程更新
git fetch origin
git merge origin/master

# 4. 如果刚才用了 stash，最后恢复
git stash pop
```

**工况E：本地有未跟踪文件，远程也有同名文件**

```bash
# 1. 先查看本地未跟踪文件
git status

# 2. 导出远程版本做对比
git show origin/master:文件名 > 远程版文件名
git diff --no-index 本地文件名 远程版文件名

# 3. 决定后处理本地文件
#    可以改名、备份、删除，或先提交来故意练冲突

# 4. 再合并远程
git merge origin/master
```

> 这是你这次 `git学习笔记.md` 的真实工况，不能直接套“已跟踪文件 diff”的那套流程。

**工况F：本地和远程各自都有提交，要先对比再融合**

```bash
# 1. 先获取远程提交
git fetch origin

# 2. 看两边各自多了什么提交
git log --oneline --left-right HEAD...origin/master

# 3. 再看文件差异
git diff HEAD origin/master

# 4. 确认后合并
git merge origin/master
```

**工况G：`push` 被拒绝（non-fast-forward）**

```bash
# 1. 先获取远程更新
git fetch origin

# 2. 看远程比本地多了什么
git log --oneline HEAD..origin/master

# 3. 合并远程
git merge origin/master

# 4. 解决完冲突后再推送
git push origin master
```

> 这种报错通常表示：远程分支比你的本地分支更新，你不能直接把自己的历史硬推上去。

**工况H：故意练一次合并冲突**

```bash
# 1. 本地先提交一次
git add 文件名
git commit -m "本地修改"

# 2. 获取并合并远程
git fetch origin
git merge origin/master

# 3. 打开冲突文件，处理冲突标记
git add 文件名
git commit -m "解决合并冲突"
```

### 6.8 实战案例：本地与远程融合

**背景：**

典型情况：
- 有一个本地仓库（可能通过U盘拷贝或其他方式创建）
- 有一个远程仓库（GitHub/GitLab）
- 两边都有修改，需要对齐合并
- **以远程为主，但保留本地有价值的文件**

实际案例：
- 两台电脑（PC和linbl）都有同一个项目的仓库
- 一台电脑（linbl）已经更新并推送到GitHub
- 另一台电脑（PC）也有一些本地修改
- 需要同步，以远程为主

**操作步骤：**

#### 步骤 1：连接远程仓库

```bash
# 1. 添加远程仓库地址
git remote add origin https://github.com/用户名/仓库名.git

# 2. 验证是否添加成功
git remote -v
```

**这一步要点：**
- `origin` 是远程仓库的别名（约定俗成的名字）
- 一个仓库可以有多个远程别名（如同时添加GitHub和Gitee）
- HTTPS 访问公共仓库时，`clone` / `fetch` 通常可以直接进行；但 `push` 仍然需要认证。私有仓库通常在访问时就需要认证（Token或SSH）

#### 步骤 2：获取远程内容

```bash
# 1. 拉取远程内容（不覆盖本地）
git fetch origin

# 2. 查看远程分支
git branch -r

# 3. 查看所有分支（本地+远程）
git branch -a
```

**这一步要点：`fetch` vs `pull`**
- `git fetch`：只下载到`.git`目录的远程追踪分支，**不覆盖本地工作区**
- `git pull`：下载 + 自动合并，**可能产生冲突，或者因为本地修改会被覆盖而被 Git 拒绝**
- **学习阶段建议用`fetch`，更安全！**

**Git的"第四区"：远程追踪分支**
```
工作区 → 暂存区 → 本地仓库 → 远程追踪分支（origin/master）
```
- 远程追踪分支是只读的
- 存在`.git`目录里
- 用`origin/master`表示远程的master分支

#### 步骤 3：对比本地和远程

```bash
# 1. 查看本地文件
ls -la                     # Git Bash / Linux / macOS
Get-ChildItem -Force       # PowerShell

# 2. 查看远程文件
git ls-tree -r origin/master --name-only

# 3. 查看远程某个文件的内容
git show origin/master:文件名

# 4. 对比具体文件的差异（适合“本地已跟踪文件”和远程版本对比）
git diff origin/master -- 文件名

# 5. 如果本地是未跟踪文件，先把远程版本导出，再对比
git show origin/master:文件名 > 远程版文件名
git diff --no-index 本地文件名 远程版文件名

# 6. 查看差异概览
git diff --stat origin/master

# 7. 查看提交历史（带作者信息）
git log --format="%an <%ae> - %s" --all
```

**这一步技巧：**
- 用`cat 文件名`或 `Get-Content 文件名` 查看本地文件内容
- 用`git show origin/master:文件名`查看远程文件内容
- 本地已跟踪文件适合直接用`git diff`
- 本地未跟踪文件更适合先导出远程版本，再用`git diff --no-index`
- 用SourceTree图形化工具更直观

**对比命令速查表：**

| 命令 | 作用 | 使用场景 |
|------|------|---------|
| `git diff --stat origin/master` | 差异概览（文件名+行数变化） | 快速了解有哪些不同 |
| `git diff origin/master -- 文件名` | 具体文件的详细差异 | 本地已跟踪文件 vs 远程版本 |
| `git diff --no-index 本地文件 远程版文件` | 两个普通文件的差异 | 本地未跟踪文件 vs 远程导出文件 |
| `git ls-tree -r origin/master --name-only` | 列出远程所有文件 | 看远程有哪些文件 |
| `git show origin/master:文件名` | 查看远程文件内容 | 对比某个文件的具体内容 |
| `cat 文件名` / `Get-Content 文件名` | 查看本地文件内容 | 和远程版本对比 |

#### 步骤 4：做出决策并融合

**这一步先确认 3 件事：**
```bash
# 1. 看工作区是否干净
git status

# 2. 确认自己当前在哪个分支
git branch --show-current

# 3. 如果已经有本地提交，先留一个备份分支
git branch backup-before-sync
```

> `git reset --hard` 只会重置已追踪的内容；如果工作区里还有未提交的文件，尤其是未追踪文件，分支备份不一定能兜住，最好先手动备份，或先 commit 留痕。

**场景A：先对比，再逐个决策（最推荐）**
```bash
# 对比某个文件
git diff origin/master -- 文件名

# 手动修改成你最终想保留的内容

# 然后提交推送
git add .
git commit -m "合并本地和远程的内容"
git push origin master
```
- 最适合学习阶段
- 能清楚知道“本地改了什么，远程改了什么”
- 不会一上来就做高风险操作

**场景B：完全以远程为准（高风险）**
```bash
git reset --hard origin/master
```
- 本地所有文件变成和远程一模一样
- ⚠️ 本地已追踪的修改会丢失
- ✅ 未追踪的文件（untracked）不会被删除

**场景C：保留本地文件，以远程为主**
```bash
# 1. 先备份本地独有的文件（手动复制到其他地方）

# 2. 重置到远程版本
git reset --hard origin/master

# 3. 把备份的文件放回来

# 4. 添加到暂存区并提交
git add .
git commit -m "添加本地独有的文件"
git push origin master
```

**场景D：故意练一次合并冲突**
```bash
# 1. 先把本地同名文件提交一次
git add git学习笔记.md
git commit -m "提交本地版笔记"

# 2. 再合并远程
git merge origin/master
```
- 如果两边改了同一个文件的同一部分，就会进入冲突状态
- 打开冲突文件，处理 `<<<<<<<` / `=======` / `>>>>>>>` 标记
- 解决后再执行：
```bash
git add git学习笔记.md
git commit -m "解决合并冲突"
```

#### 步骤 5：提交和推送

```bash
# 1. 添加到暂存区
git add .                    # 添加所有文件
git add 文件名               # 添加单个文件

# 2. 提交到本地仓库
git commit -m "提交说明"

# 3. 推送到远程仓库
git push origin master

# 4. 最后确认状态
git status
git log --oneline --decorate --graph --all -10
```

**提交历史分析：**
```bash
# 查看所有提交的作者信息（可以看出是哪台电脑/用户提交的）
git log --format="%an <%ae> - %s" --all
```

### 6.9 常见问题

**Q1：fetch会覆盖本地文件吗？**
A：不会！fetch只下载到`.git`目录的远程追踪分支，完全不动工作区。

**Q2：`git reset --hard`会删除未追踪的文件吗？**
A：不会！只会重置已追踪的文件，untracked文件保持不变。

**Q3：可以同时添加多个远程仓库吗？**
A：可以！可以添加多个别名（如origin、github、gitee），指向不同地址。

**Q4：远程仓库需要设置user.name和user.email吗？**
A：不需要！这些是本地配置，只影响提交记录里的作者信息，远程仓库本身不需要设置。

**Q5：push时提示需要认证怎么办？**
A：
- 公共仓库：通常不需要认证
- 私有仓库：需要生成Personal Access Token，push时输入Token代替密码

**Q6：为什么切换分支后，资源管理器或 VSCode 里看到的文件会变？**
A：因为 Git 会把工作区文件切换成目标分支对应的版本。你切到别的分支，看到的就是那个分支当时的文件快照。

---

## 第七阶段：图形化工具

### Sourcetree

Atlassian 出品的免费 Git GUI 工具，支持 Windows 和 Mac。

**官网下载：** https://www.sourcetreeapp.com/

**主要功能：**

| 功能 | 说明 |
|------|------|
| 可视化分支图 | 图形化查看分支、合并历史 |
| 提交历史 | 点击查看每次提交的 diff |
| 远程仓库管理 | 直接克隆 GitHub/GitLab/Bitbucket 仓库 |
| 提交/推送/拉取 | 通过按钮操作，不需要敲命令 |
| 冲突解决 | 可视化对比冲突内容 |

**连接 GitHub：**
1. 安装后打开 Sourcetree
2. 点击右上角齿轮 → 账户 → 添加 GitHub 账户
3. 授权后即可看到自己的仓库，一键克隆

**适合的场景：**
- 不习惯命令行时，用图形界面操作
- 查看分支合并历史（比 `git log --graph` 更直观）
- 解决合并冲突时，可视化对比两边的修改

> Sourcetree 是命令行的补充，不是替代。建议先掌握命令行，再用 GUI 提高效率。

**其他常见 Git GUI 工具：**

| 工具 | 特点 |
|------|------|
| GitHub Desktop | GitHub 官方出品，简洁 |
| TortoiseGit | Windows 右键菜单集成 |
| VSCode 内置 Git | 编辑器自带，最方便 |

---

## 第八阶段：常见问题整理

### 8.1 基础概念题

**Q：Git 的三区是什么？**

A：本地三区是工作区（你编辑文件的地方）、暂存区（`git add` 后）、本地仓库（`git commit` 后）。远程仓库用于协作，不属于本地三区。流程：工作区 → `git add` → 暂存区 → `git commit` → 本地仓库。

**Q：文件的四种状态？**

A：Untracked（新文件未跟踪）、Modified（已修改未暂存）、Staged（已暂存未提交）、Committed（已提交）。

**Q：`git diff` 和 `git diff --staged` 的区别？**

A：`git diff` 对比工作区 vs 暂存区（看还没 add 的改动），`git diff --staged` 对比暂存区 vs 上次提交（看已经 add 但还没 commit 的改动）。

**Q：`git reset` 三种模式的区别？**

A：`--soft` 只撤 commit，修改留在暂存区；`--mixed` 撤 commit + add，修改留在工作区；`--hard` 会把提交回退，并丢弃当前未提交修改。记忆口诀：soft 只撤提交，mixed 多撤暂存，hard 全部撤掉。

---

### 8.2 实际场景题

**Q：误删了分支怎么恢复？**

A：用 `git reflog` 找到分支最后的提交 ID，然后 `git branch <分支名> <提交ID>` 重新创建。

**Q：提交了错误的代码怎么撤回？**

A：看情况：
- 还没推送到远程 → `git reset --soft HEAD~1` 撤销提交，修改回到暂存区
- 已经推送到远程 → `git revert <commit-id>` 创建一个反向提交，不改变历史

**Q：合并冲突怎么解决？**

A：四步：1. 打开冲突文件看 `<<<<<<<` / `=======` / `>>>>>>>` 标记；2. 手动编辑决定保留哪些；3. 删掉所有冲突标记；4. `git add` → `git commit`。也可以用 `git merge --abort` 放弃合并。

**Q：`git rebase` 和 `git merge` 怎么选？**

A：`merge` 保留真实历史（有分叉有合并），`rebase` 历史更干净（线性）。个人分支整理提交用 rebase，公共分支用 merge。已经推送到远程的提交不要 rebase。

**Q：改了一半要切分支，怎么办？**

A：用 `git stash` 临时保存 → 切分支干别的事 → 切回来 → `git stash pop` 恢复。

**Q：`.gitignore` 不生效怎么办？**

A：如果文件已经被提交过，`.gitignore` 对它无效。需要先 `git rm --cached <文件名>`（或批量 `git rm --cached -r '*.ext'`）从 Git 移除跟踪，再提交。两者缺一不可：`.gitignore` 管将来，`git rm --cached` 管过去。

---

### 8.3 进阶题

**Q：HEAD 是什么？**

A：HEAD 是一个指针，指向当前分支的最新提交。`HEAD~1` 是上一次提交，`HEAD~2` 是上上次。

**Q：`git fetch` 和 `git pull` 的区别？**

A：`git fetch` 只下载不合并（安全，可以先看看再决定），`git pull` 在默认配置下等于 `git fetch` + `git merge`（方便但可能有意外冲突）；如果配置了 `pull.rebase=true`，则会改成 `fetch + rebase`。推荐先 fetch 再手动处理。

**Q：什么情况下会丢失提交？**

A：`git reset --hard` 会丢弃当前未提交修改，也会把分支指针挪回去。被挪走的提交通常还能用 `git reflog` 找回来；真正容易永久丢的是未提交且被覆盖的修改，或者 reflog 记录过期、`.git` 被删除。

**Q：`git checkout` 和 `git switch` 的区别？**

A：`git checkout` 功能太多（切分支、恢复文件、检出提交），容易混淆。`git switch` 专门用于切分支，更直观。`git switch -c 新分支` 等于 `git checkout -b 新分支`。

**Q：分支名有什么命名规范？**

A：常用格式：
- `feature/功能名` — 新功能
- `bugfix/问题描述` — 修复 bug
- `hotfix/问题描述` — 紧急修复
- `release/版本号` — 发布分支

---

## 附录：常用命令速查表

### 基础操作

| 命令 | 作用 |
|------|------|
| `git init` | 在当前目录创建 Git 仓库 |
| `git status` | 查看当前状态 |
| `git diff` | 查看未暂存的修改 |
| `git diff --staged` | 查看已暂存但未提交的修改 |
| `git diff A B` | 对比两次提交 |
| `git add <file>` | 把修改放入暂存区 |
| `git add .` | 把所有修改放入暂存区 |
| `git add -p` | 按代码块选择性暂存 |
| `git commit -m "备注"` | 提交，写备注 |
| `git log` | 查看完整提交历史 |
| `git log --oneline` | 查看精简提交历史 |
| `git log --graph` | 图形化显示分支历史 |
| `git blame <文件>` | 逐行查看文件修改记录 |
| `git status -s` | 精简状态输出（两列格式） |
| `git ls-files '*.ext'` | 查看被跟踪的指定类型文件 |
| `git ls-files '*.ext' \| wc -l` | 统计指定类型文件数量 |
| `git ls-tree -r --name-only <分支>` | 列出某分支的所有文件 |
| `git rev-list --all --count` | 查看仓库总提交数 |

### 撤销操作

| 命令 | 作用 |
|------|------|
| `git restore <file>` | 丢弃工作区的修改 |
| `git restore --staged <file>` | 撤回暂存（修改保留在工作区） |
| `git reset --soft HEAD~1` | 撤销提交，修改保留在暂存区 |
| `git reset --mixed HEAD~1` | 撤销提交，修改保留在工作区 |
| `git reset --hard HEAD~1` | 撤销提交，并丢弃当前未提交修改 |
| `git revert <commit-id>` | 创建反向提交，撤销指定提交（不改历史） |
| `git reflog` | 查看所有操作记录，找回丢失的提交 |

### 分支操作

| 命令 | 作用 |
|------|------|
| `git branch` | 查看所有分支 |
| `git branch <名称>` | 创建新分支 |
| `git checkout <分支名>` | 切换分支 |
| `git checkout -b <名称>` | 创建并切换分支 |
| `git branch -d <名称>` | 删除已合并的分支 |
| `git branch -D <名称>` | 强制删除未合并的分支 |
| `git merge <分支名>` | 合并指定分支到当前分支 |
| `git stash` | 临时保存当前修改 |
| `git stash pop` | 恢复 stash 并删除 |
| `git stash list` | 查看所有 stash |

### 标签操作

| 命令 | 作用 |
|------|------|
| `git tag v1.0` | 打轻量标签 |
| `git tag -a v1.0 -m "备注"` | 打附注标签（推荐） |
| `git tag -a v1.0 <commit-id>` | 给指定提交打标签 |
| `git tag` | 查看所有标签 |
| `git show v1.0` | 查看标签详情 |
| `git tag -d v1.0` | 删除本地标签 |

### 远程仓库操作

| 命令 | 作用 |
|------|------|
| `git remote` | 查看远程仓库列表 |
| `git remote -v` | 查看远程仓库详细地址 |
| `git remote add origin <url>` | 添加远程仓库 |
| `git remote rm origin` | 删除远程仓库别名 |
| `git clone <url>` | 克隆远程仓库 |
| `git push` | 推送到远程 |
| `git push -u origin master` | 首次推送并设置上游分支 |
| `git push origin --tags` | 推送所有标签 |
| `git fetch` | 下载远程更新（不合并） |
| `git pull` | 默认配置下下载并合并远程更新 |

### 进阶操作

| 命令 | 作用 |
|------|------|
| `git rebase <分支>` | 变基，保持提交历史线性 |
| `git rebase -i HEAD~3` | 交互式变基（合并/编辑/删除提交） |
| `git cherry-pick <commit-id>` | 把某个提交搬到当前分支 |
| `git rm --cached <文件>` | 从 Git 移除跟踪（文件保留在电脑上） |
| `git rm --cached -r '*.ext'` | 按通配符批量移除跟踪 |
| `git ls-files '*.ext' \| xargs git rm --cached` | 批量移除（glob 失效时用） |
