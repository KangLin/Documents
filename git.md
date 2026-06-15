
### Git 代码量统计命令
#### 指定用户名和起止日期

```bash
git log --since="2021-03-01" --before="2022-01-09" --author="username" --pretty=tformat: 
--numstat | awk '{ add += $1; subs += $2; loc += $1 - $2 } END 
{ printf "added lines: %s, removed lines: %s, total lines: %s\n", add, subs, loc }'
```

直接复制粘贴即可，username换成你的用户名，【since="2021-03-01" --before="2022-01-09"】中的日期换成你想要的日期。执行后会输出在当前项目内，指定用户名的用户的代码量统计信息，示例如下:

```
added lines: 15909 removed lines : 6359 total lines: 9550
```

#### 统计所有用户的行数信息

- 扫描了当前分支的每个文件，然后输出每个人的代码增删行数信息:

```bash
git log --shortstat --pretty="%cE" | sed 's/\(.*\)@.*/\1/' | grep -v "^$" | awk 'BEGIN { line=""; } !/^ / { if (line=="" || !match(line, $0)) {line = $0 "," line }} /^ / { print line " # " $0; line=""}' | sort | sed -E 's/# //;s/ files? changed,//;s/([0-9]+) ([0-9]+ deletion)/\1 0 insertions\(+\), \2/;s/\(\+\)$/\(\+\), 0 deletions\(-\)/;s/insertions?\(\+\), //;s/ deletions?\(-\)//' | awk 'BEGIN {name=""; files=0; insertions=0; deletions=0;} {if ($1 != name && name != "") { print name ": " files " files changed, " insertions " insertions(+), " deletions " deletions(-), " insertions-deletions " net"; files=0; insertions=0; deletions=0; name=$1; } name=$1; files+=$2; insertions+=$3; deletions+=$4} END {print name ": " files " files changed, " insertions " insertions(+), " deletions " deletions(-), " insertions-deletions " net";}'

```

示例如下：

```bash
k@DESKTOP-T3RLUJL MSYS /d/Source/RabbitRemoteControl
$ git log --shortstat --pretty="%cE" | sed 's/\(.*\)@.*/\1/' | grep -v "^$" | awk 'BEGIN { line=""; } !/^ / { if (line=="" || !match(line, $0)) {line = $0 "," line }} /^ / { print line " # " $0; line=""}' | sort | sed -E 's/# //;s/ files? changed,//;s/([0-9]+) ([0-9]+ deletion)/\1 0 insertions\(+\), \2/;s/\(\+\)$/\(\+\), 0 deletions\(-\)/;s/insertions?\(\+\), //;s/ deletions?\(-\)//' | awk 'BEGIN {name=""; files=0; insertions=0; deletions=0;} {if ($1 != name && name != "") { print name ": " files " files changed, " insertions " insertions(+), " deletions " deletions(-), " insertions-deletions " net"; files=0; insertions=0; deletions=0; name=$1; } name=$1; files+=$2; insertions+=$3; deletions+=$4} END {print name ": " files " files changed, " insertions " insertions(+), " deletions " deletions(-), " insertions-deletions " net";}'
kl222,: 15309 files changed, 378267 insertions(+), 220548 deletions(-), 157719 net
noreply,: 32 files changed, 1071 insertions(+), 245 deletions(-), 826 net
```

- 如下统计命令不区分用户：

```bash
git log --pretty=tformat: --numstat | awk '{ add += $1; subs += $2; loc += $1 - $2 } END { printf "added lines: %s, removed lines: %s, total lines: %s\n", add, subs, loc }'
```

示例如下：

```bash
k@DESKTOP-T3RLUJL MSYS /d/Source/RabbitRemoteControl
$ git log --pretty=tformat: --numstat | awk '{ add += $1; subs += $2; loc += $1 - $2 } END { printf "added lines: %s, removed lines: %s, total lines: %s\n", add, subs, loc }'
added lines: 379338, removed lines: 220793, total lines: 158545
```

### 恢复 `git reset --hard ` 的代码
#### 已经提交了的代码

```bash
git reflog
```
执行后会输出如下信息：

```
e1b2c3d HEAD@{0}: reset: moving to e1b2c3d
a1b2c3d HEAD@{1}: commit: some commit message
```

其中 `a1b2c3d` 是你之前提交的 commit id，执行如下命令即可恢复：

```bash
git reset --hard a1b2c3d
```
#### 已经 `git add`，但没有提交的代码

```bash
git fsck --lost-found
```
执行后会输出如下信息：

```
dangling blob 1234567
```
其中 `1234567` 是你丢失的代码的 blob id，执行如下命令即可恢复：

```bash
git show 1234567 > recovered_code.txt
```
执行后会在当前目录下生成一个 `recovered_code.txt` 文件，里面就是你丢失的代码。

- 给出一个恢复脚本：

```bash
#!/bin/bash
# 1. 列出所有悬空 blob 的哈希值
git fsck --lost-found | awk '{print $3}' > blob_hashes.txt

# 2. 逐个查看内容（交互式）
while read hash; do
    echo "=== 查看对象: $hash ==="
    git show $hash | head -20  # 显示前20行
    #echo "是否恢复这个文件？(y/n)"
    read  -t 60 -p "是否恢复这个文件？(y/N)" INPUT < /dev/tty
    if [ "${INPUT:-N}" != "Y" ] && [ "${INPUT:-N}" != "y" ]; then
		continue
	fi

    git show $hash > "recovered_$(date +%s)_${hash:0:8}.txt"
    echo "已保存为 recovered_${hash:0:8}.txt"    
done < blob_hashes.txt
```

执行后会逐个显示悬空 blob 的内容，并询问是否恢复，如果输入 `y` 或 `Y` 则会将该 blob 的内容保存到一个新的文件中，文件名格式为 `recovered_<timestamp>_<hash>.txt`。如果没有输入，则默认不恢复。
- 注意：如果你有很多悬空 blob，建议先将它们的哈希值保存到一个文件中，然后再逐个查看和恢复，以免一次性显示过多内容导致混乱。


