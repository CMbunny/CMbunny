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

**3. The ESP32 "Twist": Dual-Core**<br>
The ESP32 has two cores: Core 0 (Protocol CPU) and Core 1 (Application CPU).<br>

- When you create a task, you can use the function xTaskCreatePinnedToCore() to tell the scheduler exactly which core that task should live on.<br>
- If you don't specify, FreeRTOS will assign it to a core for you.<br>
- ***Independent Scheduling:*** Each core has its own independent scheduler and its own "Ready Lists." A high-priority task on Core 0 has absolutely no impact on what is running on Core 1.<br>

**4. What happens when priorities are equal?**<br>
If two tasks have the same priority, the scheduler uses Round-Robin Scheduling.<br>
- It will run the first task for a specific amount of time (one "tick").<br>
- When the next tick occurs, it moves that task to the back of the list and starts the second task.<br>
- This ensures that no task of the same priority gets "starved" of CPU time. <br>

**NOTES:** <br>
- ***Avoid Priority Inversion:*** If a low-priority task holds a resource (like a Mutex) that a high-priority task needs, the high-priority task will be stuck. FreeRTOS has "Priority Inheritance" to help fix this, but it is something to watch for.<br>
- ***Don't Over-Prioritize:*** A common mistake is making every task "High Priority." If everything is high priority, the scheduler has to work much harder to rotate them, which can lead to system instability. Reserve high priority only for time-critical tasks (like motor control or emergency stops).<br>

**Question:** <br>
What is blocking? How Blocking works?<br>

**Answer:**<br>
In FreeRTOS, "Blocking" is the mechanism that prevents a task from wasting CPU power while it waits for something to happen.<br>
Instead of the task staying in a "Ready" state and constantly asking the CPU "Are you done yet?" (which is called busy-waiting and ruins performance), the task tells the scheduler: "Put me to sleep, and don't wake me up until X happens."<br>

**How Blocking Works: The State Machine**<br>
When a task enters a "Blocked" state, the scheduler removes it from the "Ready List" we discussed earlier. It is effectively invisible to the scheduler until the condition it is waiting for is met.<br>

**The "Wake-up" Triggers**<br>
A task can be unblocked by several events:<br>

- ***Time Delay (vTaskDelay):*** You tell the task to sleep for a specific number of milliseconds. The scheduler keeps a timer and automatically moves the task back to "Ready" once the time expires.<br>
- ***Semaphore or Mutex:*** You want to wait for a shared resource to be free. The task blocks until another task "gives" the semaphore.<br>
- ***Queue:*** You are waiting for data to arrive from a sensor or another task. The task blocks until a message is placed in the queue.<br>
- ***Event Group:*** The task waits for a combination of "flags" (bits) to be set, indicating that specific conditions have been met.<br>

**Why Blocking is Crucial?** <br>
If you don't block your tasks, your ESP32 will run at 100% CPU usage constantly. This causes two major problems:<br>

- ***Overheating:*** Running the CPU at maximum load consumes more power and generates heat.<br>
- ***Starvation:*** If one "busy-waiting" task has a high priority, it might never let the lower-priority tasks (like the ones responsible for your Wi-Fi or Bluetooth stacks) run, causing your device to crash or disconnect.<br>

**Comparison: Busy-Waiting vs. Blocking**
|Feature|Busy-Waiting (Bad Practice)|Blocking (Best Practice)|
|--|--|--|
|CPU Usage| 100% (The CPU is spinning in circles)|0% (The task is asleep)|
|Efficiency|Terrible (Waste of battery/power)|Excellent|
|Response|Instant (but hurts everything else)|Immediate upon event (via interrupt)|
|Code |Stylewhile(sensor_not_ready);|xSemaphoreTake(xSemaphore, portMAX_DELAY);|

**The "Internal" View:** ***How the Scheduler handles this***<br>
When you call a blocking function like vTaskDelay(100):<br>

- **The Context Save:** The scheduler saves the current task's registers.<br>
- **State Change:** The scheduler changes the task status from Running to Blocked and moves it to a "Delayed List" inside the kernel.<br>
- **The Switch:** The scheduler looks at its "Ready List," picks the next highest-priority task, and starts running that instead.<br>
- **The Wake-Up:** On every "Tick" (the heartbeat of the OS), the kernel checks the "Delayed List." Once the counter reaches 100, the kernel moves that task back to the "Ready List."<br>
- **Re-scheduling:** At the next available opportunity, the scheduler sees that the task is "Ready" again and puts it back on the CPU.
