# Digital-clock-based-on-the-FPGA-EGO1-development-board
（The development platform is Vivado 2017.4.）The code implements a 24-hour counter.   Pressing button S3 lets you adjust the hour and minute digits.   Five seconds before every full hour an alarm sounds.   Slide switch SW0 upward to freeze the clock.
# EGO1数字钟项目 - Vivado 2017.4

## 项目概述

这是一个基于EGO1 FPGA开发板（Xilinx Artix-7 XC7A35T-1CSG324C）的24小时制数字钟项目。

## ⚠️ 重要说明

**EGO1数码管特性：**
- EGO1使用**共阴极**数码管
- FPGA输出**高电平有效**（位选和段选都是高电平有效）
- 有**两组独立的4位数码管**，各有独立的段选信号
  - 第一组(seg_data_0): 右边4位
  - 第二组(seg_data_1): 左边4位

## 功能特性

### 1. 基础功能
- 24小时制时间显示（HH-MM-SS格式）
- 8位7段数码管显示时、分、秒

### 2. 整点报时功能
- 整点前5秒开始提示（秒数55-59）
- 16个LED同时闪烁
- 整点到达后停止

### 3. 校时功能
- **按键S3（模式选择）**：
  - 第1次按：进入小时校时模式（小时位闪烁）
  - 第2次按：进入分钟校时模式（分钟位闪烁）
  - 第3次按：退出校时，恢复正常计时
  
- **按键S4（加1）**：
  - 在校时模式下，每按一次当前校时的数字加1
  - 小时：00-23循环
  - 分钟：00-59循环

### 4. 按键消抖
- S3和S4按键均做20ms消抖处理

## 硬件连接

| 功能 | 引脚 | FPGA Pin |
|------|------|----------|
| 系统时钟 | 100MHz | P17 |
| 复位按键S6 | FPGA_RESET | P15 |
| 模式按键S3 | PB3 | V1 |
| 加1按键S4 | PB4 | U4 |

## 文件结构

```
digital_clock_project/
├── src/
│   ├── digital_clock_top.v    # 顶层模块
│   ├── clk_divider.v          # 时钟分频模块
│   ├── key_debounce.v         # 按键消抖模块
│   ├── clocks_ctrl.v          # 时钟控制模块
│   ├── chime_module.v         # 整点报时模块
│   ├── led_controller.v       # LED控制模块
│   └── seg_display.v          # 7段数码管显示模块
├── constraints/
│   └── digital_clock.xdc      # 引脚约束文件
├── create_project.tcl         # Vivado项目创建脚本
└── README.md                  # 本说明文件
```

## Vivado操作步骤

### 方法一：使用TCL脚本创建项目

1. 打开Vivado 2017.4
2. 在Tcl Console中执行：
   ```tcl
   cd {项目所在路径}/digital_clock_project
   source create_project.tcl
   ```
3. 等待项目创建完成

### 方法二：手动创建项目

1. **创建新项目**
   - File → New Project
   - 项目名称：digital_clock
   - 选择项目路径
   - Project Type: RTL Project
   - 不勾选"Do not specify sources at this time"

2. **添加源文件**
   - Add Sources → Add or create design sources
   - 添加src目录下所有.v文件
   - 设置digital_clock_top为顶层模块

3. **添加约束文件**
   - Add Sources → Add or create constraints
   - 添加constraints/digital_clock.xdc

4. **选择器件**
   - Parts → xc7a35tcsg324-1

5. **综合与实现**
   - Flow Navigator → Run Synthesis
   - 等待综合完成
   - Run Implementation
   - 等待实现完成
   - Generate Bitstream

6. **下载到开发板**
   - Open Hardware Manager
   - Open Target → Auto Connect
   - 右键FPGA器件 → Program Device
   - 选择生成的.bit文件
   - Program

## 操作说明

1. **正常运行**
   - 上电后显示初始时间12:00:00
   - 时钟自动运行

2. **校时操作**
   - 按S3进入小时校时（小时位闪烁）
   - 按S4调整小时值
   - 再按S3进入分钟校时（分钟位闪烁）
   - 按S4调整分钟值
   - 再按S3退出校时

3. **复位**
   - 按S6复位，时间回到12:00:00

4. **整点报时**
   - 每到整点前5秒，LED自动闪烁
   - 整点时停止

## 模块说明

### clk_divider - 时钟分频模块
- 输入：100MHz系统时钟
- 输出：1Hz（秒计数）、10Hz（闪烁）、1kHz（数码管扫描）

### key_debounce - 按键消抖模块
- 20ms消抖时间
- 输出按键按下的单脉冲信号

### clocks_ctrl - 时钟控制模块
- BCD格式计数
- 支持正常计时和校时两种模式
- 状态机控制模式切换

### seg_display - 数码管显示模块
- 8位动态扫描显示（两组同时扫描）
- 共阴极数码管，高电平有效
- 支持指定位闪烁

### chime_module - 整点报时模块
- 检测xx:59:55~xx:59:59时段
- 输出报时激活信号

### led_controller - LED控制模块
- 报时时LED闪烁
- 非报时时LED熄灭

## 注意事项

1. 确保使用正确的FPGA型号：xc7a35tcsg324-1
2. EGO1数码管是**共阴极**，高电平有效
3. 两组数码管各有独立的段选信号，不能共用
4. LED为高电平点亮

## 常见问题

Q: 数码管不亮？
A: 检查位选和段选信号极性，EGO1是高电平有效

Q: 只有部分数码管亮？
A: 检查是否正确连接了两组段选信号(seg_data_0和seg_data_1)

Q: 数码管显示全8？
A: 检查段选信号是否正确，应为高电平点亮

Q: 按键无响应？
A: 检查按键消抖时间设置，确认按键引脚映射正确

Q: 时钟不走？
A: 检查时钟分频是否正确，确认复位信号状态

## 更新日志

- 修正了数码管极性（共阴极，高电平有效）
- 添加了第二组段选信号输出
- 修正了段码表
- 修正了位选信号极性
