# Lab-8: Exercises – Model Answers - ALU Lab 

## Exercises
1. Add overflow detection for addition and subtraction.  
2. Extend the ALU to 8-bit width.  
3. Implement increment and decrement operations.  
4. Use a parameterized opcode width and input size.  
5. Design a register file to connect to the ALU inputs and outputs.  

---

## Answer Model

### 1. Add overflow detection for addition and subtraction
Overflow occurs when the result exceeds the representable range.  
- **Addition**: Overflow if the carry into MSB differs from the carry out.  
- **Subtraction**: Overflow if operand signs differ and result sign is inconsistent.  

```verilog
assign overflow_add = (A[3] & B[3] & ~Result[3]) | (~A[3] & ~B[3] & Result[3]);
assign overflow_sub = (A[3] & ~B[3] & ~Result[3]) | (~A[3] & B[3] & Result[3]);
```

---

### 2. Extend the ALU to 8-bit width
Update I/O declarations:  
```verilog
input [7:0] A, B;
output [7:0] Result;
```
Carry and overflow logic must be updated for the 8th bit. The design logic remains the same, showing modular scalability.

---

### 3. Implement increment and decrement operations
```verilog
4'b1000: Result = A + 1; // Increment
4'b1001: Result = A - 1; // Decrement
```

---

### 4. Use parameterized opcode width and input size
```verilog
parameter WIDTH = 8;     
parameter OPCODE = 4;    

input [WIDTH-1:0] A, B;
input [OPCODE-1:0] alu_sel;
```
This improves reusability for 16-bit or 32-bit ALUs.

---

### 5. Design a register file for ALU
A register file provides storage for operands and results.  

```verilog
reg [7:0] regfile [0:7];
assign A = regfile[read_addr1];
assign B = regfile[read_addr2];

always @(posedge clk) begin
  if (we) regfile[write_addr] <= Result;
end
```

This enables the ALU to be integrated into a CPU datapath.

---


