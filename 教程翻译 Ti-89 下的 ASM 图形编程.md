# 教程翻译 Ti-89 下的 ASM 图形编程

原文标题：《Graphics programming on the TI-89》

原文地址：

中文翻译如下：

##     目录

-     [概述](#概述)
-     [子画面](#子画面)
-     [子程序](#子程序)
-     [显示-子画面处理](#显示-子画面处理)
-     [显示-屏幕处理](#显示-屏幕处理)


###     概述

本教程用来教读者如何在Ti-89 下编程实现子画面（Sprites）

为了做出一个给人印象深刻的游戏，你需要懂得如何进行图形编程。在TI-89上，这些图像作为子画面（sprites）来处理。
为了在屏幕上显示一个子画面，我们需要以下三件事物：

-    1、屏幕
-    2、子画面
-    3、用来把子画面放置（投）到屏幕上的程序

我假定你已经拥有屏幕，除非你把它撕掉了，那么就让我们从子画面开始吧。


###     子画面

想让计算机的一个位图图像起作用要有三个关键部分。这三部分分别是图像的大小，色彩，以及每一个独立的像素点。

一个子画面除了没有颜色外拥有同样的部件。为了进行灰度编程，多个子画面被快速地从屏幕上闪过。请看下述对于加法符号的表述：

    Plus_Sign
        dc.w            1,8
        dc.b            %00011000,%00000000
        dc.b            %00011000,%00000000
        dc.b            %00011000,%00000000
        dc.b            %11111111,%00000000
        dc.b            %11111111,%00000000
        dc.b            %00011000,%00000000
        dc.b            %00011000,%00000000
        dc.b            %00011000,%00000000

让我们简略解释一下各个部分。

`Plus_Sign`

用标号来标识你的数据通常是很重要的。这也是最好的能让你的程序更清晰一点的方法，比如在标号后面使用一个冒号
`：` 用于标识子程序，在标号后面不使用冒号 `：`用于标识数据（译者注：也就是说上述的例子中标号后面全部是数据定义，所以不加冒号 `:`）。这些会让你的程序更容易被人理解。
在你的程序里使用可识别的标号也非常有用。不必担心你的标号的长度或者你在程序中用了多少标号，因为在一个编译好的程序里标号不占空间。

    dc.w            1,8

这一句说明了你的图像的规模大小。

第一个数字表示每个点阵行的“字”（words）的数量，第二个数字表示总的点阵行的数目。（译者注：上述点阵共有8行，每行占用一个 word 大小）一定要
确保你使用的是 `dc.w` 而不是 `dc.b` 或者 `dc.l`。
你们疑惑为何我使用“字/行”（words/row），而不是“字节/行”（bytes/row），我将在下面说明：

    dc.b            %00011000,%00000000
    dc.b            %00011000,%00000000
    dc.b            %00011000,%00000000
    dc.b            %11111111,%00000000
    dc.b            %11111111,%00000000
    dc.b            %00011000,%00000000
    dc.b            %00011000,%00000000
    dc.b            %00011000,%00000000

这是真正保存图像信息的部分，在 Ti-89
的屏幕上，一个像素点对应着一个 `bit` 位的数据，因此通过修改一个 `byte` 的8 `bit` 位的数据，你能修改位于一行的整整 `8` 个像素点，1个 `bit` 位的开启或关闭将被用来描绘1个像素点，因此我们通过二进制的方式，用 `1` 来描绘 1 个像素点的打开，用 `0` 来描绘 1 个像素点的关闭。我们以二进制的格式（the `%` ）把它一次一个比特（`dc.b`）地写出来，这样既可以更容易地看到图像，而且使得我们能够讨论图像的各个组成部分具体到每个比特。

有人可能会疑惑为什么我保持了整整一列空格（译者注：此处应指每行定义之后的所有的 `%00000000`）。如果我没有保留一整列空格来适应图像的大小，由程序放到屏幕上的图像将会像下面一样显示：

    Plus_Sign
        dc.w            1,8                     ;make sure it is dc.w
        dc.b            %00011000,%00011000     ;make sure it is dc.b
        dc.b            %00011000,%11111111
        dc.b            %11111111,%00011000
        dc.b            %00011000,%00011000
        dc.b            %00101011,%11010001     ;anything after this would vary 
        dc.b            %11010101,%00110101
        dc.b            %00110100,%10100011
        dc.b            %01100101,%11011001


译者注：作者的意思是计算器内部的处理单位是按照 word 来处理，也就是每次以两个 byte 为单位，于是第一列的偶数行的 byte 就会被填充到第二列的对应奇数行中，就会导致一种错位效果，从第1行第1列到第8行第1列总共8行会被依次填充为如下形式：


第1行第1列，第2行第1列
第3行第1列，第4行第1列
第5行第1列，第6行第1列
第7行第1列，第8行第1列

现在我们的图像可用了，我们可以转移到编程的下一部分--子程序了。


###     子程序

你的子程序需要把如下内容的信息输入到子程序里去。

下面就是需要的：

-    1、准备放置图像的屏幕缓冲区的内存位置；
-    2、准备放置图像的 `x`，`y` 坐标值；
-    3、原始图像数据保存的内存位置；
-    4、图像占据内存空间的大小和维数；

让我们一步一步地开始吧。你可以把屏幕缓冲区的位置放在一个变量里，并且你可以选择是否把 `x` 坐标值和 `y` 坐标值合并到另一个变量里，而且一旦你得到图像数据保存的内存位置你也就得到了图像的大小和维数。

你可以把 `x` 坐标值和 `y` 坐标值合并的原因：是因为你真正从屏幕缓冲区内存位置得到的，用来开始放置图像的位置数据的是一个偏移量（`offset`）。另外一点，计算器在 `LCD` 上显示图像之前，用来保存显示图像数据的内存，不是内存空间里完整地串连在一起的许多行，而基本上是内存空间里的一长行，每一行之后直接换行到下一行。举例来说，如果 `d0` 是 `x` 坐标值，`d1` 是 `y` 坐标值，并且 `d0` 是你输出的变量，你可以通过下面的语句来把 `x` 坐标值和 `y` 坐标值合并到一起：

    muls.w          #240,d1
    add.l           d1,d0

上述两条语句相当于 `d0 = 240*y + x`。每行有240个像素点（TI-89只能显示其中的160个，但是为 Ti-89 放置屏幕缓冲区而设置的内存总量跟 TI-92 和 TI-92+ 是一样的，都是每行有240像素点），因此我们用行数乘以 `240`，然后把列数加上去，这样我们就得到了开始的第一个像素点的偏移量。

让我们把目前所学到的内容编写到程序里去吧：


        include tios.h
        xdef            _ti89
        xdef            _main
        xdef            _comment

    _main:
        move.l          #$4c00,a0               ;store the address of the screen into a0
        move.l          #Plus_Sign,a1   ;store the address of the image into a1
        move.l          #5,d0                   ;store the column number into d0
        move.l          #2,d1                   ;store the row number into d1
        muls.w          #240,d1
        add.l           d1,d0                   ;combine x and y into d0
                                                ; The previous 4 lines could also be written out in the following way:
                                                ;       move.l          #240*2+5,d0
        jsr             drawsprite                      ;we will make the routine later
        rts

    drawsprite:                                             ;It does nothing right now
        rts

    _comment
        dc.b            "Sprite Routine",0

    Plus_Sign               ;here's our picture
        dc.w            1,8
        dc.b            %00011000,%00000000
        dc.b            %00011000,%00000000
        dc.b            %00011000,%00000000
        dc.b            %11111111,%00000000
        dc.b            %11111111,%00000000
        dc.b            %00011000,%00000000
        dc.b            %00011000,%00000000
        dc.b            %00011000,%00000000
        end             ;required at the end of every program


如果你试着编译这个程序，并且然后去试着运行它，什么都不会发生。既然你已经在写好的程序里得到了基本的部分（即为前文
所述画图所需要的基本部分），我们需要浓缩一下这个子程序来得到你的子画面。你马上就拥有了如下信息：

`a0` = 屏幕缓冲区的内存位置

`a1` = 你的子画面的内存位置

`d0` = 与屏幕缓冲区起始内存位置相对的偏移位置

让我们通过决定下述内容来开始：图像的维数，开始放置图像到屏幕缓冲区的内存位置，以及我们准备如何来着手建造这个子画面-显示程序。


### 显示-子画面处理

既然TI-89每个 `byte` 有 `8` 个像素点，更进一步的一个 `byte` 将会把图像8像素点放置到右边。那么我们需要通过一个循环命令来移动图像。
为了做这个循环我们也需要把每个 `byte` 进行重叠交错。如果我们不这样做，我们将得到空白点。请看下列场景：

        dc.b            %00001111,%11110000

每个 `byte` 通过 `wrap-around` 向左循环移动 `2` 位，并且被用 `MOVE` 写入


        dc.b            %00111100,%11000011

每个 `byte` 为使用 `wrap-around` 向左循环移动 `2` 位，并且被用 `MOVE` 写入
   
        dc.b            %00111100,%11000000
        
把每个 `byte` 保存到一个 `word` 里，每个 `word` 向左转动 `2` 位，使用或不使用 `wrap-around`，然后用 `OR` 写入  
        dc.b            %00000000,%00111111,%11000000

每个 `word`:

        dc.b            %00000000,%00111100
        dc.b            %00000011,%11000000

根据这些分析，你需要把这些数据以其 `2` 倍大小保存起来然后循环翻转（`rotate`）它，然后通过 `OR` 把它们写到屏幕缓冲区中。但是如果你一次一个 `byte` 地操作，你将会试图把一个 `word` 写入一个奇数内存位置，这将会引起一个内存地址错误。当你在一个奇数内存位置读写一个 `word` 或一个 `longword`，一个地址错误将会发生。为了不引发这种地址内存错误，我们需要一次跳过 `2` 个 `byte` ，并且我们也需要能够 `rotate` 图像的 `15` 个像素点到右边。如果这些让你困惑不解，这还不是最坏的，不过还是让我们继续从事那些我们所拥有的，并且开始从那些我们已经得到的入手吧。

把下列代码正确地加到 `drawsprite` 和 `rts` 之间：  
        move.b          d0,d1
        and.l           #$F,d1
        and.l           #$FFFFFFF0,d0
        lsr             #3,d0

这里是不同的部分。`d1` 现在包含 `0` 到 `15` 间的一个数字，用来 `rotate` 图像到右边的像素点的数字，`d0` 现在包含了来自屏幕缓冲区内存位置开始处，画出图像所需要的到偏移地址的 `bytes` 形式的数据。现在我们吧屏幕缓冲区地址加到 `d0` 上来获取图像开始位置的 `bytes` 形式的信息。

把下一句代码加到上次增加代码的最后：

        add.1           a0,d0

为了显示我们的子画面，我们需要使用 `2` 次 `dbra` 命令。一次用在画垂直的程序，一次用在画水平的程序。使用每个程序的次数已经保存在子画面之前，但是我们还需要每次减去 `1` 因为 `dbra` 会比保存在变量中的值多执行一次。

把下列语句加入到上次所增加代码的后面：


        move.w          (a1)+,d2
        move.w          (a1)+,d3
        sub.l           #1,d2
        sub.l           #1,d3

现在有了用于 `dbra` 程序的被正确初始化的数据。

这里是程序的组成部分;

    Vert:
    Horiz:
    draw the picture
    dbra Horiz
    dbra Vert

首先我们创造分别用于垂直程序和水平程序的标号，然后将其放入 `dbra` 命令中。

把下列语句加入到上次所增加代码的后面：

    drawspritevert:
    drawspritehoriz:
        dbra.w          d2,drawspritevert
        dbra.w          d3,drawspritehoriz

既然我们需要重复水平程序，我们需要拷贝 `d2` 到另一个变量值并且使用这个变量来代替 `d2` 。

因此，在标号 `drawspritevert：` 和标号 `drawspritehoriz：` 之间增加下列语句：

        move.l          d2,d4

然后修改下列语句：

        dbra.w          d2,drawspritevert
为：
        dbra.w          d4,drawspritehoriz 
现在为了在每次我们执行水平程序之间能够跳到一整行，我们需要增加下列语句在 `move.ld2,d4`  之后： 

        move.l          d0,a2
        add.l           #30,d0

我们已经成功地迈出了更远的一步并且完成了垂直程序章节。现在开始下一部分。

我们首先需要清理那些将会被用来做变换的变量和其他的垃圾数据，然后我们需要拷贝这些变量到屏幕。还记得回到关于翻转数据和需要一次跳过一个 `word` 的部分吗？这些迟早派得上用场。

我们从子画面（`sprite`） 拷贝了一个 `word` 到 `d5` 里，使得子画面（`sprite`）前进到下一个 `word` 然后我们使用 `ror` 命令根据 `d1` 所指定的次数来循环它到右边。 但是等等！ 它已经到右边了！我们需要做的就是更换 `d5` 的上、下的 `words` 然后根据 `d1` 指定的次数循环它到右边。

让我们在 `drawspritehoriz：` 后面坚持写完我们所得到更多的内容吧：

        clr.l           d5
        move.w          (a1)+,d5
        swap            d5
        ror.l           d1,d5


###     显示-屏幕处理

现在所有我们得到的剩下部分就是把图像放置到屏幕（缓冲区？）上去了。这里你有几个选择：

-    1、坚持 sprite 使用 `OR` 逻辑运算，也就是黑上加黑，黑上加白，以及白上加黑造成黑，但是白上加白造成白。

-    2、坚持 sprite 使用 `XOR` 逻辑运算，也就是黑上加黑和摆上加白造成白，并且黑上加白或白上加黑造成黑。

-    3、通过反转 `d5` 并且使用 `AND` 逻辑运算来删除 sprite，造成的后果是黑色加到 sprite 上会在屏幕上造成白色，否则将会保持它原来的颜色。

如果你选择使用第1个，把下面这2行语句加到上次语句的最后：

        or.l            d5,(a2)
        lea             2(a2),a2

如果你选择第2个，插入下面这两行语句来替换掉刚才的2行语句：

        eor.l           d5,(a2)
        lea             2(a2),a2

如果你选择最后一个，插入下面的语句来替换：

        not             d5
        and.l           d5,(a2)
        lea             2(a2),a2

你也能使用同样的程序来做崩溃检测，通过插入下列语句来替换上面的语句以及2条 `dbra` 命令：

        and.l           (a2),d5
        lea             2(a2),a2
        cmp.l           #0,d5
        dbne            d4,drawspritehoriz
        dbne            d3,drawspritevert

现在你完成了！这个 `Drawsprite` 程序将会像下面看到的样子如果你选择了第一种方式画图到屏幕上：

    drawsprite:
        move.b          d0,d1
        and.l           #$F,d1
        and.l           #$FFFFFFF0,d0
        lsr                     #3,d0
        add.l           a0,d0
        move.l          (a1)+,d2
        move.l          (a1)+,d3
        sub.l           #1,d2
        sub.l           #1,d3

    drawspritevert:
        move.l          d2,d4
        move.l          d0,a2
        add.l           #30,d0

    drawspritehoriz:
        clr.w           d5
        move.w          (a1)+,d5
        swap            d5
        ror.l           d1,d5
        or.l            d5,(a2)
        lea                     2(a2),a2
        dbra.w          d4,drawspritevert
        dbra.w          d3,drawspritehoriz

原文在此结束。

*==========================*
完整的程序如下：
*==========================*

        include tios.h
        xdef            _ti89
        xdef            _main
        xdef            _comment
    _main:
        move.l          #$4c00,a0               ;store the address of the screen into a0
        move.l          #Plus_Sign,a1   ;store the address of the image into a1
        move.l          #5,d0                   ;store the column number into d0
        move.l          #2,d1                   ;store the row number into d1
        muls.w          #240,d1
        add.l           d1,d0                   ;combine x and y into d0
                                                                ; The previous 4 lines could also be written out in the following way:
                                                                ;       move.l          #240*2+5,d0
        jsr             drawsprite                      ;we will make the routine later
        rts
    drawsprite:                                             ;It does nothing right now
        move.b          d0,d1
        and.l           #$F,d1
        and.l           #$FFFFFFF0,d0
        lsr             #3,d0
        add.l           a0,d0
        move.w          (a1)+,d2
        move.w          (a1)+,d3
        sub.l           #1,d2
        sub.l           #1,d3
        
    drawspritevert:
        move.l          d2,d4
        move.l          d0,a2
        add.l           #30,d0
        
    drawspritehoriz:
        clr.w           d5
        move.w          (a1)+,d5
        swap            d5
        ror.l           d1,d5
        or.l            d5,(a2)
        lea             2(a2),a2
        dbra.w          d4,drawspritevert
        dbra.w          d3,drawspritehoriz
        
    _comment
        dc.b            "Sprite Routine",0
    Plus_Sign               ;here's our picture
        dc.w            1,8
        dc.b            %00011000,%00000000
        dc.b            %00011000,%00000000
        dc.b            %00011000,%00000000
        dc.b            %11111111,%00000000
        dc.b            %11111111,%00000000
        dc.b            %00011000,%00000000
        dc.b            %00011000,%00000000
        dc.b            %00011000,%00000000
        end             ;required at the end of every program


翻译者：FreeBlues

译文网络版本：https://github.com/FreeBlues/Ti-89T/blob/master/教程翻译%20Ti-89%20下的%20ASM%20图形编程.md

如果有哪些地方翻译得有问题，欢迎大家指出，确认无误后我会把更确切的翻译合入新版本，谢谢！





