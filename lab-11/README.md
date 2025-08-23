# Lab-11: Exercises – Model Answers - Multiplier Exercises 

## Exercises and Solutions

1. **Extend the multiplier to handle signed numbers using two's complement.**  
   In Verilog, declare operands and product as *signed* so that synthesis infers two's-complement arithmetic:
   ```verilog
   module mul_signed #(parameter W=8)(
     input  signed [W-1:0]  A, B,
     output signed [2*W-1:0] P
   );
     assign P = A * B;  // two's-complement multiply
   endmodule
   ```
   *Alternative approach:* compute the sign separately as `A[W-1] ^ B[W-1]`, multiply magnitudes of |A| and |B|, then negate the product if sign=1.

---

2. **Design an 8-bit multiplier using a hierarchical approach.**  
   Build 8×8 from four 4×4 blocks and add/shift their partial products:
   

   ```verilog
   // 4x4 unsigned building block
   module mul4 (input [3:0] a,b, output [7:0] p);
     assign p = a * b;
   endmodule

   module mul8_hier (
     input  [7:0] A, B,
     output [15:0] P
   );
     wire [3:0] Ah = A[7:4], Al = A[3:0];
     wire [3:0] Bh = B[7:4], Bl = B[3:0];

     wire [7:0] pLL, pLH, pHL, pHH;
     mul4 u_ll(.a(Al), .b(Bl), .p(pLL));
     mul4 u_lh(.a(Al), .b(Bh), .p(pLH));
     mul4 u_hl(.a(Ah), .b(Bl), .p(pHL));
     mul4 u_hh(.a(Ah), .b(Bh), .p(pHH));

     wire [9:0] cross = {2'b0,pLH} + {2'b0,pHL}; 
     assign P = ({pHH,8'b0}) + ({cross,4'b0}) + {8'b0,pLL};
   endmodule
   ```

---

3. **Implement a pipelined multiplier and compare performance.**  
   Register inputs and outputs to increase Fmax. On FPGAs this maps to DSP blocks with built-in pipeline stages.
   ```verilog
   module mul8_pipe #(parameter W=8)(
     input  wire              clk,
     input  wire              en,
     input  wire [W-1:0]      A, B,
     output reg  [2*W-1:0]    P
   );
     reg [W-1:0] A_r, B_r;
     always @(posedge clk) if (en) begin
       A_r <= A; B_r <= B;
     end

     wire [2*W-1:0] P_c = A_r * B_r;
     always @(posedge clk) if (en) P <= P_c;
   endmodule
   ```
   - **Combinational multiply:** 1-cycle latency, lower Fmax.  
   - **2-stage pipeline:** 2-cycle latency, but one result per cycle throughput, much higher Fmax.

---

4. **Simulate all A×B combinations for A,B ∈ [0,15].**  
   ```verilog
   module tb_mul4;
     reg  [3:0] A,B;
     wire [7:0] P_dut;
     integer a,b;
     mul_signed #(.W(4)) dut (.A($signed({1'b0,A})),
                              .B($signed({1'b0,B})), .P(P_dut));
     initial begin
       for (a=0; a<16; a=a+1)
         for (b=0; b<16; b=b+1) begin
           A=a[3:0]; B=b[3:0];
           #1;
           if (P_dut !== A*B)
             $display("Mismatch: %0d*%0d, got %0d exp %0d",
                       A,B,P_dut,A*B);
         end
       $display("Exhaustive test complete.");
       $finish;
     end
   endmodule
   ```

---

5. **Compare resource usage between combinational and sequential designs.**  

- **Combinational (array/tree):**  
  - Area ~O(n²) LUTs (if no DSPs).  
  - 1-cycle latency, lower Fmax.  
  - With DSPs: few DSPs + routing, throughput = 1 result/cycle (pipelined).  

- **Sequential (shift-add / Booth):**  
  - Area ~O(n).  
  - Latency: n cycles (shift-add).  
  - Throughput: 1 result per n cycles.  
  - Higher Fmax per stage but lower throughput.  

👉 Use DSP-backed pipelined multipliers for **throughput**, sequential designs for **area efficiency**.


