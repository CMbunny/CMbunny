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
