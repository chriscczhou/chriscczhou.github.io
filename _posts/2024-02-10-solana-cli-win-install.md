---
title: "Windows 安装 solana cli"
date: 2024-02-10T0:19:30-04:00
categories:
  - blog
tags:
  - solana
excerpt_separator: ""
---

### Windows 安装 solana cli
1. 去github查询最新的solana版本：https://github.com/solana-labs/solana/releases
2. 替换版本然后下载：https://release.solana.com/v1.17.18/solana-install-init-x86_64-pc-windows-msvc.exe
3. 下载完成后，使用管理员权限打开一个`cmd`，切换到下载目录，去执行以下命令：`solana-install-init-x86_64-pc-windows-msvc.exe v1.17.18`
   ![image](https://github.com/chriscczhou/solanaL/assets/108380177/f11399f2-add6-4b59-8ba1-7e3f034c2106)
4. 按下任意键，新建一个普通用户的`cmd`窗口输入：`solana --version`，返回正确的版本说明安装成功:
   ![image](https://github.com/chriscczhou/solanaL/assets/108380177/c843bcb0-20a5-4cbe-8f7a-758aa025fe54)
