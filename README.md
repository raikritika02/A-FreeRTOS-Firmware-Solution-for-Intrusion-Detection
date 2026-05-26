# A-FreeRTOS-Firmware-Solution-for-Intrusion-Detection
📌 Project Overview
SmartGuard is a high-performance, real-time intrusion detection system built on the STM32F446RE microcontroller running FreeRTOS. It uses an IR sensor to detect unauthorized motion or entry and immediately triggers a synchronized audio-visual alarm using an active buzzer and a red LED. Multiple RTOS tasks execute concurrently with priority-based scheduling, ensuring rapid intrusion response while all other system operations continue without interruption. A green LED blinks continuously as a system heartbeat, and all events are streamed live to the SWV Data Console with millisecond timestamps for real-time monitoring and debugging.
Key Highlights
Feature Detail
Microcontroller STM32F446RE (ARM Cortex-M4 @ 180 MHz)
RTOS FreeRTOS via CMSIS-RTOS2
Detection Method IR sensor + hardware EXTI interrupt
Alarm Output Active buzzer (PA1) + Red LED (PB2)
Status Indicator Green LED heartbeat (PB1)
User Control Push button reset (PC13)
Monitoring SWV ITM Data Console (live event logs)
RTOS ConceptsTask Management, Priority Scheduling, Interrupt Handling, Real-Time Delays

Problem Statement
Unauthorized intrusion into restricted areas such as homes, laboratories, offices, server rooms, and storage facilities is a major security concern. Traditional embedded alarm systems are built using sequential programming techniques, where operations execute one after another. Such systems suffer from:

Delayed response — the MCU is busy with one task while an intrusion goes undetected
Inefficient multitasking — alarm activation, LED indication, and logging cannot run at the same time
Poor synchronization — no deterministic guarantee on when the system will respond

SmartGuard solves this by using FreeRTOS to run all operations as independent, concurrent tasks with defined priorities, ensuring the alarm fires immediately upon detection regardless of what else the system is doing.

 Challenges & Solutions
Challenge 1 — SWV Data Console Showed No Output
Symptom: SWV ITM Data Console was completely blank even though the program was running and reaching printf() calls.
Root Cause: The SWV core clock field in the debug configuration did not match the actual SYSCLK (180 MHz). SWV requires an exactly matching clock value to decode the serial bit stream from ITM Port 0.
Solution:
Set SWV Core Clock to exactly 180 MHz in Run → Debug Configurations → Debugger
Enabled ITM Stimulus Port 0 in the SWV ITM Data Console configure window
Clicked Start Trace before pressing Resume — output appeared immediately


Challenge 2 — IR Sensor Generating Continuous False Triggers
Symptom: The alarm was activating repeatedly with no person or object in front of the sensor. intrusionDetected was being set to 1 continuously at startup.
Root Cause: The sensitivity potentiometer on the IR sensor module was set too high, causing it to detect ambient infrared radiation from room lighting and nearby electronics.
Solution:
Adjusted the onboard sensitivity potentiometer anti-clockwise until only a direct hand wave at 20–30 cm triggered the sensor
Added a 2-reading confirmation in SensorTask backup poll (two consecutive LOW reads 50 ms apart before setting the flag)
Result: zero false triggers across all subsequent test sessions


Challenge 3 — Green LED Froze During Active Alarm
Symptom: When the alarm was active, the green LED stopped blinking completely, defeating its purpose as a system heartbeat indicator.
Root Cause: AlarmTask (Real-Time priority) was not calling osDelay() inside the alarm-active branch. Without yielding, it monopolised the CPU and LEDTask (Low priority) was never scheduled — a classic priority starvation problem.
Solution:
Added osDelay(500) inside both branches of the alarm-active loop
The 500 ms delays serve dual purpose: create the 1 Hz buzzer/LED blink pattern AND yield CPU time to LEDTask
After the fix, the green LED continued its 1 Hz blink perfectly during all alarm tests
