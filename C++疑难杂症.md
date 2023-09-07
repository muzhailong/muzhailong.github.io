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
