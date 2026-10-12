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
