~~~
# Typora本身已经使用如下两种图片结构
# Markdown 格式 ![alt](path)
# HTML 格式  <img src="./assets/xxxx.png" alt="..." style="...">
# 本项目对其进行匹配查找并移动到根目录的 ./_trash_bin 文件下
# 系统环境是 windows10 
# Typora版本 1.14.9

import re
import shutil
import os
from pathlib import Path

# 获取工作路径
print("请输入你放置图片文件夹的名称: ")
print("注意，路径采用反斜杠「/」，特殊符号需要用「\」转义") # 这句话其实不用
LocalPath_input = Path(input(r"请输入路径：").strip().strip('"'))

# 检查并创建 [_trash_bin]文件夹
try:
    os.makedirs(str(LocalPath_input) + '/_trash_bin', exist_ok=True)
except ValueError:
    print("有点问题")

# 创建正则对象，方便调用，查找结构有两种
# HTML 格式  <img src="./assets/xxxx.png" alt="..." style="...">
pattern_html = re.compile(r'src="([^"]+)"')
# Markdown 格式 ![alt](path)
pattern_md = re.compile(r'!\[[^\]]*\]\(([^)]+)\)')

# md中的图片文件地址列表，set 去重
mdImg_list = set()

# 逐个读取 md 文件，查找图片文件列表
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
imgfile_list = list(LocalPath_input.rglob("*.png"))

# 创建回收站对象，用于去重
trash = LocalPath_input / "_trash_bin"

# 将没有引用的图片移至 _trash_bin
for img in imgfile_list:
    # 跳过回收站自身里的图片
    if trash in img.parents:
        continue
    # 未被引用的才移动（Path 与 Path 比较，类型保持一致）
    if img.resolve() not in mdImg_list:
        target = LocalPath_input / "_trash_bin" / img.name
        shutil.move(str(img), str(target))
        print(f"移动: {img} -> {target}")

~~~
