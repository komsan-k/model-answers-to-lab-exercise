# Lab-7: Exercises – Model Answers – Barrel Shifter and Rotator

## 1. Modify the barrel shifter to support rotation (ROL, ROR)
A rotate-left/right by variable amount can be expressed with shifts and ORs:

```verilog
function [N-1:0] rol;
  input [N-1:0] a; input [$clog2(N)-1:0] s;
  begin
    rol = (a << s) | (a >> (N - s));
  end
endfunction

function [N-1:0] ror;
  input [N-1:0] a; input [$clog2(N)-1:0] s;
  begin
    ror = (a >> s) | (a << (N - s));
  end
endfunction
```

---

## 2. Extend the barrel shifter to 16 bits with arithmetic shift support
Arithmetic right shift (ASR) preserves the sign bit. Use `>>>` with a signed operand.

```verilog
module barrel16 #(parameter N=16) (
  input      [N-1:0] din,
  input  [$clog2(N)-1:0] shamt,
  input      [1:0]   op,   // 00: LSL, 01: LSR, 10: ASR, 11: NOP
  output reg [N-1:0] dout
);
  wire [N-1:0] lsl = din << shamt;
  wire [N-1:0] lsr = din >> shamt;
  wire [N-1:0] asr = $signed(din) >>> shamt;  // sign-extend

  always @* begin
    case (op)
      2'b00: dout = lsl;
      2'b01: dout = lsr;
      2'b10: dout = asr;
      default: dout = din;
    endcase
  end
endmodule
```

---

## 3. Design a combined shifter/rotator module with a 3-bit control input
Example control encoding:  
`000 LSL`, `001 LSR`, `010 ASR`, `011 ROL`, `100 ROR`

```verilog
module shifter_rotator #(
  parameter N = 16
)(
  input      [N-1:0] din,
  input  [$clog2(N)-1:0] shamt,
  input      [2:0]   ctrl,
  output reg [N-1:0] dout
);
  wire [N-1:0] lsl = din << shamt;
  wire [N-1:0] lsr = din >> shamt;
  wire [N-1:0] asr = $signed(din) >>> shamt;
  wire [N-1:0] rol = (shamt==0) ? din : ((din << shamt) | (din >> (N - shamt)));
  wire [N-1:0] ror = (shamt==0) ? din : ((din >> shamt) | (din << (N - shamt)));

  always @* begin
    case (ctrl)
      3'b000: dout = lsl;   // logical shift left
      3'b001: dout = lsr;   // logical shift right
      3'b010: dout = asr;   // arithmetic shift right
      3'b011: dout = rol;   // rotate left
      3'b100: dout = ror;   // rotate right
      default: dout = din;  // no-op
    endcase
  end
endmodule
```

---

## 4. Synthesize the design on FPGA and verify functionality
**I/O Mapping Example (Nexys A7):**
- `din[N-1:0]` → switches `SW[N-1:0]`
- `shamt[$clog2(N)-1:0]` → lower switches
- `ctrl[2:0]` → pushbuttons
- `dout[N-1:0]` → LEDs

**Procedure:**
1. Create a top module that instantiates `shifter_rotator`.
2. Add XDC constraints for FPGA board pins.
3. Test with sample inputs:
   - `din=16'h8001`, vary `shamt`.
   - Verify LSL/LSR behavior (zero fill).
   - Verify ASR preserves sign.
   - Verify ROL/ROR end-around behavior.
   - Corner cases: `shamt=0`, `shamt=N-1`.
4. Optionally, connect `dout` to 7-segment display for easier observation.


