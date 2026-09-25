# STM32 Nucleo-F103RB Signal Analyzer

This project implements a real-time, single-channel signal analyzer using the **STM32 Nucleo-F103RB** development board. The system continuously samples an analog input and streams the digitized voltage readings over USB to a host PC for live waveform visualization.

- **Operating Range:** 0 to 3.3 V DC maximum
- **Communication:** UART via USB Virtual COM Port at 115200 baud

## Hardware Requirements

- STM32 Nucleo-F103RB Development Board (STM32F103RBT6)
- Standard female-to-male jumper wires
- External testing components:
  - Potentiometer
  - Sensor (e.g., MQ-135)
  - External 3.3 V DC power supply to generate a test signal

## Software & Hardware Configuration

This project utilizes a decoupled STMicroelectronics toolchain to define the hardware abstraction layer and compile the firmware.

- **Initialization:** Standalone STM32CubeMX (v6.18.1)
- **Development & Flashing:** STM32CubeIDE (v2.0)

### CubeMX Peripheral Setup

1. **Analog (ADC1):**
   - Enable **IN0**, which automatically routes to pin **PA0**.
   - Keep **Continuous Conversion Mode** disabled for software polling.

2. **Connectivity (USART2):**
   - Set Mode to **Asynchronous**.
   - Use the default settings:
     - Baud Rate: **115200 Bits/s**
     - Data Bits: **8 Bits**
     - Parity: **None**
     - Stop Bits: **1**

3. **System Core (SYS):**
   - Set Debug to **Serial Wire**.
   - This allocates pins **PA13** and **PA14** for the ST-Link debugger.

## Code Implementation

The core logic resides in the infinite loop of `main.c`. It utilizes blocking HAL functions to:

1. Start the ADC conversion.
2. Wait for the conversion to complete.
3. Read the 12-bit ADC value.
4. Convert the digital value back to a voltage using the 3.3 V reference.
5. Format the voltage as a string.
6. Transmit the formatted value through USART2.
7. Wait for 10 ms before taking the next sample.

### Main Loop

```c
/* Infinite loop */
/* USER CODE BEGIN WHILE */
while (1)
{
    HAL_ADC_Start(&hadc1);
    HAL_ADC_PollForConversion(&hadc1, HAL_MAX_DELAY);

    uint16_t raw_adc_value = HAL_ADC_GetValue(&hadc1);
    float voltage = ((float)raw_adc_value / 4095.0f) * 3.3f;

    int len = sprintf(uart_buf, "%.2f\r\n", voltage);
    HAL_UART_Transmit(&huart2, (uint8_t*)uart_buf, len, HAL_MAX_DELAY);

    HAL_Delay(10);

    /* USER CODE END WHILE */

    /* USER CODE BEGIN 3 */
}
/* USER CODE END 3 */
