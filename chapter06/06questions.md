
下面程序实现依次用内存0:0-0:15单元中的内容改写程序中的数据  
------
```asm
        assume cs:codesg  
        codesg segment  
                dw 0123H,0456H,0789H,0abch,0defh,0fedh,0cbah,0987h   
        start:        mov ax,0  
                mov ds,ax  
                mov bx,0          
                mov cx,8  
        s:        mov ax,[bx]  
                (                 )        
                add bx,2  
r                loop s  
                mov ax,4c00h  
                int 21h  
        codesg ends  
        end start 
```
填空 mov cs:[bx],ax
程序中数据位置`cs:[bx]`


下面程序依次用…0:0-0:15内容……改写程序中的数据,数据的传送用栈来进行.栈空间设置在程序内  
---------

```asm   
       assume cs:codesg  
        codesg segment  
               dw 0123H,0456H,0789H,0abch,0defh,0fedh,0cbah,0987h   
                dw 0,0,0,0,0,0,0,0,0,0                ;10个字单元用作栈空间即20个字节型单元  
        start:        mov ax,( cs )  
                mov ss,ax                        ;两次dw定义了0:0-0:23的数据 则空栈的SP的栈底为0:24  
                mov sp,(   )        或 mov sp,36  
                mov ax,0  
                mov ds,ax  
                mov bx,0  
                mov cx,8  
        s:        push [bx]      ;从栈底 ss:24H push入 ds:bx处内容,之后ss:22H\20H  
                (      )        ;实际上就是把push入SS:SP处的数据POP到 SS:[bx]\SS:0002\SS:0004……  
                                       ;push 是 sp=sp-2       pop 是sp=sp+2     
                add bx,2  
                loop s  
                mov ax,4c00h  
                int 21h  
        codesg ends  

end start  
```

2.第一次dw是八个数据也就是16 第二次是10个字就是20 相加是36 就是`24H`
3.之前把[bx]的数据push入栈，之后再把栈里的数据放回到SS:[bx]中

实验5
====
1.
```asm
assume cs:code,ds:data,ss:stack
data segment   

dw 0123h,0456h,0789h,0abch,0defh,0fedh,0cbah,0987hdata ends
stack segment  
dw 0,0,0,0,0,0,0,0
stack ends
code segment
start: mov ax,stack       
mov ss,ax       
mov sp,16
mov ax,data       
mov ds,ax
push ds:[0]       
push ds:[2]       
pop ds:[2]       
pop ds:[0]
mov ax,4c00h
int 21h
code ends
end start
```

整个程序的入口为（ds+10h）:0
程序返回前DATA段中的数据不改变

如果段中数据占N字节 程序加载后，该段实际占有空间为？
--------
(N/16+1)*16 [说明：N/16只取整数部分]    或   （N＋15）/ 16 ，对16取整 
一、这首先要从8086处理器寻址原理说起。  
      8086这种处理器有二十根地址线（20个用于寻址的管脚），可以使用的外部存储器空间可达1MB（00000H～FFFFFH）。 但是，8086内部的寄存器都是16位的，用任何一个寄存器（比如BX或SI），都无法直接寻址8086所支持的1M地址空间，因为16位寄存器只能表示640KB的空间范围（0000～FFFFH）。
      所以，Intel想了一个方法，设计出了CS/DS/ES/SS这几个段地址寄存器，用段地址寄存器与普通寄存器组合，来寻址1MB的地址范围，即：对于CPU取指来说，用CS:IP组合来寻址下一个要执行的指令（也在存储器中）；对于堆栈操作PUSH/POP来说，用SS:SP组合来表示当前栈指针（栈也在存储器中）；对于数据操作指令来说，用默认的DS/ES或指定的段地址（段前缀指令）与偏移量寄存器组合寻址。  
组合后的实际地址＝段寄存器内容×16＋偏移量寄存器内容  
      从这个公式可以看到，每一个段的地址都对齐在16的倍数上。比如DS=1234H，则这个段就从 1234H×16＋0000H＝12340H开始，最大到1234H×16＋0FFFFH＝2233FH为止。  
二、对同一个内存地址，有不同的段:偏移量组合方法，比如2233FH这个地址，既可以表示为1234H:0FFFFH（在1234H段中），也可以表示为2233H:0000FH（在2233H段中）。  
那么，如果汇编程序中有下面两个连续的段定义，汇编编译程序会怎么做呢？  
```name1 segment  
d1 db 0  
name1 ends  
name2 segment  
d2 db 0  
name2 ends  
```
　　编译程序可以将name1和name2编译成一个段，d1和d2在内存中连续存放，这样可以节省内存空间。比如编译程序以name1为基准，将name1作为一个段的起始，程序加载后会被放在xxxx0H的地方，那么d1就放在该段偏移地址为0字节的位置，d2就放在该段偏移为1字节的位置。  
　　这样的处理方式虽然可能会节省一点内存空间，但是对于编译器的智能化要求太高了，它必须将源程序中所有引用到name2和d2的地方，全部调整为以name1段为基准，这实在是太难了，而且也节省不了几个字节的空间，编译器是不会干这种吃力不讨好的事的。 
      编译器实际的处理方式是将name1中的所有内容放在一个段的起始地址处，name2里的所有内容放在后续一个段的起始地址处（这也是汇编指令segment的本义：将不同数据分段）。这样，即使name1中只包含一个字节，也要占一个段（16个字节），所以，一个段实际占用的空间＝（段中字节数＋15）/ 16。  
　　所以，8086处理器的内部寻址原理和汇编程序编译器共同决定了segment定义的段必须放在按16的倍数对准的段地址边界上，占用的空间也是16的倍数。




