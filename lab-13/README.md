# Lab-13: Exercises – Model Answers - Traffic Light FSM 

## Exercises and Solutions

### 1. Modify FSM for two-way intersection (Main and Side road)
Use a Moore FSM with timed stages. Example timing (seconds): Main G=10, Main Y=3, All-Red=1, Side G=7, Side Y=3, All-Red=1. Outputs depend on state; transitions depend on a 1 Hz tick.

```verilog
typedef enum logic [2:0] {
  S_MAIN_G, S_MAIN_Y, S_AR1, S_SIDE_G, S_SIDE_Y, S_AR2
} state_t;

localparam int T_MAIN_G=10, T_MAIN_Y=3, T_SIDE_G=7, T_SIDE_Y=3, T_AR=1;

state_t state, nstate;
logic [5:0] sec_cnt;     
wire tick_1hz;           

always_comb begin
  nstate = state;
  unique case (state)
    S_MAIN_G: nstate = (sec_cnt==T_MAIN_G-1) ? S_MAIN_Y : S_MAIN_G;
    S_MAIN_Y: nstate = (sec_cnt==T_MAIN_Y-1) ? S_AR1    : S_MAIN_Y;
    S_AR1   : nstate = (sec_cnt==T_AR-1)     ? S_SIDE_G : S_AR1;
    S_SIDE_G: nstate = (sec_cnt==T_SIDE_G-1) ? S_SIDE_Y : S_SIDE_G;
    S_SIDE_Y: nstate = (sec_cnt==T_SIDE_Y-1) ? S_AR2    : S_SIDE_Y;
    S_AR2   : nstate = (sec_cnt==T_AR-1)     ? S_MAIN_G : S_AR2;
    default : nstate = S_MAIN_G;
  endcase
end

always_ff @(posedge clk or posedge rst) begin
  if (rst) begin state<=S_MAIN_G; sec_cnt<=0; end
  else begin
    state <= nstate;
    if (state!=nstate) sec_cnt <= 0;
    else if (tick_1hz) sec_cnt <= sec_cnt + 1;
  end
end

assign main_g = (state==S_MAIN_G);
assign main_y = (state==S_MAIN_Y);
assign main_r = ~main_g & ~main_y;
assign side_g = (state==S_SIDE_G);
assign side_y = (state==S_SIDE_Y);
assign side_r = ~side_g & ~side_y;
```

---

### 2. Add pedestrian walk signal with button interrupt
Debounce and edge-detect `ped_btn`. When pressed, set a pending request flag served at the next all-red barrier, entering a dedicated pedestrian window (`S_PED_WALK`).

```verilog
logic ped_req, ped_pend;
always_ff @(posedge clk or posedge rst) begin
  if (rst) ped_pend <= 1'b0;
  else if (ped_req) ped_pend <= 1'b1;
  else if (state==S_PED_WALK && tick_1hz && sec_cnt==T_PED-1) ped_pend <= 1'b0;
end

localparam int T_PED=8;
typedef enum logic [2:0] { S_MAIN_G, S_MAIN_Y, S_AR1, S_PED_WALK,
                           S_SIDE_G, S_SIDE_Y, S_AR2 } state_t;

always_comb begin
  nstate = state;
  case (state)
    S_AR1:    nstate = (sec_cnt==T_AR-1) ? (ped_pend ? S_PED_WALK : S_SIDE_G) : S_AR1;
    S_PED_WALK: nstate = (sec_cnt==T_PED-1) ? S_SIDE_G : S_PED_WALK;
    S_AR2:    nstate = (sec_cnt==T_AR-1) ? S_MAIN_G : S_AR2;
    default:  ; 
  endcase
end

assign ped_walk = (state==S_PED_WALK);
assign ped_stop = ~ped_walk;
```

---

### 3. Use 1 kHz input clock with a counter-based divider for 1 Hz tick
```verilog
module tick_1hz_from_1khz (
  input  logic clk_1k, rst,
  output logic tick_1hz
);
  logic [9:0] div;
  always_ff @(posedge clk_1k or posedge rst) begin
    if (rst) begin div<=10'd0; tick_1hz<=1'b0; end
    else if (div==10'd999) begin div<=10'd0; tick_1hz<=1'b1; end
    else begin div<=div+10'd1; tick_1hz<=1'b0; end
  end
endmodule
```

---

### 4. Add buzzer output for pedestrian crossing alert
Beep during `S_PED_WALK`. Slow beep until last 3 seconds, then fast beep.

```verilog
assign buzzer_slow = (state==S_PED_WALK) && (sec_cnt < T_PED-3) && blink_2hz;
assign buzzer_fast = (state==S_PED_WALK) && (sec_cnt >= T_PED-3) && blink_5hz;
assign buzzer = buzzer_slow | buzzer_fast;
```

---

### 5. Display countdown on 7-segment display for each light
Compute remaining seconds for each active state, convert to BCD, and drive display.

```verilog
function automatic [5:0] t_state_max(input state_t s);
  case (s)
    S_MAIN_G:   t_state_max = T_MAIN_G;
    S_MAIN_Y:   t_state_max = T_MAIN_Y;
    S_AR1:      t_state_max = T_AR;
    S_PED_WALK: t_state_max = T_PED;
    S_SIDE_G:   t_state_max = T_SIDE_G;
    S_SIDE_Y:   t_state_max = T_SIDE_Y;
    S_AR2:      t_state_max = T_AR;
    default:    t_state_max = 0;
  endcase
endfunction

wire [5:0] remain = (t_state_max(state) > sec_cnt+1) ?
                    (t_state_max(state) - 1 - sec_cnt) : 6'd0;
```

Multiplex digits at ~1 kHz to avoid flicker.

---

## Verification Tips
- Test the 1 Hz divider.
- Ensure transitions only occur on tick.
- Assert vehicle greens are exclusive.
- Simulate pedestrian button presses at random times.


