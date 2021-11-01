# ch2

## Q1
必做：
ppt:33 页的三条红线位置，请在 runtime 的代码中找到(可以使用课上介绍的编译、反编译工具查找)

### write close panic
code :
package main
func main() {
	ch := make(chan int)
	close(ch)
	ch <- 1
}

运行程序可得到报错的位置信息
➜  ch2 git:(main) ✗ go run main.go 
panic: send on closed channel

goroutine 1 [running]:
main.main()
	/Users/harukilv/go/src/github.com/izaya/GoCNStudyNote/ch2/main.go:5 +0x65
exit status 2

得到报错的行数
➜  ch2 git:(main) ✗ go tool compile -S main.go |grep ":5"
  0x0049 00073 (main.go:5)	MOVQ	"".ch+24(SP), AX
	0x004e 00078 (main.go:5)	MOVQ	AX, (SP)
	0x0052 00082 (main.go:5)	LEAQ	""..stmp_0(SB), AX
	0x0059 00089 (main.go:5)	MOVQ	AX, 8(SP)
	0x005e 00094 (main.go:5)	PCDATA	$1, $0
	0x005e 00094 (main.go:5)	NOP
	0x0060 00096 (main.go:5)	CALL	runtime.chansend1(SB)


断点找到报错的地方
➜  ch2 git:(main) ✗  dlv exec ./main
Type 'help' for list of commands.
(dlv) b runtime.chansend1
Breakpoint 1 set at 0x10039a0 for runtime.chansend1() /usr/local/go/src/runtime/chan.go:142
(dlv) c
> runtime.chansend1() /usr/local/go/src/runtime/chan.go:142 (hits goroutine(4):1 total:2) (PC: 0x10039a0)
Warning: debugging optimized function
> runtime.chansend1() /usr/local/go/src/runtime/chan.go:142 (hits goroutine(3):1 total:2) (PC: 0x10039a0)
Warning: debugging optimized function
   137:		return c.qcount == c.dataqsiz
   138:	}
   139:	
   140:	// entry point for c <- x from compiled code
   141:	//go:nosplit
=> 142:	func chansend1(c *hchan, elem unsafe.Pointer) {
   143:		chansend(c, elem, true, getcallerpc())
   144:	}
   145:	
   146:	/*
   147:	 * generic single channel send/recv




### close nil panic
code :
package main
func main() {
	var ch chan int
	close(ch)
}

➜  ch2 git:(main) ✗ go run  main_02.go 
panic: close of nil channel

goroutine 1 [running]:
main.main()
	/Users/harukilv/go/src/github.com/izaya/GoCNStudyNote/ch2/main_02.go:4 +0x2a
exit status 2



➜  ch2 git:(main) ✗ go build main_02.go 
➜  ch2 git:(main) ✗ ls
ch2.md     main       main.go    main.o     main_02    main_02.go main_3.go
➜  ch2 git:(main) ✗ dlc exec main_02
zsh: command not found: dlc
➜  ch2 git:(main) ✗ dlv exec main_02
Type 'help' for list of commands.
(dlv) b main.main
Breakpoint 1 set at 0x105e10f for main.main() ./main_02.go:2
(dlv) c
> main.main() ./main_02.go:2 (hits goroutine(1):1 total:1) (PC: 0x105e10f)
Warning: debugging optimized function
     1:	package main
=>   2:	func main() {
     3:		var ch chan int
     4:		close(ch)
     5:	}
     6:	
     7:	
(dlv) disass
TEXT main.main(SB) /Users/harukilv/go/src/github.com/izaya/GoCNStudyNote/ch2/main_02.go
	main_02.go:2	0x105e100	65488b0c2530000000	mov rcx, qword ptr gs:[0x30]
	main_02.go:2	0x105e109	483b6110		cmp rsp, qword ptr [rcx+0x10]
	main_02.go:2	0x105e10d	7625			jbe 0x105e134
=>	main_02.go:2	0x105e10f*	4883ec10		sub rsp, 0x10
	main_02.go:2	0x105e113	48896c2408		mov qword ptr [rsp+0x8], rbp
	main_02.go:2	0x105e118	488d6c2408		lea rbp, ptr [rsp+0x8]
	main_02.go:4	0x105e11d	48c7042400000000	mov qword ptr [rsp], 0x0
	main_02.go:4	0x105e125	e81662faff		call $runtime.closechan
	main_02.go:5	0x105e12a	488b6c2408		mov rbp, qword ptr [rsp+0x8]
	main_02.go:5	0x105e12f	4883c410		add rsp, 0x10
	main_02.go:5	0x105e133	c3			ret
	main_02.go:2	0x105e134	e8c7b2ffff		call $runtime.morestack_noctxt
	.:0		0x105e139	ebc5			jmp $main.main
  
(dlv) b runtime.closechan
Breakpoint b set at 0x1004353 for runtime.closechan() /usr/local/go/src/runtime/chan.go:355
(dlv) b runtime.closechan
(dlv) c
> [b] runtime.closechan() /usr/local/go/src/runtime/chan.go:355 (hits goroutine(1):1 total:1) (PC: 0x1004353)
Warning: debugging optimized function
   350:		src := sg.elem
   351:		typeBitsBulkBarrier(t, uintptr(dst), uintptr(src), t.size)
   352:		memmove(dst, src, t.size)
   353:	}
   354:	
=> 355:	func closechan(c *hchan) {
   356:		if c == nil {
   357:			panic(plainError("close of nil channel"))
   358:		}
   359:	
   360:		lock(&c.lock)


### close closed panic
code :
package main
func main() {
	 ch := make(chan int)
  
	close(ch)
	close(ch)
}


➜  ch2 git:(main) ✗ go run main_03.go 
panic: close of closed channel

goroutine 1 [running]:
main.main()
	/Users/harukilv/go/src/github.com/izaya/GoCNStudyNote/ch2/main_03.go:6 +0x57
exit status 2
➜  ch2 git:(main) ✗ dlv exec main_03
Type 'help' for list of commands.

(dlv) b main.main
Breakpoint 1 set at 0x105e10f for main.main() ./main_03.go:2
(dlv) c
> main.main() ./main_03.go:2 (hits goroutine(1):1 total:1) (PC: 0x105e10f)
Warning: debugging optimized function
     1:	package main
=>   2:	func main() {
     3:		 ch := make(chan int)
     4:	  
     5:		close(ch)
     6:		close(ch)
     7:	}
(dlv) disass
TEXT main.main(SB) /Users/harukilv/go/src/github.com/izaya/GoCNStudyNote/ch2/main_03.go
	main_03.go:2	0x105e100	65488b0c2530000000	mov rcx, qword ptr gs:[0x30]
	main_03.go:2	0x105e109	483b6110		cmp rsp, qword ptr [rcx+0x10]
	main_03.go:2	0x105e10d	7652			jbe 0x105e161
=>	main_03.go:2	0x105e10f*	4883ec28		sub rsp, 0x28
	main_03.go:2	0x105e113	48896c2420		mov qword ptr [rsp+0x20], rbp
	main_03.go:2	0x105e118	488d6c2420		lea rbp, ptr [rsp+0x20]
	main_03.go:3	0x105e11d	488d059c6c0000		lea rax, ptr [rip+0x6c9c]
	main_03.go:3	0x105e124	48890424		mov qword ptr [rsp], rax
	main_03.go:3	0x105e128	48c744240800000000	mov qword ptr [rsp+0x8], 0x0
	main_03.go:3	0x105e131	e84a56faff		call $runtime.makechan
	main_03.go:3	0x105e136	488b442410		mov rax, qword ptr [rsp+0x10]
	main_03.go:3	0x105e13b	4889442418		mov qword ptr [rsp+0x18], rax
	main_03.go:5	0x105e140	48890424		mov qword ptr [rsp], rax
	main_03.go:5	0x105e144	e8f761faff		call $runtime.closechan
	main_03.go:6	0x105e149	488b442418		mov rax, qword ptr [rsp+0x18]
	main_03.go:6	0x105e14e	48890424		mov qword ptr [rsp], rax
	main_03.go:6	0x105e152	e8e961faff		call $runtime.closechan
	main_03.go:7	0x105e157	488b6c2420		mov rbp, qword ptr [rsp+0x20]
	main_03.go:7	0x105e15c	4883c428		add rsp, 0x28
	main_03.go:7	0x105e160	c3			ret
	main_03.go:2	0x105e161	e89ab2ffff		call $runtime.morestack_noctxt
	.:0		0x105e166	eb98			jmp $main.main
(dlv) b runtime.closechan
Breakpoint 2 set at 0x1004353 for runtime.closechan() /usr/local/go/src/runtime/chan.go:355
(dlv) c 
> runtime.closechan() /usr/local/go/src/runtime/chan.go:355 (hits goroutine(1):1 total:1) (PC: 0x1004353)
Warning: debugging optimized function
   350:		src := sg.elem
   351:		typeBitsBulkBarrier(t, uintptr(dst), uintptr(src), t.size)
   352:		memmove(dst, src, t.size)
   353:	}
   354:	
=> 355:	func closechan(c *hchan) {
   356:		if c == nil {
   357:			panic(plainError("close of nil channel"))
   358:		}
   359:	
   360:		lock(&c.lock)
(dlv) 




## Q2
选做：
修正压缩包中的 ast_map_expr 文件夹中的 test，使 test 可以通过

