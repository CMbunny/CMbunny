# Topic 1 – What a PLC actually is (and how it thinks)
A PLC is a specialised industrial computer that is built to:
- Read sensors and buttons (inputs) continuously
- Make decisions according to the program you wrote
- Turn motors, valves, lights, contactors, VFDs, etc. on or off (outputs)
- Do this extremely reliably, even in harsh factory environments

*It is **not like a normal PC**. It runs the same small program over and over, thousands of times per second, in a fixed cycle called the scan cycle.*<br>

**Simple analogy**<br>
Think of the PLC as a very fast, very disciplined worker standing in front of a machine:<br>
1.He looks at every sensor and button (reads inputs).<br>
2.He thinks “what should I do right now according to the rules I was given?” (executes the program).<br>
3.He turns the correct motors, lights, valves on or off (writes outputs).<br>
4.He immediately starts again from step 1.<br>

He never gets bored, never forgets, and never skips a step.<br>

***Why this matters for elevators?***<br>
An elevator has many sensors (floor position, door open/closed, overload, call buttons, etc.) and many outputs (motor up/down, door open/close, indicators, brakes). The PLC must constantly look at all of them and decide the next safe action. That continuous looking-and-deciding is the scan cycle.

### Quiz 1 – PLC Basics (5 questions)
Choose the best answer for each. Reply with the letter (A/B/C/D) for each question. Correct options are bold.

1.What is the main job of a PLC in a machine?<br>
A) Play videos and browse the internet<br>
**B) Continuously read inputs, make decisions, and control outputs**<br>
C) Only store data like a hard disk<br>
D) Replace all sensors so the machine needs no sensors<br>

2.A PLC runs its program…<br>
A) Only once when you press start<br>
**B) Continuously in a repeating cycle called the scan cycle**<br>
C) Only when an operator tells it to<br>
D) Randomly whenever it feels like it<br>

3.Which of these is a typical PLC input for an elevator?<br>
A) The motor that moves the cabin<br>
**B) A floor call button pressed by a passenger**<br>
C) The door motor itself<br>
D) The cabin light<br>

4.Why is a PLC preferred over a normal computer for controlling elevators and machines?<br>
A) It is cheaper and prettier<br>
**B) It is designed for reliable, continuous, real-time industrial control**<br>
C) It can run Windows games<br>
D) It does not need any program<br>

5.In the worker analogy, what does the worker do after he has turned the outputs on or off?<br>
A) He goes home<br>
B) He waits for the operator to tell him what to do next<br>
**C) He immediately starts looking at the sensors again**<br>
D) He turns all outputs off and stops<br>

# Topic 2 – Scan Cycle + Inputs vs Outputs (the heart of every PLC)
Every PLC works in a strict repeating loop called the scan cycle.It never stops while the PLC is running.<br>

**Typical scan cycle steps:**

**1.Read Inputs** <br>
The PLC looks at every physical input terminal and copies the real-world state into its internal memory (input image table).<br>
Example elevator inputs: call buttons, door open sensors, floor position sensors, overload switch, emergency stop, etc.<br>

**2.Execute Program**<br>
The PLC runs your logic from top to bottom (or according to the program organization).<br>
It decides what the outputs should be, based only on the input image it just read and the internal memory (flags, timers, counters, etc.).<br>

**3.Write Outputs**<br>
The PLC copies the decided output states to the physical output terminals.<br>
Example elevator outputs: motor up, motor down, door open, door close, floor indicators, cabin light, brake release, etc.<br>

**4.Housekeeping / Communications**<br>
Internal tasks, communication with HMI, other PLCs, diagnostics, etc.<br>

***Then it immediately goes back to step 1 and repeats.***
A modern PLC can complete this cycle in a few milliseconds (sometimes less than 1 ms).

#### Key points you must never forget:
- The program only “sees” the input values that were captured at the beginning of the scan.
- Outputs only change at the end of the scan.
- If a sensor changes state in the middle of the scan, the PLC will not notice it until the next scan.

##### Inputs vs Outputs – simple and clear
|**Type**|**Direction**|**Examples in Elevator**|**PLC sees it as**|
|---|---|---|---|
|Input|From field → PLC|Call button, door closed sensor, floor sensor|0 or 1|
|Output|From PLC → field|Motor up, motor down, door open command|0 or 1|

- Inputs are read-only for the program (you cannot force them in normal logic).
- Outputs are written by the program.

**Why this matters for our 2-elevator, 10-floor project?** <br>
We will have many inputs (buttons on every floor for both cars, door sensors, position sensors, overload, etc.) and many outputs (two independent motors, two door systems, indicators, etc.). The scan cycle must read all of them reliably and decide the correct action for each elevator every few milliseconds.<br>

### Quiz 2 – Scan Cycle + Inputs/Outputs (5 questions)
Reply with the letter for each answer (example: 1A 2C 3B 4D 5A).Correct option are marked bold.

1.In a normal PLC scan cycle, when does the PLC actually change the physical outputs?<br>
A) At the very beginning of the scan<br>
B) Continuously while the program is running<br>
**C) At the end of the scan, after the program has finished**<br>
D) Only when the operator presses a special button<br>

2.What happens if a floor call button is pressed exactly in the middle of a scan?<br>
A) The PLC sees it immediately<br>
**B) The PLC will see it only in the next scan**<br>
C) The button is ignored forever<br>
D) The elevator stops<br>

3.Which of the following is an input for an elevator PLC?<br>
A) Command to open the door<br>
B) Motor “Up” contactor<br>
**C) Door closed sensor**<br>
D) Cabin light<br>

4.Which of the following is an output for an elevator PLC?<br>
A) Overload switch<br>
B) Emergency stop button<br>
C) Floor position sensor<br>
**D) Motor “Down” command**<br>

5.Why do we say the PLC works with an “input image table”?<br>
A) Because it takes a photograph of the elevator<br>
**B) Because it freezes a copy of all input states at the start of the scan and works only with that copy**<br>
C) Because inputs are never important<br>
D) Because the PLC has a camera<br>

# Topic 3 – NO / NC Contacts & the absolute basics of Ladder Logic
Ladder logic looks like an electrical schematic (rungs of a ladder).<br>

#### Two basic contact types:

**1. NO – Normally Open contact**
- Draws as: `| |`
- In the real world the contact is open when the device is not activated.
- In the PLC: the contact becomes TRUE (conducts) only when the linked bit/tag is 1.
- Example: A call button is pressed → bit becomes 1 → NO contact closes in the logic.

**2. NC – Normally Closed contact**
- Draws as: `|/|`
- In the real world the contact is closed when the device is not activated.
- In the PLC: the contact becomes TRUE when the linked bit/tag is 0.
- Example: An Emergency Stop is healthy (not pressed) → bit is 0 → NC contact stays closed and allows the circuit to run. When E-Stop is pressed, bit becomes 1 and the NC contact opens → everything stops.

**Very simple elevator example (just to show the idea):** <br>
Imagine one rung that allows the elevator motor to run only if:
- No Emergency Stop is active
- Door is closed
- A call exists

In Ladder it would look conceptually like:
```
   E-Stop    Door_Closed   Call_Request          Motor_Up
----|/|--------| |------------| |--------------------( )----
```

- `|/|` = NC contact of Emergency Stop (must be healthy)
- `| |` = NO contact of Door Closed sensor
- `| |` = NO contact of Call Request
- `( )` = Coil (output) that turns the motor on

### Quiz 3 – NO / NC Contacts (5 questions)
Reply with the letters only (example: 1A 2B 3C 4D 5A).Correct option are in bold.

1.What does a NO contact do in Ladder logic?<br>
A) It is always closed<br>
**B) It conducts only when its bit is 1**<br>
C) It conducts only when its bit is 0<br>
D) It is used only for outputs<br>

2.What does a NC contact do in Ladder logic?<br>
A) It conducts only when its bit is 1<br>
B) It is always open<br>
**C) It conducts only when its bit is 0**<br>
D) It turns the output on automatically<br>

3.In an elevator, which contact would you normally use for the Emergency Stop in the main enable circuit?<br>
A) NO contact of E-Stop<br>
**B) NC contact of E-Stop**<br>
C) Neither – Emergency Stop is never used in logic<br>
D) Both at the same time<br>

4.A floor call button is pressed (bit becomes 1). Which contact will close?<br>
A) NC contact of that button<br>
**B) NO contact of that button**<br>
C) Both<br>
D) Neither<br>

5.Why do we use an NC contact for Emergency Stop in the logic?<br>
A) Because it looks nicer<br>
**B) So that when the E-Stop is healthy (not pressed) the circuit can run, and when it is pressed the circuit opens and stops**<br>
C) Because Emergency Stop is an output<br>
D) So the elevator always runs<br>

# Topic 4 – Simple Ladder “Seal-in / Latching” logic (very important for elevators)
Many things in elevators need to “remember” that a button was pressed even after the passenger releases the button.<br>

This is done with **seal-in** (also called latching or holding) logic.<br>

***Classic example – Motor Start/Stop:***
```
   Start     Stop       Motor
----| |------|/|--------( )----
              |
   Motor      |
----| |-------|
```
- Press Start (NO) → Motor coil turns on.
- The Motor contact (NO) in parallel with Start keeps the coil on even after you release the Start button.
- Press Stop (NC) → the circuit opens and the motor stops.

*In elevators we use the same idea for:*
- Registering a floor call (the call stays latched until the elevator serves that floor)
- Keeping the motor running while travelling
- Door open/close sequences, etc.

**We will use this pattern many times.**

### Quiz 4 – NO/NC + Seal-in (5 questions)
Reply with letters only. Correct answers are marked bold.

1. For a normal floor call button, which contact do we use in the logic?<br>
**A) NO**<br>
B) NC<br>
C) Both<br>
D) None<br>

2. For the Emergency Stop in the main enable circuit, which contact do we use?<br>
A) NO<br>
**B) NC**<br>
C) Both<br>
D) None<br>

3. What is the purpose of the “seal-in” (holding) contact parallel to the Start button?<br>
A) To make the button look nicer<br>
**B) To keep the output on after the Start button is released** <br>
C) To stop the motor automatically<br>
D) To read the input faster<br>

4.If I use a NO contact for Emergency Stop in the main circuit, what happens?<br>
A) Everything works safely<br>
**B) The elevator can only run when E-Stop is pressed (dangerous and wrong)** <br>
C) The PLC will not scan<br>
D) Nothing changes<br>

5.In the classic Start/Stop circuit, the Stop button is programmed as:<br>
A) NO contact<br>
**B) NC contact** <br>
C) A timer<br>
D) An output coil<br>

# Topic 5 – Seal-in (Latching) logic – deeper understanding
Seal-in is one of the most used patterns in industrial PLC programming (including elevators).<br>

**How it works step-by-step:** <br>
1. Operator presses Start button (momentary).
2. The Start NO contact closes for a short time.
3. The output coil (Motor) turns ON.
4. The Motor’s own NO contact (placed in parallel with Start) also closes.
5. Even after the operator releases the Start button, the parallel Motor contact keeps the circuit closed → Motor stays ON.
6. When Stop is pressed, the NC contact opens → circuit breaks → Motor turns OFF and the seal-in contact also opens.

**Elevator examples where we use seal-in:** <br>
- Floor call registration (call stays remembered until the car arrives)
- Cabin moving up / down
- Door opening sequence
- “Car in service” flag

*We almost never let a momentary button directly drive a motor without some form of memory (seal-in or Set/Reset).*

### Quiz 5 – 7 questions (all previous topics + seal-in)
This quiz mixes everything we have covered so far. Reply with the letters only (example: 1A 2B 3C … 7D). Correct are marked in bold.<br>

1. What is the correct contact type for a normal floor call button?<br>
**A) NO**<br>
B) NC<br>
C) Both<br>
D) Timer contact<br>

2. What is the correct contact type for Emergency Stop in the main enable circuit?<br>
A) NO<br>
**B) NC<br>**
C) Both<br>
D) Output coil<br>

3. In the classic Start/Stop circuit, the Stop button is programmed as:<br>
A) NO contact<br>
**B) NC contact<br>**
C) Seal-in contact<br>
D) Output<br>

4. What is the main purpose of the seal-in (holding) contact?<br>
A) To stop the motor faster<br>
**B) To keep the output ON after the Start button is released<br>**
C) To read inputs more accurately<br>
D) To replace the Emergency Stop<br>

5. If a floor call button is pressed (bit = 1), which contact closes?<br>
A) NC contact of the button<br>
**B) NO contact of the button<br>**
C) Both<br>
D) Neither<br>

6. When does the PLC update the physical outputs?<br>
A) At the beginning of the scan<br>
B) Continuously during the program<br>
**C) At the end of the scan<br>**
D) Only when a button is pressed<br>

7. Why is using a NO contact for Emergency Stop dangerous?<br>
**A) Because the elevator will only be allowed to run when the E-Stop is pressed<br>**
B) Because the PLC will crash<br>
C) Because inputs will not be read<br>
D) Because timers will not work<br>

# Topic 6 – Timers (the next essential tool)
Timers are used almost everywhere in elevator control.<br>

**Most common timer: TON (Timer ON-Delay)** <br>
How a TON works:<br>
1. When the input condition (IN) becomes TRUE, the timer starts counting time.<br>
2. It counts until it reaches the preset time (PT).<br>
3. When the elapsed time ≥ preset time, the timer output (Q) becomes TRUE.<br>
4. If the input condition goes FALSE before the preset is reached, the timer resets to zero.<br>

**Simple picture in your mind:**
- IN = “start counting”
- PT = “how long to wait” (example 3 seconds)
- Q = “time is finished → do something”

**Other common timers (just names for now):**
- TOF → Timer OFF-Delay (output stays on for a time after the input turns off)
- TP → Pulse timer (gives a fixed pulse)

For elevators we mostly use **TON** at the beginning.<br>

**Real elevator examples where TON is used:**
- Door open time (keep doors open for 5–8 seconds)
- Door closing delay
- Short delay before the motor starts moving after doors are closed
- Floor arrival delay (wait a moment before opening doors)
- Anti-nuisance delay (ignore very short button presses)
- Travel timeout (if the elevator takes too long to reach the next floor → fault)

**Very simple Ladder idea (conceptual):**
```
Door_Closed          TON_Door_Delay
----| |--------------[TON  PT:=3s]----
                      |
                      Q
                     | |------------( Door_Open_Command )
```
When the door is confirmed closed, wait 3 seconds, then allow the next action.

### Quiz 6 – Timers (7 questions)
Reply with the letters only.

1. What does a TON timer do?<br>
A) Turns the output on immediately and turns it off after a delay<br>
**B) Starts counting when the input becomes TRUE and turns its output TRUE after the preset time** <br>
C) Counts how many times a button is pressed<br>
D) Always runs continuously<br>

2. What happens if the input of a TON goes FALSE before the preset time is reached?<br>
A) The timer continues counting<br>
B) The timer output becomes TRUE<br>
**C) The timer resets to zero** <br>
D) The PLC stops scanning<br>

3. In an elevator, which of these is a typical use of a TON timer?<br>
A) Counting the number of floors<br>
**B) Keeping the doors open for a fixed time** <br>
C) Reading the call buttons<br>
D) Turning the cabin light on permanently<br>

4. What does “PT” mean on a timer?<br>
A) Present Time<br>
**B) Preset Time (the time we want to wait)** <br>
C) Power Time<br>
D) Program Time<br>

5. Why do we often put a short TON delay after the doors are confirmed closed before starting the motor?<br>
A) To make the elevator slower<br>
**B) To give a small safety/settling time and avoid mechanical stress** <br>
C) Because the PLC is slow<br>
D) To count passengers<br>

6. Which timer output becomes TRUE only after the preset time has elapsed?<br>
A) The IN input<br>
**B) The Q output of a TON** <br>
C) The PT value<br>
D) The Emergency Stop<br>

7. If I want the elevator doors to stay open for 6 seconds after arriving at a floor, which timer is the most natural choice?<br>
A) A counter <br>
**B) A TON timer** <br>
C) A seal-in contact only<br>
D) An NC contact<br>

# Topic 7 – Counters (short and clear)
Counters are simpler than timers.<br>

**Most common: CTU (Count Up)** <br>
- Every time the input (CU) has a rising edge (goes from 0→1), the counter increases by 1.
- When the current value (CV) reaches the preset value (PV), the output (Q) becomes TRUE.
- There is usually a Reset input to set the count back to zero.

***Elevator examples:***
- Counting how many times a door has opened (for maintenance)
- Simple passenger counting (rare and not accurate)
- Counting floors passed (sometimes used in older systems)
- Anti-nuisance: count repeated button presses in short time

For our project we will use counters very little at the beginning. Timers and seal-in logic are far more important.<br>

**Quick difference:**
- Timer → measures time
- Counter → measures number of events

### Quiz 7 – Mixed Foundation + Counters (7 questions)
This is a mixed revision quiz. Answer all 7 with letters. Correct answers are in bold.

1. A TON timer output (Q) becomes TRUE when:<br>
A) The input becomes TRUE<br>
**B) The elapsed time reaches the preset time** <br>
C) The PLC is powered on<br>
D) A counter reaches its value <br>

2. What happens to a TON if its input goes FALSE before the preset time?<br>
A) It continues counting<br>
**B) It resets to zero** <br>
C) Its output stays TRUE<br>
D) The PLC faults<br>

3. Which contact do we normally use for a floor call button?<br>
**A) NO** <br>
B) NC <br>

4. Which contact do we normally use for Emergency Stop in the main circuit?<br>
A) NO<br>
**B) NC** <br>

5. What is the main job of a seal-in contact?<br>
A) To measure time<br>
**B) To keep an output ON after the momentary button is released**<br>
C) To count floors<br>
D) To reset a timer<br>

6. A CTU counter increases its value when:<br>
A) Time passes<br>
**B) Its count input has a rising edge (0→1)** <br>
C) The preset is reached<br>
D) The PLC scan ends<br>

7. In an elevator, which is more commonly used for “keep doors open for 6 seconds”?<br>
A) Counter<br>
**B) TON timer**<br>
C) Only seal-in<br>
D) NC contact of the door<br>

# Topic 8 Starting the Elevator Project (Single Car first)
We will not jump into 2 elevators yet.<br>

Real industrial practice is:<br>
**First make one elevator work correctly → then expand to multi-car group control.**<br>

### High-level view of one elevator (10 floors)
**Main jobs the PLC must do:** <br>

<ol> <li>Accept hall calls (up/down buttons on each floor) and car calls (buttons inside the cabin)</li>
<li>Remember the calls (seal-in / memory)</li>
<li>Decide direction and next target floor</li>
<li>Control the door sequence (open → wait → close)</li>
<li>Control the motor (up / down / stop)</li>
<li>Monitor safety conditions continuously (door closed, E-Stop, overload, limits, etc.)</li>
<li>Handle faults and recovery</li> </ol>

**Typical high-level sequence for one trip:** <br>
_IDLE → New call registered → Decide direction → Close doors → Travel → Arrive at floor → Open doors → Wait → Close doors → Go to next call or return to IDLE_<br>

### Quiz 8 – 10 Questions (Interview-style + All topics)
Answer with letters only (1A 2B 3C … 10D).Correct options are in bold. <br>

1. In a PLC scan cycle, when are the physical output terminals actually updated? <br>
A) At the beginning of the scan<br>
B) Continuously while the logic is executing<br>
**C) At the end of the scan after the program has run** <br>
D) Only when a force is applied<br>

2. A field device is wired to a digital input. The PLC program needs to react when this device is activated. Which type of contact is normally used in Ladder? <br>
A) NC contact<br>
**B) NO contact** <br>
C) Timer contact<br>
D) Counter contact<br>

3. Why is an Emergency Stop almost always programmed with an NC contact in the main enable circuit?<br>
A) So the machine runs only when E-Stop is pressed<br>
**B) So the circuit is closed when E-Stop is healthy and opens when E-Stop is activated** <br>
C) Because E-Stop is an output<br>
D) To increase scan time<br>

4. Explain the purpose of a seal-in (holding) contact parallel to a momentary Start button.<br>
A) It measures the time the button is pressed<br>
**B) It keeps the output energised after the Start button is released** <br>
C) It resets the timer<br>
D) It counts the number of starts<br>

5. A TON timer is used to keep elevator doors open for 7 seconds after arrival. When does the timer’s Q output become TRUE?<br>
A) Immediately when the door open command is given<br>
**B) After the preset time has elapsed while the input condition remains TRUE** <br>
C) When the door closed sensor activates<br>
D) When a counter reaches 7<br>

6. What happens to a TON timer if its enabling input goes FALSE before the preset time is reached?<br>
A) The timer continues from where it left off<br>
B) The accumulated time is retained and Q stays TRUE<br>
**C) The timer resets and the accumulated time returns to zero** <br>
D) The PLC generates a fault<br>

7. In industrial PLC programming, what is the main difference between a Timer and a Counter?<br>
**A) Timer measures time, Counter measures number of events** <br>
B) Timer is only for outputs, Counter is only for inputs<br>
C) They are exactly the same<br>
D) Counter is faster than Timer<br>

8. A CTU counter increments its value on:<br>
A) Every PLC scan<br>
**B) A rising edge (0 → 1) of its count input** <br>
C) When the preset value is reached<br>
D) When the Reset input is TRUE<br>

9. Which of the following is the most correct statement about PLC inputs and outputs in an elevator system?<br>
A) Call buttons and door sensors are outputs; motor commands are inputs<br>
**B) Call buttons and door sensors are inputs; motor and door commands are outputs** <br>
C) Everything is an input<br>
D) Everything is an output<br>

10. You need a short delay (1.5 s) after the doors are confirmed closed before allowing the elevator motor to start. Which instruction is the most appropriate?<br>
A) CTU counter<br>
B) Seal-in contact only<br>
**C) TON timer** <br>
D) NC contact of the door sensor<br>

# Topic 9 Preliminary I/O Thinking for One 10-Floor Elevator
**Digital Inputs (examples):** <br>

- Hall Call Up buttons (Floor 1 to 9)
- Hall Call Down buttons (Floor 2 to 10)
- Car Call buttons (Floor 1 to 10) inside the cabin
- Door Closed sensor
- Door Open sensor (or Fully Open)
- Floor level sensors / position signals (or encoder later)
- Overload sensor
- Emergency Stop
- Safety chain / landing door locks (simplified for learning)
- Inspection / Manual mode selector (optional at start)

**Digital Outputs (examples):** <br>

- Motor Up command
- Motor Down command
- Door Open command
- Door Close command
- Cabin light
- Floor indicators / direction arrows
- Call registered lights (optional)
- Brake release (if separate)

**Important notes for learning version:** <br>

- We will start with simplified position feedback (one sensor or flag per floor).
- Real elevators use absolute encoders or multi-sensor systems + safety PLC. We stay in standard control logic for learning.
- Safety signals (E-Stop, door locks, overload) will be treated as hard interlocks — the car must never move if they are not healthy.

We will build the logic in this order:<br>

1. Call registration (hall + car calls) using seal-in<br>
2. Door sequence (Open → Wait → Close) with timers<br>
3. Simple direction & travel logic<br>
4. Arrival and floor serving<br>
5. Basic interlocks and faults<br>
6. Later expand to 2 elevators<br>

### Quiz 9 – 10 Questions (Interview style + All topics)
Correct options are market in bold.<br>

1. In a standard PLC scan cycle, the physical outputs are written:<br>
A) Before reading the inputs<br>
B) During the execution of each rung<br>
**C) At the end of the scan** <br>
D) Only on the first scan after power-up<br>

2. A push-button is connected to a digital input. When the button is pressed you want the logic to become TRUE. Which contact should you use?<br>
A) NC contact<br>
**B) NO contact** <br>
C) Coil<br>
D) Timer Q contact<br>

3. Why is Emergency Stop normally evaluated with an NC contact in the main safety/enable circuit?<br>
A) To allow the machine to run only while the button is pressed<br>
**B) So that a healthy (not activated) E-Stop keeps the circuit closed, and activation opens the circuit** <br>
C) Because E-Stop is configured as an output<br>
D) To reduce the number of inputs required<br>

4. What is the industrial purpose of placing a seal-in contact in parallel with a momentary Start button?<br>
A) To measure how long the button was pressed<br>
**B) To maintain the output after the operator releases the button** <br>
C) To automatically reset all timers<br>
D) To convert the input into an analog signal<br>

5. A TON timer is enabled. After the preset time elapses, which part of the timer becomes TRUE?<br>
A) The PT value<br>
B) The IN input<br>
**C) The Q output** <br>
D) The Reset input<br>

6. If the enabling condition of a TON goes FALSE before the preset time is reached, the timer will:<br>
A) Hold its accumulated value<br>
B) Continue counting in the background<br>
**C) Reset the accumulated time to zero** <br>
D) Force the Q output TRUE<br>

7. In PLC terms, what is the fundamental difference between a Timer and a Counter?<br>
**A) Timers measure elapsed time; Counters measure the number of events** <br>
B) Timers can only be used with outputs; Counters only with inputs<br>
C) They function identically<br>
D) Counters are only available in Structured Text<br>

8. A CTU (Count Up) counter increments its current value on:<br>
A) Every PLC scan while the input is TRUE<br>
**B) A rising edge (0 to 1 transition) of the count input** <br>
C) When the preset value is loaded<br>
D) When the Reset input is TRUE<br>

9. In an elevator control system, which of the following are typically inputs to the PLC?<br>
A) Motor Up command and Door Open command<br>
**B) Hall call buttons, door closed sensor, overload sensor** <br>
C) Cabin light and direction arrows<br>
D) Brake release and floor indicators<br>

10. You need a 1.5 second delay after the door closed sensor becomes TRUE before allowing the travel motor to start. The most appropriate instruction is:<br>
A) CTU counter<br>
B) Simple seal-in without time<br>
**C) TON timer** <br>
D) NC contact of the door sensor only<br>

# Topic 10 Single Elevator – First Professional Step: I/O List
In real industrial projects we always start with a clear I/O list before writing any logic.<br>

**_Here is a practical learning-version I/O list for one 10-floor elevator._** <br>

**Digital Inputs (PLC reads these)**
|Tag / Name|Description|Type|Notes|
|---|---|---|---|
|HallCall_Up_1 to _9|Hall Up buttons on floors 1–9|NO|Passenger wants to go up|
|HallCall_Down_2 to _10|Hall Down buttons on floors 2–10|NO|Passenger wants to go down|
|CarCall_1 to _10|Buttons inside the cabin|NO|Destination from inside|
|DoorClosed|Door fully closed sensor|NO/NC|Critical interlock|
|DoorOpen|Door fully open sensor|NO|Optional but useful|
|FloorSensor_1 to _10|Simplified floor position sensors|NO|Learning version|
|Overload|Cabin overload switch|NC|Safety – stop when overloaded|
|E_Stop|Emergency Stop|NC|Hard safety interlock|
|SafetyChain|Landing door locks + other safeties|NC|Must be healthy to move|

**Digital Outputs (PLC controls these)**
|Tag / Name|Description
|--|--|
|Motor_Up|Command to drive motor upward|
|Motor_Down|Command to drive motor downward|
|Door_Open_Cmd|Command to open doors|
|Door_Close_Cmd|Command to close doors|
|Cabin_Light|Cabin illumination|
|Dir_Arrow_Up|Direction indicator|
|Dir_Arrow_Down|Direction indicator|
|Floor_Indicator|Current floor display (simplified)|

**Internal Memory we will need soon** <br>
- Call registered flags (one for each hall up, hall down, car call)
- Current floor
- Target floor
- Direction (Up / Down / Idle)
- Door sequence states
- Travel enable
- Various timers (door open time, start delay, etc.)

**Important learning note:** <br>
_This is a simplified version suitable for learning. Real elevators use absolute encoders, safety PLCs, redundant sensors, and certified safety functions. We stay in standard control logic._<br>

**Next Logic Block: Call Registration** <br>
This is the first real piece of elevator logic we will design.<br>
**Goal:** When a passenger presses a Hall Call or Car Call button, the call must be remembered (latched) until the elevator serves that floor. After serving, the call is cleared.<br>

We will use classic seal-in (or Set/Reset) for this.<br>

### Quiz 10 – 10 Questions (Interview style + All topics so far)
Correct options are in bold.<br>

1. In a PLC scan cycle, physical outputs are updated:<br>
A) At the start of the scan<br>
B) Continuously during logic execution<br>
**C) At the end of the scan** <br>
D) Only after a power cycle<br>

2. A momentary push-button is connected to a digital input. To detect that the button is pressed, which contact is normally used in Ladder?<br>
A) NC contact<br>
**B) NO contact** <br>
C) Timer contact<br>
D) Counter contact<br>

3. Why is an Emergency Stop evaluated with an NC contact in the main enable circuit?<br>
A) So the machine runs only while the button is held<br>
**B) So a healthy E-Stop keeps the circuit closed and activation opens it** <br>
C) Because E-Stop is an output<br>
D) To save input points<br>

4. What is the purpose of a seal-in contact placed in parallel with a momentary Start button?<br>
A) To measure press duration<br>
**B) To keep the output energised after the button is released** <br>
C) To reset all timers automatically<br>
D) To convert the signal to analog<br>

5. When does the Q output of a TON timer become TRUE?<br>
A) As soon as the IN input becomes TRUE<br>
**B) After the preset time has elapsed while IN remains TRUE** <br>
C) When the Reset input is activated<br>
D) On every rising edge of the input<br>

6. If the IN condition of a TON goes FALSE before the preset time is reached, the timer will:<br>
A) Retain the accumulated time<br>
B) Continue counting<br>
**C) Reset the accumulated time to zero** <br>
D) Force Q TRUE<br>

7. What is the fundamental difference between a Timer and a Counter in PLC programming?<br>
**A) Timer measures time; Counter measures number of events** <br>
B) Timer works only with outputs; Counter only with inputs<br>
C) They are functionally identical<br>
D) Counters are only available in Structured Text<br>

8. A CTU counter increments its current value on:<br>
A) Every scan while the input is TRUE<br>
**B) A rising edge (0→1) of the count input** <br>
C) When the preset is loaded<br>
D) When the Reset input is TRUE<br>

9. In an elevator system, which of the following are typically PLC inputs?<br>
A) Motor Up and Door Open commands<br>
**B) Hall call buttons, door closed sensor, overload switch** <br>
C) Cabin light and direction arrows<br>
D) Brake release and floor indicators<br>

10. You need a short time delay after the door closed sensor is TRUE before allowing the motor to start. The most appropriate instruction is:<br>
A) CTU counter<br>
B) Seal-in only<br>
**C) TON timer** <br>
D) NC contact of the door sensor alone

# Topic 11
