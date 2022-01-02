---
title: Trie树
date: 2021-09-19 16:02:32 
tags: 算法
---

# 介绍  

Trie:高效地存储和查找字符串集合的数据结构

<!-- more -->  

# Trie树

维护一个字符串集合，支持两种操作：

1. `I x` 向集合中插入一个字符串 xx；
2. `Q x` 询问一个字符串在集合中出现了多少次。

共有 N 个操作，输入的字符串总长度不超过 10^5，字符串仅包含小写英文字母。

------

实现：

```c++
#include<iostream>
using namespace std;
const int N = 100010;
int son[N][26], cnt[N], idx; //下标是0的点，既是根节点，也是空节点
char str[N];
void insert(char str[])
{
    int p = 0;
    for(int i = 0; str[i]; i++)
    {
        int u = str[i] - 'a';
        if(!son[p][u]) son[p][u] = ++ idx;
        p = son[p][u];
    }
    cnt[p] ++;
}

int query(char str[])
{
    int p = 0;
    for(int i = 0; str[i]; i++)
    {
        int u = str[i] - 'a';
        if(!son[p][u]) return 0;
        p = son[p][u];
        
    }
    return cnt[p];
}

int main()
{
    int n;
    scanf("%d", &n);
    while(n--)
    {
        char op[2];
        scanf("%s%s",op,str);
        if(op[0] == 'I') insert(str);
        else printf("%d\n",query(str));
        
    }
    return 0;
}
```

