# Lab-3: Exercises – Model Answers

## Questions and Exercises

1. **Modify the testbench to include more delays and corner cases.**  
   In the testbench, additional delays can be added using the `#` operator (e.g., `#10`, `#50`).  
   Corner cases include input combinations such as all zeros, all ones, and alternating bits.  
   For example, testing `0000` and `1111` ensures the design handles extremes correctly.

2. **What is the functional difference between XOR and XNOR?**  
   - **XOR (Exclusive OR):** Outputs `1` only when the number of high inputs is odd (e.g., `0 ⊕ 1 = 1`).  
   - **XNOR:** The complement of XOR. It outputs `1` when the inputs are equal (both 0 or both 1).  

3. **Describe a real-world application where NAND gates are preferred.**  
   NAND gates are widely used in digital circuit design because they are *functionally complete*,  
   meaning any logic function can be built from them.  
   A practical example is in memory circuits (SRAM/DRAM), where NAND structures are used to build storage cells.

4. **Write a 2-input multiplexer using only logic gates in Verilog.**  

   ```verilog
   module mux2to1 (input a, input b, input sel, output y);
       assign y = (a & ~sel) | (b & sel);
   endmodule


