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
                loop s  
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


