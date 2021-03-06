读取文件示例
===========================

该示例从 **SD卡根目录** 中找到文件 **target.txt** 并读取其全部内容，然后用 **UART** 发送出去。如果 **UART** 连接到 电脑，可以看到文件内容。

该示例需要用到以下几个源文件：
* **./RTL** 下的 [top.sv](https://github.com/WangXuan95/FPGA-SDcard/blob/master/example/ReadFile/RTL/top.sv "top.sv")，是该示例的顶层
* **../../RTL** 下的 [SDFileReader.sv](https://github.com/WangXuan95/FPGA-SDcard/blob/master/RTL/SDFileReader.sv "SDFileReader.sv")、[SDDirParser.sv](https://github.com/WangXuan95/FPGA-SDcard/blob/master/RTL/SDDirParser.sv "SDDirParser.sv")、[SDReader.sv](https://github.com/WangXuan95/FPGA-SDcard/blob/master/RTL/SDReader.sv "SDReader.sv")、[SDCmdCtrl.sv](https://github.com/WangXuan95/FPGA-SDcard/blob/master/RTL/SDCmdCtrl.sv "SDCmdCtrl.sv")
* **../../UART** 下的 [uart_tx.sv](https://github.com/WangXuan95/FPGA-SDcard/blob/master/UART/uart_tx.sv "uart_tx.sv")、[ram.sv](https://github.com/WangXuan95/FPGA-SDcard/blob/master/UART/ram.sv "ram.sv")

你需要手动建立工程并为 **top.sv** 分配引脚。详见 **top.sv** 里的注释.

另外，**./Quartus** 目录中提供了一个基于 **Altera Cyclone IV FPGA** 的工程示例。但你多半需要重新为它调整器件型号和引脚分配，才能在你的 FPGA 板子上正确的运行。

# 推荐硬件电路

要运行该示例，你需要一个带有 **SD卡槽** 的 FPGA开发板（如Nexys4开发板、荔枝糖开发板），且推荐该开发板上有如下图的电路(注意上拉电阻)。

> 注: 很多 **SoC FPGA** 开发板也带有 **SD卡槽** ，但多数情况下该卡槽是与 **硬核处理器** 连接的。我们需要的是与 FPGA 连接的 **SD卡槽**。

![推荐硬件电路](https://github.com/WangXuan95/FPGA-SDcard/blob/master/images/sch.png)

# 运行结果

我用了手头所有能用的 SD卡 测试了该示例的兼容性，如下表：

| |  SDv1.1 2GB card | SDv2 128MB card  | SDv2 2GB card  | SDHCv2 8GB card |
| :------: | :------------: | :------------: | :------------: | :-----------: |
| **FAT16** | :heavy_check_mark:  |  :heavy_check_mark: | :heavy_check_mark:  | NaN\* |
| **FAT32** | :heavy_check_mark:  |  :heavy_check_mark: | :heavy_check_mark:  | :heavy_check_mark: |

>  注：因为 SDHCv2 似乎无法在 Windows 中格式化成 FAT16 格式，所以也没法测试

下图展示了使用 **SDHCv2 card** + **FAT32** 测试的结果。图中的 8 个 LED 来自 top.sv 中的输出端口 **output [7:0] led** ，编码为 **11111110** ， 最前面两个 **11** 代表SD卡类型为 **SDHCv2** ； 紧接着的两个 **11** 代表文件系统为 **FAT32** ；再接着的一个 *1* 代表找到目标文件（本示例中为 **target.txt** ，你可以修改）；最后的三个位 **110** 代表任务结束，这仅仅是 **SDFileReader.sv** 内的状态机码而已。 这 8 位 LED 的具体含义请见 **top.sv** 中的注释。

![测试结果照片](https://github.com/WangXuan95/FPGA-SDcard/blob/master/images/ReadFile.png)

另外， **最关键** 的是，该示例将 目标文件( **target.txt** ) 中的内容通过 **UART** 发送了出去( **波特率=115200** )，如果你的 FPGA 开发板有 UART，请将 **top.sv** 的 **output uart_tx** 引脚正确的分配，以在上位机中观察读到的文件内容。

最后注意，clk 的频率需要给 50MHz，如果你没办法给 50MHz，请按照注释修改一些参数后，程序照样能正确运行。另外， rst_n 信号是低电平复位的，你可以把它连接到按钮上，每次复位都能重新读取一遍 SD 卡。如果你的 FPGA 开发板没有按钮，请将 rst_n 赋值为 **1'b1**

