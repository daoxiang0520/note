---
title: 常用组合逻辑电路
date: 2026-06-15
tags: [数字逻辑, 武汉大学, 组合逻辑, 加法器, 乘法器, 多路选择器, 译码器, 编码器, 比较器]
---

# 常用组合逻辑电路

## 4.4 常用组合逻辑电路

---

## 1. 加法器 (Adders)

### 1.1 半加器 (Half Adder, HA)

半加器用于两个1位二进制数相加，产生和(S)与进位(C)。

| A | B | S | C |
|---|---|---|---|
| 0 | 0 | 0 | 0 |
| 0 | 1 | 1 | 0 |
| 1 | 0 | 1 | 0 |
| 1 | 1 | 0 | 1 |

**逻辑表达式：**
- `S = A XOR B = A + B`
- `C = A AND B = AB`

**Verilog 实现：**

```verilog
module half_adder(A, B, S, C);
    input A, B;
    output S, C;
    assign S = A ^ B;
    assign C = A & B;
endmodule
```

### 1.2 全加器 (Full Adder, FA)

全加器将两个加数位和来自低位的进位相加，产生和(S)与进位(Ci+1)。

| Ai | Bi | Ci-1 | Fi | Ci |
|----|----|------|----|----|
| 0  | 0  | 0    | 0  | 0  |
| 0  | 0  | 1    | 1  | 0  |
| 0  | 1  | 0    | 1  | 0  |
| 0  | 1  | 1    | 0  | 1  |
| 1  | 0  | 0    | 1  | 0  |
| 1  | 0  | 1    | 0  | 1  |
| 1  | 1  | 0    | 0  | 1  |
| 1  | 1  | 1    | 1  | 1  |

**逻辑表达式：**
- `Fi = Ai XOR Bi XOR Ci-1 = sum(m(1,2,4,7))`
- `Ci = AiBi + Ci-1(Ai XOR Bi) = sum(m(3,5,6,7))`
- `Ci = AiBi + AiCi-1 + BiCi-1`

**Verilog 实现：**

```verilog
module adder_full(ia, ib, ic, os, oc);
    input ia, ib, ic;
    output os, oc;
    assign {oc, os} = ia + ib + ic;
endmodule
```

**用两个半加器构成全加器：**

```verilog
module full_adder(A, B, Cin, S, Co);
    input A, B, Cin;
    output S, Co;
    wire C1, C2, S1;
    half_adder u1(A, B, S1, C1);
    half_adder u2(Cin, S1, S, C2);
    assign Co = C1 | C2;
endmodule
```

### 1.3 串行进位加法器 (Ripple Carry Adder, RCA)

n个全加器串联，进位逐级传递。

```
S1 = A1 XOR B1 XOR C0
C1 = A1B1 + C0(A1 XOR B1)

S2 = A2 XOR B2 XOR C1
C2 = A2B2 + C1(A2 XOR B2)

S3 = A3 XOR B3 XOR C2
C3 = A3B3 + C2(A3 XOR B3)

S4 = A4 XOR B4 XOR C3
C4 = A4B4 + C3(A4 XOR B4)
```

### 1.4 超前进位加法器 (Carry Look-ahead Adder, CLA)

通过并行计算所有进位，减少延迟。

**定义：**
- 进位产生函数: `Gi = Ai * Bi`
- 进位传递函数: `Pi = Ai XOR Bi`

**进位公式：**

```
C1 = G1 + P1*C0
C2 = G2 + P2*G1 + P2*P1*C0
C3 = G3 + P3*G2 + P3*P2*G1 + P3*P2*P1*C0
C4 = G4 + P4*G3 + P4*P3*G2 + P4*P3*P2*G1 + P4*P3*P2*P1*C0
```

### 1.5 4位超前进位加法器 74LS283

74LS283 是一种常用的4位二进制超前进位加法器芯片。

```
           S4 S3 S2 S1
             |
      74LS283
C4 --|        |-- C0
     A4A3A2A1 B4B3B2B1
```

### 1.6 乘法器 (Multipliers)

**2位二进制乘法器 (A=A1A0, B=B1B0)：**

```
P = A * B = A1A0 * B1B0

P0 = A0 * B0
P1 = A1*B0 XOR A0*B1
P2 = A1*B1 + C1(C1为A1*B0 + A0*B1的进位)
P3 = C2(C2为A1*B1 + C1的进位)
```

---

## 2. 编码器 (Encoders)

### 2.1 8-3 线编码器

将 8 个输入信号编码为 3 位二进制输出。

**逻辑表达式：**
- `Y2 = I4 + I5 + I6 + I7`
- `Y1 = I2 + I3 + I6 + I7`
- `Y0 = I1 + I3 + I5 + I7`

**Verilog 实现：**

```verilog
module encoder8_3(iIN_N, oY_N);
    input [7:0] iIN_N;
    output reg [2:0] oY_N;

    always @(iIN_N)
    begin
        case(iIN_N)
            8'b01111111: oY_N = 3'b000;
            8'b10111111: oY_N = 3'b001;
            8'b11011111: oY_N = 3'b010;
            8'b11101111: oY_N = 3'b011;
            8'b11110111: oY_N = 3'b100;
            8'b11111011: oY_N = 3'b101;
            8'b11111101: oY_N = 3'b110;
            8'b11111110: oY_N = 3'b111;
            default: oY_N = 3'bxxx;
        endcase
    end
endmodule
```

### 2.2 优先编码器 74LS148

8-3线优先编码器，优先权最高的输入线有最高优先级。

**引脚功能：**
- `I0~I7` — 输入（低电平有效），I7 优先级最高
- `Y0~Y2` — 编码输出（低电平有效）
- `ST` — 使能输入（低电平有效）
- `YS` — 选通输出
- `YEX` — 扩展输出

**功能表：**

| ST | 输入 | 输出 |
|----|------|------|
| 1  | 任意 | Y=111, YS=1, YEX=1 |
| 0  | 有输入 | 输出对应编码, YS=1, YEX=0 |
| 0  | 无输入 | Y=111, YS=0, YEX=1 |

**两片 74LS148 级联构成 16-4 线编码器：**

将两片 74LS148 级联，可扩展为 16-4 线编码器。

- 输出编码范围：0000~0111（芯片2），1000~1111（芯片1）
- `Y3 = 1`（芯片1有编码请求时）
- `Y2 = Y22 + Y12`
- `Y1 = Y21 + Y11`
- `Y0 = Y20 + Y10`

---

## 3. 译码器 (Decoders)

### 3.1 n-2^n 线译码器

将 n 位输入代码翻译为 2^n 位输出，每个输出对应一个最小项。

**3-8 译码器真值表：**

| A | B | C | Y0| Y1| Y2| Y3| Y4| Y5| Y6| Y7 |
|---|---|---|---|---|---|---|---|---|---|---|
| 0 | 0 | 0 | 1 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| 0 | 0 | 1 | 0 | 1 | 0 | 0 | 0 | 0 | 0 | 0 |
| 0 | 1 | 0 | 0 | 0 | 1 | 0 | 0 | 0 | 0 | 0 |
| 0 | 1 | 1 | 0 | 0 | 0 | 1 | 0 | 0 | 0 | 0 |
| 1 | 0 | 0 | 0 | 0 | 0 | 0 | 1 | 0 | 0 | 0 |
| 1 | 0 | 1 | 0 | 0 | 0 | 0 | 0 | 1 | 0 | 0 |
| 1 | 1 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 1 | 0 |
| 1 | 1 | 1 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 1 |

**输出表达式：**
- `Y0 = ABC`, `Y1 = ABC`, `Y2 = ABC`, `Y3 = ABC`
- `Y4 = ABC`, `Y5 = ABC`, `Y6 = ABC`, `Y7 = ABC`

### 3.2 3-8 译码器 74LS138

**引脚功能：**
- `A2, A1, A0` — 地址输入
- `Y0~Y7` — 译码输出（低电平有效）
- `STA` — 使能输入1（高电平有效）
- `STB, STC` — 使能输入2、3（低电平有效）

**功能表：**

| STA | STB | STC | A2 | A1 | A0 | 输出 |
|-----|-----|-----|----|----|----|------|
| 0   | x   | x   | x  | x  | x  | 全1 |
| x   | 1   | x   | x  | x  | x  | 全1 |
| x   | x   | 1   | x  | x  | x  | 全1 |
| 1   | 0   | 0   | 0  | 0  | 0  | Y0=0，其余1 |
| 1   | 0   | 0   | 0  | 0  | 1  | Y1=0，其余1 |
| ... | ... | ... | ... | ... | ... | ... |
| 1   | 0   | 0   | 1  | 1  | 1  | Y7=0，其余1 |

**Verilog 实现：**

```verilog
module decoder3_8(iSTA, iSTB_N, iSTC_N, iA, oY_N);
    input iSTA, iSTB_N, iSTC_N;
    input [2:0] iA;
    output [7:0] oY_N;
    reg [7:0] m_y;
    assign oY_N = m_y;

    always @(iSTA, iSTB_N, iSTC_N, iA)
    begin
        if (iSTA && !(iSTB_N || iSTC_N))
            case(iA)
                3'b000: m_y = 8'b01111111;
                3'b001: m_y = 8'b10111111;
                3'b010: m_y = 8'b11011111;
                3'b011: m_y = 8'b11101111;
                3'b100: m_y = 8'b11110111;
                3'b101: m_y = 8'b11111011;
                3'b110: m_y = 8'b11111101;
                3'b111: m_y = 8'b11111110;
            endcase
        else
            m_y = 8'hff;
    end
endmodule
```

### 3.3 两片 74LS138 构成 4-16 线译码器

将 A3 作为片选信号，实现 4-16 线译码。

- A3=0：芯片2工作，输出 0000~0111 (0~7)
- A3=1：芯片1工作，输出 1000~1111 (8~15)

### 3.4 译码器实现逻辑函数

利用译码器产生全部最小项的特性，可以实现任意逻辑函数。

**示例：用 3-8 译码器实现 F(A,B,C) = sum(m1,m4,m5,m7)**

```
F = Y1 + Y4 + Y5 + Y7
```

连接方式：将 74LS138 的输出 Y1、Y4、Y5、Y7 通过与非门(或与门)输出。

### 3.5 BCD-十进制译码器 74LS42

4 线输入（BCD 码），10 线输出（低电平有效）。

| A3 A2 A1 A0 | 0~9 输出 |
|-------------|----------|
| 0000~1001   | 对应位为0，其余为1 |
| 1010~1111   | 全1（伪码） |

### 3.6 七段数码管译码器 74LS48

将 BCD 码转换为七段数码管显示驱动信号。

| 输入 | a | b | c | d | e | f | g | 显示 |
|------|---|---|---|---|---|---|---|---|
| 0000 | 1 | 1 | 1 | 1 | 1 | 1 | 0 | 0 |
| 0001 | 0 | 1 | 1 | 0 | 0 | 0 | 0 | 1 |
| 0010 | 1 | 1 | 0 | 1 | 1 | 0 | 1 | 2 |
| 0011 | 1 | 1 | 1 | 1 | 0 | 0 | 1 | 3 |
| 0100 | 0 | 1 | 1 | 0 | 0 | 1 | 1 | 4 |
| 0101 | 1 | 0 | 1 | 1 | 0 | 1 | 1 | 5 |
| 0110 | 0 | 0 | 1 | 1 | 1 | 1 | 1 | 6 |
| 0111 | 1 | 1 | 1 | 0 | 0 | 0 | 0 | 7 |
| 1000 | 1 | 1 | 1 | 1 | 1 | 1 | 1 | 8 |
| 1001 | 1 | 1 | 1 | 1 | 0 | 1 | 1 | 9 |

**Verilog 实现 (带共阳极/共阴极控制)：**

```verilog
module seg_decoder(iflag, iA, oY);
    input iflag;          // 1:共阴极, 0:共阳极
    input [3:0] iA;       // BCD码输入
    output reg [6:0] oY;  // 7段码输出

    always @(iflag, iA)
    begin
        case(iA)
            4'b0000: oY = 7'h3f;   // 0
            4'b0001: oY = 7'h06;   // 1
            4'b0010: oY = 7'h5b;   // 2
            4'b0011: oY = 7'h4f;   // 3
            4'b0100: oY = 7'h66;   // 4
            4'b0101: oY = 7'h6d;   // 5
            4'b0110: oY = 7'h7d;   // 6
            4'b0111: oY = 7'h27;   // 7
            4'b1000: oY = 7'h7f;   // 8
            4'b1001: oY = 7'h6f;   // 9
            4'b1010: oY = 7'h77;   // A
            4'b1011: oY = 7'h7c;   // b
            4'b1100: oY = 7'h58;   // c
            4'b1101: oY = 7'h5e;   // d
            4'b1110: oY = 7'h79;   // E
            4'b1111: oY = 7'h71;   // F
        endcase
        if (!iflag)
            oY = ~oY;  // iflag=0 时取反(共阳极)
    end
endmodule
```

---

## 4. 数据选择器 (Multiplexer, MUX)

### 4.1 4选1 数据选择器

数据选择器根据地址选择信号，从多路输入中选择一路输出。

**功能：**
```
F = A1A0 * D0 + A1A0 * D1 + A1A0 * D2 + A1A0 * D3
  = sum(mi * Di)  for i = 0 to 3
```

**Verilog 实现：**

```verilog
module mux4(iD, iS, oQ);
    input [3:0] iD;    // 4路数据输入
    input [1:0] iS;    // 地址选择
    output reg oQ;     // 输出

    always @(iD, iS)
    begin
        case(iS)
            2'b00: oQ = iD[0];
            2'b01: oQ = iD[1];
            2'b10: oQ = iD[2];
            2'b11: oQ = iD[3];
        endcase
    end
endmodule
```

### 4.2 8选1 数据选择器 74LS151

**引脚功能：**
- `A2, A1, A0` — 地址选择
- `D0~D7` — 8路数据输入
- `Y` — 原码输出
- `W` — 反码输出
- `EN` — 使能（低电平有效）

**输出表达式：**
```
Y = A2A1A0*D0 + A2A1A0*D1 + ... + A2A1A0*D7
  = sum(mi * Di)  for i = 0 to 7
```

### 4.3 用 MUX 实现逻辑函数

利用 MUX 的 `sum(mi * Di)` 特性，将 Di 接固定电平(0/1)或输入变量，可实现任意逻辑函数。

### 4.4 数据分配器 (Demultiplexer, DMUX)

数据分配器将一路输入数据分配到多路输出中的某一特定通道。

**Verilog 1-8 数据分配器：**

```verilog
module dmux_8(iEN, iS, iD, oY);
    input iEN;              // 使能
    input iD;               // 数据输入
    input [2:0] iS;         // 地址选择
    output reg [7:0] oY;    // 输出

    always @(iD, iEN, iS)
    begin
        oY = 8'b11111111;
        if (iEN)
            case(iS)
                3'b000: oY[0] = iD;
                3'b001: oY[1] = iD;
                3'b010: oY[2] = iD;
                3'b011: oY[3] = iD;
                3'b100: oY[4] = iD;
                3'b101: oY[5] = iD;
                3'b110: oY[6] = iD;
                3'b111: oY[7] = iD;
            endcase
    end
endmodule
```

---

## 5. 数值比较器 (Comparators)

### 5.1 1位比较器

比较两个1位二进制数 A 和 B。

| A | B | FA>B | FA=B | FA<B |
|---|---|------|------|------|
| 0 | 0 | 0    | 1    | 0    |
| 0 | 1 | 0    | 0    | 1    |
| 1 | 0 | 1    | 0    | 0    |
| 1 | 1 | 0    | 1    | 0    |

**逻辑表达式：**
- `FA>B = AB + AB = A`
- `FA=B = AB + AB = A XOR B`
- `FA<B = AB = B`

### 5.2 4位数值比较器 74LS85

**级联输入：** `I(A>B)`, `I(A=B)`, `I(A<B)`

**输出逻辑：**
```
FA>B = (A1>B1) + (A1=B1)(A0>B0) + (A1=B1)(A0=B0)I(A>B)
FA=B = (A1=B1)(A0=B0)I(A=B)
FA<B = (A1<B1) + (A1=B1)(A0<B0) + (A1=B1)(A0=B0)I(A<B)
```

### 5.3 多位比较器 Verilog 实现（参数化）

```verilog
module compare_n(A, B, AGB, ALB, AEB);
    parameter n = 4;
    input [n-1:0] A, B;
    output reg AGB, ALB, AEB;

    always @(A or B)
    begin
        AGB = 0;
        ALB = 0;
        AEB = 0;
        if (A > B)      AGB = 1;
        else if (A == B) AEB = 1;
        else             ALB = 1;
    end
endmodule
```

---

## 习题参考

### P168 #9
用 Verilog 实现 8-3 线编码器。

### P168 #18
用 74LS138 实现逻辑函数：
1. `f(A1,B2,C3) = sum(m(0,2,3,4,5,7))`
2. `f(A1,B2,C3) = sum(m(1,2,3,5,6))`

### 用 74LS138 实现全加器
利用 74LS138 的输出和与非门实现全加器的和 Si 与进位 Ci+1。
