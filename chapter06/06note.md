
包含多个段的程序
========
在代码中使用数据
-------
```asm
assume cs:code
code segment
dw 0123h,0456h,0abch,0defh,0fedh,0cdah,0789h  //dw用于定义字型数据   dw定义的数据位于代码段最开始，所以偏移地址为0
（start）   mov bx,0
            mov ax,0
            mov cx,8
            s:add ax,cx:[bx]   //8个数据位于代码段中，CS用于存放代码段的段地址
            add bx,2
            loop s

mov ax,4c00h
int 21h
code ends
end(start)
```

程序加载到内容中后，所占内存空间的前16个单元存放在定义的数据中，后面才开始放机器指令
`end`通知编译器程序的入口
`dw`占16个字节

在代码中使用栈
-------
```asm
assume cs:code
code segment
dw 0123h,0456h,0abch,0defh,0fedh,0cdah,0789h 
dw 0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0  //用dw定义16个字型数据，将获得16个字的内存空间，存放之前的16个数据，后面的程序把这段空间当作栈
```

将数据，代码，栈放进不同的段
--------
*一个段的最大容量是64KB
```asm
assume cs:code,ds:data,ss:stack
data segment
     dw 0123h,0456h,0abch,0defh,0fedh,0cdah,0789h 
data ends

stack segment
      dw 0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0
stack ends

code segment

start: mov ax,stack
       mov ss,ax
       mov sp,20h   //设置栈顶SS:SP 指向stack:20
       mov ax,data
       mov ds,ax    //da指向data段
       mov bx,0     //ds:bx指向data段的第一个单元
       mov cx,8
       
s:     push [bx]
       add bx,2
       loop s
       
       mov bx,0
       mov cx,8
       
s0:    pop [bx]
       add bx,2
       loop s0
       
mov ax,4c00h
int 21h
code ends
end start
```
1.段名代表了段地址 偏移地址要看数据在数据中的位置
       
       

