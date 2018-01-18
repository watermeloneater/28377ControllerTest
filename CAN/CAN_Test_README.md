# CAN通信测试-程序修改和调试记录  
* **原程序-TI例程**  
程序名称：`can_external_transmit`  
文件路径：`ti\controlSUITE\device_support\F2837xD\v210\F2837xD_examples_Cpu1`  
程序功能：CAN-A模块发送数据到CAN-B模块，CAN-B模块以中断方式接收数据  

* **修改后程序的功能**  
  * CAN-A模块的邮箱1设置为发送邮箱，CAN-A模块的邮箱2设置为接收邮箱  
  * 上位机通过USB-CAN卡发送数据到接收邮箱，DSP程序检测到接收邮箱有新数据后，将接收到的数据的每个字节加1，然后再通过发送邮箱将数据发送回上位机  
  
* **修改说明**  
主要修改了例程中的两个文件，[can_external_transmit.c](https://github.com/eecsfuture/28377ControllerTest/blob/master/CAN/can_external_transmit.c)和[F2837xD_SysCtrl.c](https://github.com/eecsfuture/28377ControllerTest/blob/master/CAN/F2837xD_SysCtrl.c)，具体说明如下：
1. 修改时钟频率设定函数  
TI例程使用的外部晶振频率为20MHz，系统时钟频率设定为200MHz，用如下函数进行设置：  
`InitSysPll(XTAL_OSC, IMULT_20, FMULT_0, PLLCLK_BY_2);`  
而测试控制器使用的外部晶振频率为30MHz，将系统时钟频率设置为100MHz，需要修改时钟频率设定函数：  
`InitSysPll(XTAL_OSC, IMULT_20, FMULT_0, PLLCLK_BY_6);`  
2. 修改CAN通信引脚定义  
将CAN-A模块的接收发送引脚由30和31改为70和71，取消CAN-B引脚初始化定义  
    ```
    //    GPIO_SetupPinMux(30, GPIO_MUX_CPU1, 1); //GPIO30 -  CANRXA
    //    GPIO_SetupPinOptions(30, GPIO_INPUT, GPIO_ASYNC);
    //    GPIO_SetupPinMux(31, GPIO_MUX_CPU1, 1); //GPIO31 - CANTXA
    //    GPIO_SetupPinOptions(31, GPIO_OUTPUT, GPIO_PUSHPULL);
    //    GPIO_SetupPinMux(10, GPIO_MUX_CPU1, 2); //GPIO10 -  CANRXB
    //    GPIO_SetupPinOptions(10, GPIO_INPUT, GPIO_ASYNC);
    //    GPIO_SetupPinMux(8, GPIO_MUX_CPU1, 2);  //GPIO8 - CANTXB
    //    GPIO_SetupPinOptions(8, GPIO_OUTPUT, GPIO_PUSHPULL);
        GPIO_SetupPinMux(70, GPIO_MUX_CPU1, 5); //GPIO70 -  CANRXA
        GPIO_SetupPinOptions(70, GPIO_INPUT, GPIO_ASYNC);
        GPIO_SetupPinMux(71, GPIO_MUX_CPU1, 5); //GPIO71 - CANTXA
        GPIO_SetupPinOptions(71, GPIO_OUTPUT, GPIO_PUSHPULL);
    ```  
3. 修改CAN-A模块波特率设置函数（系统时间频率由200MHz改为100MHz）  
    ```
    //CANBitRateSet(CANB_BASE, 200000000, 500000);
      CANBitRateSet(CANA_BASE, 100000000, 500000);  
    ```  
4. 屏蔽CAN-B的初始化和使能语句  
    ```
    //CANInit(CANB_BASE);
    //CANClkSourceSelect(CANB_BASE, 0);   // 500kHz CAN-Clock
    //CANBitRateSet(CANB_BASE, 200000000, 500000);
    //CANIntEnable(CANB_BASE, CAN_INT_MASTER | CAN_INT_ERROR | CAN_INT_STATUS);
    //CANGlobalIntEnable(CANB_BASE, CAN_GLB_INT_CANINT0);
    //CANEnable(CANB_BASE);
    ```
5. 修改接收邮箱编号（原为1，改为2）、ID号（原为0x5555，改为0xAAAA）和设置函数  
    ```
    #define RX_MSG_OBJ_ID    2
    sRXCANMessage.ui32MsgID = 0xAAAA;
    //    CANMessageSet(CANB_BASE, RX_MSG_OBJ_ID, &sRXCANMessage,
    //                  MSG_OBJ_TYPE_RX);
    CANMessageSet(CANA_BASE, RX_MSG_OBJ_ID, &sRXCANMessage,
                  MSG_OBJ_TYPE_RX);
    ```
6. 屏蔽原有的发送函数，增加新的发送和接收函数  
    ```
    while(1)
    {
    	uint32_t status;

    	//
    	// Read the CAN-A new data status
    	//
    	status = CANStatusGet(CANA_BASE, CAN_STS_NEWDAT);

    	//
    	// If the message object 2 has recieved a new data ,then get the received message
    	//
    	if(status == 2)
    	{
    		//
			// Get the received message
			//
			CANMessageGet(CANA_BASE, RX_MSG_OBJ_ID, &sRXCANMessage, true);

			//
			// Copy the recieved data to the txMsgData[]
			//
			for(i = 0; i < MSG_DATA_LENGTH; i++)
			{
				txMsgData[i] = rxMsgData[i] + 1;
			}

			//
			// Transmit Message
			//
			CANMessageSet(CANA_BASE, TX_MSG_OBJ_ID, &sTXCANMessage,
						  MSG_OBJ_TYPE_TX);

    	}
    }
    ```  
7. 屏蔽了while后的停止语句  
`//asm("   ESTOP0");`  

* **测试结果**  
![CAN通信测试结果-上位机界面](https://github.com/eecsfuture/28377ControllerTest/blob/master/images/CAN_Test_Result_20180118.png)  
**说明**：  

	序号 | 发送数据 | 返回数据
	------- | -------- | --------
	2和3 | 01 02 03 04 | 02 03 04 05
	4和5 | 05 06 07 08 | 06 07 08 09
