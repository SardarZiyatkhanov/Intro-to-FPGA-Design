# Objective

The goal of the lab is to reflect on a bit sequence given to the FPGA board using an input key. If after series of inputs, there is a sequence of 1011, one of the LED displays must turn on. The behavior of the algorithm is polished with the clock. With each clock cycle the board checks for the state of the state machine and shows its order on a 7-segment display. The clock also dictates for how long the LED stays on – exactly 1 clock cycle. Additionally, the reset button brings the state back to the beginning. The logic will become much clearer in the next section of the report.

# Background

As it was stated earlier, 1011 is the desired sequence. Let’s see what the diagram for Moore’s state machine for this exact sequence looks like on paper:

![image](Pasted%20image%2020260113190458.png)

The graph above shows that the states change from S0 to S4 as long as the sequence changes from 1 to 0 to 1 to 1. The output is zero on any change of the state except for transition S3->S4 where the sequence 1011 requirements are fulfilled. This is where our LED shows up indicating success

# Results

Main code is carefully explained in the texted version after the screenshot:

![image](Pasted%20image%2020260113190512.png)

```verilog
// Declaring the module
module seq1011_moore (
    input  wire       clk,   //clock input
    input  wire       rst_n, // reset input
    input  wire       x,     //serial input
    output reg        z,     //output to define LED state
    output reg  [2:0] state  //state of the state machine
);

    //instantiating states
    localparam [2:0]
        S0 = 3'd0,
        S1 = 3'd1,
        S2 = 3'd2,
        S3 = 3'd3,
        S4 = 3'd4;

    reg [2:0] next_state;

    // set z to 1 if state 4 was reached
    always @(*) begin
        z = (state == S4);
    end

    //switch statement to see what state to go to depending on input
    always @(*) begin
        next_state = state;
        case (state)
            S0: next_state = (x ? S1 : S0);
            S1: next_state = (x ? S1 : S2);
            S2: next_state = (x ? S3 : S0);
            S3: next_state = (x ? S4 : S2);
            S4: next_state = S0;
            default: next_state = S0;
        endcase
    end

    //get back to state 0 on negative edge of the clock cycle
    // and go to the next state otherwise
    always @(posedge clk or negedge rst_n) begin
        if (!rst_n)
            state <= S0;
        else
            state <= next_state;
    end

endmodule
```

Testbench:

```verilog
`timescale 1ns/1ps

// Module declaration
module tb_seq1011_moore;

    reg clk, rst_n, x;          // input bits
    wire z;                     // output bit
    wire [2:0] state;           // states check

    // main module instance
    seq1011_moore dut (
        .clk   (clk),
        .rst_n (rst_n),
        .x     (x),
        .z     (z),
        .state (state)
    );

    // clock
    initial clk = 0;
    always #5 clk = ~clk;

    // task for reseting sequence for 2 clock cycles
    // used to set a delay in between cases
    task do_reset;
        begin
            rst_n = 0;
            x = 0;
            repeat (2) @(posedge clk);
            rst_n = 1;
            @(posedge clk);
        end
    endtask

    // the task is to run different sequences for different amount of matches one by one
    // it was done for visual intuitiveness
    task run_case;
        //following are the parameters passed to the task
        input [127:0] name;     //name of the case
        input integer nbits;    //number of bits we want to passs
        input [255:0] stream;   // stream of bits themselves
        integer i;

        begin
            for (i = 0; i < nbits; i = i + 1) begin
                @(negedge clk)
                    x = stream[nbits-1-i]; // set x up
                @(posedge clk);
                #1;
            end
        end
    endtask

    // run all 3 cases
    initial begin
        do_reset();
        run_case("CASE1 no match", 12, 12'b000100010000);

        do_reset();
        run_case("CASE2 one match", 10, 10'b0010110000);

        do_reset();
        run_case("CASE3 multi non-overlap", 9, 9'b101101011);

        $finish;
    end

endmodule
```

Waveform:

![image](Pasted%20image%2020260113190623.png)

![image](Pasted%20image%2020260113190629.png)

If the waveform is examined carefully it is clear that it is divided into three parts with a 2 clock cycles gap: the first sequence that does not show signs of the desired sequence, hence the 0 output (LED is off), the second and third sequences that have one and two 1011 detected respectively. Moreover, with each clock cycle waveform shows the updates in the state number.

The decoder:

![image](Pasted%20image%2020260113190647.png)

Just like any other BCD to 7 segment decoder that we implemented in the previous lab, this one converts inputs to the corresponding configuration of segments on a display on the FPGA board.

Top module:

```verilog
module top (
    input  wire       clk50,         // clock
    input  wire       reset_n_key,   // reset key
    input  wire       key1_n,        // input toggle
    output wire       led0,          // LED
    output wire [7:0] dig,           // 7-seg segments
    output wire [5:0] sel            // displays selection
);

    // pressing key1 sends input 1 pulse
    wire x = ~key1_n;

    // slow clock for enhanced user experience
    parameter integer CLK_HZ = 50000000;
    parameter integer BIT_HZ = 2;
    localparam integer DIV = (CLK_HZ/(2*BIT_HZ));

    reg clk_bit;
    reg [25:0] cnt;

    // clk50 -> clk_bit divider
    always @(posedge clk50 or negedge reset_n_key) begin
        if (!reset_n_key) begin
            cnt <= 0;
            clk_bit <= 0;
        end else if (cnt == DIV-1) begin
            cnt <= 0;
            clk_bit <= ~clk_bit;
        end else begin
            cnt <= cnt + 1;
        end
    end

    wire z;
    wire [2:0] state;

    // moore state machine instantiation
    seq1011_moore u_fsm (
        .clk   (clk_bit),
        .rst_n (reset_n_key),
        .x     (x),
        .z     (z),
        .state (state)
    );

    assign led0 = z; // if z=1 LED is lightened up

    // display state number using the decoder
    wire [7:0] seg_raw;
    sevenseg_ca_active_low u_7seg (
        .val({1'b0,state}),
        .seg(seg_raw)
    );

    assign dig = seg_raw;
    assign sel = 6'b111110; // enable only SEL0 digit

endmodule
```

# Final steps

After all of our codes are ready we do the pin assignment according to the manual and run the code using programmer tool. The results may be observed in the following youtube video:  
https://youtu.be/1eSseFv4Cfc

# Analysis

After understanding the Moore FSM behavior, we implement the same logic in Verilog. A 1 at the start moves the machine from S0 to S1. Releasing the button for one clock (input 0) advances it from S1 to S2. Pressing the button for the next two clocks (input 1, then 1) moves the state from S2 to S3 and then S3 to S4, where the LED turns on for one clock cycle. After reaching S4, state machine returns to S0 regardless of whether input was 0 or 1.

# Conclusion

In the 5th lab we used Verilog with Quartus prime tool to simulate the Moore’s state machine for sequence 1011. We used KEY1 as a serial input to generate the sequence of bits based on which FPGA display outputs the number of states in the state machine and reset button to reset the states showed on a display. An LED emits light if our sequence is detected. Every process is governed by the clock on its positive edge.

- **Note: This Lab was done in Collaboration with my teammate, Said Ahadov, so the link to the video leads to his YouTube channel**
