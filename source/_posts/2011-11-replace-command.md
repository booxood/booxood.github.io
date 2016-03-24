title: 文本替换命令
date: 2011-11-01 17:54:47
tags: Linux
categories: Linux
---

在 Unix/Linux 系统中经常需要在命令中替换某一类文件中的内容，例如：把 *.xml 文件里的 aaaa 替换成 bbbb 这样的操作。


下面介绍几种方法。

- bash

封装一个 bash 脚本来完成这个操作

```bash
#!/usr/bin/bash

old_str=$1
new_str=$2
file_name=$3

echo "old_str:[$old_str]"
echo "new_str:[$new_str]"
echo "file_name:[$file_name]"

change_fun()
{
  old_str1=$1
  new_str1=$2
  file_name1=$3
  (echo "vi ${file_name1} <<-!";
  echo ":%s:$old_str1:$new_str1:g";
  echo ":wq";
  echo "!";
  )|sh
}

for filename in `ls ./*${file_name}*`
do
  change_fun "$old_str" "$new_str"  "$filename"
  #echo "change_fun \"$old_str\" \"$new_str\"  \"$filename\""
done
```

脚本的步骤是： vi 打开文件 -> 使用 `%s` 正则匹配替换 -> 保存并关闭

- sed

```
sed -i 's/old_str/new_str/g' *filename*
```

如果是sed的老版本不支持 -i 参数，那就不能修改源文件，只能是先把替换的结果用重定向符输出到一个临时文件，然后再覆盖源文件了。

- perl

```
perl -pi -e 's/old_str/new_str/g' *filename*
```
