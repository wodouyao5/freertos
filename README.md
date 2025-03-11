# 7.移植Freertos

环境：

clion 2023.2.4+STM32CubeMx

stm32f103c8t6

ST--Link

**tips：**

在cubemx生成工程时，由于Freertos使用的是系统嘀嗒，所以需要将系统的时钟换成基本的定时器作为时钟来源

[ HAL和FreeRTOS基础时钟](https://www.bilibili.com/video/BV1DH4y1L77k/?spm_id_from=333.788&vd_source=77dca8402aab32aa41c9a00a66cce231)



## 1.在裸机工程里创建Freertos文件夹，并创建如下文件夹


## 2.在Freertos官网下载源码包



## 3.对官网源码进行选取

### 1.include文件夹所需文件

将官网源码的include全部复制过来，另外还需添加**FreeRTOSConfig.h**文件，到**FreeRTOSv9.0.0\FreeRTOS\Demo** 中寻找自己对应板子型号或者相同内核的**FreeRTOSConfig.h**



这里我是用的f103c8t6所以选择**CORTEX_STM32F103_GCC_Rowley**





### 2.portable文件夹所需文件

因为采用mingw对c语言进行编译，故选择**GCC**；c8t6内核为M3，故选择**ARM_CM3**

**MemMang** 内存管理选择**heap_4**



### 3.source文件夹所需文件

将圈红部分复制至source文件夹



## 4.工程实现

### 1.Cmake修改

因为引入了新的头文件和源文件，对于cmake编译需要自行添加新增文件的具体路径

圈出部分为新增文件路径，对应左侧含有头文件和源文件的文件夹

修改后需刷新cmake文件



### 2.FreeRTOSConfig.h修改

添加如下代码

```
// 用于启动第一个任务的中断
# define  xPortPendSVHandler    PendSV_Handler
// 用于每次任务切换中断
#define  xPortSysTickHandler    SysTick_Handler
// 定时器回调函数
#define  vPortSVCHandler        SVC_Handler
```



### 4.stm32f1xx_it.c修改

将stm32原有的三个中断函数注释掉，并添加新的定义

```
// 用于启动第一个任务的中断
# define  xPortPendSVHandler    PendSV_Handler
// 用于每次任务切换中断
#define  xPortSysTickHandler    SysTick_Handler
// 定时器回调函数
#define  vPortSVCHandler        SVC_Handler
```



### 5.完成led灯翻转和串口打印字符串

在main.c中，添加

```
#include "FreeRTOS.h"
#include "task.h"
```



功能代码：对任务进行定义

```
void led1_task(void* arg)
{
    while(1)
    {
        HAL_GPIO_TogglePin(GPIOC, GPIO_PIN_13); //翻转led灯
        vTaskDelay(500);
    }
}


void print_task(void* arg)
{
    while(1)
    {
        HAL_UART_Transmit(&huart1, (uint8_t *)"HelloWorld\r\n", 12, 100);//串口打印
        vTaskDelay(1000);
    }
}

```



主函数 int main(void)

```
    //led翻转任务
    xTaskCreate(led1_task, "led1_task", 64, NULL, 3, NULL);
   
    // 创建串口打印任务
    xTaskCreate(print_task, "print_task", 128, NULL, 5, NULL);

    // 启动任务调度
    vTaskStartScheduler();
```



最后下载进板子，led灯闪烁

串口成功打印！


### 6.Bug汇总
