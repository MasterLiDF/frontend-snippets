# README (中文)

## Git 实用操作教程（按开发工作流整理）
本文按**日常开发实际使用顺序**梳理Git核心操作，剔除重复内容、修正表述歧义，涵盖从仓库初始化到代码回滚的全流程，适配90%以上开发场景，指令标注优先级和注意事项。

## 一、前期准备：基础配置与仓库操作
### 1. 全局用户信息配置（首次使用必做）
提交代码前必须配置，关联提交记录的身份信息。
#### 设置用户名
git config --global user.name "Your Name"
#### 设置邮箱
git config --global user.email "your.email@example.com"
#### 查看所有配置，验证是否生效
git config --list
#### 生成密钥
ssh-keygen -t ed25519 -C "your.email@example.com"
#### 复制公钥
cat ~/.ssh/id_ed25519.pub | clip
✨ 说明：去掉--global仅对当前仓库生效，多账号开发可按需配置。

### 2. 仓库初始化/克隆
#### 方式1：本地新建仓库
#### 在当前目录创建空Git仓库
git init
#### 关联远程仓库（后续推送用）
git remote add origin 远程仓库地址(HTTPS/SSH)

#### 方式2：克隆已有远程仓库（最常用）
#### HTTPS方式（无需配置，每次推送需输账号密码）
git clone https://github.com/username/repo.git
#### SSH方式（需配置SSH密钥，免密登录，推荐）
git clone git@github.com:username/repo.git
#### 克隆并指定本地目录名
git clone 远程仓库地址 my-repo

### 3. 查看远程仓库关联信息
git remote -v

## 二、日常开发：分支操作（Git核心）
开发需基于分支进行，主分支（main/master）仅用于发布，功能开发在子分支完成。
### 1. 查看分支
#### 查看本地分支（* 标记当前所在分支）
git branch
#### 查看所有分支（本地+远程）
git branch -a

### 2. 创建并切换分支（最常用）
#### 传统指令
#### 先创建再切换
git branch 分支名(如feature/payment)
git checkout 分支名
#### 一步创建并切换（推荐）
git checkout -b 分支名

#### Git 2.23+ 新指令（更语义化，替代checkout）
#### 切换已有分支
git switch 分支名
#### 一步创建并切换（推荐）
git switch -c 分支名

### 3. 拉取远程分支最新代码
开发前先拉取，避免代码冲突。
#### 拉取远程指定分支代码并自动合并（日常常用）
git pull origin 分支名
#### 仅拉取代码不合并（先查看差异，再手动合并，适合复杂场景）
git fetch origin 分支名

## 三、开发完成：代码暂存与提交
### 1. 查看文件状态（开发中高频使用）
确认修改、新增、未跟踪的文件，避免漏提交/错提交。
#### 详细输出（推荐，清晰展示文件状态）
git status
#### 简化输出（一行一条，快速查看）
git status -s

### 2. 暂存修改文件
将工作区的修改提交到暂存区，为后续提交做准备。
#### 暂存指定单个/多个文件
git add 文件名1.txt 文件名2.py
#### 暂存所有修改/新增文件（推荐，日常开发最常用）
git add .
#### 暂存所有修改（包括删除的文件，不含新增未跟踪文件）
git add -u
#### 交互式暂存（可选择文件的部分内容暂存，适合精细化提交）
git add -p

### 3. 提交暂存区代码
提交到本地仓库，生成提交记录，**提交信息需遵循规范**（如feat/fix/docs前缀，便于追溯）。
#### 提交并手动输入提交信息（需进入编辑界面，保存退出）
git commit
#### 直接提交并填写提交信息（推荐，最常用）
git commit -m "feat: 新增支付页面功能"
#### 修正最后一次提交（修改提交信息/补充暂存的文件，未推远程时使用）
git commit --amend -m "fix: 修复支付页面样式问题"

### 4. 查看提交记录
验证提交是否成功，或追溯历史修改。
#### 查看所有提交记录（按时间倒序，详细信息）
git log
#### 简化输出（一行一条，显示提交ID和信息，推荐）
git log --oneline
#### 查看指定文件的提交记录
git log 文件名
#### 可视化分支合并图（查看所有分支的提交历史和合并关系）
git log --graph --oneline --all

## 四、功能完成：推送远程+合并分支
### 1. 推送本地分支到远程
#### 首次推送（关联本地和远程分支，后续推送无需加-u）
git push -u origin 分支名
#### 非首次推送（直接推送）
git push origin 分支名

### 2. 合并分支到主分支（如main）
推荐使用--no-ff参数，保留分支历史，便于后续回滚，符合开发规范。
#### 1. 先切换到主分支
git switch main
#### 2. 拉取主分支最新代码（避免合并冲突）
git pull origin main
#### 3. 合并功能分支到主分支（推荐--no-ff方式）
git merge --no-ff 功能分支名(如feature/payment)
#### 执行合并操作，但不会自动创建合并提交（commit）
git pull feature/a --no-commit
#### 4. 若出现冲突，解决后执行以下命令继续合并
git add .
git merge --continue
#### 5. 合并完成后，推送主分支到远程
git push origin main

### 3. 删除分支（功能合并后清理）
#### 删除本地已合并的分支（安全，会校验是否合并）
git branch -d 功能分支名
#### 强制删除本地未合并的分支（谨慎，会丢失未合并代码）
git branch -D 功能分支名
#### 删除远程分支（功能合并后同步清理）
git push origin --delete 功能分支名

## 五、问题处理：撤销/回滚操作（救急必备）
按**从轻到重**排序，优先使用低风险方式，**已推远程的代码禁止使用--hard强制回滚**（会导致团队历史不一致）。
### 1. 撤销工作区修改（未暂存，最轻量）
恢复文件到最近一次提交/暂存的状态，未提交的修改会被覆盖。
#### 传统指令
git checkout -- 文件名
#### Git 2.23+ 推荐指令（更语义化）
git restore 文件名

### 2. 撤销暂存区修改（已暂存，未提交）
将暂存区的文件退回工作区，保留修改内容。
#### 传统指令
git reset HEAD 文件名
#### Git 2.23+ 推荐指令
git restore --staged 文件名

### 3. 回滚本地提交（已提交，未推远程）
#### 方式1：保留工作区修改（仅撤销提交，可重新提交）
#### 回滚到上一次提交
git reset --soft HEAD^
#### 回滚到指定提交（commit_id通过git log --oneline查看）
git reset --soft 提交ID

#### 方式2：彻底回滚（删除工作区所有未提交修改，谨慎使用）
#### 回滚到上一次提交
git reset --hard HEAD^
#### 回滚到指定提交
git reset --hard 提交ID

### 4. 回滚远程提交（已推远程，推荐使用git revert，无风险）
新增**反向提交**抵消目标提交的修改，保留原有提交历史，团队协作友好。
#### 撤销单个普通提交
git revert 提交ID
#### 撤销--no-ff方式的合并提交（-m 1表示保留主分支代码，丢弃功能分支代码）
git revert -m 1 合并提交ID
✨ 注意：多提交撤销需**从后往前**执行，避免代码冲突。

## 六、版本管理：标签（tag）操作
用于标记版本发布节点（如v1.0.0），便于版本追溯和回退，操作需同步远程。
### 1. 打标签（关联指定提交ID，推荐加注释）
#### 1. 查看提交ID（确定版本对应的提交）
git log --oneline
#### 2. 打带注释的标签（-a=标签名，-m=标签注释，推荐）
git tag -a 标签名(如v1.0.1) 提交ID -m "版本1.0.1：优化支付页面样式"
#### 3. 推送标签到远程
git push origin 标签名

### 2. 删除标签
#### 1. 删除本地标签
git tag -d 标签名
#### 2. 删除远程标签
git push origin --delete 标签名

### 3. 根据标签回退版本（生产环境谨慎使用！）
回退后需强制推送，**仅适用于紧急情况**，需提前和团队沟通。
#### 1. 切换到目标分支（如main）
git switch main
#### 2. 重置分支到标签对应的提交（--hard覆盖工作区和暂存区）
git reset --hard 标签名
#### 3. 强制推送到远程（生产环境禁止随意使用！）
git push -f origin main

## 七、核心操作速查（日常开发高频使用）
### 1. 基础提交流程
git status → git add . → git commit -m "提交信息" → git push

### 2. 分支管理流程
git switch -c 功能分支 → 开发提交 → git push -u origin 功能分支 → 切换主分支git switch main → git pull origin main → git merge --no-ff 功能分支 → git push origin main → git branch -d 功能分支+git push origin --delete 功能分支

### 3. 常用救急指令
- 撤销工作区修改：git restore 文件名
- 回滚本地未推提交：git reset --soft 提交ID
- 回滚远程已推提交：git revert 提交ID

## 八、重要注意事项
1. 主分支（main/master）禁止直接开发，所有功能在子分支完成后合并；
2. 推送代码前务必拉取远程最新代码，避免冲突；
3. 已推远程的代码，**绝对禁止**使用git reset --hard强制回滚；
4. 强制推送（git push -f）、强制删除分支（git branch -D）仅在紧急情况使用，需提前和团队沟通；
5. 提交信息、分支命名遵循团队规范，便于协作和追溯。

##  储藏代码
git stash





