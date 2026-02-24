# 根据输入-输出的映射关系反推布尔函数
![boolean](image.png)
首先使用简单的NOT和AND对所有的输出为1的语句进行拟合，然后用OR把他们连接起来并且进行简化。

$$
X \ OR \ Y = NOT(NOT(X)\ AND \ NOT(Y))
$$
我们可以使用and 和 not实现所有的布尔函数
# NAND
$$
x \ nand\ y=not(x \ and \ y) \\
not \ x = x \ nand \ x \\
x and y = not(x \ nand \ y)
$$
我们可以仅仅用nand就实现所有的布尔函数

# 逻辑门——使用HDL构建
要求：真值表
使用HDL 在 chip 层面实现逻辑门应该干什么。
![hdl](image-1.png)

# Hardware simulation
在硬件构建的过程中，一般有两类分工。
system architects 提供芯片API，test和cmp的脚本
而developer 则利用HDL语言构造对应的逻辑门并且根据系统架构师提供的内容验证自己实现的正确性。

# Multi-bit Buses
可以在芯片里面实现类似于数组的操作。但是和数组本质上不同。
多根线绑在一起，当成一个整体进行传输。把多个信号打包成一个整体进行传输。
在索引的时候，0在最右边。

# multiplexing / demultiplexing
比如在一些网络信息传输的时候，可以利用mux和dmux实现信息的整合和解码，只要mux和dmux传入的sel是一样的就可以了。
![mux](image-2.png)

