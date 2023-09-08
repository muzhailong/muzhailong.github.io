#### 1. 把不相关的目标文件放在一起编译成的可执行文件（静态库）会包含所有的符号
func.cpp
```C++
int add(int a, int b){
  return a+b;
}
```
编译`g++ -c func.cpp -o func.o`并查看符号
```bash
$ nm -C func.o
0000000000000000 T add(int, int)
```
main.cpp
```C++
int main(){
  return 0;
}
```
编译`g++ -c main.cpp -o main.o`并查看符号
```bash
$ nm -C main.o
0000000000000000 T main
```
一起生成可执行文件, 并查看符号
```bash
$ g++ main.o func.o -o main
$ nm -C main
0000000000004010 B __bss_start
0000000000004010 b completed.8061
                 w __cxa_finalize@@GLIBC_2.2.5
0000000000004000 D __data_start
0000000000004000 W data_start
0000000000001070 t deregister_tm_clones
00000000000010e0 t __do_global_dtors_aux
0000000000003df8 d __do_global_dtors_aux_fini_array_entry
0000000000004008 D __dso_handle
0000000000003e00 d _DYNAMIC
0000000000004010 D _edata
0000000000004018 B _end
00000000000011c8 T _fini
0000000000001120 t frame_dummy
0000000000003df0 d __frame_dummy_init_array_entry
0000000000002154 r __FRAME_END__
0000000000003fc0 d _GLOBAL_OFFSET_TABLE_
                 w __gmon_start__
0000000000002004 r __GNU_EH_FRAME_HDR
0000000000001000 t _init
0000000000003df8 d __init_array_end
0000000000003df0 d __init_array_start
0000000000002000 R _IO_stdin_used
                 w _ITM_deregisterTMCloneTable
                 w _ITM_registerTMCloneTable
00000000000011c0 T __libc_csu_fini
0000000000001150 T __libc_csu_init
                 U __libc_start_main@@GLIBC_2.2.5
0000000000001129 T main
00000000000010a0 t register_tm_clones
0000000000001040 T _start
0000000000004010 D __TMC_END__
0000000000001138 T add(int, int)
```
可以看到最终产生的可执行文件中包含了`add(int,int)`符号
**结论： 将所有的目标文件放到编译的输入列表中，产生的目标文件（可执行文件或者静态库）会包含所有的符号**

#### 2. 链接时应该把具体的实现放在后面
a.cpp
```c++
#include <iostream>

int add(int, int);
int main(){
  std::cout<<add(1,1);
  return 0;
}
```
b.cpp
```c++
int add_impl(int ,int);
int add(int a, int b){
  return add_impl(a, b);
}
```
c.cpp
```c++
int add_impl(int a, int b){
  return a + b;
}
```
编译
```bash
$ g++ -c a.cpp -o a.o
$ g++ -c b.cpp -o b.o
$ ar crv libb.a ./b.o
$ g++ -c c.cpp -o c.o
$ ar crv libc.a ./c.o
```
得到`a.o` `libb.a` `libc.a`

链接1
```bash
g++ a.o libb.a libc.a -o a
```
OK, success

链接2 
```bash
g++ a.o libc.a libb.a -o a
```
会得到
```
/usr/bin/ld: libb.a(b.o): in function `add(int, int)':
b.cpp:(.text+0x1d): undefined reference to `add_impl(int, int)'
collect2: error: ld returned 1 exit status
```
**结论： 应该将具体实现的lib放到链接序列的后面**


