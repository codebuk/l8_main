# Fix no check of return values on startup 

---
	goto check_fr;
	/* USER CODE END 2 */
---

and also 

---

    check_fr:
    osStatus ret;
    ret = osKernelInitialize();
    if (ret != osOK ){
        printf ("FreeRTOS osKernelInitialize failed\n");
    }
        /* Call init function for freertos objects (in freertos.c) */
        MX_FREERTOS_Init();
        /* Start scheduler */
    ret = osKernelStart();
    if (ret != osOK ){
        printf ("FreeRTOS osKernelStart failed\n");
    }

    /* We should never get here as control is now taken by the scheduler */
    /* Infinite loop */

        while (1) {

        }
---
#stackcove flow 

add config in FreeRTOS IOC

remove from freertos.c

    void vApplicationStackOverflowHook(TaskHandle_t xTask, signed char *pcTaskName)
    {
    /* Run time stack overflow checking is performed if
    configCHECK_FOR_STACK_OVERFLOW is defined to 1 or 2. This hook function is
    called if a stack overflow is detected. */
    }

add to main.c

    void vApplicationStackOverflowHook(TaskHandle_t *pxTask, signed char *pcTaskName) {
        (void) pxTask;
        printf("\r\nStack overflow detected. Task name: %s", pcTaskName);
    }

# runtime stats 

https://stm32world.com/wiki/STM32_FreeRTOS_Statistics


1. setup timer to fire at , say 10kHz

2. add to main!!!

    /* * USER CODE BEGIN PV */

    volatile unsigned long ulHighFrequencyTimerTicks;

    /* USER CODE END PV */

and...

    /* USER CODE BEGIN 0 */

    void configureTimerForRunTimeStats(void)
    {
        ulHighFrequencyTimerTicks = 0;
        HAL_TIM_Base_Start_IT(&htim6);
    }

    unsigned long getRunTimeCounterValue(void)
    {
        return ulHighFrequencyTimerTicks;
    }

in stm32f4xx_it.c

3.
    void TIM6_DAC_IRQHandler(void)
    {
    /* USER CODE BEGIN TIM6_DAC_IRQn 0 */

    // Needed for freertos stats
    ulHighFrequencyTimerTicks++;

    /* USER CODE END TIM6_DAC_IRQn 0 */
    HAL_TIM_IRQHandler(&htim6);
    /* USER CODE BEGIN TIM6_DAC_IRQn 1 */

    /* USER CODE END TIM6_DAC_IRQn 1 */
    }
---
    /* USER CODE BEGIN EV */

    extern volatile unsigned long ulHighFrequencyTimerTicks;

    /* USER CODE END EV */
---
5. MX seems to ad this automagically to FreeRTOSConfig.h

    /* USER CODE BEGIN 0 */
    extern void configureTimerForRunTimeStats(void);
    extern unsigned long getRunTimeCounterValue(void);
    /* USER CODE END 0 */





#define configTOTAL_HEAP_SIZE                    ((size_t)20000)
#define configMAX_TASK_NAME_LEN                  ( 32 )
#define configCHECK_FOR_STACK_OVERFLOW           2
#define configRECORD_STACK_HIGH_ADDRESS          1
#define configUSE_OS2_THREAD_ENUMERATE       0

