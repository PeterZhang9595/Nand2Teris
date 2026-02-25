# Project2:Boolean Arithmetic
Nand2Tetris的第二课Project需要我们逐步利用逻辑门实现基本的运算逻辑单元。
## HalfAdder
实现一个单Bit的加法器，输入a和b，输出sum和carry(进位)。
这是一个再简单不过的加法运算了，然而如果我们想用已有的逻辑门完成还需要导一下。
![halfadder](image.png)
经过观察，carry符合AND门的输出规律，sum符合XOR门的输出规律。调用原有API即可。
```hdl
/**
 * Computes the sum of two bits.
 */
CHIP HalfAdder {
    IN a, b;    // 1-bit inputs
    OUT sum,    // Right bit of a + b 
        carry;  // Left bit of a + b

    PARTS:
    And(a=a,b=b,out=carry);
    Xor(a=a,b=b,out=sum);
}
```
## FullAdder
实现3个bit相加，这里我们调用前面已经完成好的HalfAdder即可。但由于加法的省略机制，这里要注意处理好进位。
我们同样需要注意的一点是，虽然在实现的时候我们是按照存在先后顺序的方式完成的，但是进位不会和sum产生任何纠葛。
```hdl
/**
 * Computes the sum of three bits.
 */
CHIP FullAdder {
    IN a, b, c;  // 1-bit inputs
    OUT sum,     // Right bit of a + b + c
        carry;   // Left bit of a + b + c

    PARTS:
    HalfAdder(a=a,b=b,sum=t1,carry=t2);
    HalfAdder(a=t1,b=c,sum=sum,carry=t3);
    HalfAdder(a=t2,b=t3,sum=carry,carry=t4);
}
```

## Add16
实现两个16bit数字的相加，难点在于进位的处理。
我们采用15个FullAdder+1个HalfAdder的构建方式。此处不列出代码了。
**要注意我们在现成的HDL语言中无法直接使用特定数字为变量赋值。看了一下大神的笔记，原来可以用True or False赋值。**

## Inc16
一个实现给输入加1的累加器。
```hdl
/**
 * 16-bit incrementer:
 * out = in + 1
 */
CHIP Inc16 {
    IN in[16];
    OUT out[16];

    PARTS:
    Add16(a=in,b[0]=true,out=out);
}
```
这里值得一提的是HDL中给一个变量赋值，要使用布尔值。同时对于一个多bit变量，其各位数值默认为false(0),如果要指定为true(1)要利用角标手动访问指定。

## ALU
最后是经典的黑盒，ALU。
```hdl
/**
 * ALU (Arithmetic Logic Unit):
 * Computes out = one of the following functions:
 *                0, 1, -1,
 *                x, y, !x, !y, -x, -y,
 *                x + 1, y + 1, x - 1, y - 1,
 *                x + y, x - y, y - x,
 *                x & y, x | y
 * on the 16-bit inputs x, y,
 * according to the input bits zx, nx, zy, ny, f, no.
 * In addition, computes the two output bits:
 * if (out == 0) zr = 1, else zr = 0
 * if (out < 0)  ng = 1, else ng = 0
 */
// Implementation: Manipulates the x and y inputs
// and operates on the resulting values, as follows:
// if (zx == 1) sets x = 0        // 16-bit constant
// if (nx == 1) sets x = !x       // bitwise not
// if (zy == 1) sets y = 0        // 16-bit constant
// if (ny == 1) sets y = !y       // bitwise not
// if (f == 1)  sets out = x + y  // integer 2's complement addition
// if (f == 0)  sets out = x & y  // bitwise and
// if (no == 1) sets out = !out   // bitwise not

CHIP ALU {
    IN  
        x[16], y[16],  // 16-bit inputs        
        zx, // zero the x input?
        nx, // negate the x input?
        zy, // zero the y input?
        ny, // negate the y input?
        f,  // compute (out = x + y) or (out = x & y)?
        no; // negate the out output?
    OUT 
        out[16], // 16-bit output
        zr,      // if (out == 0) equals 1, else 0
        ng;      // if (out < 0)  equals 1, else 0

    PARTS:
    Mux16(a=x,b=false,sel=zx,out=x1);
    Not16(in=x1,out=negateX);
    Mux16(a=x1,b=negateX,sel=nx,out=x2);

    Mux16(a=y,b=false,sel=zy,out=y1);
    Not16(in=y1,out=negateY);
    Mux16(a=y1,b=negateY,sel=ny,out=y2);

    Add16(a=x2,b=y2,out=xPlusy);
    And16(a=x2,b=y2,out=xAndy);
    Mux16(a=xAndy,b=xPlusy,sel=f,out=fxy);

    Not16(in=fxy,out=notFxy);
    Mux16(a=fxy,b=notFxy,sel=no,out=out,out[15]=ng,out[0..7]=outCopy1,out[8..15]=outCopy2);

    Or8Way(in=outCopy1,out=allZero1);
    Or8Way(in=outCopy2,out=allZero2);
    Or(a=allZero1,b=allZero2,out=sel);
    Mux(a=true,b=false,sel=sel,out=zr);
}
```
代码实现还是比较简单的，但是要仔细一点。
- 关注```Mux16(a=fxy,b=notFxy,sel=no,out=out,out[15]=ng,out[0..7]=outCopy1,out[8..15]=outCopy2);```的写法。我们必须在这里把总线拆开分给后面的Or8Way。因为在Nand2Tetris的编程语言中，out是只写的。
- Or8Way是好东西，别忘了这个的存在。
- 判断二进制数是否负数就看第一位；判断是否为0就看所有位是否都是0.

第二次project到此就基本完成了。任务量和难度比第一次都要小一点，更多是一些编程的练习。
**出发，Project3!**
