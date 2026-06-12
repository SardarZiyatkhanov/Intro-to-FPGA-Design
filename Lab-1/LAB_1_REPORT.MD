### Objective

The objective of our lab was to write the Verilog code in Quartus Prime software and let it design the representation of two circuits, which are **Half-Adder** and **Full-Adder**. What distinguishes those two Adders from each other is that Half-Adder basically adds two one-bit inputs and gives the results in a form of sum and carry, while Full-Adder allows us to introduce the third input in a form of input-carry and have three 1-bit inputs, resulting in 2 outputs - sum and output-carry. In this report I will walk through all the fundamental knowledge required to accomplish this task.

### Background

To represent both the Half-Adder and Full-Adder in a form of a circuit of logic gates and understand the logic behind it, is important to be familiar with basic Boolean algebra, with the concepts of minterms and Maxterms, and have the ability to construct truth tables and elaborate on them using minterms or Maxterms.

##### *Half-Adder*

| A   | B   | C-out | Sum |
| --- | --- | ----- | --- |
| 0   | 0   | 0     | 0   |
| 0   | 1   | 0     | 1   |
| 1   | 0   | 0     | 1   |
| 1   | 1   | 1     | 0   |

above, I have constructed truth table based on basic Boolean algebra properties, and here it is clear that we have output carry only when both summands are 1.

Now we can easily express the logic, using minterms or Maxterms. Considering carry, it is more rational to use minterms, as we have '1' only once, therefore less logic gates will be involved in a circuit, making it cost-efficient. And considering sum results, it is does not matter which to choose, since we have equal numbers of 0's and 1's, but I will focus on minterms, since I have used them in a carry representation.

So, basically, minterm is the AND of all variables ("multiplication") that should result in 1 for each truth-table row.

1. It is obvious that both A and B, that are responsible for carry to be 1, are 1 as well, so we do not negate them:
**C-out = AB** 

, additionally, since AB are 11, in decimal 11 is 3, so it is represented as m3, therefore:
C-out = m3

2. In the first 1 in Sum, only A is 0, so we negate it (and 01 = 1), m1 = A'B
In the second 1, B is 0, and we negate it this time (and 10 = 2), m2 = AB'
**Sum = m1 + m2 = A'B + AB'**
, and by definition we know that this is NOR gate, so
**Sum = A⨁B**

For manual visual representation, we can create a circuit in Logisim:

![image](Pasted%20image%2020250930194036.png)

So, basically, that it. Now it is easy to draw a circuit manually or represent it in Quartus Prime using Verilog as we understood the logic behind it, which is basically the aim of this lab. I will come to this point very soon.

##### *Full-Adder*
When we come to the Full-Adder, here we start with the truth table as well, introducing the third summand - carry-input.

| A   | B   | C-in | C-out | Sum |
| --- | --- | ---- | ----- | --- |
| 0   | 0   | 0    | 0     | 0   |
| 0   | 0   | 1    | 0     | 1   |
| 0   | 1   | 0    | 0     | 1   |
| 0   | 1   | 1    | 1     | 0   |
| 1   | 0   | 0    | 0     | 1   |
| 1   | 0   | 1    | 1     | 0   |
| 1   | 1   | 0    | 1     | 0   |
| 1   | 1   | 1    | 1     | 1   |

In C-out we can focus on either minterms or Maxterms, and in Sum as well, therefore I will focus on minterms.

1. So,  for C-out:
C-out = m3+m5+m6+m7 = A'BC+AB'C+ABC'+ABC = (A'B+AB')C + AB(C'+C) = **(A⨁B)C + AB**

2. And, for Sum:
Sum = m1+m2+m4+m7 = A'B'C+A'BC'+AB'C'+ABC = B'(A'C+AC') + B(A'C'+AC) = B'(A⨁C) + B(A⨁C)' = (B⨁(A⨁C)) = A⨁B⨁C = **(A⨁B)⨁C**

So, in Logisim we get something like that:

![image](Pasted%20image%2020250930195359.png)

and if we look closer, we can see there the familiar pattern - two Half-Adder, continued by OR gate:

![image](Screenshot%202025-09-30%20195802_updated.png)

Now, having all this knowledge, we finally can move to visualization of the discussed Adders in Quartus Prime, using Verilog.

### Results
##### *Half Adder in Quartus Prime*

![image](Pasted%20image%2020251002134355.png)

Here is the code for Half-Adder:
```Verilog title:"Half_Adder"
module very_half_adder(
    input a,       // First input bit
    input b,       // Second input bit
    output sum,    // Sum output (XOR of a and b)
    output c_out   // Carry output (AND of a and b)
);
    xor gate_xor (sum, a, b);   // XOR gate for sum
    and gate_and (c_out, a, b); // AND gate for carry
endmodule

```

First, I will walk through the code, trying to explain the "language" itself.

1. We defined a circuit block using `module` and gave it the name of "**very_half_adder**" (since I had some issues creating new file, I had to rename the "half_adder" as "very_half_adder" and "full_adder" as "half_adder").
2. In the created "very_half_adder" module, we declare out input signals a and b, writing `input a` and `input b`, then we declare output signals: `output sum` and `output c_out`. Everything within module block, enclosed by parenthesis following with semicolon.
3. Now we can use built-in Verilog primitives to perform `xor` and `and` gates' functions. `xor gate_xor(sum, a, b);` means the xor of "a" and "b" will be "sum". Same logic goes with `and` where we write `and gate_and (c_out, a, b);`
4. Finally, we mark the end of the module, writing `endmodule` at the bottom.

##### *Full Adder in Quartus Prime*

![image](Pasted%20image%2020251002134420.png)

![image](Pasted%20image%2020251002134449.png)

As we saw earlier, it is possible to construct the **Full-Adder** by combining **two Half-Adders and an OR gate**, making it possible to add three binary inputs (`a`, `b`, and `c_in`) and produce both a `sum` and a `carry-out`.

```Verilog title:"Full Adder"
module half_adder(   // actually full adder
    input a,         
    input b,         
    input c_in,      
    output sum,      
    output c_out     
);
    wire s1, c1, c2; 

    // First half adder
    very_half_adder HA1(
        .a(a),
        .b(b),
        .sum(s1),
        .c_out(c1)
    );

    // Second half adder
    very_half_adder HA2(
        .a(s1),
        .b(c_in),
        .sum(sum),
        .c_out(c2)
    );
    or gate_or(c_out, c1, c2);
endmodule
```

1. Just like the half adder, we start with the `module` keyword and name it `"half_adder"` (though it really is a **full adder**, due to naming restrictions explained earlier). The inputs are `a`, `b`, and `c_in`, and the outputs are `sum` and `c_out`.
2. Inside, we declare three internal wires (`s1, c1, c2`) that temporarily hold intermediate results between the components.
3. The **first half adder (`HA1`)** takes inputs `a` and `b` and produces two outputs: `s1` (partial sum) and `c1` (carry from this addition).
4. The **second half adder (`HA2`)** then adds `s1` and the third input `c_in`. Its sum output becomes the final `sum`, and it generates another carry `c2`.
5. Finally, both carries (`c1` and `c2`) are combined using an **OR gate**, because if either half adder generates a carry, the overall full adder must produce a carry-out.
6. The `endmodule` marks the end of this circuit description.

