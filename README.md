

### A20信号与CPUA20信号

在vl82c311的数据手册中，存在2个代表地址20的信号线。**A20**与**CPUA20**。

对于A20信号线的描述存在矛盾的点。根据推荐的连接框图可以看到

![1](picture\1.PNG)

386CPU的A20信号线连接的是VL82C311的A20信号pin。但是在手册中的时序图中，对于CPU的时序部分是这样描述的：

![](picture/2.png)

可以看到，在时序图中，地址20对应的是==CPUA20==信号，这与上边的框图明显冲突。所以下边需要分析两个信号具体的描述来确认信号的连接方式。

+ 对于A20的描述是这样的：  

> ![](picture/3.png)

A20在CPU ,non-Master DMA,Refresh Mode三种模式下是作为输出使用的。只在Master Mode模式下才作为了输入，这说明在正常使用过程中，A20与CPU不能接在一起。

+ CPUA20的描述是这样的

> ![](picture/4.png)

CPUA20是CPU地址20的输入脚，在Master Mode和DMA周期下作为输出。



虽然CPUA20的描述中，说法并不是在CPU模式下作为输入，但是通过 *CPU Address Bit 20 input* 也可以确定它与CPU的连接关系了。

另外一个最有力的证据是在使用这款芯片组的主板上可以看出连接关系。

> ![](picture/5.png)

图片中很明显的62号CPUA20 pin是直接通向CPU的A20脚附近的。==所以证实了CPUA20才是连接CPU的pin==。

> 为什么需要采用两种A20引脚呢：
>
> 1. 首先A20是作为ISA BUS的地址引脚，用于连接外部设备。在正常主板输出地址的周期中A20都是在正确输出地址。
>
> 2. 在CPU启动后将进入实模式，所访问的地址空间应该是\$00000-\$FFFFF共1MB。地址的计算是16位的CS左移4位加上IP寄存器的值。对于8086来说地址由A0-A19表示，当值超过\$FFFFF时，比如\$100001，此时访问的地址超过了1MB的空间，CPU将舍弃最高位从而访问\$00001。但对于386来说，地址的计算与8086相同，不过因为存在地址A20，实际地址由A0-A20表示，所以可以访问的地址变成了\$000000-\$1FFFFF共2MB。这对于8086上使用某些通过忽略溢出来访问内存的程序来说是错误的，所以vl82c311引入了A20引脚，这个引脚由上面图片中说明的，可以通过内部逻辑强行拉低，从而将不会出现访问到\$100000后内存的情况。
>
>    

