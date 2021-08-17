---
title: 前缀树
date: 2021-08-12T15:26:23+08:00
hero: head.svg
menu:
  sidebar:
    name: 前缀树
    identifier: algorithm-trie
    parent: algorithm
    weight: 4002
---

---

![Golang](https://img.shields.io/badge/-Golang-blue)
![算法](https://img.shields.io/badge/-%E7%AE%97%E6%B3%95-green)

---

> 前缀树又称字典树

## Trie的应用

1. 自动补全,例如：在百度搜索的输入框中,输入一个单词的前半部分,能够自动补全出可能的单词结果。
2. 拼写检查，例如：在word中输入一个拼写错误的单词, 能够自动检测出来。
3. IP路由表，在IP路由表中进行路由匹配时, 要按照最长匹配前缀的原则进行匹配。
4. T9预测文本，在大多手机输入法中, 都会用9格的那种输入法. 这个输入法能够根据用户在9格上的输入,自动匹配出可能的单词。
5. 填单词游戏，相信大多数人都玩过那种在横竖的格子里填单词的游戏。
