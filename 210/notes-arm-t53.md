
1. how to boot a arm board ?
   1.1 Boot device sequence(selection)
  	* nandflash		OM[5:0] = 000010	
		* sd/mmc		OM[5:0] = 001100	
		hardware connection
		led.bin(204bytes) -> sd-led.bin(8K)
		1. size = 8K
		2. byte 0x8-0xb -> checksum (sum of led.bin all bytes)
		3. if 1 && 2, then BL0 thought BL1 is integrity.

		/dev/sdb -> /dev/mmcblk0

   1.2 Device driver
	Watchdog -> 
	Interrupt -> 
	gpio(led) ->
	uart -> 
	clock ->
	sdram ->
	flash ->

   1.3 Bootloader
    uart -> getc, putc -> gets, puts, printf -> shell -> 
	command -> load/go
	command -> xmodem -> tftp -> usb
	command -> nand read/write/erase/mtd
        command -> linux boot

2. linux c program (vs) arm c program ?
   2.1 API (vs) SFR
   2.2 VA (vs) PA
   2.3 ELF (vs) BIN -> Load method


minicom
=======
0. sudo apt-get install minicom
1. sudo minicom -s
	pc: /dev/ttyS0
	usb->com: /dev/ttyUSB0

Interrupt
=========
1. test app1-EINT-demo.bin
	press K1
2. test by hands
	go 0x0
	go 0x8
	go 0x18
	md 0xd0037400
	mw 0xd0037418 0x21000000
	loadx (test.bin)
	go 0x18

Q & A
=====
1. 什么是异常？什么是中断？
	1) 属于内核 vs SoC
	2) 异常的分类 7种
	3) 异常模式 5种
	4) 异常向量表 32字节,8个入口,0x14, 指令,跳转, B, 0xEA000000, offset = (handler-pc)/4 (pos+8=pc)
	5) 异常的响应/现场保存和恢复/处理/返回
	-----------------------
	1) 属于板级（或芯片级）
	2) 中断源 和 中断控制器
	3) 响应，现场的保存和恢复，mask/pending, 写1清0
	4) 中断控制器的设计演变
		a) 跳转到异常向量表入口后跳转到handler
		b) 识别中断源
		c) 跳转到相应的ISR

2. 中断是怎样产生的？	*表示充分条件
	1) 中断源
		a) 使能中断源
		b) 设置触发方式
		c) 设置掩码允许通过
		*d) Pending 位清除
	2) 中断控制器
		a) 设置 Enable 位允许通过
			*设置 Clear 位屏蔽其他位
		*b) 设置 Mod 位选择 IRQ/FIQ
		*c) 设置优先级
	3) 内核
		a) CPSR 中的 I-bit 禁止位, 0=使能 
		*b) VIC 控制器使能向量工作方式
	4) 用户
		按键触发中断

3. 中断产生之后是如何处理的？
	0) 注册中断向量
		VIC0ADDR16 = asm_handler	(现场/模式返回)
	------------------------------------------------
	1) 现场保存
		ldr sp, =0xD0034000		
			(sp是IRQ模式下的栈，要初始化)
		sub lr, lr, #4	
			(返回被中断的那条，而不是下一条)
		stmfd sp!, {r0-r12, lr}
			(lr是否保存？取决于是否用到 BL 跳转或被修改)
	2) 中断处理
		a) putchar('a');
		b) 清除中断源的pending，IRQstatus 自动清0了
		c) 清除中断控制器的 VICADDR 最后硬件识别的向量(写一下)
	3) 中断的返回
		模式和地址必须同时返回
		a) 直接
			ldmfd sp!, {r0-r12, pc}^
			
		b) 间接
			ldmfd sp!, {r0-r12, lr}
			movs pc, lr


Timer 定时器
============

1. 控制器的工作时钟是怎么产生的？
	1) PCLK source
	2) level1: 8-bit prescaler
	3) level2: 2/4/8/16 divider

2. 定时中断是怎么产生的？
	1) If the down-counter reaches zero, 
	2) Timer Count Buffer register (TCNTBn)
	3) down-counter--
	4) set the timer enable bit of TCON
	5) set interrrupt enable bit

3. PWM 信号是怎么产生的？
	1) value of the TCMPBn register
	2) down-counter value matches the value of the compare register
	3) reload function












