# Lab-6: Exercises – Model Answers – Encoder and Decoder

## Exercises and Model Answers

### 1. Add an enable input to both encoder and decoder modules

**2-to-4 decoder with enable (active-high):**
```verilog
module dec2to4_en (
  input        en,
  input  [1:0] a,
  output reg [3:0] y
);
  always @* begin
    if (!en) y = 4'b0000;
    else case (a)
      2'b00: y = 4'b0001;
      2'b01: y = 4'b0010;
      2'b10: y = 4'b0100;
      2'b11: y = 4'b1000;
    endcase
  end
endmodule
```

**4-to-2 encoder with enable:**
```verilog
module enc4to2_en (
  input        en,
  input  [3:0] d,
  output reg [1:0] y
);
  always @* begin
    if (!en) y = 2'b00;
    else case (1'b1)
      d[3]: y = 2'b11;
      d[2]: y = 2'b10;
      d[1]: y = 2'b01;
      d[0]: y = 2'b00;
      default: y = 2'b00;
    endcase
  end
endmodule
```

---

### 2. Modify the encoder to output a valid flag
```verilog
module enc4to2_valid (
  input  [3:0] d,
  output reg [1:0] y,
  output       valid
);
  assign valid = |d;
  always @* begin
    case (1'b1)
      d[3]: y = 2'b11;
      d[2]: y = 2'b10;
      d[1]: y = 2'b01;
      d[0]: y = 2'b00;
      default: y = 2'b00;
    endcase
  end
endmodule
```

---

### 3. Create a hierarchical 3-to-8 decoder using 2-to-4 decoders
```verilog
module dec3to8_hier (
  input        en,
  input  [2:0] a,
  output [7:0] y
);
  wire [3:0] g;

  dec2to4_en u_msb (.en(en), .a({1'b0, a[2]}), .y(g));

  wire [3:0] y_lo, y_hi;
  dec2to4_en u_lo (.en(g[0]), .a(a[1:0]), .y(y_lo));
  dec2to4_en u_hi (.en(g[1]), .a(a[1:0]), .y(y_hi));

  assign y = { y_hi, y_lo };
endmodule
```

---

### 4. Compare structural, behavioral, and hierarchical models

- **Structural**: Gate-level or module interconnections. Precise, less readable for large designs.  
- **Behavioral**: Describes function using `case` or `if`. Concise and readable, less gate-level control.  
- **Hierarchical**: Composes designs from submodules. Improves reusability, readability, and verification.  

**Summary**: Use behavioral for clarity, structural for control, hierarchical for scalability.

---

### 5. Use conditional operators (?:) to implement a 4-to-2 encoder
```verilog
module enc4to2_ternary (
  input  [3:0] d,
  output [1:0] y,
  output       valid
);
  assign valid = |d;
  assign y = d[3] ? 2'b11 :
             d[2] ? 2'b10 :
             d[1] ? 2'b01 :
             d[0] ? 2'b00 : 2'b00;
endmodule
```


