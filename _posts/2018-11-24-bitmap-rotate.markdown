---
layout: post
title : "从 bit 反转到 2D bitmap 旋转反射"
author: "siahuat0727"
catalog: true
tags:
    - 2D bitmap rotate
    - 2D bitmap reflex
    - 2D 位图旋转
    - 2D 位图反射
---

## 前言

之前写黑白棋 AI，棋盘的 data structure 用的是 bitmap（2 个 64-bit 分别储存黑棋与白棋的分布），在加速搜索的过程中，用了 transposition table 减少重复运算。
而若将某次搜索结果通过**棋盘旋转、反射**等记录在 transposition table 中，可进一步减少重复的运算。
当时想要快速进行这些旋转、反射，搜了各种关键字，什么 rotate 2D bitmap， rotate bit matrix， 位图旋转，位 2D 旋转竟然都找不到相关资料。
最后终于在 [Nugnikoll 的 GitHub repository](https://github.com/Nugnikoll/MyReversi/blob/3fc9edf19f838e77bcdf03ae226add1e70422c5d/cpp/reversi.h#L253)（目前 Botzone 黑白棋冠军）找到对应的实现：
```cpp
static void rotate_r(ull& brd){
    brd = (brd & 0xf0f0f0f000000000) >> 4  | (brd & 0x0f0f0f0f00000000) >> 32
        | (brd & 0x00000000f0f0f0f0) << 32 | (brd & 0x000000000f0f0f0f) << 4;
    brd = (brd & 0xcccc0000cccc0000) >> 2  | (brd & 0x3333000033330000) >> 16
        | (brd & 0x0000cccc0000cccc) << 16 | (brd & 0x0000333300003333) << 2;
    brd = (brd & 0xaa00aa00aa00aa00) >> 1  | (brd & 0x5500550055005500) >> 8
        | (brd & 0x00aa00aa00aa00aa) << 8  | (brd & 0x0055005500550055) << 1;
}
```
理解其原理后写了这篇文章。
找不到或许只是我不知道这动作的专业术语，若有缘人知道欢迎留言告诉啦！

## 内容

### Bit 反转
先从 bit 反转说起，[8 bit 反转](https://stackoverflow.com/questions/2602823/in-c-c-whats-the-simplest-way-to-reverse-the-order-of-bits-in-a-byte)可以这么做：
```cpp
unsigned char reverse(unsigned char b) {
   b = (b & 0xAA) >> 1 | (b & 0x55) << 1;
   b = (b & 0xCC) >> 2 | (b & 0x33) << 2;
   b = (b & 0xF0) >> 4 | (b & 0x0F) << 4;
   return b;
}
```
8-bit 位置变化： $b_7b_6b_5b_4b_3b_2b_1b_0$  $\Rightarrow$ $b_0b_1b_2b_3b_4b_5b_6b_7$
eg. $11011010_2$  $\Rightarrow$  $01011011_2$

过程：

$b_7\ b_6\ b_5\ b_4\ b_3\ b_2\ b_1\ b_0$

以 1 bit 为单位，相邻两个互换：

$b_6\ b_7\ b_4\ b_5\ b_2\ b_3\ b_0\ b_1$

两两一组，相邻两组互换：

$b_4\ b_5\ b_6\ b_7\ b_0\ b_1\ b_2\ b_3$

四四一组，相邻两组互换：

$b_0\ b_1\ b_2\ b_3\ b_4\ b_5\ b_6\ b_7$

完成！

---

这件事情其实也可以这么分析，绿色数字表示该数与目标位置的距离，向右取正：


$\begin{matrix}
\small\color{forestgreen}{+7}  & \small\color{forestgreen}{+5}  & \small\color{forestgreen}{+3}  & \small\color{forestgreen}{+1}  & \small\color{forestgreen}{-1}  & \small\color{forestgreen}{-3}  & \small\color{forestgreen}{-5}  & \small\color{forestgreen}{-7}  \\\
b_7 & b_6 & b_5 & b_4 & b_3 & b_2 & b_1 & b_0
\end{matrix}$

分解目标距离：

$\begin{matrix}
\tiny\color{forestgreen}{+4}  & \tiny\color{forestgreen}{+4}  & \tiny\color{forestgreen}{+4}  & \tiny\color{forestgreen}{+4}  & \tiny\color{forestgreen}{-4}  & \tiny\color{forestgreen}{-4}  & \tiny\color{forestgreen}{-4}  & \tiny\color{forestgreen}{-4}\\\
\tiny\color{forestgreen}{+2}  & \tiny\color{forestgreen}{+2}  & \tiny\color{forestgreen}{-2}  & \tiny\color{forestgreen}{-2}  & \tiny\color{forestgreen}{+2}  & \tiny\color{forestgreen}{+2}  & \tiny\color{forestgreen}{-2}  & \tiny\color{forestgreen}{-2}\\\
\tiny\color{forestgreen}{+1}  & \tiny\color{forestgreen}{-1}  & \tiny\color{forestgreen}{+1}  & \tiny\color{forestgreen}{-1}  & \tiny\color{forestgreen}{+1}  & \tiny\color{forestgreen}{-1}  & \tiny\color{forestgreen}{+1}  & \tiny\color{forestgreen}{-1}\\\
b_7 & b_6 & b_5 & b_4 & b_3 & b_2 & b_1 & b_0
\end{matrix}$


#### 第一次位移：


$\begin{matrix}
\tiny\color{forestgreen}{+4}  & \tiny\color{red}{+4}  & \tiny\color{forestgreen}{+4}  & \tiny\color{red}{+4}  & \tiny\color{forestgreen}{-4}  & \tiny\color{red}{-4}  & \tiny\color{forestgreen}{-4}  & \tiny\color{red}{-4}\\\
\tiny\color{forestgreen}{+2}  & \tiny\color{red}{+2}  & \tiny\color{forestgreen}{-2}  & \tiny\color{red}{-2}  & \tiny\color{forestgreen}{+2}  & \tiny\color{red}{+2}  & \tiny\color{forestgreen}{-2}  & \tiny\color{red}{-2}\\\
\small\color{forestgreen}{+1}  & \small\color{red}{-1}  & \small\color{forestgreen}{+1}  & \small\color{red}{-1}  & \small\color{forestgreen}{+1}  & \small\color{red}{-1}  & \small\color{forestgreen}{+1}  & \small\color{red}{-1}\\\
\color{forestgreen}{b_7} & \color{red}{b_6} & \color{forestgreen}{b_5} & \color{red}{b_4} & \color{forestgreen}{b_3} & \color{red}{b_2} & \color{forestgreen}{b_1} & \color{red}{b_0}
\end{matrix}$

$$\small\color{forestgreen}{+1}$$ mask：$$10101010_2 \Rightarrow AA_{16}$$
$$\small\color{red}{-1}$$ mask：$$01010101_2 \Rightarrow 55_{16}$$



经过 `b = (b & 0xAA) >> 1 | (b & 0x55) << 1` ：

$\begin{matrix}
\tiny\color{red}{+4}  & \tiny\color{forestgreen}{+4}  & \tiny\color{red}{+4}  & \tiny\color{forestgreen}{+4}  & \tiny\color{red}{-4}  & \tiny\color{forestgreen}{-4}  & \tiny\color{red}{-4}  & \tiny\color{forestgreen}{-4}\\\
\tiny\color{red}{+2}  & \tiny\color{forestgreen}{+2}  & \tiny\color{red}{-2}  & \tiny\color{forestgreen}{-2}  & \tiny\color{red}{+2}  & \tiny\color{forestgreen}{+2}  & \tiny\color{red}{-2}  & \tiny\color{forestgreen}{-2}\\\
\color{red}{b_6} & \color{forestgreen}{b_7} & \color{red}{b_4} & \color{forestgreen}{b_5} & \color{red}{b_2} & \color{forestgreen}{b_3} & \color{red}{b_0} & \color{forestgreen}{b_1}
\end{matrix}$

#### 第二次位移：

$\begin{matrix}
\tiny\color{forestgreen}{+4}  & \tiny\color{forestgreen}{+4}  & \tiny\color{red}{+4}  & \tiny\color{red}{+4}  & \tiny\color{forestgreen}{-4}  & \tiny\color{forestgreen}{-4}  & \tiny\color{red}{-4}  & \tiny\color{red}{-4}\\\
\small\color{forestgreen}{+2}  & \small\color{forestgreen}{+2}  & \small\color{red}{-2}  & \small\color{red}{-2}  & \small\color{forestgreen}{+2}  & \small\color{forestgreen}{+2}  & \small\color{red}{-2}  & \small\color{red}{-2}\\\
\color{forestgreen}{b_6} & \color{forestgreen}{b_7} & \color{red}{b_4} & \color{red}{b_5} & \color{forestgreen}{b_2} & \color{forestgreen}{b_3} & \color{red}{b_0} & \color{red}{b_1}
\end{matrix}$

$$\small\color{forestgreen}{+2}$$ mask：$$11001100_2 \Rightarrow CC_{16}$$
$$\small\color{red}{-2}$$ mask：$$00110011_2 \Rightarrow 33_{16}$$

经过 `b = (b & 0xCC) >> 2 | (b & 0x33) << 2` ：

$\begin{matrix}
\tiny\color{red}{+4}  & \tiny\color{red}{+4}  & \tiny\color{forestgreen}{+4}  & \tiny\color{forestgreen}{+4}  & \tiny\color{red}{-4}  & \tiny\color{red}{-4}  & \tiny\color{forestgreen}{-4}  & \tiny\color{forestgreen}{-4}\\\
\color{red}{b_4} & \color{red}{b_5} & \color{forestgreen}{b_6} & \color{forestgreen}{b_7} & \color{red}{b_0} & \color{red}{b_1} & \color{forestgreen}{b_2} & \color{forestgreen}{b_3}
\end{matrix}$

#### 第三次位移：

$\begin{matrix}
\small\color{forestgreen}{+4}  & \small\color{forestgreen}{+4}  & \small\color{forestgreen}{+4}  & \small\color{forestgreen}{+4}  & \small\color{red}{-4}  & \small\color{red}{-4}  & \small\color{red}{-4}  & \small\color{red}{-4}\\\
\color{forestgreen}{b_4} & \color{forestgreen}{b_5} & \color{forestgreen}{b_6} & \color{forestgreen}{b_7} & \color{red}{b_0} & \color{red}{b_1} & \color{red}{b_2} & \color{red}{b_3}
\end{matrix}$

$$\small\color{forestgreen}{+4}$$ mask：$$11110000_2 \Rightarrow F0_{16}$$
$$\small\color{red}{-4}$$ mask：$$00001111_2 \Rightarrow 0F_{16}$$

经过 `b = (b & 0xF0) >> 4 | (b & 0x0F) << 4` ：

$\begin{matrix}
\color{red}{b_0} & \color{red}{b_1} & \color{red}{b_2} & \color{red}{b_3} & \color{forestgreen}{b_4} & \color{forestgreen}{b_5} & \color{forestgreen}{b_6} & \color{forestgreen}{b_7}
\end{matrix}$

另外，通过这种分解，可以发现每一次的位移都不会影响到每个位置的目标距离，也就是说，**三次位移的顺序任意对调，是不会影响结果的。**

### 2D bitmap 旋转

回顾 bit 反转，我们可以先以最小单位（1-bit）为一组两两进行反转，再以次小单位(2-bit)为一组两两进行反转，……，最终得到整个 bit 反转的结果。

对于 2D bitmap 旋转，同样的特性还是存在的。
下图显示 2D 顺时旋转 90$^\circ$ 的结果：

| ![](https://i.imgur.com/YgMVxoh.png) | <br><br><br><br> $\Rightarrow$ | ![](https://i.imgur.com/PRm8oyK.png) |

分解过程：
小单位（4 分之 1 原图）自行顺时旋转 90$^\circ$，再视小单位为一体进行顺时旋转 90$^\circ$。

| ![](https://i.imgur.com/9ZS3ygS.png) | <br><br><br><br> $\Rightarrow$ | ![](https://i.imgur.com/DMjx93c.png) | <br><br><br><br> $\Rightarrow$ | ![](https://i.imgur.com/4LNwQds.png) |

而第一步的自行顺时旋转若不是最小旋转单位（2\*2 bitmap）则一直分解下去。

以下以 16-bit（4\*4 bitmap）顺时旋转 90 度为例。

16-bit 位置变化： $b_Fb_Eb_Db_Cb_Bb_Ab_9b_8b_7b_6b_5b_4b_3b_2b_1b_0 \Rightarrow  b_3b_7b_Bb_Fb_2b_6b_Ab_Eb_1b_5b_9b_Db_0b_4b_8b_C$
eg. $1111101111001111_2 \Rightarrow 111111011011101_2$

图解：
$\begin{matrix}
b_F & b_E & b_D & b_C &             & b_3 & b_7 & b_B & b_F  \\\
b_B & b_A & b_9 & b_8 & \Rightarrow & b_2 & b_6 & b_A & b_E  \\\
b_7 & b_6 & b_5 & b_4 &             & b_1 & b_5 & b_9 & b_D  \\\
b_3 & b_2 & b_1 & b_0 &             & b_0 & b_4 & b_8 & b_C  \\\
\end{matrix}$



观察其目标距离：

$\begin{matrix}
\small\color{forestgreen}{+3}  &             & \small\color{forestgreen}{+6}  & & \small\color{forestgreen}{+9}  &             & \small\color{forestgreen}{+12}\\\
{b_F} & & {b_E} & & {b_D} & & {b_C} \\\
\\\
\small\color{forestgreen}{-2}  &             & \small\color{forestgreen}{+1}  & & \small\color{forestgreen}{+4}  &             & \small\color{forestgreen}{+7}\\\
{b_B} & & {b_A} & & {b_9} & & {b_8} \\\
\\\
\small\color{forestgreen}{-7}  &             & \small\color{forestgreen}{-4}  & & \small\color{forestgreen}{-1}  &             & \small\color{forestgreen}{+2}\\\
{b_7} & & {b_6} & & {b_5} & & {b_4} \\\
\\\
\small\color{forestgreen}{-12}  &             & \small\color{forestgreen}{-9}  & & \small\color{forestgreen}{-6}  &             & \small\color{forestgreen}{-3}\\\
{b_3} & & {b_2} & & {b_1} & & {b_0} \\\
\end{matrix}$

分解目标距离：（由上分解过程可以得出此分解结果）

$\begin{matrix}
\tiny\color{forestgreen}{+2}  &             & \tiny\color{forestgreen}{+2}  & & \tiny\color{forestgreen}{+8}  &             & \tiny\color{forestgreen}{+8}\\\
\tiny\color{forestgreen}{+1}  &             & \tiny\color{forestgreen}{+4}  & & \tiny\color{forestgreen}{+1}  &             & \tiny\color{forestgreen}{+4}\\\
{b_F} & & {b_E} & & {b_D} & & {b_C} \\\
\\\
\tiny\color{forestgreen}{+2} & & \tiny\color{forestgreen}{+2}  & & \tiny\color{forestgreen}{+8} & & \tiny\color{forestgreen}{+8} & \\\
\tiny\color{forestgreen}{-1} & & \tiny\color{forestgreen}{-4}  &             & \tiny\color{forestgreen}{-1}  & & \tiny\color{forestgreen}{-4}  \\\
{b_B} & & {b_A} & & {b_9} & & {b_8} \\\
\\\
\tiny\color{forestgreen}{-8}  &             & \tiny\color{forestgreen}{-8}  & & \tiny\color{forestgreen}{-2}  &             & \tiny\color{forestgreen}{-2}\\\
\tiny\color{forestgreen}{+1}  &             & \tiny\color{forestgreen}{+4}  & & \tiny\color{forestgreen}{+1}  &             & \tiny\color{forestgreen}{+4}\\\
{b_7} & & {b_6} & & {b_5} & & {b_4} \\\
\\\
\tiny\color{forestgreen}{-8}  & & \tiny\color{forestgreen}{-8}  & & \tiny\color{forestgreen}{-2}  & & \tiny\color{forestgreen}{-2}\\\
\tiny\color{forestgreen}{-4}  &             & \tiny\color{forestgreen}{-1}  & & \tiny\color{forestgreen}{-4}  &             & \tiny\color{forestgreen}{-1}\\\
{b_3} & & {b_2} & & {b_1} & & {b_0} \\\
\end{matrix}$

#### 第一次位移：

$\begin{matrix}
\tiny\color{forestgreen}{+2}  &             & \tiny\color{red}{+2}  & & & \tiny\color{forestgreen}{+8}  &             & \tiny\color{red}{+8}\\\
\small\color{forestgreen}{+1}  &             & \small\color{red}{+4}  & & & \small\color{forestgreen}{+1}  &             & \small\color{red}{+4}\\\
\color{forestgreen}{b_F} & \rightarrow & \color{red}{b_E} & & & \color{forestgreen}{b_D} & \rightarrow & \color{red}{b_C} \\\
\uparrow & & \downarrow & & & \uparrow & & \downarrow\\\
\tiny\color{orange}{+2}  &             & \tiny\color{blue}{+2}  & & & \tiny\color{orange}{+8}  &             & \tiny\color{blue}{+8}\\\
\small\color{orange}{-4}  &             & \small\color{blue}{-1}  & & & \small\color{orange}{-4}  &             & \small\color{blue}{-1}\\\
\color{orange}{b_B} & \leftarrow  & \color{blue}{b_A} & & & \color{orange}{b_9} & \leftarrow  & \color{blue}{b_8} \\\
\\\
\tiny\color{forestgreen}{-8}  &             & \tiny\color{red}{-8}  & & & \tiny\color{forestgreen}{-2}  &             & \tiny\color{red}{-2}\\\
\small\color{forestgreen}{+1}  &             & \small\color{red}{+4}  & & & \small\color{forestgreen}{+1}  &             & \small\color{red}{+4}\\\
\color{forestgreen}{b_7} & \rightarrow & \color{red}{b_6} & & & \color{forestgreen}{b_5} & \rightarrow & \color{red}{b_4} \\\
\uparrow & & \downarrow & & & \uparrow & & \downarrow\\\
\tiny\color{orange}{-8}  &             & \tiny\color{blue}{-8}  & & & \tiny\color{orange}{-2}  &             & \tiny\color{blue}{-2}\\\
\small\color{orange}{-4}  &             & \small\color{blue}{-1}  & & & \small\color{orange}{-4}  &             & \small\color{blue}{-1}\\\
\color{orange}{b_3} & \leftarrow  & \color{blue}{b_2} & & & \color{orange}{b_1} & \leftarrow  & \color{blue}{b_0} \\\
\end{matrix}$

$$\small\color{forestgreen}{+1}$$ mask：$$1010\_0000\_1010\_0000_2 \Rightarrow A0A0_{16}$$
$$\small\color{red}{+4}$$ mask：$$0101\_0000\_0101\_0000_2 \Rightarrow 5050_{16}$$
$$\small\color{orange}{-4}$$ mask：$$0000\_1010\_0000\_1010_2 \Rightarrow 0A0A_{16}$$
$$\small\color{blue}{-1}$$ mask：$$0000\_0101\_0000\_0101_2 \Rightarrow 0505_{16}$$

经过 `b = (b & 0xA0A0) >> 1 | (b & 0x5050) >> 4 | (b & 0x0A0A) << 4 | (b & 0x0505) << 1`：



$\begin{matrix}
\tiny\color{orange}{+2}  &             & \tiny\color{forestgreen}{+2}  & & \tiny\color{orange}{+8}  &             & \tiny\color{forestgreen}{+8}\\\
\color{orange}{b_B} &             & \color{forestgreen}{b_F} & & \color{orange}{b_9} &             & \color{forestgreen}{b_D} \\\
\\\
\tiny\color{blue}{+2}  &             & \tiny\color{red}{+2}  & & \tiny\color{blue}{+8}  &             & \tiny\color{red}{+8} \\\
\color{blue}{b_A} &             & \color{red}{b_E} & & \color{blue}{b_8} &             & \color{red}{b_C} \\\
\\\
\tiny\color{orange}{-8}  &             & \tiny\color{forestgreen}{-8}  & & \tiny\color{orange}{-2}  &             & \tiny\color{forestgreen}{-2}\\\
\color{orange}{b_3} &             & \color{forestgreen}{b_7} & & \color{orange}{b_1} &             & \color{forestgreen}{b_5} \\\
\\\
\tiny\color{blue}{-8}  &             & \tiny\color{red}{-8}  & & \tiny\color{blue}{-2}  &             & \tiny\color{red}{-2} \\\
\color{blue}{b_2} &             & \color{red}{b_6} & & \color{blue}{b_0} &             & \color{red}{b_4} \\\
\end{matrix}$


#### 第二次位移：

$\begin{matrix}
\small\color{forestgreen}{+2} & & \small\color{forestgreen}{+2} & & \small\color{red}{+8} & & \small\color{red}{+8}\\\
\color{green}{b_B} & & \color{green}{b_F} & & \color{red}{b_9} & & \color{red}{b_D} \\\
 & & & \rightarrow\\\
\small\color{forestgreen}{+2} & & \small\color{forestgreen}{+2} & & \small\color{red}{+8} & & \small\color{red}{+8}\\\
\color{green}{b_A} & & \color{green}{b_E} & & \color{red}{b_8} & & \color{red}{b_C} \\\
 & \uparrow & & & & \downarrow\\\
\small\color{orange}{-8} & & \small\color{orange}{-8} & & \small\color{blue}{-2} & & \small\color{blue}{-2}\\\
\color{orange}{b_3} & & \color{orange}{b_7} & & \color{blue}{b_1} & & \color{blue}{b_5} \\\
 & & & \leftarrow\\\
\small\color{orange}{-8} & & \small\color{orange}{-8} & & \small\color{blue}{-2} & & \small\color{blue}{-2}\\\
\color{orange}{b_2} & & \color{orange}{b_6} & & \color{blue}{b_0} & & \color{blue}{b_4} \\\
\end{matrix}$

$$\small\color{forestgreen}{+2}$$ mask：$$1100\_1100\_0000\_0000_2 \Rightarrow CC00_{16}$$
$$\small\color{red}{+8}$$ mask：$$0011\_0011\_0000\_0000_2 \Rightarrow 3300_{16}$$
$$\small\color{orange}{-8}$$ mask：$$0000\_0000\_1100\_1100_2 \Rightarrow 00CC_{16}$$
$$\small\color{blue}{-2}$$ mask：$$0000\_0000\_0011\_0011_2 \Rightarrow 0033_{16}$$

经过 `b = (b & 0xCC00) >> 2 | (b & 0x3300) >> 8 | (b & 0x00CC) << 8 | (b & 0x0033) << 2`：


$\begin{matrix}
\color{orange}{b_3} &             & \color{orange}{b_7} & & \color{green}{b_B} &             & \color{green}{b_F} \\\
\\\
\color{orange}{b_2} &             & \color{orange}{b_6} & & \color{green}{b_A} &             & \color{green}{b_E}\\\
\\\
\color{blue}{b_1} &             & \color{blue}{b_5} & & \color{red}{b_9} &             & \color{red}{b_D} \\\
\\\
\color{blue}{b_0} &             & \color{blue}{b_4} & & \color{red}{b_8} &             & \color{red}{b_C} \\\
\end{matrix}$

2D 顺时旋转 90$^\circ$ 完成！

```cpp
unsigned int cw_rotate_16(unsigned int b)
    b = (b & 0xA0A0) >> 1 | (b & 0x5050) >> 4
      | (b & 0x0A0A) << 4 | (b & 0x0505) << 1;
    b = (b & 0xCC00) >> 2 | (b & 0x3300) >> 8
      | (b & 0x00CC) << 8 | (b & 0x0033) << 2;
    return b;
}
```

再次说明，通过观察分解目标距离，可以发现**顺序对调**是**不会影响结果**的。

再看 64-bit 的版本，相信不难理解。

```cpp
static void rotate_r(ull& brd){
    brd = (brd & 0xf0f0f0f000000000) >> 4  | (brd & 0x0f0f0f0f00000000) >> 32
        | (brd & 0x00000000f0f0f0f0) << 32 | (brd & 0x000000000f0f0f0f) << 4;
    brd = (brd & 0xcccc0000cccc0000) >> 2  | (brd & 0x3333000033330000) >> 16
        | (brd & 0x0000cccc0000cccc) << 16 | (brd & 0x0000333300003333) << 2;
    brd = (brd & 0xaa00aa00aa00aa00) >> 1  | (brd & 0x5500550055005500) >> 8
        | (brd & 0x00aa00aa00aa00aa) << 8  | (brd & 0x0055005500550055) << 1;
}
```

对角反射、上下反射、逆时旋转、180$^\circ$ 旋转等都可以通过相同想法一步步实现出来。

## 结语

突然发现四个颜色的挑选好像某科技公司，这……纯属巧合？
有写错的欢迎指正噢！
