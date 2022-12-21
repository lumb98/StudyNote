# ESP32 学习笔记



记录一些ESP32的学习。

## API

### I/O操作API

`void pinMode(uint8_t pin, uint8_t mode);`

定义I/O口模式，是输入还是输出

```cpp
//第二个参数有如下宏定义
#define INPUT             0x01
// Changed OUTPUT from 0x02 to behave the same as Arduino pinMode(pin,OUTPUT) 
// where you can read the state of pin even when it is set as OUTPUT
#define OUTPUT            0x03 
#define PULLUP            0x04
#define INPUT_PULLUP      0x05
#define PULLDOWN          0x08
#define INPUT_PULLDOWN    0x09
#define OPEN_DRAIN        0x10
#define OUTPUT_OPEN_DRAIN 0x12
#define ANALOG            0xC0
```

`void digitalWrite(uint8_t pin, uint8_t val);`

`int digitalRead(uint8_t pin);`

```cpp
//第二个参数的宏定义
#define LOW               0x0
#define HIGH              0x1
```





### RTOS相关API

实际上API基本和FreeRTOS的API一样

`xTaskCreate()`

创建任务

```cpp
static inline IRAM_ATTR BaseType_t xTaskCreate(
                            TaskFunction_t pvTaskCode,
                            const char * const pcName,     /*lint !e971 Unqualified char types are allowed for strings and single characters only. */
                            const uint32_t usStackDepth,
                            void * const pvParameters,
                            UBaseType_t uxPriority,
                            TaskHandle_t * const pxCreatedTask) PRIVILEGED_FUNCTION
```



`vTaskDelay();`

RTOS非阻塞延时。

参数是系统时钟的次数，而不是毫秒，但默认的tick计时器是1ms一次，所以默认情况下和ms无异。

```cpp
void vTaskDelay( const TickType_t xTicksToDelay ) PRIVILEGED_FUNCTION;
```



`pdMS_TO_TICKS()`

该函数返回系统定时器次数，将输入的毫秒数计算后返回。

`protTICK_PERIOD_MS`

一个宏定义，每毫秒系统定时器的次数。