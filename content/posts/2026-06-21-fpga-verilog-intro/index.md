---
title: "Verilog快速上手"
subtitle:
date: 2026-06-21T23:41:56+08:00
slug: ""
draft: false
description:
keywords:
weight: 0
categories: ["FPGA"]
collections: []
tags: ["FPGA", "Verilog", "HDL"]
summary:
password:
message:
repost:
  enable: false
  url:
---

# 介绍

**Verilog**是一种HDL(Hardware Description Language)，即**硬件描述语言**，用于设计和验证逻辑电路。

与C/C++,Java,Python等编程语言不同，HDL所描述的是电路结构和行为，相当于用结构化的语言来描述一个电路，这比电路图描述电路要更加自动化大规模化，最终综合得到的产物是可以直接映射到实际的电路结构上的。

这篇文章主要介绍verilog代码的基本写法：

## 1.1模块

`模块(module)`​是Verilog的基本结构。模块就像集成电路芯片，一个电路就是一个模块，而一个模块内部可以由若干个小模块组成，这些小模块内部也可以由更小的模块组成。实现功能的电路结构一定要包裹在`module <模块名>(<输入输出端口列表>)...endmodule`里。

下面我们用这个来描述一个电路，比如一个与门元件。与门有两个输入一个输出。所以我们将与门模块命名成`model and_gate(input a, input b, output c)`



比如一个1位半加器：

```verilog
module half_adder( 
	input a, 
	input b,
	output sum, 
	output cy	
)
assign sum = a ^ b; //和
assign cy = a & b;// 进位
endmodule
```

上面的Verilog代码模块对应的就是下面这个方框里的电路，这个方框就是我们定义的半加器模块。

![](https://img.synjadex.top/public/posts/2026-06-21-fpga-verilog-intro/half_adder.webp)

在half_adder的括号里定义的a,b,sum,cy就是端口，可以看到相当于芯片的引脚。



## 1.2 数据类型

Verilog主要有两种数据类型：

1. **线网型（wire）**：相当于导线，用于传导信号，连接各个端口和模块。
2. **寄存器型（reg）**:  表示一个存储单元，与wire最大的区别就是有**记忆存储**功能，能保存当前状态。



## 1.2 assign语句

`assign`语句用于标记**连续赋值**的两端信号，等号相当于“连接”，用一根导线将左右两边的端点相连。例如下面这个模块：

```verilog
module example(a,b,c,d,o)
	input a,b,c,d;
	output o;
	wire ab, cd;
	assign ab = a & b;
	assign cd = c ^ d;
	assign o = ~(ab | cd);
endmodule
```

