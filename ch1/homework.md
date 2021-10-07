# 使用gdb&dlv找到程序入口
## 查看编译后的进程入口地址

在mac环境下安装gdb与dlv

gdb ./main

info file

可以找到程序入口

image-20211007154134272

## 在 dlv 调试工具中，使用断点功能找到代码位置

 dlv exec ./main

(dlv) b *0x10655a0

打断点后可以看到函数执行情况

image-20211007165000718

# 查询函数的调用方
必做：runqput，runqget，globrunqput，globrunqget 选做：schedule，findrunnable，sysmon

## runqput


 runqput tries to put g on the local runnable queue.
 如果next为false, runqput将g放到队尾
 如果next为true, runqput将g放到下一次运行的位置
 如果队列满了，将g放到全局队列中
调用方：


globrunqget 从全局的队列中获取一串g

goyield_m 将当前选中的G放到P上运行

newproc 创建一个g，并放到等待队列中

ready 运行前的准备工作

## runqget
从本地队列中获取一个g

调用方

findrunnable 从本地或全局队列中获取一个g

schedule 找到并执行一个可运行的goroutine

stealWork 从任意p获取一个goroutine或者timer

## globrunqput

g放到全局队列中

调用方：

exitsyscall0 获取p失败，将g置为ready状态

goschedImpl 处理当前g的状态并运行schedule函数进行调度

## globrunqget

从全局队列中获取g

调用方：

findrunnable 查找可运行的goroutine

schedule 查找并执行可运行的goroutine
