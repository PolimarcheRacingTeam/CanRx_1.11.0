# Wheel Node (stm32f303k8) CAN-BUS Configuration (FW_1.11.0)

## Summary

1. CubeMX Configuration
2. Main.c setup
3. Tx / Rx
4. Examples

### CubeMX Configuration

- Open up STM32CubeIDE
- Select your usual Workspace Directory
- _File->new->STM32 Project_
- On the Part Number write "F303k8" and click on the only one MCU on the right list and then click "Next"
- Select a Project Name (No space admitted)
  - Targeted Language : C
  - Targeted Binary Type: Executable
  - Targeted Project Type: STM32Cube
  - "NEXT"
- Firmware Package Name and Version: V1.11.x (x can be 0 to any)
  - Code Generator Options: Copy only the necessary Files
- "Finish"

Now we have the MCU GPIO menu configuration.

In The Pinout & Configuration menu

- System Core
  - SYS	
    - Debug: Serial Wire
    - System Wake-Up 1: NOT SELECTED
    - Timebase Source: SysTick
  - RCC
    - High Speed Clock (HSE): Crystal/Ceramic Resonator
    - Master Clock Output: Not Selected

In the Clock Configuration

- Inputa Frequency: 12Mhz
- PLL Source Mux: Select HSE
- *PLLMul: X6
- System Clock Mux: PLLCLK
- APB1 Presclaler (should be red): /2

Going Back to Pinout & Configuration

- Connectivity
  - CAN
    - Select Activated, after that a Configuration menu should appear
    - Go to NVIC Settings
      - CAN RX0 Interrupt: Enable
    - Go to Parameter Settings
      - Prescaler: 8
      - Time Quanta in Bit Segment 1: 7 Times
      - Time Quanta in Bit Segment 2: 1 Time
      - now baud Rate should be 500 kbit/s (finding out how to go 1Mbit/s)
      - Time Triggered Communication: Enable
  - Now you can generate other GPIO as you want

Go to Project Manager

- Code Generator
  - Generate peripheral initialization as a pair of .c/.h files

Now it's time to generate the code! (Yellow Washer symbol) and open C perspective.

### Main.c Setup

In this part Three other files are important

- stm32f3xx_it.c
- stm32f3xx_hal_can.c (Drivers/STM32F3xx_HAL_Driver/Src/ HERE)
- m32f3xx_hal_can.h (Drivers/STM32F3xx_HAL_Driver/Inc/ HERE)

In the /* USER CODE BEGIN PV */

```
CAN_FilterTypeDef sFilterConfig ;
CAN_TxHeaderTypeDef TxHeader;
CAN_RxHeaderTypeDef RxHeader;
uint8_t Data; 
```

 In the /* USER CODE BEGIN 2 */

```
HAL_CAN_Start(&hcan); // This start all the CAN Functions
// now we configure a standard filter (no filtering)
sFilterConfig.FilterMode = CAN_FILTERMODE_IDMASK;
sFilterConfig.FilterScale = CAN_FILTERSCALE_32BIT;
sFilterConfig.FilterIdHigh = 0x0000;
sFilterConfig.FilterIdLow = 0x0000;
sFilterConfig.FilterMaskIdHigh = 0x0000;
sFilterConfig.FilterMaskIdLow= 0x0000;
sFilterConfig.FilterFIFOAssignment = CAN_FILTER_FIFO0;
sFilterConfig.FilterActivation = ENABLE; //Can't be disabled, can wouldn't work
sFilterConfig.FilterBank = 0;

HAL_CAN_ConfigFilter(&hcan, &sFilterConfig);

HAL_CAN_ActivateNotification(&hcan, CAN_IT_RX_FIFO0_MSG_PENDING); //This activate //callbacks function

```

Now we go to the _stm32f3xx_it.c_ file, this one is responsible for every Interrupt Service Rotuine

