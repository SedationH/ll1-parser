## 整体设计

![image-20201115143705239](http://picbed.sedationh.cn/image-20201115143705239.png)

## Fisrt

### 算法理论

![image-20201115153818417](http://picbed.sedationh.cn/image-20201115153818417.png)

### IMPLEMENT

值得注意的是

`const first = (firstSets[symbol] = {})`这里的设计 刚好实现了对递归遍历过程中，所有不管是终结符号还是非终结符号的 first 集计算

first 变量用于方便 firstSet 当前 symbol 值内容的获取与赋值

firstOf 是个自我递归函数，其返回当前 firstOf(symbol) 对应的 first 集合

firstOf 会通过 getProductionsForSymbol(X) 来寻找每个 grammer 首字母含有 X 的产生式子 productionsForSymbol

在所有的 productionsForSymbol 获取 getRHS(productionsForSymbol[k]) 产生式的右部 production

## Follow

```js
// 几种可能出现的情况 默认都在求follow(B)
//   1. A -> aB 那么要求follow(A)
//   2. A -> aBC 求follow(C)
//   如果非终结符号 C | A 的first集合中没有ε 则 follow(B) = first(A) | first(C)
//   如果有ε 需要接着往下找
//   A -> Bz 直接z  因为上面first对于非终结符号的处理也有，所以也和上面的操作一致了
//   对于开始符号的处理进行了特定判断
```

## IMPLEMENT

没啥好说的

注意下 在 B -> aB 中对可能造成的死循环跳出就好了

## 结果示例

```zsh
$ node index.js
Grammar:

   S -> aBc
   S -> bAB
   A -> aAb
   A -> b
   B -> b
   B -> ε

First sets: 

   S : [ 'a', 'b' ]
   a : [ 'a' ]
   b : [ 'b' ]
   A : [ 'a', 'b' ]
   B : [ 'b', 'ε' ]
   ε : [ 'ε' ]
   c : [ 'c' ]

Follow sets: 

   S : [ '$' ]
   A : [ 'b', '$' ]
   B : [ 'c', '$' ]

NonTerminals: S,A,B
Terminals: a,c,b,ε
┌───┬──────────┬────────┬──────────┬───┬────────┐
│   │ a        │ c      │ b        │ ε │ $      │
├───┼──────────┼────────┼──────────┼───┼────────┤
│ S │ S -> aBc │        │ S -> bAB │   │        │
├───┼──────────┼────────┼──────────┼───┼────────┤
│ A │ A -> aAb │        │ A -> b   │   │        │
├───┼──────────┼────────┼──────────┼───┼────────┤
│ B │          │ B -> ε │ B -> b   │   │ B -> ε │
└───┴──────────┴────────┴──────────┴───┴────────┘
┌──────┬──────────────┬────────────┬──────────┐
│ Step │ AnalyseStack │ RemainText │ Action   │
├──────┼──────────────┼────────────┼──────────┤
│ 0    │ $,S          │ babb$      │ S -> bAB │
├──────┼──────────────┼────────────┼──────────┤
│ 1    │ $,B,A,b      │ babb$      │ Match: b │
├──────┼──────────────┼────────────┼──────────┤
│ 2    │ $,B,A        │ abb$       │ A -> aAb │
├──────┼──────────────┼────────────┼──────────┤
│ 3    │ $,B,b,A,a    │ abb$       │ Match: a │
├──────┼──────────────┼────────────┼──────────┤
│ 4    │ $,B,b,A      │ bb$        │ A -> b   │
├──────┼──────────────┼────────────┼──────────┤
│ 5    │ $,B,b,b      │ bb$        │ Match: b │
├──────┼──────────────┼────────────┼──────────┤
│ 6    │ $,B,b        │ b$         │ Match: b │
├──────┼──────────────┼────────────┼──────────┤
│ 7    │ $,B          │ $          │ B -> ε   │
├──────┼──────────────┼────────────┼──────────┤
│ 8    │ $            │ $          │ Match: $ │
└──────┴──────────────┴────────────┴──────────┘
匹配成功🀄️
```



### 示例1

## 💡想法

first集和follow的集的设计逻辑都很好理解

编程的难点在于

1. 如何处理ε的出现  -> 执行细节处理
2. 如何处理非终结符  -> 递归

再通过first集和follow集来构建分析表格，数据结构：

​	 \[非终结符][终结符]  -> How to handle? 也就是选择哪一个产生式

因为是ll(1) must handle method is only one so we can use it to definitively estimate a string is ok or not



封装📦屏蔽细节处理，让🤔思考专注于难点。

## Refer

https://github.com/SinaKarimi7/LL1-Parser.git

参考了项目的整体结构，使用 EMS 进行了重组，使结构更加清晰

原 First 集合求取和所理解的不一致，进行了修改



