# Lab-9: Exercises – Model Answers - Counters and Clock Divider

## Exercises
1. Modify the counter to create a down counter and an up-down counter.
2. Implement a counter with a terminal count that resets automatically.
3. Use the clock divider to control the blinking rate of an LED on an FPGA board.
4. Design a clock divider with selectable division factor using inputs.
5. Extend the counter to 8 bits and verify wrap-around behavior.

---

## Model Answers

### 1. Down Counter and Up-Down Counter
**Down Counter (synchronous reset, enable):**
```verilog
module counter_down #(parameter WIDTH=4)(
  input               clk, rst_n, en,
  output reg [WIDTH-1:0] q
);
  always @(posedge clk) begin
    if (!rst_n) q <= {WIDTH{1'b0}};
    else if (en) q <= q - 1'b1;
  end
endmodule
```

**Up-Down Counter (dir=1: up, dir=0: down):**
```verilog
module counter_updown #(parameter WIDTH=4)(
  input               clk, rst_n, en, dir,
  output reg [WIDTH-1:0] q
);
  always @(posedge clk) begin
    if (!rst_n) q <= {WIDTH{1'b0}};
    else if (en) q <= dir ? (q + 1'b1) : (q - 1'b1);
  end
endmodule
```

---

### 2. Counter with Terminal Count Auto-Reset
A modulus-`M` counter that wraps to 0 when it reaches `M-1`:
```verilog
module counter_mod #(parameter M=10, WIDTH=$clog2(M))(
  input               clk, rst_n, en,
  output reg [WIDTH-1:0] q,
  output                   tc  // terminal count pulse when q==M-1
);
  assign tc = en && (q == M-1);
  always @(posedge clk) begin
    if (!rst_n) q <= {WIDTH{1'b0}};
    else if (en)  q <= (q == M-1) ? {WIDTH{1'b0}} : (q + 1'b1);
  end
endmodule
```

---

### 3. Clock Divider to Blink LED
Divide the system clock and toggle an LED at the terminal count:
```verilog
module led_blink #(
  parameter CLK_HZ = 100_000_000,     // e.g., Nexys A7: 100 MHz
  parameter BLINK_HZ = 2              // 2 toggles per second -> 1 Hz blink
)(
  input  clk, rst_n,
  output reg led
);
  localparam integer HALF_PERIOD = CLK_HZ/(2*BLINK_HZ);
  localparam integer W = $clog2(HALF_PERIOD);
  reg [W-1:0] cnt;

  always @(posedge clk) begin
    if (!rst_n) begin
      cnt <= 0; led <= 1'b0;
    end else if (cnt == HALF_PERIOD-1) begin
      cnt <= 0; led <= ~led;          // toggle LED
    end else begin
      cnt <= cnt + 1'b1;
    end
  end
endmodule
```

---

### 4. Clock Divider with Selectable Factor
Program the terminal value via input:
```verilog
module clk_div_sel #(
  parameter W=24
)(
  input               clk, rst_n,
  input       [W-1:0] div_val,   // runtime-programmable terminal
  output reg          tick       // 1-cycle pulse at division boundary
);
  reg [W-1:0] cnt;
  always @(posedge clk) begin
    if (!rst_n) begin
      cnt <= 0; tick <= 1'b0;
    end else if (cnt == (div_val-1)) begin
      cnt  <= 0;
      tick <= 1'b1;
    end else begin
      cnt  <= cnt + 1'b1;
      tick <= 1'b0;
    end
  end
endmodule
```

---

### 5. 8-bit Counter with Wrap-around
Parameterize width and observe rollover from `8'hFF -> 8'h00`:
```verilog
module counter_n #(parameter WIDTH=8)(
  input               clk, rst_n, en,
  output reg [WIDTH-1:0] q
);
  always @(posedge clk) begin
    if (!rst_n) q <= {WIDTH{1'b0}};
    else if (en) q <= q + 1'b1;   // natural wrap-around on overflow
  end
endmodule
```

**Testbench Snippet:**
```verilog
initial begin
  rst_n=0; en=0; #20; rst_n=1; en=1;
  q_prev = 0;
  repeat (300) begin
    @(posedge clk);
    if (q == 8'h00 && q_prev == 8'hFF) $display("Wrap OK @%0t", $time);
    q_prev = q;
  end
  $finish;
end
```



