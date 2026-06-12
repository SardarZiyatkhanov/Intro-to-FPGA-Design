Team members: Sardar Ziyatkhanov

# Objective

The objective of our final lab is to implement vending machine on FPGA board using buttons as inputs and two LED displays and two 7-segment displays as outputs. Three of the key input buttons must represent three coins of different cost that a user may insert: nickel (5¢), dime (10¢) and quarter (25¢). A reset input is used in case a user wants to go back to start. Upon inserting coins two 7-segment displays on a board will show inserted amount. Our group number is 4 so the cost of a drink to buy from a vending machine assigned to us is 45¢. When a user inserts exactly 45¢ 7-segment displays will show it, one of the LED that represents dispense will turn on for a moment and everything goes back to 00. If, however, inserted amount exceeds 45¢, another LED will turn on – this one represents change – then the change amount will be displayed for a moment and everything resets to zero. To program everything we will use Quartus Prime and for waveform simulations – ModelSim.

# Background

## Main Logic.

To implement vending machine, FSM principles similar to those utilized in the previous lab will be used. Even so, with the extended number of states and more complex functionality FSM becomes much more sophisticated. In the table below, overall process is carefully outlined:

![image](Pasted%20image%2020260113194501.png)

All the possible states are divided into several sections. Two big boxes represent two different conditions: first box + S0 state is where a user is allowed to insert a coin with a given range of values, second box + S45 state is where the machine is forced to vend. As you might have already guessed, numbers in front of letter S in states S0-S65 represent all possible configurations our vending machine can allow – they are all multiples of 5 since any of possible coin input is a multiple of 5. For visual representativeness, nickel input arrow is signified with a dot at the tip, dime input - with a rectangular shape and a quarter input – with a basic arrow (note: for non-input transitions the arrow is filled unlike the quarter input one to avoid confusion).

1) Starting from S0 we begin to insert coins  
2) We continue to insert coins as long as tracked amount does not exceed 40¢ (reminder: our drink costs 45¢). If at any point along the process we press the reset button, our machine goes to the refund state that displays the amount to refund, which is total inserted amount, and heads back to S0  
3) If upon insertion S45 was hit (identifying 45¢ that is exactly the drink price) the machine goes to vend state (aka dispense). Since 45¢ is the required amount of money to buy a drink, we quickly head back to S0 to buy a new drink or let someone else to. Additionally, vend state is responsible for ‘dispense’ LED  
4) Likewise, if inserted amount exceeds 45¢ we enter the vend state that lights up the dispense LED. However, since we do not want to scam a user, we enter the change state that displays the difference between an inserted amount and the actual cost of a drink, which mimics the user getting their money back this way. Change state is also responsible for ‘change’ LED  

## Decoder.

A subtle complication arises with displaying states’ digits (that is, total inserted amount or change) on FPGA. Since we are operating in a maximum of 2-digit numbers domain, we will need some corrections in how our decoder operates.

Basically, it all comes down to primary school arithmetic. Suppose we have arbitrary value v that consists of 2 digits at most. To find ‘tens’ of v we simply divide it by 10:  
tens = v / 10

In order to find ones of v we find a remainder of division by 10:  
ones = v % 10

This way, we can easily display digits of the number on separate displays.

We still, however, need to tread lightly at this step. You see, even though the logic behind what we need to display is trivial, there are some peculiarities behind how we should display it. 7-segment displays on a single device are normally (at least in case of our FPGA board) connected through one line. This implies that either display can only work one at a time. Let’s put particular emphasis on the phrase “one at a time” since this is basically how we mitigate the issue:

To display 2 digits as a single number we switch between 2 digits with high frequency. So, even though digits are not technically displayed simultaneously, it appears so because of our perception. The coding behind everything will be explained in “Results” section of the report.

## Debounced pulses.

Our goal is to imitate vending machine behavior as accurately as possible. When we insert a single coin, it logically follows that the displayed amount must be increased by only inserted amount. With buttons this may be tricky since they can be pressed continuously. The behavior of the buttons may come out very unpredictable unless a user was explicitly told to only press button shortly, which is not rational. That’s why we introduce debounced pulses. Roughly speaking, it acts like a filter and comes in handy in following circumstances:

1) Where signal may be detected multiple times throughout a single press on different rising edges of a clock cycle  
2) Where signal may bounce (e.g. 0 -> 1 -> 0 -> 1) and produce undesired output  

So, a signal is recognized if it is received continuously for some period. This period is long enough to avoid false inputs (aka bounces) and is short enough not to overwhelm a user by too long of a press every time they want to deposit a coin. Coding principles behind these precautions are explained in the “Results” section of the report.

# Results

Here we will share all the codes and strive to explain it line by line with all the necessary details. The order of the codes will be as follows: Debounce pulse code, binary to 7-segment code, main FSM code, testbench code that tests main FSM code, testbench results as a waveform generated with ModelSim, top code and, finally, pin planner.

## Debounce pulse.

![image](Pasted%20image%2020260113194526.png)

Lines 1-4:  
This is module initialization. CLK_HZ = 50_000_000 represents our sistem clock to 50MHz which is the default value. DEBOUNCE_MS = 20 is the threshold for debouncing that was discussed in the previous section of the report.

Lines 4-9:  
From top to bottom the parameters represent:  
1) System clock  
2) Reset. Our debouncing mechanism functions using flip-flops. Since flip￾flops remember the old value with each iteration it is an important practice to consistently reset them  
3) Keys on the FPGA  
4) Press pulse. Shows that a press must happen when countinuous pulse is confirmed  

Line 10:  
Computes how long 20ms (the debounce threshold) is in clock cycles. CLK_HZ/1000 = 50,000 is clock cycles per milliseconds. 50,000* DEBOUNCE_MS = 1,000,000 cycles. So, for press to be confirmed input must be consistent for a million clock cycles.

Lines 12-16:  
The most important part of the program. The problem with one press at a time in systems like vending machine is that human input is not discrete relative to system clock, meaning the are not synchronized (human input may come any time at any stage of the clock). This creates an issue with metastability of the input. What it means is that input is not clearly 0 or 1. key_sync1 and key_sync2 are two 1-bit registers that actually act like to D flip-flops. First flip-flop stores input that comes from a key filtering out metastability. Second flip-flop by receiving output from the first flip-flop acts like a safety measure to ensure as sufficient time as possible is given for pulse to be procesed. Thus, signal comes out as stable as possible.

Lines 18-20:  
From top to bottom the parameters represent:  
1) Counter register. when calculated $clog2(CNT_MAX+1)-1 gives the value of 19 yielding an array of size 20. Basically, if the input is consistent across the array, press is confirmed  
2) Stable input state. As opposed to raw key_n that may bounce, stable_key is already filtered input  
3) Delayed (previous) stable input state. To address new input, we need to confirm transition (0->1)  

Lines 22-28:  
On positive edge of the clock cycle reset all the parameters if reset is 1.

Lines 28-40:  
Otherwise (no reset), reset press pulse and begin:  
1) If synchronized value is equal to the stable value no change occurred. This forces to reset counter  
2) If the values differ, however, check for how long they stay different as long as counter is not equal to the maximum value (20ms). As soon as the counter reached the maximum value, accept new stable input  

Lines 42-46:  
Assign old stable value to the new stable value from the same clock cycle. If the next stable value changes it confirms press.

## Binary to 7-segment.

![image](Pasted%20image%2020260113194543.png)

Lines 1-10:  
Our constants are system clock with the default value and scan. As we discussed in the “Background section of the report, due to displays connecting through single line we need to constantly switch between them with high frequency. Scan is exactly this frequency.  
From top to bottom parameters are:  
1) Clock  
2) Reset  
3) Value to pass to the display  
4) 7 segments of the display (plus decimal point that we will not need)  
5) 6 displays  

Lines 12-30:  
This function implements the binary to 7-segment conversion that we have seen many times in previous reports.

Lines 32-34:  
Representing our value in terms of tens and ones for proper displaying. Refer to the “Background” section of the report.

Lines 36-38:  
This is a clock divider. It is, basically, how many clock cycles we need to overcome to switch to a different display. divcnt is a counter that operates just like one in the debounce pulse code. scan is a switch – a toggle that decides what display we need to switch to.

![image](Pasted%20image%2020260113194559.png)

Lines 41-53:  
This is where we toggle scan. If reset is 1 we set everything to 0. Otherwise count up to DIV and then invert scan (0->1 or 1->0).

Lines 56-67:  
This is where we display our number. If scan is equal to 0 we display first display. If scan is 1 – second display.

## Main FSM implementation.

![image](Pasted%20image%2020260113194611.png)

Lines 1-18:  
Module initialization. From top to bottom constant values are:  
1) System clock  
2) Price of a drink  
3) The next three are the time limits in milliseconds of how long we need to show values of total inserted amount, change, and refunded amount for in milliseconds  
From top to bottom parameters are: clock, reset, enable (useful in testbench), three of possible coin values, cancel for refund option, dispense and collect states for LED functionality, the value to display.

Lines 21-29:  
All possible FSM states as well as registered current state and next state. Use state diagram drawing from previous section of the report as a reference.

Lines 31-53:  
A function that converts the state to the amount of money it represents.

![image](Pasted%20image%2020260113194622.png)

Lines 56-77:  
A function that returns the state after coin addition. First, it converts current state to amount and adds an inserted coin to it. Then, it returns resulting amount as a corresponding state.

Lines 80-87:  
Similar to the pulse debounce code, it computes the number of clock cycles required for displaying of given values. Then we create a counter for each of the timers.

Lines 88-91:  
latched_total is used as a storage not to lose the value after we enter vending or refund. cur_amount simply converts the amount we have to the corresponding state. Finally, cur_change, returns difference between the amount we inserted and the actual value if a user overpayed and 0 otherwise.

![image](Pasted%20image%2020260113194636.png)

Lines 94-142:  
State transition logic. If the program is not enabled, next state is always S0. Otherwise, state transition is executed as follows:  
1) If cancel button is pressed and current amount is in the range between 0 and the price of a drink, we go to refund state and go to S0 otherwise  
2) If totally inserted amount is below the price limit we simply assign next state to the amount after inserting a coin  
3) If total inserted amount reaches or exceeds the price we go to vending state  
4) For the rest of the states we wait untill the timer runs out so we can notice the changes. If during the vending state change is 0, we go back to S0, and we go to change state otherwise. Change state displays the change to be returned to a user before going back to S0. Same applies for refund state that displays refunded amount that just happens to be exactly inserted amount  

![image](Pasted%20image%2020260113194647.png)

Lines 145-172:  
Actions to be performed on positive edge of the clock cycle. First, if we reset, all parameters are being reset.  
Next in the always block come three timers for vended amount, change and refunded amount, respectively. They are used to display the desired value after we hit the price point and operate in the same way as a timer in debounce pulse code.  
After the timer comes the latch. If in the next state we hit the price point we the store the value of this next state. If we press cancel and want a refund, on the other hand, we store current amount.

![image](Pasted%20image%2020260113194659.png)

Lines 175-214:  
The outputs produced in the process. Initially we set all the outputs to 0. Then:  
1) If a state is inbetween S0 and S40 both including the output is simply the displayed amount  
2) If a state higher than S40 the outputs are dispense LED, collect LED depending on whether we need to return the change and the displayed amount  
3) For vend state the outputs are the same as in previous case except we use total latched amount instead of current amount which, mathematically, is the same  
4) For change state we turn both dispense and collect LEDs on and display the change  
5) For refund state we turn collect LED on and display the refunded amount  

## Testbench.

![image](Pasted%20image%2020260113194711.png)

Lines 1-12:  
Nothing special. We just initialize a module and all the necessary input and output parameters. They match all the parameter in the DUT that will be instantiated later and were discussed previously in main FSM code.

Lines 15-28:  
DUT instance. Hold parameter for displaying the value were made as short as 4ms for the purposes of waveform readability. User-friendly 1200ms as it was set before would just be unreadable. The rest of the variables were already discussed previously.

Lines 30-35:  
This task is just the to set the hold time inbetween the tests. For instance, tick(2) means wait for 2 clock cycles.

Lines 38-41:  
Each line is a separate task for each type of coin input we have. For instance read pulse(5) as follows:  
1) On the negative edge set p5 to 1  
2) Wait for one positive edge  
3) On the next negative edge set p5 to 0  
This mimics coin insertion and cancel behaviour

![image](Pasted%20image%2020260113194727.png)

Lines 43-75:  
An actual test. We start from enable=0 and reset=1, wait for 2 clock cycles, set reset to 0, wait for 2 more clock cycles, set enable to 1 and the tests begin. We cover four scenarios:  
1) User pays exactly 45  
2) User pays 10 then requests a refund  
3) User overpays 20¢  
4) User overpays 10¢  
We wait for to clock cycles inbetween inserting coings and 12-16 clock cycles inbetween the tests.

## Waveform.

The results of the testbench may be observed in the waveform. Output were satisfying for given inputs, FSM logic flawlessly produced all the desired results.

![image](Pasted%20image%2020260113194736.png)

## Top.

With the top module everything is straightforward:  
1) We initialize the module  
2) We create separate DUT for each functionality (notice how there is a separate DUT for each of four input buttons we will have)  
3) We assign LEDs to corresponding output states (collect and dispense)  

![image](Pasted%20image%2020260113194748.png)

## Pin Planner.

Here we do everything according to documentation.

![image](Pasted%20image%2020260113194758.png)

## YouTube link.

https://youtu.be/rqPYg5X6sJ0

# Analysis

From the waveform we made sure that main outputs, that is, the value to display, dispense, collect, are satisfactory. However, besides our main logic we also have brand new debounce functionality and displaying 2-digit number. To evaluate their performance, we bring our FPGA and see how it performs:

1) We observe that, as the debounce code promised, presses are consistent. There are no false pulse detections, and the button sends pulse only once with each press  
2) Regarding how number is displayed, again, as expected, since the frequency of switching between displays is very high, the way number appears is not visually disturbing for human eye  

# Conclusion

The Results came out as expected because of prior testing using the test bench code and waveform produced. Let’s quickly recap:

1) Analyze what the functionality is we are aiming to achieve. In this lab’s case it’s utilizing the FSM logic just like in the previous lab  
2) Discuss the possible issues and how we can mitigate them  
3) Write the code based on this functionality  
4) Test the code with comprehensive test bench and ModelSim waveform  
5) Write the top code and do the pin assignment  
6) Connect the board to your computer, run the code and analyze the results
