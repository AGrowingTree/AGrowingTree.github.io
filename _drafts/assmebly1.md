## data section

初始化，在运行时不改变

```assembly
section.data
```



## bss section

声名变量

```assembly
section.bss
```



## text section

代码段，必须有 `global _start` 告诉程序开始的地方

```
section.text
   global _start
_start:
```



## 注释

```assembly
; This program displays a message on screen
```



## `elf32`: Executable and Linkable Format 32-bit Object Files

