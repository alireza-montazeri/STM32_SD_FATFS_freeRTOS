# STM32 FATFS on SD card using freeRTOS
Since SD Card & DMA with CubeMX generated Code doesn't work, i want to offer the solution.<br/>
This repository contains a working example of STM32L476 FATFS on an SD card using freeRTOS. The main problem is using freeRTOS and Cube generated files for FATFS automatically using DMA on SDMMC peripheral. The DMA on SDMMC has a problem while using both RX and TX channels.<br/>
I coded and tested this for STM32L476. For other microController this workflow should be also succesfull. This repository is a fully working example.<br/>
This is the CubeIDE project, and to use it, clone this repository in your directory, and then in CubeIDE, go to *File->Import->General->Import existing project* and choose your directory.<br/>
**After every CubeMX code generating, changes remain and don't need to be applied again.**<br/>
Solution in this project is using this ST community [post](https://community.st.com/s/feed/0D70X000006SpXHSA0).
<br/><br/>

#### 1. Generate Project in Cube MX
Select SDMMC & DMA2 request on one single Channel 4. Not two channels for RX TX See attached CubeMX example. Also make sure `Generate peripheral initialization as a pair of .c/.h files` is checked.

#### 2. In generated Cod
In *Core/Src/sdmmc.c* file go to `void HAL_SD_MspInit(SD_HandleTypeDef* sdHandle)` and move all thing execpt `SDMMC1 DMA Init` section into `USER CODE SDMMC1_MspInit 0` section. Use compiler command `#if(false)/#endif` to disable codes between `USER CODE End SDMMC1_MspInit 0` and `USER CODE BEGIN SDMMC1_MspInit 1`. This would help our code to remain unchanged when generating Cube.

#### 3. In *Core/Src/stm32l4xx_it.c* apply changes in `void DMA2_Channel4_IRQHandler(void)` as below.
```
void DMA2_Channel4_IRQHandler(void)
{
  /* USER CODE BEGIN DMA2_Channel4_IRQn 0 */
    if((hsd1.Context == (SD_CONTEXT_DMA | SD_CONTEXT_READ_SINGLE_BLOCK)) ||
     (hsd1.Context == (SD_CONTEXT_DMA | SD_CONTEXT_READ_MULTIPLE_BLOCK)))
      {
        BSP_SD_DMA_Rx_IRQHandler();
      }
    else if((hsd1.Context == (SD_CONTEXT_DMA | SD_CONTEXT_WRITE_SINGLE_BLOCK)) ||
          (hsd1.Context == (SD_CONTEXT_DMA | SD_CONTEXT_WRITE_MULTIPLE_BLOCK)))
     {
       BSP_SD_DMA_Tx_IRQHandler();
     }
  /* USER CODE END DMA2_Channel4_IRQn 0 */
  HAL_DMA_IRQHandler(&hdma_sdmmc1);
  /* USER CODE BEGIN DMA2_Channel4_IRQn 1 */

  /* USER CODE END DMA2_Channel4_IRQn 1 */
}
```
#### 4. In *FATFS/Target/bsp_driver_sd.c* make changes below as it is in project:
- add `static HAL_StatusTypeDef SD_DMAConfigTx(SD_HandleTypeDef *hsd)`
- add `static HAL_StatusTypeDef SD_DMAConfigRx(SD_HandleTypeDef *hsd)`
- add `void BSP_SD_DMA_Rx_IRQHandler(void)`
- add `void BSP_SD_DMA_Tx_IRQHandler(void)`
- Rewrite `uint8_t BSP_SD_WriteBlocks_DMA(uint32_t *pData, uint32_t WriteAddr, uint32_t NumOfBlocks)`
- Rewrite `uint8_t BSP_SD_ReadBlocks_DMA(uint32_t *pData, uint32_t ReadAddr, uint32_t NumOfBlocks)`
Again with using compiler command `#if(false)/#endif` you can make code resistant to changes that happen when generating Cube.
