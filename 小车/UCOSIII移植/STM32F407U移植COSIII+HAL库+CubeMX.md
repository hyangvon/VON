# STM32F407移植UCOSIII+HAL库+CubeMX
# 1、准备可用工程
利用cubemx配置stm32f407vet6芯片，在配置好时钟和引脚功能后，选择生成MDK-ARM格式的工程文件。
![](_v_images/20220804143153938_3598.png =x200)
得到一个仅配置好时钟和引脚功能的工程，可编译通过。
# 2、在工程中添加UCOS相关文件
## 2.1 在文件夹中添加文件
在工程目录中新建UCOSIII文件夹，打开后将 `UCOSIII 3.04\Micrium\Software` 中的uC-CPU、uC-LIB和uCOS-III三个文件夹复制进来，再新建两个文件夹：UCOS_BSP和UCOS_CONFIG。
| ![](_v_images/20220804144626752_32053.png =x200)    |  ![](_v_images/20220804144411066_20347.png =x200)   |
| :-: | :-: |
将 `UCOSIII 3.04\Micrium\Software\EvalBoards\ST\STM32F429II-SK\uCOS-III` 中的8个文件复制进UCOS_CONFIG文件夹中。
将 `UCOSIII 3.04\Micrium\Software\EvalBoards\ST\STM32F429II-SK\BSP` 中的2个文件复制进UCOS_BSP文件夹中。
|  ![](_v_images/20220804145113619_18281.png =400x)    |   ![](_v_images/20220804145214113_6392.png =400x)   |
| :-: | :-: |
## 2.2 在工程中添加源文件
虽然源文件已经在工程文件夹中，但仍需将其手动添加到keil的工程中才能加以编译。
打开keil，在工程中建立6个新分组，并添加对应源文件。
|  ![](_v_images/20220804150233860_19359.png =180x)   |  ![](_v_images/20220804151501521_30827.png =180x)   |  ![](_v_images/20220804152323098_23968.png =180x)   |
| :-- | :--| :-- |
|  UCOSIII_BSP分组中文件路径：<br>`"UCOSIII\UCOS_BSP"`<br>UCOSIII_CPU分组中文件路径：<br>`"UCOSIII\uC_CPU"`<br>`"UCOSIII\uC-CPU\ARM-Cortex-`<br>`M4\RealView"`<br>UCOSIII_LIB分组中文件路径：<br>`"UCOSIII\uC_LIB"`<br>`"UCOSIII\uC-LIB\Ports\ARM-`<br>`Cortex-M4\RealView"`   | UCOSIII_CORE分组中文件路径：<br>`"UCOSIII\uCOS-III\Source"`   |  UCOSIII_PORT分组中文件路径：<br>`"UCOSIII\uCOS-III\Ports\ARM-`<br>`Cortex-M4\Generic\RealView"`<br> UCOSIII_CONFIG分组中文件路径：<br>`"UCOSIII\UCOS_CONFIG"`  |
在"Options for Target->C/C++->Include Paths"选项中添加对应头文件路径。
|  ![](_v_images/20220804155742468_21981.png =x300)   |  ![](_v_images/20220804154524816_5258.png =x300)   |
| :-: | :-: |
# 3、配置文件修改
## 3.1 修改bsp.c和bsp.h文件
bsp.c文件中有很多代码，我们只需要其中关于DWT的代码，因此需要进行修改。
修改后的bsp.h文件如下：
```
#ifndef  __BSP_H__
#define  __BSP_H__

#include "stm32f4xx_hal.h"

void BSP_Init(void);

#endif
```
修改后的bsp.c文件如下：
```
#include "includes.h"

#define  BSP_REG_DEM_CR                           (*(CPU_REG32 *)0xE000EDFC)	//DEMCR寄存器
#define  BSP_REG_DWT_CR                           (*(CPU_REG32 *)0xE0001000)    //DWT控制寄存器
#define  BSP_REG_DWT_CYCCNT                       (*(CPU_REG32 *)0xE0001004)	//DWT时钟计数寄存器
#define  BSP_REG_DBGMCU_CR                        (*(CPU_REG32 *)0xE0042004)

//DEMCR寄存器的第24位,如果要使用DWT ETM ITM和TPIU的话DEMCR寄存器的第24位置1
#define  BSP_BIT_DEM_CR_TRCENA                    DEF_BIT_24

//DWTCR寄存器的第0位,当为1的时候使能CYCCNT计数器,使用CYCCNT之前应当先初始化
#define  BSP_BIT_DWT_CR_CYCCNTENA                 DEF_BIT_00

/*
*********************************************************************************************************
*                                            BSP_CPU_ClkFreq()
* Description : Read CPU registers to determine the CPU clock frequency of the chip.
* Argument(s) : none.
* Return(s)   : The CPU clock frequency, in Hz.
* Caller(s)   : Application.
* Note(s)     : none.
*********************************************************************************************************
*/
CPU_INT32U  BSP_CPU_ClkFreq (void)
{
    return HAL_RCC_GetHCLKFreq();//返回HCLK时钟频率
}

/*
*********************************************************************************************************
*                                            BSP_Tick_Init()
* Description : BSP_Tick_Init.
* Argument(s) : none.
* Return(s)   : none.
* Note(s)     : none.
*********************************************************************************************************
*/
void BSP_Tick_Init(void)
{
	CPU_INT32U cpu_clk_freq;
	CPU_INT32U cnts;
	cpu_clk_freq = BSP_CPU_ClkFreq();

	#if(OS_VERSION>=3000u)
		cnts = cpu_clk_freq/(CPU_INT32U)OSCfg_TickRate_Hz;
	#else
		cnts = cpu_clk_freq/(CPU_INT32U)OS_TICKS_PER_SEC;
	#endif
	OS_CPU_SysTickInit(cnts);
}

void BSP_Init(void)
{
	BSP_Tick_Init();//此函数会初始化OS系统时钟，如果移植了正点原子的delay文件，则与主函数中的delay_init(168)只需要调用一个即可
}

#if (CPU_CFG_TS_TMR_EN == DEF_ENABLED)
void  CPU_TS_TmrInit (void)
{
    CPU_INT32U  cpu_clk_freq_hz;

    BSP_REG_DEM_CR     |= (CPU_INT32U)BSP_BIT_DEM_CR_TRCENA; 	//使用DWT  /* Enable Cortex-M4's DWT CYCCNT reg.*/
    BSP_REG_DWT_CYCCNT  = (CPU_INT32U)0u;					 	//初始化CYCCNT寄存器
    BSP_REG_DWT_CR     |= (CPU_INT32U)BSP_BIT_DWT_CR_CYCCNTENA;	//开启CYCCNT

    cpu_clk_freq_hz = BSP_CPU_ClkFreq();
    CPU_TS_TmrFreqSet(cpu_clk_freq_hz);
}
#endif

#if (CPU_CFG_TS_TMR_EN == DEF_ENABLED)
CPU_TS_TMR  CPU_TS_TmrRd (void)
{
    return ((CPU_TS_TMR)BSP_REG_DWT_CYCCNT);
}
#endif

#if (CPU_CFG_TS_32_EN == DEF_ENABLED)
CPU_INT64U  CPU_TS32_to_uSec (CPU_TS32  ts_cnts)
{
    CPU_INT64U  ts_us;
    CPU_INT64U  fclk_freq;

    fclk_freq = BSP_CPU_ClkFreq();
    ts_us     = ts_cnts / (fclk_freq / DEF_TIME_NBR_uS_PER_SEC);

    return (ts_us);
}
#endif

#if (CPU_CFG_TS_64_EN == DEF_ENABLED)
CPU_INT64U  CPU_TS64_to_uSec (CPU_TS64  ts_cnts)
{
	CPU_INT64U  ts_us;
	CPU_INT64U  fclk_freq;

    fclk_freq = BSP_CPU_ClkFreq();
    ts_us     = ts_cnts / (fclk_freq / DEF_TIME_NBR_uS_PER_SEC);

    return (ts_us);
}
#endif
```
## 3.2 修改startup_stm32f407xx.s
将
```
                DCD     PendSV_Handler             ; PendSV Handler
                DCD     SysTick_Handler            ; SysTick Handler
```
修改为
```
                DCD     OS_CPU_PendSVHandler             ; PendSV Handler
                DCD     OS_CPU_SysTickHandler            ; SysTick Handler
```
将
```
PendSV_Handler  PROC
                EXPORT  PendSV_Handler             [WEAK]
                B       .
                ENDP
SysTick_Handler PROC
                EXPORT  SysTick_Handler            [WEAK]
                B       .
                ENDP
```
修改为
```
OS_CPU_PendSVHandler  PROC
                EXPORT  OS_CPU_PendSVHandler             [WEAK]
                B       .
                ENDP
OS_CPU_SysTickHandler PROC
                EXPORT  OS_CPU_SysTickHandler            [WEAK]
                B       .
                ENDP
```
在startup_stm32f407xx.s的174行添加如下汇编代码：
```
				IF {FPU} != "SoftVFP"
				; Enable Floating Point Support at reset for FPU
				LDR.W   R0, =0xE000ED88         ; Load address of CPACR register
				LDR     R1, [R0]                ; Read value at CPACR
				ORR     R1,  R1, #(0xF <<20); Set bits 20-23 to enable CP10 and CP11 coprocessors
				; Write back the modified CPACR value
				STR     R1, [R0]                ; Wait for store to complete
				DSB

				; Disable automatic FP register content
				; Disable lazy context switch
				LDR.W   R0, =0xE000EF34         ; Load address to FPCCR register
				LDR     R1, [R0]
				AND     R1,  R1, #(0x3FFFFFFF)  ; Clear the LSPEN and ASPEN bits
				STR     R1, [R0]
				ISB                             ; Reset pipeline now the FPU is enabled
				ENDIF

```
![](_v_images/20220805111046533_4001.png =x350)
同时需要支持浮点运算，默认支持
![](_v_images/20220805111220307_3452.png =x350)
## 3.3 串口重定向
修改usart.h文件，在CubeMX生成代码预留的 `USER CODE BEGIN Includes` 段中添加对标准库头文件的引用
```
#include <stdio.h>
#include <string.h>
```
修改后如图所示
![](_v_images/20220805112414546_27462.png =x100)
修改usart.c文件，在CubeMX生成代码预留的 `USER CODE BEGIN 1` 段中添加如下代码
```
#ifdef __GNUC__
  /* With GCC/RAISONANCE, small printf (option LD Linker->Libraries->Small printf
     set to 'Yes') calls __io_putchar() */
  #define PUTCHAR_PROTOTYPE int __io_putchar(int ch)
#else
  #define PUTCHAR_PROTOTYPE int fputc(int ch, FILE *f)
#endif /* __GNUC__ */
/**
  * @brief  Retargets the C library printf function to the USART.
  * @param  None
  * @retval None
  */
PUTCHAR_PROTOTYPE
{
  /* Place your implementation of fputc here */
  /* e.g. write a character to the EVAL_COM1 and Loop until the end of transmission */
  HAL_UART_Transmit(&huart3, (uint8_t *)&ch, 1, 0xFFFF);
  return ch;
}
```
修改后如图所示
![](_v_images/20220805113102691_10034.png =x300)
# 4、编译运行
快捷键F7编译通过，F8下载程序到开发板
![](_v_images/20220805113844578_28905.png =x200)