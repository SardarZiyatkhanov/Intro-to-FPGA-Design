# Objective

The objective of the Lab 4 was to design and simulate 3-Bit counter with Enable, and Reset pins by writing several Verilog HDL programs in a Quartus Prime software using such tools as Pin Planner and Programmer to demonstrate the functionality of the scheme on Alinx AX4010 Cyclone IV FPGA board, making sure that the code is correct and provides desired output by observing Waveform structure in ModelSim software beforehand. Overall, the system was expected to do the following on the one of the six 7-segment displays located on the board:
1. To start counting only while Enable button is pressed and kept in that position.
2. Count from decimal 0 up to decimal 7 (since the largest 3-bit number is $111_2$ = $7_{10}$) and cycle again, like: 0, 1, ..., 7, 0, 1, 2, ...., 0...
3. Hold the current value, when Enable button is released.
4. Display 0 when Reset button is pressed.

# Background

## 1. Theoretical Digital Logic Background

In this lab, for the first time, we can observe the fusion of conventional Combinational Logic that we have worked with so far and where the output depends only on current input, with the Sequential Synchronous Logic where output depends on current input and past states (has "memory") and circuit updates its state on clock edges (rise or fall). Combinational Logic is represented by the BCD-to-7-segment decoder which is used to convert 4-bit binary input to the visual representation of that input in decimal form on 7-segment display; whereas, Sequential Synchronous Logic is demonstrated by 3-bit counter, made up of 3 Flip-Flops, which can be conceived as memory cells, having the ability to store their previous states. 

### 1.1 3-Bit Binary Counter

3-bit binary counter lets us to iterate through the digits from 0 to 7, while the value increases by 1 with each clock cycle during clock edge (positive in our case) and goes back to zero, repeating the process.

In my code, which will be discussed later, I have implemented 3-bit UP Counter, which switches its states only on the positive rise of the clock. In the module, responsible for counter, I have specified `always @(posedge clk or posedge reset)`, which, as far as I figured out, refers to the D flip-flops. Below, the schematics of 3-bit UP counter using three D FFs can be observed with the waveforms showing the output of the counter during each clock cycle, triggered by positive edge:

![image](Pasted%20image%2020251204091106.png)

![image](Pasted%20image%2020251203235337.png)

By observing those images we can see that $Q_0$ corresponds to the rightmost digit and $Q_2$ to the leftmost. This can be seen more clearly if we take a look at the table below:

![image](Pasted%20image%2020251203235616.png)

As mention before, the counter in this Lab has 2 control signals:
1. Enable - to allow counting only while pressed.
2. Reset - to reset the count (go back to zero).

### 1.2 BCD to 7-segment Decoder

We have discussed the concepts related to this in previous Lab report. Shortly, the aim is to design a truth table corresponding to the segments of the display, which has the segments named by letters from A to G and DP:

![image](Pasted%20image%2020251106214108.png)

![image](Pasted%20image%2020251106213647.png)

- Depending on the 4-bit input the following decimal digits will be displayed:
	- 0 in binary is 0000 (for this, all segments should be high, except G)
	- 1 in binary is 0001 (only B and C segments should be high)
	- ...
	- 9 in binary is 1001 (all segments should be high, except E)

Although, this time, instead of 4 bits, we have input of 3 bits, resulted from 3-bit counter, FPGA board's display logic works on 4-bit representation; therefore, to overcome this, we simply can add zero from the left side of our 3-bit number, like so:

|Decimal|Binary (3-bit)|BCD (4-bit)|
|---|---|---|
|0|000|0000|
|1|001|0001|
|5|101|0101|
|7|111|0111|

This will be done in top module, where we will "glue" two modules (counter and decoder) together to achieve desired functionality.
## 2. Verilog Implementation

### 2.1 3-Bit Counter Module

![image](Screenshot%202025-12-03%20203210.png)

This module is responsible for realization of 3-bit up counter functionality which will generate slow tick (frequency by which the counting happens), counting itself, reset and enable control.

```verilog 
module _3bit_countr #(
    parameter integer DIVIDER_MAX = 25_000_000 - 1
)(
    input  wire clk,
    input  wire reset,
    input  wire count_enable,
    output reg  [2:0] count_value
);
```

- Parameter `DIVIDER_MAX` with configurable value of 24,999,999 is defined, so at 50 MHz (system clock of our board) 25 million cycles will take approximately 0.5 seconds . Why did not define exactly 25 millions? -> clock-divider register starts at 0, not 1, and when it will reach 24,999,999, we will have exactly 25 million clock cycles. 
- as mentioned before, `clk` input is the system clock running at the 50 MHz frequency.
- `reset` input - when active high will lead counter to go to zero.
- `count_enable` input will allow counter to change when 1, and tell counter to hold it current value even when tick occurs when 0. Connected to enable button.
- `count_value` output is 3 bits of width and declared as register since assigned in "always" block

```verilog
    reg  [24:0] clk_divider_counter;
    wire slow_tick_reached;
    assign slow_tick_reached = (clk_divider_counter == DIVIDER_MAX);

```
- `clk_divider_counter` has a width of 25 bits, so it can count up to 25 millions ($2^{25}$ = 33,554,431) and is used to generate ticks.
- `slow_tick_reached` - this becomes 1 for one clock cycle when the divider counter hits maximum value set before. 


- By `always @(posedge clk or posedge reset) begin` we define sequential process triggered on rising edge of clk or reset. Rest is explained by comments in code block:
```verilog
    always @(posedge clk or posedge reset) begin
	    //when reset is pressed (=1), we reset the divider to 0
	    //and counter's value to 0 (start the process from the scratch)
        if (reset) begin
            clk_divider_counter <= 25'd0;
            count_value         <= 3'd0;
        end 
        //if reset is not pressed we operate normally
        else begin
	        //when 0.5 seconds've elapsed, we restart divider 
            if (slow_tick_reached) begin
                clk_divider_counter <= 25'd0;
                
                // on the tick (rising edge) - update 3-bit counter.
                // Happens only when divider reached max and 
                // enable button is pressed.
                if (count_enable) begin
	                // if reached 7, repeat with 0
                    if (count_value == 3'd7)
                        count_value <= 3'd0;        
                    // other time - increment
                    else
                        count_value <= count_value + 3'd1;
                end
            end 
            //when 0.5 seconds haven' elapsed, we continue counting for
            // divider value 
            else begin
                clk_divider_counter <= clk_divider_counter + 25'd1;
            end
        end
    end
```

### 2.2 3-Bit Counter Testbench Module

![image](Screenshot%202025-12-03%20203217.png)

by the following testbench our goal is to define if reset zeros the counter, enable allows count only when high, counter increments periodically, counter holds value when enable is released, counter cycles after 7 going back to 0.

Testbench code is straightforward. Explained using comments:

```verilog
`timescale 1ns/1ps

module tb_3bit_countr;

    //tb signals
    reg clk_tb;
    reg reset_tb;
    reg enable_tb;
    wire [2:0] count_tb;

    // Device Under Test
    // using small divider (5 cycles) so simulation is quick
    _3bit_countr #(
        .DIVIDER_MAX(4)
    ) dut (
        .clk          (clk_tb),
        .reset        (reset_tb),
        .count_enable (enable_tb),
        .count_value  (count_tb)
    );

    // creating 50mhz clk toggling every 10ns with 20ns period
    initial begin
        clk_tb = 1'b0;
        forever #10 clk_tb = ~clk_tb;
    end

    initial begin
        reset_tb  = 1'b1; //reset is active
        enable_tb = 1'b0; // enable is not active

        // holdin reset active for 50ns
        #50;
        reset_tb  = 1'b0;   // release reset
        enable_tb = 1'b1;   // enable active -> start counting

        // count for 1ms
        #1000;

        // release enable, test if holds value
        enable_tb = 1'b0;
        #300;

        // activate enable -> continue counting
        enable_tb = 1'b1;
        #1000;

        // activating reset
        reset_tb = 1'b1;
        #100;
        reset_tb = 1'b0; 

        #200;
        $stop; //ending tb
    end

endmodule

```

Waveforms resulted during testbench will be observed in Results. 

### 2.3 BCD to 7 Segment module

In this module we convert 4-bit BCD digit into 8-bit pattern controlling display. (here we use conventional BCD with 4-bit input. Although we have 3-bit input, as mentioned previously, we will consider this in our top module, which will act as a connector between counter and decoder modules)

![image](Screenshot%202025-12-03%20203225.png)

On our board we use a common-anode display, which means that segment turns ON when line is 0 and vice versa; so, the bits of the output correspond to `seg_pattern = {DP, G, F, E, D, C, B, A}`

```verilog
input  wire [3:0] bcd_digit,
output reg  [7:0] seg_pattern
```
- `bcd_digit` 4-bit input from 0 to 9.
- `seg_pattern` 8-bit output for segments

This part is intuitively straightforward, we just mirror the output of the truth table above horizontally and negate digits (since common-anode) and write the code :
```verilog
always @* begin
    case (bcd_digit)
        4'd0: seg_pattern = 8'b1100_0000;
        4'd1: seg_pattern = 8'b1111_1001;
        4'd2: seg_pattern = 8'b1010_0100;
        ...
        default: seg_pattern = 8'b1111_1111;
    endcase
end
```

### 2.4 Top Module

Finally, we have reached the top module, where the functionalities of two separate modules merge into one logic. Additionally, with the help of top module, we can actually connect to board pins and physical components.
In top module we receive 50MHz clock, reset button, enable button signals and result in an output of 7-segm LED signals and digit select lines. 

Code will be explained using comments in code block below:
```verilog
module top_3bit_countr (
    input  wire       clk_50mhz, //system clock
    input  wire       btn_reset_n, //reset button
    input  wire       btn_enable_n, //enable button

    output wire [7:0] seg_pins, // display segments
    output wire [5:0] digit_sel     // six 7-segm displays
);

    //inverting board buttons (active-low) into active-high internal signals
    wire reset_active;
    wire enable_active;

    assign reset_active  = ~btn_reset_n;
    assign enable_active = ~btn_enable_n;

    // counter outputs
    wire [2:0] counter_value;
    wire [3:0] bcd_digit;

    // zero-extend 3-bit counter to 4-bit BCD (most signif vit is always 0)
    assign bcd_digit = {1'b0, counter_value};

    //  counter
    _3bit_countr #(
        .DIVIDER_MAX(25_000_000 - 1)  // ~0.5 sec per count at 50 MHz
    ) counter_block (
        .clk          (clk_50mhz),
        .reset        (reset_active),
        .count_enable (enable_active),
        .count_value  (counter_value)
    );
    // decoder
    bcd_to_7segm_disp disp_driver (
        .bcd_digit   (bcd_digit),
        .seg_pattern (seg_pins)
    );
    // will use the leftmost display
    assign digit_sel = 6'b111110;

endmodule

```

# Results & Analysis

After writing all codes necessary for the Lab and compiling them in Quartus prime making sure that we have no errors, next step is to move to ModelSim and observe waveforms resulted out of TB code:

![image](Screenshot%202025-12-03%20203424.png)

After making sure that waveforms correspond to the scenarios coded in testbench, we can return to Quartus prime and work on pin assignment.

Before doing pin assignment, it is important not to forget about setting top module file as top in hierarchy of the project. After that, we should compile once again, and finally, can open Pin Planner tool of Quartus, where, thanks to our top module code, we can see all the Node names ready to be assigned to actual board pins. Relying on the manual in documentation of our Cyclone IV, we can confidently assign correct Location. Also, it is important to set I/O standard as 3.3V, due to our board requirements.  

![image](Screenshot%202025-12-03%20203435.png)

Finally, we can close pin planner and connect our board physically to PC, then open Programmer tool and start programming the board. After a successful finish of programming, we can finally make sure that we have done everything correctly.

Here is the video showing the result of this Lab:
https://youtube.com/shorts/Npmv3ueEJGI?feature=share

# Conclusion

In conclusion, during this lab, we understood how to implement the functionality of Sequential logic by realization of 3-bit Up counter and how to combine this functionality with a traditional Combinational logic which was represented by regular BCD to 7-segment display decoder very familiar to us. Another important part of this lab was to design a top module, where two modules will achieve communication and where 3-bit output out of counter will be adapted for 4-bit input requirements of decoder. By doing everything right, we achieved the functionality required in this lab - count from 0 to 7 while enable is pressed, hold the value when enable is released and reset to 0 whet reset is pressed.
