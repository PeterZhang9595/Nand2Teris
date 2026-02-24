# Project1:Elementary Logic Gates
在Nand2Tetris的第一个project中，我们将使用基础的HDL语言逐步实现一系列基础的逻辑门。
## 实现方式
在作业中我们通过编写底层的HDL代码实现逻辑门以满足更高的抽象层面对逻辑门的功能要求。

使用课程提供的HardwareSimulator运行chip代码。利用课程编写好的**script**(脚本文件)以及**cmp**(对比文件)对HDL代码进行测试。

## 逻辑门具体实现思路
为了一步步地构建起所有的逻辑门，我们需要有一个逻辑门作为其它逻辑门的基础，而在本次课程中我们使用的便是Built-In 地逻辑门**Nand**，因为从理论上可证明Nand可以表示所有的布尔函数。

具体可参照课程教材原文：
![nand](image.png)

**注意，以下的逻辑门建构顺序很重要，要关注前后的依赖关系**
### Not
显然有
$$Not \ x = x \ Nand \ x$$

```hdl
/**
 * Not gate:
 * if (in) out = 0, else out = 1
 */
CHIP Not {
    IN in;
    OUT out;

    PARTS:
    Nand(a=in,b=in,out=out);
}
```
### And
```hdl
/**
 * And gate:
 * if (a and b) out = 1, else out = 0 
 */
CHIP And {
    IN a, b;
    OUT out;
    
    PARTS:
    Nand(a=a,b=b,out=aAndb);
    Not(in=aAndb,out=out);
}
```
### Or
由德摩根定律可知
$$
a \ Or \ b = Not(Not(a) \ And \ Not(b))
$$
```hdl
/**
 * Or gate:
 * if (a or b) out = 1, else out = 0 
 */
CHIP Or {
    IN a, b;
    OUT out;

    PARTS:
    Not(in=a,out=nota);
    Not(in=b,out=notb);
    And(a=nota,b=notb,out=notaAndnotb);
    Not(in=notaAndnotb,out=out);
}
```
**有了Not,And,Or三剑客，我们就可以在它们的基础上实现各种各样新的逻辑门而不依赖于Nand了。这也正是计算机设计的巧妙之处——单元测试与抽象API调用。**
### Xor
| a | b | out |
|---|---|-----|
| 0 | 0 | 0   |
| 0 | 1 | 1   |
| 1 | 0 | 1   |
| 1 | 1 | 0   |

我们使用课程中教授的布尔函数反推方法。
选取out为1的行。然后拟合它们。
可以得到
$$
(not(a) \ and \ b) \ or \ (a \ and \ not(b))
$$
这里之所以要用or进行连接并且只连接给定的标准输出为1的布尔表达式，我认为原因在于外界给我们要设计的Xor的限制就是在输入为表格中的那几种情况的时候它应当可以作出对应的回应，剩余情况（虽然这里其实已经便利了所有输入组合，但为了方便我们也可以视为还存在剩余情况）无所谓，因此我们只需要拟合多个表达式让它们满足映射条件。而在每一轮输入中只可能输入这其中的一种情况，因此我们每次只用满足一个就可以了，所以要用Or进行连接。又由于对那些输出为0的行，即使输入满足了，相应也应当是0，所以不用把它们加入到Or连接的不同布尔表达式中去。
```hdl
/**
 * Exclusive-or gate:
 * if ((a and Not(b)) or (Not(a) and b)) out = 1, else out = 0
 */
CHIP Xor {
    IN a, b;
    OUT out;

    PARTS:
    Not (in=a,out=nota);
    Not (in=b,out=notb);
    And (a=a,b=notb,out=aAndnotb);
    And (a=nota,b=b,out=notaAndb);
    Or (a=aAndnotb,b=notaAndb,out=out);
}
```

### Mux
这是一个比较新颖的逻辑门概念。简单理解它就是一个“条件判断型”逻辑门。

![mux](image-1.png)

| a | b |sel|out|
|---|---|---|---|
| 0 | 0 | 0 | 0 |
| 0 | 0 | 1 | 0 |
| 0 | 1 | 0 | 0 |
| 0 | 1 | 1 | 1 |
| 1 | 0 | 0 | 1 |
| 1 | 0 | 1 | 0 |
| 1 | 1 | 0 | 1 |
| 1 | 1 | 1 | 1 |

直接使用传统艺能，难免发现，这个表达式连接起来未免太长了，需要我们进行简化。

$$
((not a) \ and \ b \ and \ sel) \\
Or \\
(a \ and \ not(b) \ and\ not(sel)) \\
Or \\
(a \ and \ b \ and \ not(sel)) \\
Or \\
(a \ and \ b \ and \ sel)
$$
发现在markdown里面手打单词和空格太难受了也不清晰，接下来将使用逻辑符号。

我们对前面的逻辑表达式进行简化合并。合并Or式子的时候，我们可以看式子中每个变量是否一样，如果一样的话那么这个变量就保留，反之则删除。以此得到
$$
(a\land\neg sel) \lor (b \land sel)
$$

```hdl
/** 
 * Multiplexor:
 * if (sel = 0) out = a, else out = b
 */
CHIP Mux {
    IN a, b, sel;
    OUT out;

    PARTS:
    Not(in=sel,out=notsel);
    And(a=a,b=notsel,out=w1);
    And(a=b,b=sel,out=w2);
    Or(a=w1,b=w2,out=out);
}
```
这样我们就实现了基本的条件判断语句。

### DMux
这是我们遇到的第一个多输出逻辑门。sel相同时，它可以视作是Mux的逆运算。

|in |sel| a | b |
|---|---|---|---|
| 0 | 0 | 0 | 0 |
| 0 | 1 | 0 | 0 |
| 1 | 0 | 1 | 0 |
| 1 | 1 | 0 | 1 |

我们其实可以拆解为a和b分别进行输出，这样问题就转化为两个简单的构造。

$$
a = in\land (\neg sel)\\
b= in \land sel
$$

```hdl
/**
 * Demultiplexor:
 * [a, b] = [in, 0] if sel = 0
 *          [0, in] if sel = 1
 */
CHIP DMux {
    IN in, sel;
    OUT a, b;

    PARTS:
    Not(in=sel,out=notsel);
    And(a=in,b=notsel,out=a);
    And(a=in,b=sel,out=b);
}
```

**实现了前面几个基础的单bit位逻辑门，接下来我们就要进行多bit位逻辑门的研究了。**
多 bit 位逻辑门本质上是对总线中每一位信号并行地应用相同的单 bit 逻辑运算，各位之间相互独立，不存在跨位影响。这里可能会混淆的一点是，虽然在功能描述中写的是for循环，但是我们实际上运行的时候是并行运行的空间复制。

### Not16
我们从Not16入手进行逐元素取反。
```hdl
/**
 * 16-bit Not gate:
 * for i = 0, ..., 15:
 * out[i] = Not(a[i])
 */
CHIP Not16 {
    IN in[16];
    OUT out[16];

    PARTS:
    Not(in=in[0],  out=out[0]);
    Not(in=in[1],  out=out[1]);
    ...
    Not(in=in[15], out=out[15]);
}
```
### And16
很遗憾，由于我们没有实现Nand16，所以我们不得不在实现And16
的时候也采取逐元素取反的方式。

### Or16
我们现在可以调用前面的16位运算，不用再费劲写逐元素运算了。
```hdl
/**
 * 16-bit Or gate:
 * for i = 0, ..., 15:
 * out[i] = a[i] Or b[i] 
 */
CHIP Or16 {
    IN a[16], b[16];
    OUT out[16];

    PARTS:
    Not16(in=a, out=nota);
    Not16(in=b, out=notb);
    And16(a=nota, b=notb, out=temp);
    Not16(in=temp, out=out);
}
```
**在输入输出的声明中，需要声明[16]来指定总线数量，但是在Parts部分，如果要对总线进行整体调用，就不用再加上[]声明了。**

### Mux16
自由选择你的实现方式吧！可以逐元素，也可以调用前面的16bit专门API。

**接下来我们将实现有多个输入的逻辑门。**
### Or8Way
使用最基础的平衡二叉树结构进行两两Or运算。
```hdl
/**
 * 8-way Or gate: 
 * out = in[0] Or in[1] Or ... Or in[7]
 */
CHIP Or8Way {
    IN in[8];
    OUT out;

    PARTS:
    Or(a=in[0],b=in[1],out=w1);
    Or(a=in[2],b=in[3],out=w2);
    Or(a=in[4],b=in[5],out=w3);
    Or(a=in[6],b=in[7],out=w4);
    Or(a=w1,b=w2,out=p1);
    Or(a=w3,b=w4,out=p2);
    Or(a=p1,b=p2,out=out);
}
```

### Multi-Way\Multi-Bit Mux
一个更全面的分支选择器。原理很简单，关键在于底层如何利用sel与输入的运算来实现“选择”这一功能。
![muti mux](image-2.png)
由于我们只实现过2个输出的16bit形式，因此我们仍应采用类似于平衡二叉树的并行方式进行选择。
```hdl
/**
 * 4-way 16-bit multiplexor:
 * out = a if sel = 00
 *       b if sel = 01
 *       c if sel = 10
 *       d if sel = 11
 */
CHIP Mux4Way16 {
    IN a[16], b[16], c[16], d[16], sel[2];
    OUT out[16];
    
    PARTS:
    Mux16(a=a,b=b,sel=sel[0],out=w1);
    Mux16(a=c,b=d,sel=sel[0],out=w2);
    Mux16(a=w1,b=w2,sel=sel[1],out=out);
}
```
实现了4way版本之后，再写8way的时候我们就可以调用4way版本了。这其实也是一种“递归”思想指导函数实现的体现。
```hdl
/**
 * 8-way 16-bit multiplexor:
 * out = a if sel = 000
 *       b if sel = 001
 *       c if sel = 010
 *       d if sel = 011
 *       e if sel = 100
 *       f if sel = 101
 *       g if sel = 110
 *       h if sel = 111
 */
CHIP Mux8Way16 {
    IN a[16], b[16], c[16], d[16],
       e[16], f[16], g[16], h[16],
       sel[3];
    OUT out[16];

    PARTS:
    Mux4Way16(a=a,b=b,c=c,d=d,sel=sel[0..1],out=w1);
    Mux4Way16(a=e,b=f,c=g,d=h,sel=sel[0..1],out=w2);
    Mux16(a=w1,b=w2,sel=sel[2],out=out);
}
```
**有点意思。**

### Multi-way\Multi-bit DMux
这个有一点难度。我们之前使用的是由叶子到根的归一化的平衡二叉树，而现在要使用的是从根到叶子的发散的平衡二叉树，也就是说，我们要不断地通过调用DMux来判断唯一的输入in应当落在哪个子区域，然后在这个子区域内不断地进一步递归调用。
```hdl
/**
 * 4-way demultiplexor:
 * [a, b, c, d] = [in, 0, 0, 0] if sel = 00
 *                [0, in, 0, 0] if sel = 01
 *                [0, 0, in, 0] if sel = 10
 *                [0, 0, 0, in] if sel = 11
 */
CHIP DMux4Way {
    IN in, sel[2];
    OUT a, b, c, d;

    PARTS:
    DMux(in=in,sel=sel[1],a=t1,b=t2);
    DMux(in=t1,sel=sel[0],a=a,b=b);
    DMux(in=t2,sel=sel[0],a=c,b=d);
}
```

以上就是Project1的全部实现思路及代码了。其实编程的很多基本思想，比如递归、抽象层等等在这个project里面就已经涉及到了。期待下一个project。

