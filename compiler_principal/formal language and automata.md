[TOC]

# 形式语言与自动机

## 第二章 有穷自动机

### 一、确定型有穷接收器

2、已知 `Σ` = {a,b}，分别构造接受下面集合的`dfa`:

(a)、只有一个a的字符串

> <img src="./picture/形式语言与自动机-第二章-(一)2a.png" style="zoom:60%;" />

(e)、恰有两个a并且多于2个b的字符串。

> <img src="./picture/形式语言与自动机-第二章-(一)2e.png" style="zoom:60%;" />

5、构造分别接受下面语言的dfa：

(a)、L={ab^5^wb^2^ | w ∈ {a,b}^*^}

> <img src="./picture/形式语言与自动机-第二章-(一)5a.png" style="zoom:67%;" />

(b)、L={w~1~abw~2~ | w~1~∈{a,b}^*^,w~2~∈{a,b}^*^}

> <img src="./picture/形式语言与自动机-第二章-(一)5b.png" style="zoom:60%;" />

6、描述下列自动机接受的语言集合

<img src="./picture/形式语言与自动机-第二章-(一)6.png" style="zoom:60%;" />

> 解答：L = {a}∪{aw~1~b | w~1~∈{a,b}^*^}}

7、在Σ={a,b}上构造分别接受下面语言的dfa:

(b) L={w | |w| mod 5 ≠ 0} 

> <img src="./picture/形式语言与自动机-第二章-(一)7b.png" style="zoom:50%;" />

(c) L={w | n~a~(w) mod 3 > 1} 

> <img src="./picture/形式语言与自动机-第二章-(一)7c.png" style="zoom:60%;" />

(e) L={w | (n~a~(w) - n~b~(w)) mod 3 > 0}

> ==公式：(a - b) % p = (a % p - b % p + p) % p==
> <img src="./picture/形式语言与自动机-第二章-(一)7e.png" style="zoom:60%;" />

(f) L={w | (n~a~(w) + 2*n~b~(w)) mod 3 < 2}

> ==公式：(a + b) % p = (a % p + b % p) % p==
>
> <img src="./picture/形式语言与自动机-第二章-(一)7f.png" style="zoom:60%;" />

8、一个符号串上的`“run”`指的是符号串中包含由至少两个相同符号构成的子串。比如，符号串`abbbaab`包含一个符号`b`的长度为`3`的`“run”`和一个符号`a`的长度为`2`的`“run”`。在 Σ = {a,b}，中，分别构造下列语言的dfa：

(a) L = {w | w不包含长度小于4的“run”} => ==w中包含的 |run| ≥ 4==

> <img src="./picture/形式语言与自动机-第二章-(一)-8a.png" style="zoom:60%;" />

(d) L={w | 每个符号串就包含两个长度为3的“run”}

> <img src="./picture/形式语言与自动机-第二章-(一)8d.png" style="zoom:80%;" />

11、证明：L={vwv | v，w∈{a,b},|v| = 2}是正则语言

> <img src="./picture/形式语言与自动机-第二章(一)11.png" style="zoom:60%;" />

四、减少有穷自动机中状态的化简

> 定义2.8 对于所有的w ∈Σ^*^，如果
>
> δ^*^(p,w) ∈ F ∩ δ^*^(q,w) ∈ F ∩ δ^*^(p,w) ∉ F ∩ δ^*^(q,w) ∉ F
>
> 则称 dfa的两个状态`p`和`q`是==不可区分的==。反之，如果
>
> ∃ w ∈Σ^*^   δ^*^(p,w) ∉ F 并且 δ^*^(q,w) ∈ F，则`p`和`q`==是可区分的==

> 程序mark：
>
> 1、去掉所有==不可达状态==
>
> 2、考察所有状态对(p,q)。判断是否可区分
>
> 3、重复下述过程：
>
> 对于`∀(p,q)∧∀a ∈Σ`，计算δ(p,a) =p~a~和δ(q,a) =p~b~，如果(p~a~,p~b~)是可区分的，那么(p,q)也标记为可区分

2、寻找接受下面语言的最小dfa

(a) L={a^n^b^m^ | n≥2，m≥1}

> <img src="./picture/形式语言与自动机-第二章-(四)2a.png" style="zoom:60%;" />

(b) L={a^n^b| n≥0}∪{b^n^a | n≥1}

> <img src="./picture/形式语言与自动机-第二章-(四)2b.png" style="zoom:60%;" />

## 第三章 正则语言与正则文法
