# Lab-4: Exercises – Model Answers

## Exercises

1. Modify the full adder to display overflow condition in 4-bit addition.  
2. Implement a 4-bit subtractor using full adders and 2's complement logic.  
3. Extend the design to an 8-bit adder.  
4. Write behavioral Verilog code for the full adder and compare with structural style.  
5. Analyze propagation delay in a ripple carry adder.  

---

## Model Answers

### 1. Modify the full adder to display overflow condition in 4-bit addition
There are two notions of overflow:

- **Unsigned overflow**: occurs when the carry out of the MSB (`C4`) is `1`.  
  `Vu = C4`  

- **Signed (two's-complement) overflow**: occurs when the carry into the MSB differs from the carry out of the MSB.  
  `Vs = C3 ^ C4`  

**Verilog (flag logic only):**
```verilog
assign unsigned_overflow = C4;
assign signed_overflow   = C3 ^ C4;
```

---

### 2. Implement a 4-bit subtractor using full adders and 2's complement logic
Use the identity `A - B = A + (~B) + 1`.

```verilog
wire [3:0] Bn = ~B;
wire       Cin = 1'b1;
{Cout, S} = A + Bn + Cin;
```

- Borrow out for unsigned subtraction: `~Cout`.  
- Signed overflow: `Vs = C3 ^ C4`.

---

### 3. Extend the design to an 8-bit adder
Cascade two 4-bit ripple-carry adders:

```verilog
// Lower nibble
{C4, S[3:0]} = A[3:0] + B[3:0] + Cin;
// Upper nibble
{Cout, S[7:4]} = A[7:4] + B[7:4] + C4;
```

- Signed overflow for 8-bit: `Vs = C6 ^ C7`  
- Unsigned overflow: `Vu = C8`

---

### 4. Behavioral vs Structural Verilog for Full Adder

**Behavioral FA (1-bit):**
```verilog
module fa_beh(input a, b, cin, output sum, cout);
  assign {cout, sum} = a + b + cin;
endmodule
```

**Structural FA (using gates):**
```verilog
module fa_struct(input a,b,cin, output sum, cout);
  wire s1, c1, c2;
  assign s1  = a ^ b;
  assign sum = s1 ^ cin;
  assign c1  = a & b;
  assign c2  = s1 & cin;
  assign cout = c1 | c2;
endmodule
```

**Comparison**:  
- Behavioral → concise, synthesis infers hardware.  
- Structural → exposes gate-level logic, better for learning and timing analysis.

---

### 5. Propagation Delay in Ripple Carry Adder
Let:
- `txor = XOR delay`  
- `tand = AND delay`  
- `tor = OR delay`

- Carry delay per stage: `tc ≈ max(tand + tor, txor + tand + tor)`  
- Sum delay: `ts ≈ 2 * txor`  

For an N-bit ripple-carry adder:
- Worst-case MSB sum delay: `Tsum ≈ (N-1)*tc + ts`  
- Worst-case Cout delay: `Tcout ≈ N*tc`  

**Conclusion**: Delay grows linearly with `N`; faster adders (carry lookahead, carry-select) reduce this growth.


