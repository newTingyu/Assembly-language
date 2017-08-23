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

在代码中使用栈
-------
```asm
assume cs:code
code segment
dw 0123h,0456h,0abch,0defh,0fedh,0cdah,0789h 
dw 0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0  //用dw定义16个字型数据，将获得16个字的内存空间，存放之前的16个数据，后面的程序把这段空间当作栈
```



