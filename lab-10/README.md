# Lab-10: Exercises – Model Answers – Stopwatch / Counter Design

## 1. Modify the design to count down from 99 to 00
Use two BCD digits (`tens`, `ones`) with borrow on wrap. When the count reaches 00, hold (or wrap to 99 if needed).

```verilog
// 2-digit BCD down-counter: 99 -> 00 (hold at 00)
module bcd2_down (
  input  wire clk, rst_n, en,
  output reg  [3:0] tens, ones,
  output wire       at_zero
);
  assign at_zero = (tens==4'd0) && (ones==4'd0);

  always @(posedge clk or negedge rst_n) begin
    if (!rst_n) begin
      tens <= 4'd9; ones <= 4'd9;
    end else if (en) begin
      if (at_zero) begin
        tens <= tens; ones <= ones;
      end else if (ones==4'd0) begin
        ones <= 4'd9;
        tens <= tens - 4'd1;
      end else begin
        ones <= ones - 4'd1;
      end
    end
  end
endmodule
```

---

## 2. Add a pause/resume button
Gate the counter’s enable with a latched pause state.

```verilog
module pause_ctrl (
  input  wire clk, rst_n,
  input  wire btn,      // debounced, pulse preferred
  output reg  run_en
);
  always @(posedge clk or negedge rst_n) begin
    if (!rst_n) run_en <= 1'b1;
    else if (btn) run_en <= ~run_en;
  end
endmodule
```

---

## 3. Add a clock divider for 1 Hz counting
Divide the FPGA clock (e.g., 100 MHz) to 1 Hz and use the pulse for counting.

```verilog
module clk_div_1hz #(
  parameter integer CLK_HZ = 100_000_000
)(
  input  wire clk, rst_n,
  output reg  tick_1hz
);
  localparam integer TOP = CLK_HZ - 1;
  localparam integer W   = $clog2(CLK_HZ);
  reg [W-1:0] cnt;

  always @(posedge clk or negedge rst_n) begin
    if (!rst_n) begin cnt<=0; tick_1hz<=1'b0; end
    else if (cnt == TOP) begin cnt<=0; tick_1hz<=1'b1; end
    else begin cnt<=cnt+1'b1; tick_1hz<=1'b0; end
  end
endmodule
```

---

## 4. Expand the design to 4 digits (0000–9999)
Cascading BCD counters with 7-segment multiplex display.

```verilog
// 4-digit BCD down-counter
module bcd4_down (
  input  wire clk, rst_n, en,
  output reg  [3:0] d3, d2, d1, d0,
  output wire       at_zero
);
  assign at_zero = (d3|d2|d1|d0)==4'b0000;

  task dec1(input reg [3:0] in, output reg [3:0] out, output reg borrow);
    begin
      if (in==4'd0) begin out=4'd9; borrow=1'b1; end
      else          begin out=in-4'd1; borrow=1'b0; end
    end
  endtask

  reg [3:0] n3,n2,n1,n0; reg b0,b1,b2;
  always @* begin
    n3=d3; n2=d2; n1=d1; n0=d0;
    b0=1'b0; b1=1'b0; b2=1'b0;
    if (en && !at_zero) begin
      dec1(d0, n0, b0);
      if (b0) dec1(d1, n1, b1); else n1=d1;
      if (b0 && b1) dec1(d2, n2, b2); else n2=d2;
      if (b0 && b1 && b2) dec1(d3, n3, b2); else n3=d3;
    end
  end

  always @(posedge clk or negedge rst_n) begin
    if (!rst_n) begin d3<=4'd9; d2<=4'd9; d1<=4'd9; d0<=4'd9; end
    else begin d3<=n3; d2<=n2; d1<=n1; d0<=n0; end
  end
endmodule
```

---

## 5. Display hexadecimal values (0–FF)
Use hex decoder instead of BCD.

```verilog
function [6:0] seg7_hex;
  input [3:0] h;
  case (h)
    4'h0: seg7_hex = 7'b1111110;
    4'h1: seg7_hex = 7'b0110000;
    4'h2: seg7_hex = 7'b1101101;
    4'h3: seg7_hex = 7'b1111001;
    4'h4: seg7_hex = 7'b0110011;
    4'h5: seg7_hex = 7'b1011011;
    4'h6: seg7_hex = 7'b1011111;
    4'h7: seg7_hex = 7'b1110000;
    4'h8: seg7_hex = 7'b1111111;
    4'h9: seg7_hex = 7'b1111011;
    4'hA: seg7_hex = 7'b1110111;
    4'hB: seg7_hex = 7'b0011111;
    4'hC: seg7_hex = 7'b1001110;
    4'hD: seg7_hex = 7'b0111101;
    4'hE: seg7_hex = 7'b1001111;
    4'hF: seg7_hex = 7'b1000111;
  endcase
endfunction
```

---




