# Lab-12: Exercises – Model Answers - FSM Serial Adder Exercises 

## Exercises and Solutions

### 1. Modify the FSM to support 8-bit serial addition
Extend the FSM counter to 3 bits (0–7). Iterate the add-shift cycle for 8 cycles. Use two 8-bit shift registers (A, B), one 8-bit result register (S), and a carry flip-flop. At each step:
- Take LSBs of A and B.
- Add them with carry.
- Shift result into S, and shift A and B right.
- Update carry.

---

### 2. Add carry-out as a separate output from the most significant bit
The final carry after the 8th addition becomes `carry_out`. It can be latched once the FSM completes the last cycle.

```verilog
if (bit_cnt==3'd7) carry_out <= carry;
```

---

### 3. Add a ready signal to prevent multiple triggers of the FSM while busy
Introduce a `ready` flag that is high only in `IDLE` or `DONE`. Gate the `start` input with `ready` to prevent retriggering.

```verilog
assign ready = (state==IDLE) || (state==DONE);
```

---

### 4. Convert the FSM from Mealy to Moore implementation and compare
- **Mealy FSM:** Outputs depend on state and inputs. Faster, but can glitch.
- **Moore FSM:** Outputs depend only on state. More stable, but may require extra states.

For serial adders:
- Mealy can assert `done` immediately on the last step.
- Moore uses a separate `DONE` state for clean output signaling.

---

### 5. Implement the serial adder using shift registers and a single full adder module

```verilog
module fa1(input a,b,cin, output sum, cout);
  assign {cout, sum} = a + b + cin;
endmodule

module serial_adder_8 (
  input        clk, rst, start,
  input  [7:0] A_in, B_in,
  output [7:0] S_out,
  output       carry_out, ready, done
);
  reg [7:0] A, B, S;
  reg [2:0] bit_cnt;
  reg carry;
  reg [1:0] state, nstate;

  localparam IDLE=2'd0, STEP=2'd1, DONE=2'd2;

  wire a0 = A[0], b0 = B[0], s0, cnext;
  fa1 u_fa(.a(a0), .b(b0), .cin(carry), .sum(s0), .cout(cnext));

  assign ready = (state==IDLE)||(state==DONE);
  assign done  = (state==DONE);
  assign S_out = S;
  assign carry_out = (state==DONE) ? carry : 1'b0;

  always @* begin
    nstate = state;
    case (state)
      IDLE: if (start) nstate = STEP;
      STEP: nstate = (bit_cnt==3'd7) ? DONE : STEP;
      DONE: nstate = IDLE;
    endcase
  end

  always @(posedge clk) begin
    if (rst) begin
      state <= IDLE; A<=0; B<=0; S<=0; carry<=0; bit_cnt<=0;
    end else begin
      state <= nstate;
      case (state)
        IDLE: if (start) begin
          A <= A_in; B <= B_in; S <= 8'b0; carry <= 0; bit_cnt <= 0;
        end
        STEP: begin
          S <= {s0, S[7:1]};
          A <= {1'b0, A[7:1]};
          B <= {1'b0, B[7:1]};
          carry <= cnext;
          bit_cnt <= bit_cnt + 1;
        end
      endcase
    end
  end
endmodule
```

---

## Summary
- The FSM can be extended to 8 bits with minimal modification.  
- Carry-out, ready signaling, and Moore vs. Mealy implementations improve reliability.  
- Using shift registers and a single full adder provides a resource-efficient implementation suitable for FPGA synthesis.


