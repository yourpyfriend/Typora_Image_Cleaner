# 使用前提｜Prerequisites

> 中文：本文翻译使用无敌帅气的 AI「豆包」豆姐大人 sama 
>
> 日本語：本稿の翻訳は無敵でかっこいい AI「豆包」お姉さま sama による 
>
> English：This document is translated by the invincible and cool AI "Doubao", Sister‑sama



1. Typora开启设置「复制图片到./assets 文件夹」，**并确保所有图片都复制到./assets文件夹中**（Typora只会对开启选项后的图片进行该操作，不会应用于历史图片）。

   > In Typora, enable **Copy images to ./assets folder**. Ensure all images are saved under the `./assets` directory. Note: This setting only applies to newly‑inserted images; existing historical images will not be moved automatically.

2. 勾选图片语法偏好「优先使用相对路径」、「为相对路径添加./」

   > Enable image syntax preferences: **Prefer relative paths** and **Add ./ for relative paths**.

3. 本项目中提到的Typora图片文件均本地插入，并未尝试互联网图片插入的场景，请不要在备份前尝试。

   > This tool only handles locally‑inserted images. Online remote images are not supported. Always back up your files before running.

# 警告｜Warnings

- **会造成图片文件的跨文档移动**

  > Images will be moved across directories.

- **无法复原**

  > Operations cannot be undone.

- **请备份后尝试**

  > **Please make backups before use.**

# 项目说明｜Project Description

Typora没有图片整理功能，本项目基于「图片清理」功能设计，纯个人使用。（配合一些文件夹同步功能，多设备用户很省心）

> Typora lacks built‑in image‑organization features. This script implements image cleanup for personal use. Works great with folder synchronization for multi‑device workflows.

```
# Typora本身已经使用如下两种图片结构
# Markdown 格式 ![alt](path)
# HTML 格式  <img src="./assets/xxxx.png" alt="..." style="...">
# 本项目对其进行匹配查找并移动到根目录的 ./_trash_bin 文件下
# 系统环境是 windows10 
# Typora版本 1.14.9

# Typora supports two image embedding formats
# Markdown: ![alt](path)
# HTML: <img src="./assets/xxxx.png" alt="..." style="...">
# This script parses both patterns and moves unreferenced images into ./_trash_bin at root.
# Tested on Windows 10, Typora version 1.14.9
import re
import shutil
import os
from pathlib import Path

# 获取工作路径
# Get working directory
print("请输入你放置图片文件夹的名称: ")
print("Please enter your image‑containing root folder path: ")
LocalPath_input = Path(input(r"请输入路径：｜Please input path: ").strip().strip('"'))

# 检查并创建 [_trash_bin]文件夹
# Create _trash_bin directory if not exists
try:
    os.makedirs(str(LocalPath_input) + '/_trash_bin', exist_ok=True)
except ValueError:
    print("有点问题｜Encountered an error")

# 创建正则对象，方便调用，查找结构有两种
# Regex patterns for two image formats
# HTML 格式  <img src="./assets/xxxx.png" alt="..." style="...">
# For HTML <img src="...">
pattern_html = re.compile(r'src="([^"]+)"')
# Markdown 格式 ![alt](path)
# For markdown ![alt](image-path)
pattern_md = re.compile(r'!\[[^\]]*\]\(([^)]+)\)')

# md中的图片文件地址列表，set 去重
# Store all referenced absolute image paths, use set for deduplication
mdImg_list = set()

# 逐个读取 md 文件，查找图片文件列表
# Traverse all markdown files and collect referenced image paths
for md_file in LocalPath_input.rglob("*.md"):
    # 获取文本
    md_str = md_file.read_text(encoding="utf-8")
    paths = pattern_html.findall(md_str) + pattern_md.findall(md_str)
    print(str(md_file))
    print(paths)
    # Paths 拼接自动处理 ./，再 resolve 成绝对路径
    for p in paths:
        full = (md_file.parent / p).resolve()
        mdImg_list.add(full)

# rglob遍历 寻找 img 文件
# Scan all png image files recursively
imgfile_list = list(LocalPath_input.rglob("*.png"))

# 创建回收站对象，用于去重
trash = LocalPath_input / "_trash_bin"

# 将没有引用的图片移至 _trash_bin
# Move unreferenced images into _trash_bin
for img in imgfile_list:
    # 跳过回收站自身里的图片
    # Skip files already inside trash bin
    if trash in img.parents:
        continue
    # 未被引用的才移动（Path 与 Path 比较，类型保持一致）
    if img.resolve() not in mdImg_list:
        target = LocalPath_input / "_trash_bin" / img.name
        shutil.move(str(img), str(target))
        print(f"移动｜Moved: {img} -> {target}")
```
