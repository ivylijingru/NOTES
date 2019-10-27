## TCS chapter 2

歧义性是非确定性的来源

### 2.18

消除歧义 

$S \rightarrow TS\ |\ T \\T \rightarrow aTb\ |\ ab$

### 2.19

$RC(A)=\{yx\ |\ xy\in A\}$

如何处理栈的问题？如果 x 已经堆栈，y 需要弹出。但以 y 开始时栈为空。这种情况下把符号压进去

CNF：先把文法变成乔姆斯基范式，然后利用上节课的引理、推论。

### 2.20

$A = \{2^n0^n1^m3^m\}\\ CUT(A)\cap\{0^*1^*2^*3^*\}=\{0^n1^m2^n3^m\}$

如果 CUT 封闭，那么交上正则语言后得到的也应该上下文无关，矛盾。

### 2.31

$\overline{L(G)} = \{a^nb^n|n\ge 0\}$

### 2.32

$A/B = \{x|\exist y,xy\in A, y\in B\}$

| A/B  | A    | B            |
| ---- | ---- | ------------ |
| REG  | REG  | all language |
| CFL  | CFL  | REG          |
| CFL  | CFL  | CFL(?)       |

?: 一个栈不能模拟两个栈（一个栈能同时模拟 DFA 和栈吗？）

### 2.33

$\{x|x\in \{a,b\}^*, |x|_a=2|x|_b\}\\if\ |x|_a = |x|_b:S\rightarrow aSb|bSa|SS \\\because\ abba\ could\ not\ be \ generated\ without\ SS\\ S\to aSaSbS|aSbSaS|bSaSaS|\epsilon$

技巧：列出所有情况，往里面加 S

### 2.34

$C = \{x\#y\ |\ x\neq y\}\\ S_0\to S1K\\ S\to \Sigma S\Sigma\ |\ T\\T\to 0K\# \\ K\to 0K\ |\ 1K\ |\ \epsilon\\ \Sigma\to 0|1$

（补上交换0，1位置的情况，我的方法。。上课讲的方法是：）

$S\to \hat{S_0}\hat{S_1}|\hat{S_1}\hat{S_0}\\ \hat{S_0}\to \hat{C}S_0C|C\hat{S_0}C|CS_0\hat{C}\\C\to 0|1\\ \hat{C}\to 0\#|\#0|1\#|\#1$

其中S_0的定义和2.35相同。想法是在2.35中任意位置加一个井号都应该是满足的。

### 2.35

$D=\{xy\ |\ |x|=|y|, x\neq y\}\\ S\to S_0S_1|S_1S_0\\ S_0 \to \Sigma S_0\Sigma |0\\S_1\to \Sigma S_1\Sigma |1$

### 2.36

$E=\{a^ib^j\ |\ i\neq j\and2i\neq j\}\\ 1.\ j<i\\2. \ i<j<2i\\ 3.\ j>2i \\1.\  S\to aSb|T, T\to aT \\2.\ T\to aaTb|aaSb, S\to aSb|ab\\ 3.S\to aaSb|T, T\to aT $

### 2.37

左旋时保留一半，右旋时保留两个。（修改2.19中的文法变换方法）

$A\to BC\\\hat{C}\to \hat{A}\\ \hat{B}\to C\hat{A}$

### 2.40

$(a)\ \{w|w=uv, \forall u,v, |u|_a\ge|u|_b\}$

$S\to TaS|T \\T\to aTbT|\epsilon$

$(b)\ \{w||w|_a=|w|_b\}$

$S\to aBS|bAS|\epsilon\\A\to a|bAA\\B\to b|aBB$

配对关系清楚。

$\\(c)\ \{w||w|_a\ge |w|_b\}$

$S\to S_{(b)}aS_{(a)}|S_{(b)}$

中间的 a 就是栈底出不来的 a。

### 2.41

如果不是固有歧义，

$a^{2p!}b^{2p!}c^{3p!}$ 和 $a^{3p!}b^{2p!}c^{2p!}$ 都能泵出 $a^{3p!}b^{3p!}c^{3p!}$。如果非歧义则这个串由一颗分析树生成，ab bc中各存在一个泵；如果都泵的话，将有 x 个 a，x+y  个 b，y 个 c。这个串不在文法当中。

### 2.42

$(a)\ 0^n1^n0^n1^n$ 每次泵都会落在相邻的两类。

$(d)\ t_1 = t_2$，泵 t_1

### 2.43

回文，个数相等分别都上下文无关；其交不上下文无关。泵引理能控制泵的位置，从开头处泵导致0变多，从中间泵导致1变多。$0^p1^{2p}0^p$

### 2.45

$a^pb^{(p+1)(p+2)...(2p-1)+1}\\wilson:(p-1)!=1(mod\ p)\\p+k(0<k<p)\nmid (p+1)...(2p-1)+k+1$

### 2.46

$0^*\#0^*\#0^*\cup 0^n\#0^{2n}$，最小泵长度时4

### 2.47

b 个变元乔姆斯基范式树，如果树高为 k，则最多有 $2^k-1$ 个内层节点。如果派生至少用了 $2^b$ 步，则至少有 $2^b$ 个内层节点。则树高至少为 $b+1$。存在一条从根到树叶，包含 $b+1$ 个变元的路径。由抽屉原理，存在一个变元出现了两次。那么类似泵引理证明，可以构造无穷个在 $L(G)$ 中的串。

### 2.50

PERFECT SHUFFLE：一张插一张；

### 2.51

SHUFFLE：一摞插一摞；通过语言交上 $(0a)*(0b)*(1b)*$ 变成 Perfect shuffle

### 2.52

$ab^kcd^ke,\ b\neq\epsilon 时,ab^*正则；b=\epsilon时，acd^* 正则.$