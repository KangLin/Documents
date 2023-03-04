
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
