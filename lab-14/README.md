# Lab-14: Exercises – Model Answers - UART Communication

1.  **Add parity bit generation and checking.**\
    Parity ensures simple error detection. For data bus D\[DBITS-1:0\]:\

```{=html}
<!-- -->
```

    even parity bit p_EVEN = XOR(D)
    odd parity bit  p_ODD  = NOT(XOR(D))

**Verilog Example:**

``` verilog
module uart_parity #(
  parameter integer DBITS = 8,
  parameter ODD = 0
)(
  input  wire [DBITS-1:0] data,
  input  wire             p_rx,
  output wire             p_tx,
  output wire             ok
);
  wire x = ^data;
  assign p_tx = ODD ? ~x : x;
  assign ok = (ODD) ? (^{data, p_rx}) : ~ (^{data, p_rx});
endmodule
```

2.  **Support 7-bit or 9-bit data.**\
    Parameterize data width:

``` verilog
module uart_core #(
  parameter integer DATA_BITS = 8,
  parameter integer STOP_BITS = 1
)(
  input  wire clk, rst, baud_tick,
  input  wire [DATA_BITS-1:0] tx_data,
  input  wire tx_start,
  output reg  tx_busy,
  output wire tx_o,
  input  wire rx_i,
  output reg  [DATA_BITS-1:0] rx_data,
  output reg  rx_valid
);
  // Implementation
endmodule
```

3.  **Full-duplex UART loopback.**

``` verilog
module uart_loopback #(
  parameter integer DATA_BITS = 8
)(
  input  wire clk, rst, baud_tick,
  input  wire [DATA_BITS-1:0] tx_data,
  input  wire tx_start,
  output wire tx_busy,
  output wire rx_valid,
  output wire [DATA_BITS-1:0] rx_data
);
  wire tx_line;
  uart_core #(.DATA_BITS(DATA_BITS)) U (
    .clk(clk), .rst(rst), .baud_tick(baud_tick),
    .tx_data(tx_data), .tx_start(tx_start), .tx_busy(tx_busy), .tx_o(tx_line),
    .rx_i(tx_line), .rx_data(rx_data), .rx_valid(rx_valid)
  );
endmodule
```

4.  **FSM sends "HELLO".**

``` verilog
module uart_hello (
  input  wire clk, rst, baud_tick,
  output wire tx_o
);
  localparam string S = "HELLO\r\n";
  localparam N = 7;
  reg [7:0] rom [0:N-1];
  initial begin
    rom[0]="H"; rom[1]="E"; rom[2]="L"; rom[3]="L";
    rom[4]="O"; rom[5]=8'h0D; rom[6]=8'h0A;
  end
  // FSM logic
endmodule
```

5.  **UART to 7-Segment Display.**

``` verilog
module uart_to_7seg (
  input  wire clk, rst, baud_tick, rx_i,
  output wire seg_a,seg_b,seg_c,seg_d,seg_e,seg_f,seg_g,
  output wire dig0_en, dig1_en
);
  // Latch received byte and display hex
endmodule
```

------------------------------------------------------------------------

**Baud generator reference:**\
`DIV = floor(Fclk / BAUD)`


