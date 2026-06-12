# Objective

- The objective of this lab was to write appropriate codes in Quartus Prime, simulate the testbench code in MultiSim, correctly implement the pin allocation in Pin Planner (inside Quartus Prime) , and finally to run it on our Altera Cyclone IV FPGA board through Programming section of Quartus Prime to demonstrate how particular 4-bit binary numbers can be visually represented as decimal numbers on one of multiple 7-segment displays on the board     

# Background

- We can see in the picture below, the truth table for 7-segment display, displaying decimal digits (0-9): 

![image](Pasted%20image%2020251106213647.png)

- We can construct this table knowing the pins for each segment of the display, as depicted in that image:

![image](Pasted%20image%2020251106214108.png)

- Well, we will show digits 0, 1, 2, 3, ..., 9:
	- 0 in binary is 0000 (for this, all segments should be high, except G)
	- 1 in binary is 0001 (only B and C segments should be high)
	- ...
	- 9 in binary is 1001 (all segments should be high, except E)

- With that logic we design truth table for this particular task.

# Results

- When we talk about 7-segment display, the first thing that should come to our mind is a decoder. Having ready truth table we can construct our decoder. We know that for n input lines of a decoder, we can get up to 2^n output lines; however here we just need 10 outputs, to display 10 digits (0-9). 

- Now we can construct simulate our decoder behavior using Verilog coding (explained using comments in code block):

![image](Pasted%20image%2020251106215834.png)

```verilog
module seven_segment_for_FPGA(
    input [3:0] num,  // here 4 bits for input numbers
    output reg [6:0] seg // here is 7 bits for output
);

    always @ (num) begin
        case (num)
        // from our truth table we can be sure what to write 
            4'b0000: seg = 7'b1111110;    // 0
            4'b0001: seg = 7'b0110000;  // 1
            4'b0010: seg = 7'b1101101;   // 2
            4'b0011: seg = 7'b1111001;    // 3
            4'b0100: seg = 7'b0110011;     // 4
            4'b0101: seg = 7'b1011011;   // 5
            4'b0110: seg = 7'b1011111;  // 6
            4'b0111: seg = 7'b1110000;    // 7
            4'b1000: seg = 7'b1111111;   // 8
            4'b1001: seg = 7'b1111011;  // 9
            default: seg = 7'b0000000; // when input is unexpected (unknown) we select default - everything is off
        endcase
    end

endmodule

```


- After that we have to make sure that our code runs correctly, therefore, write testbench code with desired inputs:

![image](Pasted%20image%2020251106215734.png)

- and verify it looking at the waveform we got:

![image](Pasted%20image%2020251106215913.png)

- Next, we can write the "top.v" code to define all our desired input and output pins in a Pin Planner and then assign them correctly relying on a table from Cyclone IV documentation. We have 4 buttons: RESET, KEY1, KEY2, KEY3, which we also specified in top code, and using which we will control the output on the board. Here, also, we have specified on which display our digits will be displayed (on the leftmost). And it is important to specify the top.v code as a top level code in our project, as the name and the functionality suggest.  

![image](Pasted%20image%2020251106220553.png)

![image](Pasted%20image%2020251106220610.png)

- In location column we specify correct pin names for each input/output, again, according to the documentation table (and specify 3.3V - for this model of board it is required).

- After that we can compile the code once again, and then, move to the Programmer section in Quartus. 

- Connect mini-USB cable for power supply, and mini-USB Blaster cable for data transmission to the board, and other ends, regular USBs, into our PC's USP ports. Then, find device in Programmer and, finally, run it, pressing start button. 

# Analysis

- After we run it we can be sure for the correctness of our program. You can check it by watching the video after following the link below:
https://youtu.be/LCVO8Dp0fMc

# Conclusion

Overall, we can see how we can demonstrate decimal visualization of binary numbers using keys on an FPGA board and its 7-segment display. We wrote Verilog code in Quartus prime to translate 4-bit inputs in binary to outputs for the segments of 7-segment display to show decimal digits from 0 to 9. After, we made sure that code is right, running testbench code with specified inputs in MultiSim and analyzing the waveform for double check. Then wrote top level code, did pin assignment, compiled everything once more, physically connected our board using cables, looked for and found it in programmer section and start it, all that in Quartus prime. After that, we can press 4 physical buttons on the board (1 when pressed, 0 when released) and see how digits appear on the display.  
