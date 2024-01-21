删除第一行

```text-plain
sed -i '1d'  test.txt
```

删除倒数第一行

```text-plain
sed -i '$d' test.txt
```

删除倒数第n行

```text-plain
sed -i ''$[$(cat test.txt|wc -l)-n+1]'d' test.txt
```

删除倒数第1行到倒数第n行

```text-plain
sed -i '$,'$[$(cat test.txt|wc -l)-n+1]'d' test.txt
```