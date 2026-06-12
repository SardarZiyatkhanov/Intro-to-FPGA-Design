# Objective

As an objective for this lab, we were required to simulate multiplexer (4x1) and the decoder in the Quartus Prime software with test bench code and waveforms, simulated for both of them, as well, in order to see how they are working. 

# Background
Before we jump into the code and simulation, it will be useful to have some background information related to the principle of work, both, MUX and Decoder. 

## Multiplexer 
- Multiplexer is a digital circuit which allows to select one input from many of other existing inputs and send it to the single output, based on a Control Signals' parameters, which act as switches.

![image](Pasted%20image%2020251016081717.png)

- For instance, in 4x1 MUX we have 4 inputs, 2 Select lines (control), and 1 output:
	- When S=00, Y = I0
	- When S=01, Y = I1
	- When S=10, Y = I2
	- When S=11, Y = I3
- Overall, to all Mux applies the following formula: $N = 2^m$, where m represents the number of Select lines, and N - number of input lines. Output is always 1.

## Decoder
- Decoder is also a digital circuit that works in a similar way like MUX; it takes input (like Select lines in MUX) and produces corresponding to that inputs outputs (the one that is Regular Input in MUX).
![[Pasted image 20251016082926.png]]
	![[Pasted image 20251016083045.png]]
	- When Input Lines are 00 - Output is 0001
	- When Input lines are 01 - Output is 0010
	- When Input Lines are 10 - Output is 0100
	- When Input Lines are 11 - Output is 1000

# Results and Analysis

Now, having enough background, we can move to the review of our Verilog codes of both circuits' and their testbenches with Waveforms.

## Multiplexer

- First, we will focus on the 4 to 1 MUX code, I will explain the logic here chunk-by-chunk, going into the details, so anybody could grasp the logic.

![image](Pasted%20image%2020251016110114.png)

```verilog 
module mux4to1(
    input [3:0] data_in,
    input [1:0] sel,
    output reg y_out
);
```

- Although indentations in Verilog are not so important, for instance, as in Python, it's still visually easy to understand the code with them.
- First, we define our mode name with descriptive language `mux4to1` and it is important to make sure that the file's name in Quartus corresponds to the module's name.
- Now, inside our module block we can first introduce our input values. We have two set of inputs:
	- Regular Inputs `data_in` which are essentially 4-bit input bus, as an array (`data_in[3]` ... `data_in[0]`)
	- Select Inputs `sel` (`sel[0]` and `sel[1]`)
- And lastly, we declare our output as `output reg y_out`. It is declared as `reg` since it is assigned inside the `always` block.

```Verilog
always @ (data_in [3:0])
begin
```
- `always` here is the sensitivity list, the block executes only when `data_in` changes and this ensures that we use combinational logic where no clock is involved.
- by `begin` we mark the beginning of the block of code associated with `always` statement.

```Verilog
case (sel[1:0])
		2'b00: y_out = data_in[0];
		2'b01: y_out = data_in[1];
		2'b10: y_out = data_in[2];
		2'b11: y_out = data_in[3];
endcase
```

- by `case(sel[1:0])` this statement checks the value of 2-bit `sel` to decide which action is needed to be taken in order to run correctly.
- In `2'b00: y_out = data_in[0];` if `sel` is 00 we assign the value of `data_in[0]` to `y_out`. Same logic for `2'b01`, `2'b10`, `2'b11`.
- by `endcase` we end the `case`

```
end
endmodule
```

- by `end` we end the `always` block
- by `endmodule`, we, as name suggests, end conclude the module.

- When we compile that code in Quartus Prime and run RTL Simulation, we can see the following:

![image](Pasted%20image%2020251016115054.png)

### Multiplexer's Testbench Code

![image](Pasted%20image%2020251016113910.png)

- The simulation time settings are specified in the very first line of the code. The time unit is 1ns, nanoseconds, which indicates how long the simulation should wait after "#n" is triggered. The precision is 1 ps, picoseconds; in other words, the simulation tracks the code to an accuracy of 1 ps.  
- The test bench module is then declared and given the name `tb_mux4to1` (and the file we name correspondingly)
- We initialize our variables in the next three lines. Our data and selection inputs are register arrays, and our output is a wire. A wire is required to track progress with the waveforms that will be covered in more detail in the results section, but it does not explicitly store any value. Registers are usually used for storage.
- Following the inputs and outputs, we initialize a multiplexer of type `mux4to1` that originates from the module in our main 4x1 multiplexer code and call it `yut` (note: it was a typo; `yut` will be referred to as `dut` (Device Under Test) throughout the report).  
- We link multiplexers to the test bench variables in `dut`.  
- The executable code section surrounded by `begin` and `end`, followed by the actual execution block initial, which signifies that the program will be executed once.
- The `#10` command specifies an interval of ten time units (10 nanoseconds) for each of the 4 data inputs and the selection input values in segment 4 from 00 to 11. Following each test, the console specified by the `$display` command shows the values of our inputs and the output result matching to them.  
- The program is completed by the block `endmodule`.

### Running Testbench and Observing Waveforms.

![image](Pasted%20image%2020251016120003.png)

Here the ModelSim waveform provides us with a visual verification of the functionality of `mux4to1` module. The simulation executes the testbench code, which applies a series of predefined inputs and allows us to observe the corresponding output over time. The key signals displayed are the 4-bit data input `data_in`, the 2-bit select line `sel`, and the single-bit output `y_out`. Each change in the inputs occurs at 10 nanosecond intervals, as specified by the `#10` delay in the testbench code.

- **Input Signals** (`data_in` & `sel`): The testbench cycles through four specific test cases.

1. `sel` = 00, `data_in` = 0001
2. `sel` = 01, `data_in` = 0010
...
3. `sel` = 11, `data_in` = 1000

The `sel` input systematically covers all four possible binary combinations and makes sure that every selection path is tested. The waveforms for the individual bits of `data_in` and `sel` show a **logic high** (a "bump") when value is 1, and a **logic low** when their value is 0.

- **Output Signal** `y_out`: the `y_out` waveform is the result of the multiplexer's logic.

- When `sel` is 00, the MUX selects `data_in[0]`. Since `data_in` is 0001, `data_in[0]` is `1`, so `y_out` becomes 1.
- When `sel` is 1, the MUX selects `data_in[1]`. Since `data_in` is 0010, `data_in[1]` is 1, so `y_out` remains 1.

This pattern continues for all test cases. The testbench was cleverly designed so that the select lines always point to the specific bit in `data_in` that is asserted high. Because of this, the output `y_out` appropriately stays at logic 1 throughout these tests.


## Decoder

![image](Pasted%20image%2020251016123841.png)

- here the logic is essentially the same as in MUX. We:
	- declare new module `decoder`
	- declare input `A` as an array and output reg as `D`.
	- `always` also executes the block only when `A` changes.
	- behavior inside `case` is clear and straightforward.
	- by `default` we simply avoid the issue when the output is undefined, returning 0000.

- RTL simulation design:

![image](Pasted%20image%2020251016124552.png)

### Decoder's Testbench Code

![image](Pasted%20image%2020251016124330.png)

- Logic here is mostly the same as in MUX, no need to explain.

### Running Testbench and Observing Waveforms.

![image](Pasted%20image%2020251016125830.png)

The new red waves are one exception to the same idea. According to the fifth and final test in our test bench code, the red lines in this case indicate an unknown input. Because of the default output parameters that were specified in our main decoder code, it seems that the dut has no trouble handling unknown input. The output is just set to 0000 by the dut.

# Conclusion
- The simulation outputs of both the 2-by-4 Decoder and the 4-by-1 Multiplexer were accurate and agreed perfectly with the expected theoretical outcome. The experiment has demonstrated the function of the multiplexers and the decoders effectively through Quartus Prime in Verilog.
