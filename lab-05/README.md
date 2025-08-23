# Lab-5: Exercises – Model Answers

## Exercises
### 1. Convert the MUX to an 8-to-1 design using nested case statements
An 8:1 multiplexer can be built using nested `case` blocks:

```verilog
module mux8_nested (
  input  [7:0] d,
  input  [2:0] sel,
  output reg   y
);
  always @* begin
    case (sel[2])
      1'b0: begin
        case (sel[1:0])
          2'b00: y = d[0];
          2'b01: y = d[1];
          2'b10: y = d[2];
          2'b11: y = d[3];
        endcase
      end
      1'b1: begin
        case (sel[1:0])
          2'b00: y = d[4];
          2'b01: y = d[5];
          2'b10: y = d[6];
          2'b11: y = d[7];
        endcase
      end
    endcase
  end
endmodule
```

---

### 2. Extend the DEMUX to 1-to-8 with an enable input
The DEMUX only drives outputs when `en=1`.

```verilog
module demux1to8_en (
  input        din,
  input        en,
  input  [2:0] sel,
  output reg [7:0] y
);
  always @* begin
    y = 8'b0;
    if (en) begin
      case (sel)
        3'd0: y[0] = din;
        3'd1: y[1] = din;
        3'd2: y[2] = din;
        3'd3: y[3] = din;
        3'd4: y[4] = din;
        3'd5: y[5] = din;
        3'd6: y[6] = din;
        3'd7: y[7] = din;
        default: y = 8'b0;
      endcase
    end
  end
endmodule
```

---

### 3. Modify the MUX to handle a default input when select is out of range
Use a `default` branch to ensure safe behavior:

```verilog
always @* begin
  case (sel)
    3'd0: y = d[0];
    3'd1: y = d[1];
    3'd2: y = d[2];
    3'd3: y = d[3];
    3'd4: y = d[4];
    3'd5: y = d[5];
    3'd6: y = d[6];
    3'd7: y = d[7];
    default: y = d[0]; // safe fallback
  endcase
end
```

---

### 4. Discuss how `casez` could simplify DEMUX code
`casez` allows don’t-care (`?`) patterns, simplifying enable logic:

```verilog
always @* begin
  y = 8'b0;
  casez ({en, sel})
    4'b1_000: y = 8'b0000_0001;
    4'b1_001: y = 8'b0000_0010;
    4'b1_010: y = 8'b0000_0100;
    4'b1_011: y = 8'b0000_1000;
    4'b1_100: y = 8'b0001_0000;
    4'b1_101: y = 8'b0010_0000;
    4'b1_110: y = 8'b0100_0000;
    4'b1_111: y = 8'b1000_0000;
    4'b0_???: y = 8'b0000_0000; // all disabled cases
    default : y = 8'b0000_0000;
  endcase
end
```

---

### 5. Simulate with different input values and validate all paths
A testbench ensures correctness:

```verilog
module tb;
  reg  [7:0] d;   reg [2:0] sel;  wire y;
  reg        din; reg       en;   wire [7:0] y_demux;

  mux8_nested    u_mux  (.d(d), .sel(sel), .y(y));
  demux1to8_en   u_dem  (.din(din), .en(en), .sel(sel), .y(y_demux));

  initial begin
    d   = 8'b1010_1101;
    en  = 1'b0; din = 1'b0; sel = 3'd0;

    repeat (8) begin #5 sel = sel + 1; end

    en = 1'b1; sel = 3'd0;
    repeat (8) begin
      din = 1'b0; #5;
      din = 1'b1; #5;
      sel = sel + 1;
    end

    $display("Simulation completed"); #10 $finish;
  end
endmodule
```

**Validation Tips:**
- Use randomized input patterns for `d`.  
- Assert one-hot property in DEMUX outputs (`$onehot0`).  
- Include edge cases with `sel='x` to test default behavior.  


