---
layout: post
title: 利用优先队列建立高频词表
categories: Deep-Learning
---

## 前言

在构建分类器之前，我们必须先把文本的特征处理一下。如果将所有词作为作为特征，那么输入向量的维度会很长，而且会很稀疏。考虑到这一点，我们可以先筛选出高频词汇建立词汇表，例如频率最高的前5000个单词。

为了完成这一任务，第一时间想到的是[优先队列](http://algs4.cs.princeton.edu/24pq/)，但是 C++ 标准库中的函数 [priority_queue](http://en.cppreference.com/w/cpp/container/priority_queue) 并不能满足我们的需求（后文讲为什么），所以本文改进了普通的优先队列来进行高频词的统计。用 `Python` 写算法或者数据结构总是感觉怪怪的，所以本文采用 `C++`。

本文环境：

+ [C++ 11](https://en.wikipedia.org/wiki/C%2B%2B11)
+ [C++ boost filesystem module](http://www.boost.org/doc/libs/1_62_0/libs/filesystem/doc/index.htm)

## 数据存储方式

本文数据是已经完成中文分词，词之间用空格隔开，过滤掉标点符号的 `.txt` 文件，存放在同一个文件夹下。

![数据所在文件夹](https://raw.githubusercontent.com/hychn/hychn.github.io/master/img/voca_data.png)

![数据样式](https://raw.githubusercontent.com/hychn/hychn.github.io/master/img/voca_data2.png)

有一点要说明的是，因为任务的重点是统计词频，不涉及字符串编码转换，所以我们还是用 `srting` 来存储，但是对于大部分的汉字，如存储一个字 “闻”，`string` 对象的长度是**3**，也就是说需要三个字节存储一个汉字，但是这无关紧要，我们在代码中最多比较字符串是否相等。

## 读取文件夹中的文件

`C++ 11` 标准库中还没有类似 [Python OS](https://docs.python.org/2/library/os.html) 模块的部分，如果我们想要做，判断文件夹是否存在，列出某文件夹下所有文件等操作，就需要用到 [boost filesystem module](http://www.boost.org/doc/libs/1_38_0/libs/filesystem/doc/index.htm)。顺便一提，`C++ 17` 标准库已经有该[模块](http://en.cppreference.com/w/cpp/experimental/fs)了。

Mac 下安装 `boost` 推荐使用 [Homebrew](http://brew.sh/)。在国内 Homebrew 下载实在太慢了，终端走 socks5 代理推荐使用该[方法](http://www.jianshu.com/p/16d7275ec736)。这样还是太慢了，因为 boost 还是挺大的，再安利一款终端下载神器 [aria2](https://aria2.github.io/)。首先`brew install boost`  把 brew 要下载的文件链接复制下来然后 `control + c` 结束该命令，再用 aria2 把文件下载到  `/Users/UserName/Library/Caches/Homebrew` 文件下，然后运行 `brew install boost` ，Homebrew 会自动找到下载的文件解压安装。

## 代码实现

[优先队列](http://algs4.cs.princeton.edu/24pq/)在 [Sedgewick](http://www.cs.princeton.edu/~rs) 老先生的 *Algorithms 4th Edition* 一书中介绍的非常详细，本文不再详细介绍。

这是优先队列的存储结构：

![](https://raw.githubusercontent.com/hychn/hychn.github.io/master/img/voca_pq1.png)

这是优先队列的四个基本操作：

![](https://raw.githubusercontent.com/hychn/hychn.github.io/master/img/voca_pq2.png)

本文基本思路是，利用 `[string, frequncy]` 的结构（下文称之为  `token`）来存储数据，`string `  是单词，`frequency` 是该单词的词率。然后根据 `frequency` 的大小来决定 `token` 在堆中的位置。普通的优先队列不保存 `token` 在堆中的索引，如果某一单词已经存在堆中，要找到该单词增加 `frequency` 的最坏情况下需要遍历整个数组。

所以本文问利用哈希表为 `token` 保存一个索引，实时记录 `token` 在堆中的位置变化

```c++
#include <boost/filesystem.hpp>
#include <string>
#include <iostream>
#include <fstream>
#include <unordered_map>
#include <vector>
using namespace std;
using namespace boost::filesystem;

//定义结构体
struct token {
    string str;
    int num;
    token (string s, int n) {
        str = s;
        num = n;
    }
};

//利用pq数组储存
vector<token> pq;

//为每一个单词记录在pq中的位置
unordered_map<string, int> table;
int len = 0; // pq 的长度

//交换i,j元素，并更改索引
void exch (int i, int j) {
    table[pq[i].str] = j;
    table[pq[j].str] = i;
    token t = pq[i];
    pq[i] = pq[j];
    pq[j] = t;
}

//上浮操作
void swim (int i) {
    while (i > 1) {
        if (pq[i].num > pq[i/2].num)
            exch (i, i/2);
        else
            break;
        i = i/2;
    }
}

//下沉操作
void sink (int i) {
    while (i <= len/2) {
        int j = i*2;
        if (j+1 <= len && pq[j].num < pq[j+1].num)
            j = j+1;
        if (pq[i].num < pq[j].num)
            exch (i, j);
        else
            break;
        i = j;
    }
}

//添加字符串
void push (string s) {
  	//如果字符串已经存在，找到索引，更改词频，再更新位置
    if (table.count(s)) {
        int index = table[s];
        pq[index].num++;
        swim (index);
        sink (index);
        return;
    }
    len++;
    table[s] = len;
    pq.push_back(token (s, 1));
    swim (len);
}

//每次删除一个词频最大的token
token pop () {
    token t = pq[1];
    exch (1, len);
    sink (1);
    table.erase (t.str);
    pq.pop_back();
    len--;
    return t;
}


int main (int argc, char* argv[]) {
    std::ofstream fout ("vocabulary.out");
    if (argc < 2) {
        cout << "No Argument!" << endl;
        return 1;
    }
  	//利用argv[1]参数构建path对象
    path p (argv[1]);

  	//pq[0]不使用
    token t ("Start", -1);
    pq.push_back (t);
    //c++11 迭代器
  	for (directory_entry& x : directory_iterator(p)) {
		//x.path().string()是目录p下文件的绝对路径
      	std::ifstream fin (x.path().string());
        string pattern;
        while (fin >> pattern) {
          	//将所有单词添加到堆
            push (pattern);
        }
        fin.close();
    }
  
  	//输出单词和词频到目标文件
    int n = min (5000, len);
    while (n--) {
       t = pop();
       fout << t.str <<' ' << t.num << endl;
    }
    fout.close();
    return 0;
}

```

## 运行结果

```powershell
$ g++ -std=c++11 -lboost_filesystem -lboost_system vocabulary.cpp
$ ./a.out ~/Public/forums
$ less vocabulary.out
```

![](https://raw.githubusercontent.com/hychn/hychn.github.io/master/img/voca_re2.png)



