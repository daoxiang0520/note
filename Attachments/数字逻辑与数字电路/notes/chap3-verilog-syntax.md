---
title: Verilog HDL 基本语法
date: 2026-06-15
tags: [数字逻辑, 武汉大学, Verilog, HDL语法]
---

# Verilog HDL 基本语法

## 1. 模块结构 (Module Structure)

Verilog HDL 的基本设计单元是模块 (module)。

```verilog
module <模块名> (<端口列表>);
    // 端口声明
    input 端口1, 端口2, ...;
    output 端口1, 端口2, ...;
    inout 双向端口1, ...;

    // 数据类型声明
    wire 连线名;
    reg 寄存器名;
    parameter 参数名 = 参数值;

    // 功能描述
    // assign 连续赋值
    assign <wire变量> = <表达式>;

    // 模块实例化
    <模块名> <实例名> (<端口连接>);

    // always 过程块
    always @(<敏感事件>)
    begin
        // 过程赋值
        // if-else, case
        // for, while, repeat, forever
        // task, function 调用
    end
endmodule
```

### 1.1 端口声明

| 端口方向 | 关键字 | 说明 |
|---------|--------|------|
| 输入 | `input` |  |
| 输出 | `output` |  |
| 双向 | `inout` |  |

端口可以指定位宽：`input [msb:lsb] 端口名`

示例：
```verilog
module adder1 (A, B, CI, S, CO);
    input A, B, CI;
    output S, CO;
endmodule

module BCD_adder (A, B, CIN, SUM, COUT);
    input [3:0] A, B;
    output [3:0] SUM;
    input CIN;
    output COUT;
endmodule
```

### 1.2 注释

- 单行注释: `//`
- 多行注释: `/* ... */`

### 1.3 语句结束符

每条语句以分号 `;` 结束。`endmodule` 后无分号。

---

## 2. 数据类型

### 2.1 wire (连线)

- 表示结构实体之间的物理连线
- 不能存储值，必须由驱动源连续驱动
- 用于 `assign` 连续赋值语句、模块实例化端口连接

```verilog
wire [2:0] a, b, c;  // 3位宽wire
wire abc;             // 1位宽wire
```

### 2.2 reg (寄存器)

- 表示数据存储单元
- 在 `always` 或 `initial` 块中被赋值
- 保持最后一次赋值的值

```verilog
reg [4:1] sum;  // 4位reg (bit 4 down to 1)
reg [2:0] a, b, c;  // 3位reg
```

### 2.3 其他数据类型

- `integer` — 32位有符号整数
- `real` — 64位浮点数
- `time` — 64位无符号时间
- `memory` — 存储器类型

---

## 3. 常量与参数

### 3.1 数值表示

格式：`<位宽>'<进制><数值>`

| 进制 | 符号 | 示例 |
|------|------|------|
| 二进制 | `b` / `B` | `8'b10110001` |
| 十进制 | `d` / `D` | `125` |
| 十六进制 | `h` / `H` | `8'hf5` |
| 八进制 | `o` / `O` |  |

### 3.2 字符串

用双引号括起，如 `"Hello World!"`，每个字符占 8 位。

### 3.3 参数 (parameter)

```verilog
parameter PI = 3.14;
parameter sign = "Hello World";
```

---

## 4. 运算符

### 4.1 算术运算符

| 运算符 | 说明 |
|--------|------|
| `+` | 加 |
| `-` | 减 |
| `*` | 乘 |
| `/` | 除 |
| `%` | 取模 |

### 4.2 逻辑运算符

| 运算符 | 说明 |
|--------|------|
| `&&` | 逻辑与 |
| `\|\|` | 逻辑或 |
| `!` | 逻辑非 |

### 4.3 关系运算符

| 运算符 | 说明 |
|--------|------|
| `>` | 大于 |
| `<` | 小于 |
| `>=` | 大于等于 |
| `<=` | 小于等于(注意:与非阻塞赋值区分) |
| `==` | 逻辑相等 |
| `!=` | 逻辑不等 |
| `===` | 全等(含x和z) |
| `!==` | 不全等 |

### 4.4 位运算符

| 运算符 | 说明 |
|--------|------|
| `~` | 按位取反 |
| `&` | 按位与 |
| `\|` | 按位或 |
| `^` | 按位异或 |
| `^~` / `~^` | 按位同或 |

### 4.5 移位运算符

| 运算符 | 说明 |
|--------|------|
| `>>` | 右移，高位补0 |
| `<<` | 左移，低位补0 |

### 4.6 拼接运算符

`{ , }` — 将多个信号拼接为一个总线信号。

```verilog
X = {a[7:4], b[3], c[2:0]};
// X = a[7:4] + b[3] + c[2:0]
```

### 4.7 条件运算符

`? :` — 三目运算符。

```verilog
d = (a == 1) ? b : c;
// a=1 时 d=b，否则 d=c
```

### 4.8 运算符优先级

| 优先级 | 运算符 |
|--------|--------|
| 最高 | `!` `~` |
| | `*` `/` `%` |
| | `+` `-` |
| | `<<` `>>` |
| | `<` `<=` `>` `>=` |
| | `==` `!=` `===` `!==` |
| | `&` `~&` |
| | `^` `^~` |
| | `\|` `~\|` |
| | `&&` |
| 最低 | `\|\|` |
| | `? :` |

---

## 5. 电路建模方式

### 5.1 门级建模 (Gate-Level)

使用 Verilog 内置的基本门元件。

```verilog
// 语法: <门类型> <实例名>(输出, 输入1, 输入2, ...);
and a1(out, in1, in2, in3);
or  o1(out, in1, in2);
not n1(out, in);
```

**2选1 MUX 门级实现示例：**

```verilog
module MUX1(out, a, b, sel);
    output out;
    input a, b, sel;
    wire sel_, a1, a2;
    not (sel_, sel);
    and (a1, a, sel_);
    and (a2, b, sel);
    or (out, a1, a2);
endmodule
```

### 5.2 连续赋值 (Assign)

```verilog
assign L_wire = R_expression;
```

- 左值必须是 `wire` 类型
- 右值变化时自动重新计算

**2选1 MUX assign 实现示例：**

```verilog
module MUX1(out, a, b, sel);
    output out;
    input a, b, sel;
    assign out = (a && !sel) || (b && sel);
endmodule
```

### 5.3 过程块 (always)

```verilog
always @(<敏感事件表达式>)
begin
    // 过程赋值
    // if-else, case, casex, casez
    // while, repeat, for
    // task, function 调用
end
```

- 被赋值的变量必须是 `reg` 类型
- 多条语句使用 `begin...end` 块

### 5.4 敏感事件列表

| 敏感事件 | 说明 |
|----------|------|
| `always @(a)` | a 变化时触发 |
| `always @(a or b)` | a 或 b 变化时触发 |
| `always @(posedge clock)` | clock 上升沿触发 |
| `always @(negedge clock)` | clock 下降沿触发 |
| `always @(posedge clk or negedge reset)` | clk上升沿或reset下降沿触发 |

---

## 6. 赋值语句

### 6.1 连续赋值 (assign)

用于组合逻辑，对 `wire` 赋值。

```verilog
assign #1 y = a & b & c & d;
// 输入变化后延迟1个时间单位更新y
```

### 6.2 阻塞赋值 (`=`)

- 顺序执行，前面的赋值完成后才执行下一条
- 在 `always` / `initial` 块中使用

```verilog
always @(posedge clk)
begin
    b = a;  // b 先得到 a 的值
    c = b;  // c 得到更新后的 b (=a)
end
// 结果: a=b=c
```

### 6.3 非阻塞赋值 (`<=`)

- 并行执行，所有右值在块开始时读取，块结束时更新
- 用于描述时序逻辑

```verilog
always @(posedge clk)
begin
    b <= a;
    c <= b;
end
// 结果: b=a, c=b(旧值)
```

---

## 7. 条件与分支语句

### 7.1 if-else

```verilog
if (条件1)
    语句1;
else if (条件2)
    语句2;
else if (条件3)
    语句3;
else
    语句n;
```

多条语句用 `begin...end` 括起。

### 7.2 case

```verilog
case (表达式)
    值1: 语句1;
    值2: 语句2;
    ...
    default: 默认语句;
endcase
```

**4选1 MUX case 实现示例：**

```verilog
module mux4_1(out, in0, in1, in2, in3, sel);
    output out;
    input in0, in1, in2, in3;
    input [1:0] sel;
    reg out;

    always @(in0 or in1 or in2 or in3 or sel)
    begin
        case(sel)
            2'b00: out = in0;
            2'b01: out = in1;
            2'b10: out = in2;
            2'b11: out = in3;
            default: out = 2'bx;
        endcase
    end
endmodule
```

**七段数码管译码器 case 实现示例：**

```verilog
module decode4_7(decodeout, indec);
    output [6:0] decodeout;
    input [3:0] indec;
    reg [6:0] decodeout;

    always @(indec)
    begin
        case(indec)
            4'd0: decodeout = 7'b1111110;
            4'd1: decodeout = 7'b0110000;
            4'd2: decodeout = 7'b1101101;
            4'd3: decodeout = 7'b1111001;
            4'd4: decodeout = 7'b0110011;
            4'd5: decodeout = 7'b1011011;
            4'd6: decodeout = 7'b1011111;
            4'd7: decodeout = 7'b1110000;
            4'd8: decodeout = 7'b1111111;
            4'd9: decodeout = 7'b1111011;
            default: decodeout = 7'bx;
        endcase
    end
endmodule
```

---

## 8. 循环语句

### 8.1 forever

- 无条件持续执行
- 配合 `initial` 使用，常用于生成时钟信号

```verilog
initial
begin
    clk = 1'b0;
    forever #10 clk = ~clk;  // 产生周期20的时钟
end
```

### 8.2 repeat

- 重复固定次数

```verilog
repeat (次数)
    语句;
```

### 8.3 while

```verilog
while (条件)
    语句;
```

### 8.4 for

```verilog
for (初始赋值; 条件; 循环变量增减)
begin
    // 循环体
end
```

**7人投票器 for 循环实现示例：**

```verilog
module voter7(pass, vote);
    output pass;
    input [6:0] vote;
    reg [2:0] sum;
    integer i;
    reg pass;

    always @(vote)
    begin
        sum = 0;
        for (i = 0; i <= 6; i = i + 1)
            if (vote[i]) sum = sum + 1;

        if (sum[2]) pass = 1;  // 若sum>=4
        else pass = 0;
    end
endmodule
```

---

## 9. task 与 function

### 9.1 task (任务)

- 可以包含时序控制 (`#`, `@`, `wait`)
- 可以有多个输出
- 可以调用其他 task 和 function

```verilog
task <任务名>;
    input 输入端口;
    output 输出端口;
    inout 双向端口;
    begin
        // 任务体
    end
endtask
```

**任务调用示例：**

```verilog
module add16(a, b, cin, sum, cout);
    input [15:0] a, b;
    input cin;
    output [15:0] sum;
    output cout;
    reg [15:0] sum;
    reg cout;
    reg c8;

    always @(a or b or cin)
    begin
        adder8(a[7:0], b[7:0], cin, sum[7:0], c8);
        adder8(a[15:8], b[15:8], c8, sum[15:8], cout);
    end

    task adder8;
        input [7:0] ta, tb;
        input tcin;
        output [7:0] tsum;
        output tcout;
        begin
            {tcout, tsum} = ta + tb + tcin;
        end
    endtask
endmodule
```

### 9.2 function (函数)

- 不能包含时序控制
- 只有一个返回值（与函数名同名的寄存器）
- 至少有一个输入

```verilog
function <位宽范围> <函数名>;
    input 输入端口;
    begin
        // 函数体
        <函数名> = 返回值;
    end
endfunction
```

**函数调用示例：**

```verilog
function [2:0] get0;
    input [7:0] x;
    reg [2:0] count;
    integer i;
    begin
        count = 0;
        for (i = 0; i <= 7; i = i + 1)
            if (x[i] == 1'b0) count = count + 1;
        get0 = count;  // 返回 x 中 0 的个数
    end
endfunction
```

### 9.3 task 与 function 的区别

| 比较项 | task | function |
|--------|------|----------|
| 时序控制 | 可以包含 | 不能包含 |
| 返回值 | 多个输出端口 | 一个返回值(函数名) |
| 调用 | 可调用其他task/function | 只能调用function |
| 输入 | 可无输入 | 至少一个输入 |

---

## 10. 半加器与全加器模块设计示例

```verilog
module half_adder(A, B, S, C);  // 半加器
    input A, B;
    output S, C;
    assign S = A ^ B;
    assign C = A & B;
endmodule

module full_adder(A, B, Cin, S, Co);  // 全加器
    input A, B, Cin;
    output S, Co;
    wire C1, C2, S1;
    half_adder u1(A, B, S1, C1);
    half_adder u2(Cin, S1, S, C2);
    assign Co = C1 | C2;
endmodule
```

---

## 编译预处理指令

| 指令 | 说明 |
|------|------|
| `` `define `` | 宏定义 |
| `` `include `` | 文件包含 |
| `` `ifdef `` / `` `else `` / `` `endif `` | 条件编译 |
