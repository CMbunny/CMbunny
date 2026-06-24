# FREERTOS
## What is FreeRTOS?
FreeRTOS (Free Real-Time Operating System) is an open-source, lightweight Real-Time Operating System kernel designed specifically for microcontrollers and small microprocessors.
Unlike a desktop OS (like Windows or Linux) which manages complex multitasking and heavy applications, FreeRTOS is designed for embedded systems where resources (RAM, CPU power) are extremely limited. It provides the "logic" that allows a tiny chip to do several things at once—like reading a sensor, updating a display, and sending data to a server—all while ensuring critical tasks happen exactly when they are supposed to.

## What is it Used For?
FreeRTOS is the "conductor of the orchestra" in embedded devices. You use it when your project becomes too complex to manage with a simple "super-loop" (a while(1) loop in C). It is used for:<br>
***Multitasking:*** Giving each feature of your device its own "Task" (a separate piece of code running as if it has its own CPU).<br>
***Determinism:*** Ensuring that time-sensitive events (like turning off a motor) happen within a guaranteed time frame.<br>
***Resource Management:*** Safely sharing hardware (like an I2C bus) between different parts of your code so they don't crash into each other.

## How It Works: The "Under the Hood" Mechanics
FreeRTOS works by rapidly switching the CPU between different tasks. To the user, it looks like everything is happening at once, but the CPU is actually just switching back and forth millions of times per second.

**1. The Scheduler**
The heart of FreeRTOS is the Scheduler. It decides which task is allowed to run on the CPU at any given moment. It follows a "Priority-based Preemptive" strategy:<br>
***Priority:*** You assign each task a number. Higher priority tasks always run first.<br>
***Preemptive:*** If a high-priority task suddenly needs to run (e.g., an interrupt happens), the scheduler will "pause" the lower-priority task, save its state, run the important task, and then jump back to where it left off.<br>

**2. The Task States**
Tasks in FreeRTOS are always in one of four states:<br>
***Running:*** The task currently has control of the CPU.<br>
***Ready:*** The task is ready to run but is waiting for a higher-priority task to finish.<br>
***Blocked:*** The task is waiting for an event (like a timer to expire or data to arrive from a sensor). It uses 0% CPU in this state.<br>
***Suspended:*** The task is manually turned off until you tell it to wake up.<br>

**3. Context Switching**
When the scheduler switches tasks, it performs a Context Switch. It saves the values of the CPU registers (the "context") of the current task into that task's private stack, then loads the registers from the next task's stack. This makes the switch invisible to the code being executed.

## When is it Called?
FreeRTOS is not "called" like a regular library function; it is initialized and then it takes over.<br>
***Initialization:*** In your main() function, you create your tasks and define their priorities.<br>
***Starting the Kernel:*** You call vTaskStartScheduler().<br>
***The Takeover:*** Once the scheduler starts, the main() function essentially ends. The scheduler now handles all execution flow.<br>
***The Tick:** A hardware timer (the "System Tick") fires at a set interval (e.g., every 1ms). Every time this tick fires, the scheduler wakes up to check if it needs to switch tasks.

**Question:** <br> how a freertos assign or prioritise work when it is called by an esp32 chip like what and how it decides which task is of greater priority ?<br>
**Answer:** <br>
When you run FreeRTOS on an ESP32, you are essentially moving from a single-track program to a multi-threaded one. The ESP32 (which is dual-core) runs an instance of the FreeRTOS scheduler on both cores.<br>

Here is how the "decision-making" process works regarding priorities and task management.<br>

**1. The Priority System**<br>
In FreeRTOS, priorities are numerical values assigned to tasks when they are created.<br>
***The Scale:*** By default, priorities usually range from 0 (the lowest) to configMAX_PRIORITIES - 1 (the highest).<br>
***The Golden Rule:*** The scheduler will always run the task with the highest priority that is in the "Ready" state.<br>
***Preemption:*** If a high-priority task suddenly becomes "Ready" (e.g., an interrupt signals that a sensor has data), the scheduler will immediately pause the lower-priority task, save its state (context), and switch to the high-priority task. This is called Preemptive Multitasking.<br>

**2. How the Scheduler Decides (The "Ready List")** <br>
Internally, FreeRTOS keeps track of tasks using Ready Lists. Imagine these as a set of buckets, where each bucket is labeled with a priority number.<br>

- When you call xTaskCreate(), you assign a priority.<br>

- FreeRTOS places that task into the corresponding "bucket" (a linked list).<br>

- ***The Decision:*** Every time a system tick occurs (or a function like vTaskDelay() is called), the scheduler looks at the buckets starting from the highest priority number downward.<br>

- It picks the first task it finds in the highest-numbered bucket that has tasks in it and gives it the CPU.<br>
