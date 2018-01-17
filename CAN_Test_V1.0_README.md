# 程序修改记录  
* **原程序-TI例程**  
程序名称：`can_external_transmit`  
文件路径：`ti\controlSUITE\device_support\F2837xD\v210\F2837xD_examples_Cpu1`  
程序功能：CAN-A模块发送数据到CAN-B模块，CAN-B模块以中断方式接收数据  
  
* **修改说明**  
1. 修改时钟频率设定函数  
TI例程使用的外部晶振频率为20MHz，系统时钟频率设定为200MHz，用如下函数进行设置：  
`InitSysPll(XTAL_OSC, IMULT_20, FMULT_0, PLLCLK_BY_2);`  
而测试控制器使用的外部晶振频率为30MHz，为了保证系统时钟频率为200MHz不变，需要修改时钟频率设定函数：  
`InitSysPll(XTAL_OSC, IMULT_20, FMULT_0, PLLCLK_BY_6);`  
2. 修改CAN通信引脚定义  
将CAN-A模块的接收发送引脚由30和31改为70和71，取消CAN-B引脚初始化定义  
    ```
    //    GPIO_SetupPinMux(30, GPIO_MUX_CPU1, 1); //GPIO30 -  CANRXA
    //    GPIO_SetupPinOptions(30, GPIO_INPUT, GPIO_ASYNC);
    //    GPIO_SetupPinMux(31, GPIO_MUX_CPU1, 1); //GPIO31 - CANTXA
    //    GPIO_SetupPinOptions(31, GPIO_OUTPUT, GPIO_PUSHPULL);
        GPIO_SetupPinMux(70, GPIO_MUX_CPU1, 1); //GPIO70 -  CANRXA
        GPIO_SetupPinOptions(70, GPIO_INPUT, GPIO_ASYNC);
        GPIO_SetupPinMux(71, GPIO_MUX_CPU1, 1); //GPIO71 - CANTXA
        GPIO_SetupPinOptions(71, GPIO_OUTPUT, GPIO_PUSHPULL);
    //    GPIO_SetupPinMux(10, GPIO_MUX_CPU1, 2); //GPIO10 -  CANRXB
    //    GPIO_SetupPinOptions(10, GPIO_INPUT, GPIO_ASYNC);
    //    GPIO_SetupPinMux(8, GPIO_MUX_CPU1, 2);  //GPIO8 - CANTXB
    //    GPIO_SetupPinOptions(8, GPIO_OUTPUT, GPIO_PUSHPULL);
    ```  
3. 屏蔽CAN-B的初始化  
    ```
    //CANInit(CANB_BASE);
    //CANClkSourceSelect(CANB_BASE, 0);   // 500kHz CAN-Clock
    //CANBitRateSet(CANB_BASE, 200000000, 500000);
    //CANIntEnable(CANB_BASE, CAN_INT_MASTER | CAN_INT_ERROR | CAN_INT_STATUS);
    //CANGlobalIntEnable(CANB_BASE, CAN_GLB_INT_CANINT0);
    ```
4. 