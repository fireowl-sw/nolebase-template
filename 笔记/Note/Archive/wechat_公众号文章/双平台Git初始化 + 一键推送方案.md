# 🚀 双平台Git初始化 + 一键推送方案

## 🔐 第一步：SSH配置（GitHub必需）

```bash
# 生成SSH密钥（邮箱替换为你的GitHub绑定的邮箱）
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
# 一路回车即可（或设置自定义密码）

# 启动SSH代理
eval "$(ssh-agent -s)"

# 添加密钥到代理
ssh-add ~/.ssh/id_rsa

# 复制公钥（终端会显示公钥内容，选中复制）
cat ~/.ssh/id_rsa.pub
```

然后去 GitHub和Gitee：Settings → SSH and GPG keys → New SSH key → 添加复制的公钥

## 📦 第二步：Git初始化

```bash

# 配置用户信息（替换为你的用户名和邮箱）
git config --global user.name "your_username"
git config --global user.email "your_email@example.com"

# 查看设置的内容是否正确
git config --list

# 初始化仓库
git init

# 如果想设置默认分支为 main
  # 1. 删除.git文件夹
  rm -rf .git

  # 2. 设置全局默认分支为main
  git config --global init.defaultBranch main

  # 3. 重新初始化（现在默认就是main分支）
  git init

  # 4. 验证分支
  git branch
  # 显示 * main


# 添加远程仓库（替换为你的仓库地址）
git remote add gitee https://gitee.com/your_username/your_repo.git
git remote add github git@github.com:your_username/your_repo.git

# 修改远程仓库为SSH（避免gitee每次输入账号密码）
git remote set-url gitee git@gitee.com:your_username/your_repo.git
git remote set-url github git@github.com:your_username/your_repo.git


# 创建.gitignore（指定不需要版本控制的文件/目录，也可以自己创建个文件，把这些文字写上去）
echo -e "build/\ninstall/\nlog/\n__pycache__/\n*.pyc\n.vscode/" > .gitignore
```

![[costmap_function_explained-1.png]]

## 🚀 第三步：首次推送

```bash
# 添加所有文件到暂存区
git add .

# 提交到本地仓库, 双引号是你想描述当前项目是什么样的情况的内容
git commit -m "first commit"

# 推送到远程仓库（-u 参数设置上游分支，后续可直接用 git push）
git push -u gitee main
git push -u github main
```

## ⚡ 一键推送方法

### 方法A：设置推送别名

```bash
# 创建全局 Git 别名，一键推送到两个平台
git config --global alias.pushall '!git push gitee main && git push github main'
# 使用方法：git pushall
```

### 方法B：创建推送脚本

```bash
# 创建可执行脚本
echo '#!/bin/bash
git push gitee main && git push github main' > push.sh

# 添加执行权限
chmod +x push.sh
# 使用方法：./push.sh
```

## 🔄 日常使用

```bash
# 日常推送流程
git add .                        # 添加所有修改到暂存区
git commit -m "更新描述"         # 提交修改并添加说明
git pushall                      # 推送到两个平台
# 或使用脚本：./push.sh

# 日常拉取流程
# 1. 先从远程拉取最新代码
git pull gitee main

# 2. 进行代码修改...

# 3. 提交并推送到两个平台
git add .
git commit -m "更新内容"
git pushall

```
