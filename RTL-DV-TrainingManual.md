# RTL/DV Training Manual for Graduate Engineers

Nov 12, 2026 · @Prof S

## About this manual

This is a 3-week hands-on course that takes an EE/ECE graduate from basic Verilog to a first protocol design (APB), a FIFO project and clock-domain-crossing circuits. Every module has the same layout: **Why it matters**, **Concepts**, **Worked example**, **Lab assignments** (taken from the course sheet), **Checkpoint**, and **Common mistakes**.

**Outcomes.** After the course a trainee can:

- write synthesizable RTL for combinational, sequential, FSM and datapath blocks,
- build a self-checking testbench and read a waveform to debug,
- simulate at RTL, zero-delay gate level and unit-delay gate level,
- explain and implement the APB protocol and a metastability-safe pulse synchronizer.

**Prerequisites.** Digital logic (gates, latches, flip-flops, K-maps), basic C or Python, and a Linux command line.

### Schedule

| Module | Topic | Start | End | Course-sheet note |
| --- | --- | --- | --- | --- |
| 1 | Verilog basics and Verilator | 2026-07-31 | 2026-08-01 | 1.5 days |
| 2 | UDPs, shift register, counters | 2026-08-03 | 2026-08-05 | 3 days |
| 3 | Counters, level-to-pulse, gate-level netlists, divisibility | 2026-08-06 | 2026-08-10 | Assignments 1 to 5 |
| 4 | Mealy/Moore and FSMs | 2026-08-06 | 2026-08-10 | Assignments 6 to 7 |
| 5 | Rate/width conversion and multiplier | 2026-08-06 | 2026-08-10 | Assignments 8 to 10 |
| 6 | Special Verilog topics | 2026-08-11 | 2026-08-13 | Read and understand |
| 7 | APB protocol and slave testbench | 2026-08-11 | 2026-08-13 | Read, then implement |
| 8 | Design project: FIFO flush | 2026-08-14 | 2026-08-18 | From FIFO Flush Specification.pdf |
| 9 | Pulse synchronizer and arbiter | 2026-08-19 | 2026-08-22 | Assignment 6 reference |

Modules 3 to 5 share one window because the course sheet lists ten assignments for 2026-08-06 to 2026-08-10, and Modules 6 and 7 share 2026-08-11 to 2026-08-13. The sheet lists the netlist assignment (item 3) twice with identical text; this manual treats it as one assignment and uses the second slot for the gate-level delay study in Module 3.

### Tool setup

All labs use open-source tools, so trainees can work from any Linux machine.

| Tool | Used for | Install (Debian/Ubuntu) |
| --- | --- | --- |
| Verilator | Fast RTL simulation and lint | `sudo apt install verilator` |
| Icarus Verilog (`iverilog`, `vvp`) | Event-driven simulation, UDPs, gate-level delays | `sudo apt install iverilog` |
| GTKWave | Waveform viewing | `sudo apt install gtkwave` |
| Yosys | Synthesis to a gate netlist and schematics | `sudo apt install yosys` |

Use Icarus for anything with `#` delays, UDPs or tri-state strength, because Verilator is a cycle-based simulator and ignores or restricts them. Use Verilator for RTL-level checks and lint.

### Lab conventions

- One folder per assignment: `m3_a2_counter/` holding `rtl/`, `tb/`, `sim/` and a `Makefile`.
- Every submission contains RTL, testbench, a waveform screenshot, and a five-line note: what it does, the key signals, what you checked, what surprised you.
- File and module names match; one module per file; lowercase with underscores.
- Clock is `clk`, active-low async reset is `rst_n`, and registers use non-blocking assignments (see Appendix).

## Module 1: Verilog basics and Verilator

*2026-07-31 to 2026-08-01 (1.5 days)*

The same circuit can be described as gates, as a dataflow equation, or as behavior, and a simulator treats each differently. This module teaches the three styles on six small circuits and shows how to run and view them.

### Concepts

- **Module, ports, nets and variables.** `wire` carries a value driven by `assign` or a module output; `reg` holds a value assigned inside `always` or `initial`. `reg` does not mean a flip-flop.
- **Four-state logic.** Every bit is `0`, `1`, `x` (unknown) or `z` (undriven). Tri-state buses and pull-ups need `z`.
- **Three abstraction levels.** Gate primitives (`nand g1(y,a,b)`), dataflow (`assign y = ~(a&b)`), and behavioral (`always @*`).
- **Strength.** When two drivers fight, the stronger one wins. `pullup (weak1)` gives a net a default 1 that any real driver overrides.
- **Latch vs flip-flop.** A latch is level-sensitive (`always @(en or d)`); a flip-flop is edge-sensitive (`always @(posedge clk)`).

### Worked example: one NAND, three styles

```verilog
module nand2_gate (input a, b, output y);
  nand g1 (y, a, b);
endmodule

module nand2_df (input a, b, output y);
  assign y = ~(a & b);
endmodule

module nand2_beh (input a, b, output reg y);
  always @* y = ~(a & b);
endmodule
```

Write one testbench that instantiates all three and checks they agree on all four input combinations. This is the first habit to build: compare implementations against each other.

### Worked example: tri-state inverter with weak pull-up

```verilog
module tri_inv_pu (input a, en, output y);
  assign y = en ? ~a : 1'bz;
  pullup (weak1) pu (y);
endmodule
```

| en | a | tri\_inv y | tri\_inv\_pu y |
| --- | --- | --- | --- |
| 1 | 0 | 1 | 1 |
| 1 | 1 | 0 | 0 |
| 0 | any | z | 1 (weak) |

### Running a simulation

```bash
# Verilator 5: compile, build and run a self-contained testbench with waveforms
verilator --binary --trace --timing -Wall tb.v tri_inv_pu.v --top-module tb
./obj_dir/Vtb
gtkwave tb.vcd

# Icarus Verilog: use this when z, x, strengths or delays matter
iverilog -o sim tb.v tri_inv_pu.v && vvp sim
```

Verilator is a two-state, cycle-oriented simulator, so it cannot show `x` or `z` on a net the way Icarus can, and its strength handling is limited. Run the tri-state labs in both tools and note any difference; that difference is part of the lesson.

### Lab assignments

Purpose from the course sheet: understand and execute Verilog module simulations at different abstraction levels using Verilator. For each of the six circuits write the RTL, a testbench that prints a truth table, and a waveform.

1. Tri-state inverter
2. 2-input NAND
3. 2-input mux
4. D-latch
5. D flip-flop
6. Tri-state inverter with weak pull-up

For the latch and flip-flop, drive `d` so it changes while the enable is high and while the clock edge occurs, and explain what each does.

### Checkpoint

- What does the tri-state inverter output when `en = 0`, with and without the pull-up?
- Why can a latch pass a glitch on `d` but a flip-flop cannot?
- Why does the same `always @*` block describe a mux in one case and a latch in another (hint: incomplete `if`)?

### Common mistakes

- Calling a `reg` a register: it only describes how the signal is assigned.
- Forgetting that `a == b` gives `x` when either side has `x`; use `===` in testbenches.
- Missing `else` or `default` in a combinational block, which infers a latch.

## Module 2: UDPs, shift registers and counters

*2026-08-03 to 2026-08-05 (3 days)*

This module moves from single gates to clocked structures. Trainees write primitives from truth tables, then build the shift register and counters that every later module reuses.

### Concepts

- **User-defined primitives (UDPs).** A UDP is a truth table. Combinational UDPs list input combinations and an output; sequential UDPs add a current-state column and use edge notation such as `(01)` for a rising edge. `?` matches any value, `-` means no change.
- **Synchronous vs asynchronous reset.** Async reset sits in the sensitivity list (`or negedge rst_n`) and acts immediately. Sync reset is tested only at the clock edge.
- **Ripple vs synchronous counters.** A ripple counter clocks each stage from the previous stage's output, so bits change one after another and the output passes through wrong values. A synchronous counter clocks all bits together.
- **Modulo-N counting.** Count from `0` to `N-1` and wrap. The course sheet says "count up to N(6)"; this manual reads that as the values 0 to 5 and asks trainees to confirm that reading with the trainer.

### Worked example: UDPs

```verilog
primitive mux2_udp (y, a, b, s);
  output y; input a, b, s;
  table
  //  a b s : y
      0 ? 0 : 0;
      1 ? 0 : 1;
      ? 0 1 : 0;
      ? 1 1 : 1;
      0 0 x : 0;   // both data inputs agree, so s does not matter
      1 1 x : 1;
  endtable
endprimitive
```

The last two rows matter: without them, an `x` on the select makes the output `x` even when both data inputs are equal. Ask trainees to remove them and watch the waveform.

```verilog
primitive dff_udp (q, d, clk, rst_n);
  output q; reg q;
  input d, clk, rst_n;
  table
  //  d  clk   rst_n : q : q+
      ?  ?     0     : ? : 0;   // async reset
      0  (01)  1     : ? : 0;
      1  (01)  1     : ? : 1;
      ?  (1?)  1     : ? : -;   // falling clock: hold
      *  ?     1     : ? : -;   // data change: hold
      ?  ?     (?1)  : ? : -;   // reset release: hold
  endtable
endprimitive
```

### Worked example: shift register and ripple counter

```verilog
module shift8 (input clk, rst_n, sin, output reg [7:0] q);
  always @(posedge clk or negedge rst_n)
    if (!rst_n) q <= 8'b0;
    else        q <= {q[6:0], sin};
endmodule

module tff (input clk, rst_n, output reg q);
  always @(negedge clk or negedge rst_n)
    if (!rst_n) q <= 1'b0;
    else        q <= ~q;
endmodule

module ripple4_async (input clk, rst_n, output [3:0] q);
  tff t0 (.clk(clk),  .rst_n(rst_n), .q(q[0]));
  tff t1 (.clk(q[0]), .rst_n(rst_n), .q(q[1]));
  tff t2 (.clk(q[1]), .rst_n(rst_n), .q(q[2]));
  tff t3 (.clk(q[2]), .rst_n(rst_n), .q(q[3]));
endmodule
```

For the sync-reset version, change each `tff` to test `rst_n` only at its own clock edge. Trainees will see that stages 1 to 3 do not clear until their own clock edges arrive. This is the main reason ripple counters rarely use sync reset.

### Worked example: modulo-N up/down counter and pulse counter

```verilog
module mod_n_updown #(parameter N = 6, parameter W = $clog2(N))
  (input clk, rst_n, up, output reg [W-1:0] cnt);
  always @(posedge clk or negedge rst_n)
    if (!rst_n)  cnt <= 0;
    else if (up) cnt <= (cnt == N-1) ? 0 : cnt + 1;
    else         cnt <= (cnt == 0)   ? N-1 : cnt - 1;
endmodule

module mod_n_pulse #(parameter N = 5, parameter W = $clog2(N))
  (input clk, rst_n, output reg pulse);
  reg [W-1:0] cnt;
  always @(posedge clk or negedge rst_n)
    if (!rst_n) begin cnt <= 0; pulse <= 1'b0; end
    else begin
      cnt   <= (cnt == N-1) ? 0 : cnt + 1;
      pulse <= (cnt == N-2);      // high for one cycle while cnt == N-1
    end
endmodule
```

The pulse is registered so it is glitch-free. It requires `N >= 2`.

### Lab assignments

1. Study Verilog UDPs. Write UDPs for (a) a 2:1 mux and (b) a D flip-flop.
2. Write an 8-bit shift register with asynchronous reset.
3. Write a 4-bit ripple counter with async reset and, separately, with sync reset. Draw an equivalent schematic for each.
4. Write a modulo-N counter with N = 6 that counts both up and down based on an input signal.
5. Write a modulo-5 counter that outputs a single-cycle pulse every 5 clock cycles.

### Checkpoint

- Draw the waveform of `q[3:0]` in a ripple counter and mark the glitch when the count goes from 0111 to 1000.
- What is the maximum clock frequency of an N-bit ripple counter, and why is it lower than a synchronous one?
- In `mod_n_pulse`, how many flip-flops does the design need for N = 5?

### Common mistakes

- Mixing edge and level conditions in one UDP row; a row may contain only one edge.
- Using blocking `=` in a clocked block, which makes results depend on statement order.
- Writing `cnt - 1` on an unsigned counter at 0 and forgetting the wrap case.

## Module 3: Counters, level-to-pulse and gate-level simulation

*2026-08-06 to 2026-08-10, assignments 1 to 5*

This module turns a counter into a useful block (a mid-point marker), builds three ways to make a one-cycle pulse, and then re-simulates the design as a gate netlist to see what RTL hides.

### Concepts

- **Level-to-pulse (L2P).** Converts a signal that stays high into a one-cycle pulse on its rising edge. The usual form is `pulse = a & ~a_delayed`, where `a_delayed` is `a` one clock later.
- **Registered vs combinational outputs.** A decoded output such as `cnt == K` can glitch while several count bits change at once. A registered output is clean but one cycle later.
- **Zero-delay vs unit-delay gate simulation.** With zero delay all gates settle in the same time step, so only logic function is checked. With a unit delay (`#1` per gate) hazards and ordering show up.
- **X propagation.** A flip-flop with no reset starts at `x`. Any gate fed by `x` usually returns `x`, so `x` can persist forever.

### Worked example: mid\_pt in three styles

For an N-bit free-running counter the maximum value is 2^N - 1 and the middle is 2^(N-1). Using the counter's MSB as the level (high for the whole upper half) makes the L2P exercise meaningful. Decoding `cnt == mid` already gives a one-cycle pulse and teaches nothing about L2P, so trainees should see that contrast.

```verilog
module cnt_mid #(parameter N = 4)
  (input clk, rst_n, output reg [N-1:0] cnt, output mid_pt_a, mid_pt_b);
  reg msb_d;
  always @(posedge clk or negedge rst_n)
    if (!rst_n) begin cnt <= 0; msb_d <= 1'b0; end
    else begin cnt <= cnt + 1'b1; msb_d <= cnt[N-1]; end

  // 2a: L2P with a flop (edge detector)
  assign mid_pt_a = cnt[N-1] & ~msb_d;
  // 2b: combinational decode of the middle value
  assign mid_pt_b = (cnt == {1'b1, {(N-1){1'b0}}});
endmodule
```

Both outputs are high for one cycle when the count reaches the middle value. The difference is cost and timing: 2a needs one extra flop and a 2-input gate; 2b needs an N-input comparator.

### Worked example: L2P without flops

```verilog
module l2p_noflop #(parameter D = 2) (input a, output p);
  wire a_n;
  assign #D a_n = ~a;      // modeled cell delay
  assign p = a & a_n;      // high for D time units after a rises
endmodule
```

The pulse width equals the delay `D`, not a clock period. This is an asynchronous pulse generator: it works in simulation but in silicon its width depends on process and temperature, so it must be used with care. Run it in Icarus Verilog, since Verilator does not model `#` delays in this way.

### Worked example: gate-level 3-bit counter

```verilog
`ifdef UNIT
  `define GD #1
`else
  `define GD
`endif

module dff_nr (input clk, d, output reg q);          // no reset
  always @(posedge clk) q <= d;
endmodule

module dff_r (input clk, rst_n, d, output reg q);    // async reset
  always @(posedge clk or negedge rst_n)
    if (!rst_n) q <= 1'b0; else q <= d;
endmodule

module cnt3_gate (input clk, output q0, q1, q2);
  wire d0, d1, d2, c1;
  not `GD g0 (d0, q0);
  xor `GD g1 (d1, q1, q0);
  and `GD g2 (c1, q1, q0);
  xor `GD g3 (d2, q2, c1);
  dff_nr f0 (clk, d0, q0);
  dff_nr f1 (clk, d1, q1);
  dff_nr f2 (clk, d2, q2);
endmodule
```

Run it as `iverilog -DUNIT ...` for unit delay and without `-DUNIT` for zero delay. With `dff_nr` the outputs start at `x` and stay there, because `~x`, `x ^ x` and so on are all `x`. Trainees fix the testbench by writing initial values into the flops (`initial {f0.q, f1.q, f2.q} = 0;` through the hierarchy), then compare with the `dff_r` version.

### Worked example: divisibility detector

A binary number is divisible by 2, 4 or 8 exactly when its lowest 1, 2 or 3 bits are zero.

```verilog
module div_detect #(parameter W = 8)
  (input [W-1:0] x, input [1:0] sel, output reg ok);
  always @* begin
    case (sel)
      2'd1:    ok = ~x[0];          // divisible by 2
      2'd2:    ok = ~|x[1:0];       // divisible by 4
      2'd3:    ok = ~|x[2:0];       // divisible by 8
      default: ok = 1'b0;           // sel = 0 is unused
    endcase
  end
endmodule
```

The schematic is two NOR gates feeding a 4:1 mux. Zero counts as divisible; trainees should decide and document that.

### Lab assignments

1. Write a modulo-N counter that gives a single-cycle pulse every N clock cycles (reuse `mod_n_pulse` from Module 2 and add a parameter test for N = 2, 3, 7, 16).
2. Write a behavioral parameterized N-bit counter with async reset and a `mid_pt` output asserted for one clock when the count reaches the middle of the maximum value.
   - 2a. Implement `mid_pt` with an edge detector built from flops (level to pulse).
   - 2b. Implement it with combinational logic.
   - 2c. In general, implement an L2P that works without flops, using modeled cell delays.
3. Create a netlist of the above behavior for 3 bits. Simulate at gate level with zero delay and with unit delay. Replace the resettable flops with non-resettable flops. Describe the behavior.
4. The course sheet repeats item 3 with the same text. Use this slot to compare the zero-delay and unit-delay waveforms side by side and mark every glitch on `mid_pt`.
5. Write a module that detects whether its input is divisible by 2, 4 or 8, chosen by another input signal. Draw a schematic.

### Checkpoint

- Why is `cnt == mid` a poor test case for an L2P circuit?
- In the unit-delay simulation, where does a glitch appear on `mid_pt_b`, and why?
- Why does a counter built from non-resettable flops never leave `x`, while a real chip would settle to some value?

### Common mistakes

- Writing the `mid_pt` comparison against `N/2` instead of `2^(N-1)`.
- Using `#` delays inside synthesizable RTL; keep them in gate models and testbenches only.
- Declaring victory on a zero-delay run; always repeat with delays.

## Module 4: Mealy and Moore state machines

*2026-08-06 to 2026-08-10, assignments 6 and 7*

A Moore machine's output depends only on the current state; a Mealy machine's output depends on the state and the current input. That one difference changes timing, state count and glitch behavior, and it is the first design decision in any FSM.

### Concepts

|  | Moore | Mealy |
| --- | --- | --- |
| Output depends on | State only | State and input |
| Output timing | One cycle after the input that caused it | Same cycle as the input |
| Number of states | Usually more | Usually fewer |
| Glitches | None if the output is decoded from a register | Input glitches reach the output |
| Typical use | Control outputs that must be clean | Fast reactions, protocol responses |

- **Two-block coding style.** One clocked block holds `state <= next`. One combinational block computes `next` and, for Mealy, the output. Give `next` a default value at the top so no latch is inferred.
- **State encoding.** Use `localparam` names, never raw numbers in the case items. Always include a `default` branch that returns to the idle state.
- **Overlapping detection.** In `1011011` the final `11` of the first match starts the next one. Decide up front whether overlap is allowed; this manual allows it.

### Worked example: 1011 detector, Moore

States record the longest suffix seen so far: none, `1`, `10`, `101`, `1011`.

```verilog
module seq1011_moore (input clk, rst_n, din, output detected);
  localparam S0 = 3'd0, S1 = 3'd1, S10 = 3'd2, S101 = 3'd3, S1011 = 3'd4;
  reg [2:0] state, next;

  always @(posedge clk or negedge rst_n)
    if (!rst_n) state <= S0;
    else        state <= next;

  always @* begin
    next = state;
    case (state)
      S0:      next = din ? S1    : S0;
      S1:      next = din ? S1    : S10;
      S10:     next = din ? S101  : S0;
      S101:    next = din ? S1011 : S10;
      S1011:   next = din ? S1    : S10;   // overlap: trailing 1 starts a new match
      default: next = S0;
    endcase
  end

  assign detected = (state == S1011);
endmodule
```

### Worked example: 1011 detector, Mealy

The Mealy version drops the `S1011` state and raises the output on the arrival of the last bit.

```verilog
module seq1011_mealy (input clk, rst_n, din, output detected);
  localparam S0 = 2'd0, S1 = 2'd1, S10 = 2'd2, S101 = 2'd3;
  reg [1:0] state, next;

  always @(posedge clk or negedge rst_n)
    if (!rst_n) state <= S0;
    else        state <= next;

  always @* begin
    next = state;
    case (state)
      S0:      next = din ? S1   : S0;
      S1:      next = din ? S1   : S10;
      S10:     next = din ? S101 : S0;
      S101:    next = din ? S1   : S10;
      default: next = S0;
    endcase
  end

  assign detected = (state == S101) & din;
endmodule
```

For the stream `1 0 1 1 0 1 1` both machines must report two matches, ending at bits 4 and 7. The Mealy output goes high in the cycle where bit 4 or 7 is on `din`; the Moore output goes high one cycle later.

### Worked example: a checker that does not share the design's logic

```verilog
reg [3:0] hist;
always @(posedge clk) hist <= {hist[2:0], din};
wire expect_mealy = ({hist[2:0], din} == 4'b1011);
```

Compare `detected` from the Mealy design against `expect_mealy` every cycle, and the Moore design against `expect_mealy` delayed by one clock. This checker is a shift register and a compare, so a bug in the FSM cannot also hide in the checker. Feed it both the directed stream above and a few thousand random bits.

### Lab assignments

6. Study and understand the difference between Mealy and Moore state machines and their practical implications. Write one page: draw the state diagrams for the 1011 detector in both styles and state which you would pick for (a) a status LED and (b) a bus-grant response.
7. Write a Verilog state machine that detects the sequence 1011 in an incoming serial bit stream. The output must be high when 1011 is detected.

### Checkpoint

- How many flip-flops does each version need for its state register, and why is the Mealy output risky when `din` comes from another clock domain?
- What happens in each design if overlapping detection is turned off? Change the transitions out of the last state to prove it.
- Which design would you choose if the output feeds a flip-flop in the next block, and why does that choice remove most of the Mealy timing advantage?

### Common mistakes

- Forgetting the default assignment `next = state`, which creates a latch.
- Sending a Mealy output directly off-chip or across a clock boundary without a register.
- Returning to `S0` after a match instead of to the correct suffix state, which loses overlapping matches.

## Module 5: Width and rate conversion, and a sequential multiplier

*2026-08-06 to 2026-08-10, assignments 8 to 10*

These three assignments move data between widths and speeds without losing a bit, and trade area for time in a multiplier. The bandwidth check comes first: 32 bits at 100 MHz and 16 bits at 200 MHz are both 3.2 Gbit/s, so neither direction needs buffering beyond one word.

### Concepts

- **Related clocks.** Here `clk100` and `clk200` come from the same source and their rising edges line up, so no synchronizer is needed. If the clocks were unrelated, this would be a clock-domain crossing and need a FIFO or handshake (see Module 9).
- **Framing.** Splitting or joining words needs a rule for which half comes first. With no `valid` or `first` signal, the rule has to come from reset, which is fragile. Real designs carry a framing bit.
- **Race at a shared edge.** When both clocks rise together, a flop in one domain may sample the old or the new value of a flop in the other. In simulation, create the slow clock from the fast one (`always @(posedge clk200) clk100 <= ~clk100;`) so the order is the same every run.
- **Time for area.** A combinational multiplier is large and fast. A shift-and-add multiplier uses one adder and takes N cycles.

### Worked example: 32 bits at 100 MHz to 16 bits at 200 MHz

The slow domain captures each word. The fast domain watches a toggle bit from the slow domain to learn which half to send, so no clock is used as data.

```verilog
module w32_to_w16 (input clk100, clk200, rst_n, input [31:0] din,
                   output reg [15:0] dout);
  reg [31:0] hold;
  reg        tog100, tog_q;

  always @(posedge clk100 or negedge rst_n)
    if (!rst_n) begin hold <= 0; tog100 <= 0; end
    else        begin hold <= din; tog100 <= ~tog100; end

  always @(posedge clk200 or negedge rst_n)
    if (!rst_n) begin tog_q <= 0; dout <= 0; end
    else begin
      tog_q <= tog100;
      dout  <= (tog100 ^ tog_q) ? hold[31:16] : hold[15:0];
    end
endmodule
```

In the cycle after a new word is captured, `tog100` and `tog_q` differ and the upper half goes out. In the next cycle they match and the lower half follows. Output order is therefore `[31:16]`, then `[15:0]`, for every input word.

### Worked example: 16 bits at 200 MHz to 32 bits at 100 MHz

```verilog
module w16_to_w32 (input clk200, clk100, rst_n, input [15:0] din,
                   output reg [31:0] dout);
  reg        phase;
  reg [15:0] hi;
  reg [31:0] word;

  always @(posedge clk200 or negedge rst_n)
    if (!rst_n) begin phase <= 0; hi <= 0; word <= 0; end
    else begin
      phase <= ~phase;
      if (!phase) hi   <= din;            // first word is the upper half
      else        word <= {hi, din};      // second word completes the pair
    end

  always @(posedge clk100 or negedge rst_n)
    if (!rst_n) dout <= 0;
    else        dout <= word;
endmodule
```

`word` is stable for two fast cycles (10 ns), so the slow domain samples a whole word whatever the alignment. Pairing relies on the first word after reset being an upper half; trainees should write that assumption in the file header.

### Worked example: shift-and-add multiplier

```verilog
module mult_seq #(parameter N = 8)
  (input clk, rst_n, start, input [N-1:0] a, b,
   output reg [2*N-1:0] p, output reg done);
  reg [2*N-1:0] acc, mcand;
  reg [N-1:0]   mplier;
  reg [$clog2(N+1)-1:0] cnt;
  reg busy;

  always @(posedge clk or negedge rst_n)
    if (!rst_n) begin
      busy <= 0; done <= 0; p <= 0; acc <= 0; mcand <= 0; mplier <= 0; cnt <= 0;
    end else begin
      done <= 1'b0;
      if (start && !busy) begin
        busy <= 1; acc <= 0; mcand <= {{N{1'b0}}, a}; mplier <= b; cnt <= 0;
      end else if (busy) begin
        if (mplier[0]) acc <= acc + mcand;
        mcand  <= mcand << 1;
        mplier <= mplier >> 1;
        cnt    <= cnt + 1'b1;
        if (cnt == N-1) begin
          busy <= 0; done <= 1;
          p    <= mplier[0] ? acc + mcand : acc;
        end
      end
    end
endmodule
```

After the load cycle the multiplier uses exactly N cycles, and `done` pulses for one cycle when `p` is valid. Trainees verify it against `a * b` for all values at N = 4, and for corner cases (0, 1, all ones) at N = 16.

### Lab assignments

8. Write a Verilog module that takes 32-bit data at 100 MHz and issues 16-bit data at 200 MHz.
9. Write a Verilog module that takes 16-bit data at 200 MHz and issues 32-bit data at 100 MHz.
10. Implement a multiplier for two N-bit operands using an adder. The computation concludes in N cycles.

For 8 and 9, loop one module's output into the other and check that the 32-bit stream comes back unchanged after the fixed latency. Use a queue-based scoreboard in the testbench.

### Checkpoint

- What changes if `clk100` and `clk200` come from different oscillators?
- Which signal does the fast domain sample in `w32_to_w16`, and why is that safe here?
- How many adders does `mult_seq` use, and how could its adder width be cut from 2N to N bits?

### Common mistakes

- Generating both clocks with independent `always #` blocks, which makes the shared-edge race change between simulators.
- Dropping the first word after reset because the pairing logic starts in the wrong phase.
- Finishing the multiplier one cycle early or late: the counter runs 0 to N-1, and the last partial product is added in the same cycle that raises `done`.

## Module 6: Special Verilog topics

*2026-08-11 to 2026-08-13, "Read and understand special Verilog topics"*

These are the language features that separate a trainee who can write a counter from one who can write a reliable testbench. Read each topic, run the snippet, then do the exercises at the end of the module.

### Topic map

| Topic | What to learn | Why it matters |
| --- | --- | --- |
| Blocking vs non-blocking | `=` updates now, `<=` updates at the end of the time step | Decides whether clocked code behaves like hardware |
| Event scheduling | Active, inactive and NBA regions; `$display` vs `$strobe` vs `$monitor` | Explains most simulation races |
| Tasks and functions | Tasks can wait and take time; functions return one value in zero time | Reusable bus-drive and check routines |
| `generate` | Build repeated structure from a parameter | Parameterized RTL without copy-paste |
| `casex`, `casez` | `?` and `z` as don't-care; `casex` also treats `x` as don't-care | `casex` can hide bugs in simulation |
| Delays | Inertial `assign #3 y = a;` swallows short pulses; `y <= #3 a;` passes them | Gate models and L2P in Module 3 |
| File and memory I/O | `$readmemh`, `$fopen`, `$fscanf` | Loading stimulus and expected results |
| Random stimulus | `$random(seed)`, constrained by masks and modulo | Cheap randomized tests |
| Simulation control | `$finish`, `$stop`, `$time`, `$dumpfile`, `$dumpvars`, ``  `timescale `` | Every testbench uses these |

### Worked example: blocking assignments in clocked code

```verilog
// Wrong for hardware: both end up with the old value of b
always @(posedge clk) begin a = b; b = a; end

// Correct: a and b swap
always @(posedge clk) begin a <= b; b <= a; end
```

Rule of thumb: `<=` in clocked blocks, `=` in combinational `always @*` blocks, and never both kinds in one block.

### Worked example: a self-checking testbench

```verilog
`timescale 1ns/1ps
module tb_adder8;
  reg  [7:0] a, b;
  wire [8:0] sum;
  integer    errors = 0;

  adder8 dut (.a(a), .b(b), .sum(sum));    // assign sum = a + b;

  task check(input [8:0] expected);
    if (sum !== expected) begin
      errors = errors + 1;
      $display("%0t FAIL a=%0d b=%0d sum=%0d expected=%0d",
               $time, a, b, sum, expected);
    end
  endtask

  initial begin
    $dumpfile("tb_adder8.vcd");
    $dumpvars(0, tb_adder8);
    repeat (200) begin
      a = $random; b = $random; #1;
      check({1'b0, a} + {1'b0, b});
    end
    if (errors == 0) $display("PASS");
    else             $display("FAIL: %0d errors", errors);
    $finish;
  end

  initial begin #1_000_000; $display("TIMEOUT"); $finish; end   // watchdog
endmodule
```

Four habits to keep from this example: compare with `!==` so `x` fails; print the inputs with every failure; end with one clear PASS or FAIL line; add a watchdog so a hung test still ends.

### Worked example: `generate`

```verilog
module add_n #(parameter N = 8)
  (input [N-1:0] a, b, input ci, output [N-1:0] s, output co);
  wire [N:0] c;
  assign c[0] = ci;
  genvar i;
  generate
    for (i = 0; i < N; i = i + 1) begin : fa
      assign s[i]   = a[i] ^ b[i] ^ c[i];
      assign c[i+1] = (a[i] & b[i]) | (c[i] & (a[i] ^ b[i]));
    end
  endgenerate
  assign co = c[N];
endmodule
```

### Worked example: loading a memory image

```verilog
reg [7:0] rom [0:15];
initial $readmemh("rom.hex", rom);   // one hex byte per line
```

### Lab exercises

1. Run the swap example with `=` and with `<=`, and explain the waveform difference in two sentences.
2. Build `add_n` for N = 4, 8 and 16 and check each against `+` with the testbench above.
3. Write a task `apb_write(addr, data)` that drives a bus for two clock cycles; Module 7 reuses it.
4. Create `rom.hex`, load it with `$readmemh`, and read every location in a testbench.
5. Write a 4-input priority encoder with `casez`, then with `casex`, and drive `x` into the input to show how `casex` can hide a bug.
6. Print one signal with `$display`, `$strobe` and `$monitor` in the same time step as a non-blocking update, and explain which value each prints.

### Checkpoint

- Which scheduler region executes a non-blocking update, and why does that avoid read-before-write races between flip-flops?
- Why is `if (sum == expected)` a weaker check than `!==`?
- What does a `timescale` of `1ns/1ps` mean for `#5`?

### Common mistakes

- Driving DUT inputs in a testbench with `<=` at the same edge the DUT samples them; use a short delay or drive on the opposite edge.
- Putting delays inside synthesizable RTL.
- Leaving out `$finish`, so the simulator never stops.

## Module 7: AMBA APB protocol and slave testbench

*2026-08-11 to 2026-08-13, "Read through the APB protocol" and "Implement an APB read/write using a test bench around this APB slave"*

APB is the simple peripheral bus in AMBA: every transfer takes at least two clock cycles, a setup cycle and an access cycle. The reading for this module is the [AMBA 3 APB specification](https://web.eecs.umich.edu/~prabal/teaching/eecs373-f12/readings/ARM_AMBA3_APB.pdf) (ARM IHI 0024B), chapters 2 to 4, and the reference slave [APB\_Slave.v](https://github.com/maomran/APB-Slave/blob/master/APB_Slave.v).

### Protocol summary

| Signal | Driven by | Meaning |
| --- | --- | --- |
| PCLK | Clock source | All transfers are timed on its rising edge |
| PRESETn | System | Active-low reset |
| PADDR | Bridge | Address, up to 32 bits |
| PSELx | Bridge | One select per slave |
| PENABLE | Bridge | High from the second cycle of a transfer onward |
| PWRITE | Bridge | High for write, low for read |
| PWDATA | Bridge | Write data, up to 32 bits |
| PRDATA | Slave | Read data, up to 32 bits |
| PREADY | Slave | Slave pulls it low to extend the transfer (wait states) |
| PSLVERR | Slave | Transfer failed; optional, tied low if absent |

The bus has three states. **IDLE** is the default, with `PSEL` and `PENABLE` low. **SETUP** lasts exactly one cycle with `PSEL` high and `PENABLE` low. **ACCESS** follows with `PENABLE` high; address, write, select and write data must stay stable from SETUP into ACCESS. In ACCESS the bus stays while `PREADY` is low. When `PREADY` is high the transfer ends, and the bus goes to IDLE or straight to SETUP for a back-to-back transfer.

`PSLVERR` is only meaningful in the last cycle of a transfer, when `PSEL`, `PENABLE` and `PREADY` are all high. A failed write may or may not have changed the peripheral, and a failed read may return unreliable data.

### Reading the reference slave

`APB_Slave.v` is a 256-word by 32-bit memory with a three-state FSM (`IDLE`, `W_ENABLE`, `R_ENABLE`). Trainees should read it line by line and answer the questions below before writing any testbench.

- It has no `PENABLE` input, so the setup/access distinction does not exist; the FSM supplies its own two-step sequence.
- It clocks on the falling edge of `PCLK` (`always @(negedge PRESETn or negedge PCLK)`), not the rising edge the spec requires.
- `PREADY` is set after the first access and is cleared only by reset, so it is not a per-transfer handshake.
- `PRDATA` is loaded on the second edge and cleared on the next one, so read data is visible for about one cycle.
- `RAM` has no reset, so unwritten locations read as `x`.
- There is no `PSLVERR`.

It is a useful teaching example because it works, and the gaps are the lesson. The first lab drives it as it is; the second fixes it.

### Lab 7.1: testbench around the reference slave

The slave samples on the falling edge and needs `PSEL`, `PWRITE`, `PADDR` and `PWDATA` held across two falling edges. The tasks below drive on rising edges, half a cycle away from sampling, so there is no race.

```verilog
`timescale 1ns/1ps
module tb_apb_slave;
  reg         PCLK = 0, PRESETn = 0, PWRITE = 0, PSEL = 0;
  reg  [7:0]  PADDR = 0;
  reg  [31:0] PWDATA = 0;
  wire [31:0] PRDATA;
  wire        PREADY;
  reg  [31:0] rd;
  integer     i, errors = 0;

  APB_Slave dut (.PCLK(PCLK), .PRESETn(PRESETn), .PADDR(PADDR), .PWRITE(PWRITE),
                 .PSEL(PSEL), .PWDATA(PWDATA), .PRDATA(PRDATA), .PREADY(PREADY));

  always #5 PCLK = ~PCLK;

  task apb_write(input [7:0] addr, input [31:0] data);
    begin
      @(posedge PCLK); PSEL = 1; PWRITE = 1; PADDR = addr; PWDATA = data;
      @(posedge PCLK);                    // FSM moved to W_ENABLE on the falling edge
      @(posedge PCLK); PSEL = 0; PWRITE = 0;   // write happened on the second falling edge
    end
  endtask

  task apb_read(input [7:0] addr, output [31:0] data);
    begin
      @(posedge PCLK); PSEL = 1; PWRITE = 0; PADDR = addr;
      @(negedge PCLK);                    // FSM moves to R_ENABLE
      @(negedge PCLK); #1;                // PRDATA loaded on this edge
      data = PRDATA;
      @(posedge PCLK); PSEL = 0;
    end
  endtask

  initial begin
    $dumpfile("tb_apb_slave.vcd"); $dumpvars(0, tb_apb_slave);
    #22 PRESETn = 1;
    for (i = 0; i < 16; i = i + 1) apb_write(i, 32'hA5A50000 + i);
    for (i = 0; i < 16; i = i + 1) begin
      apb_read(i, rd);
      if (rd !== 32'hA5A50000 + i) begin
        errors = errors + 1;
        $display("FAIL addr %0d: got %h", i, rd);
      end
    end
    apb_read(8'd200, rd);                 // never written
    if (rd !== 32'bx) $display("NOTE: unwritten location returned %h", rd);
    if (errors == 0) $display("PASS"); else $display("FAIL: %0d errors", errors);
    $finish;
  end
endmodule
```

Extend it with back-to-back writes, a write followed by a read of the same address, a reset in the middle of a transfer, and overwriting a location.

### Lab 7.2: a spec-compliant slave and a proper bus master task

Write `apb3_slave` with `PENABLE`, rising-edge clocking and `PREADY` and `PSLVERR` outputs. A first version has zero wait states:

```verilog
module apb3_slave #(parameter AW = 8, parameter DW = 32)
  (input PCLK, PRESETn, PSEL, PENABLE, PWRITE,
   input  [AW-1:0] PADDR, input [DW-1:0] PWDATA,
   output [DW-1:0] PRDATA, output PREADY, output PSLVERR);
  reg [DW-1:0] mem [0:(1<<AW)-1];
  wire access = PSEL & PENABLE;

  assign PREADY  = 1'b1;
  assign PSLVERR = 1'b0;
  assign PRDATA  = (PSEL && !PWRITE) ? mem[PADDR] : {DW{1'b0}};

  always @(posedge PCLK)
    if (access && PWRITE) mem[PADDR] <= PWDATA;
endmodule
```

The master tasks follow SETUP, ACCESS, then wait for `PREADY`:

```verilog
task apb3_write(input [7:0] a, input [31:0] d);
  begin
    @(posedge PCLK); #1; PSEL = 1; PENABLE = 0; PWRITE = 1; PADDR = a; PWDATA = d;  // SETUP
    @(posedge PCLK); #1; PENABLE = 1;                                               // ACCESS
    @(posedge PCLK); while (!PREADY) @(posedge PCLK);                               // wait states
    #1; PSEL = 0; PENABLE = 0; PWRITE = 0;
  end
endtask

task apb3_read(input [7:0] a, output [31:0] d);
  begin
    @(posedge PCLK); #1; PSEL = 1; PENABLE = 0; PWRITE = 0; PADDR = a;
    @(posedge PCLK); #1; PENABLE = 1;
    @(posedge PCLK); while (!PREADY) @(posedge PCLK);
    d = PRDATA;                                    // sampled in the last ACCESS cycle
    #1; PSEL = 0; PENABLE = 0;
  end
endtask
```

Then extend the slave in two steps and re-run the same testbench each time:

1. Add wait states: hold `PREADY` low for a configurable number of cycles after `PENABLE` rises.
2. Add `PSLVERR`: raise it in the last ACCESS cycle for addresses at or above `8'hF0`, and make the testbench check it only when `PSEL`, `PENABLE` and `PREADY` are all high.

Add assertion-style checks to the testbench: `PADDR`, `PWRITE` and `PWDATA` must not change between SETUP and the end of ACCESS, and `PENABLE` must never be high in the cycle `PSEL` rises.

### Checkpoint

- What is the minimum number of cycles for one APB transfer, and how many with two wait states?
- Why must `PADDR` be stable through ACCESS even though the slave may only look at it in the last cycle?
- List three ways `APB_Slave.v` differs from the APB3 specification and say which one would break a real bridge first.

### Common mistakes

- Raising `PENABLE` in the same cycle as `PSEL`, which skips SETUP.
- Sampling `PRDATA` before `PREADY`.
- Treating `PSLVERR` as valid in every cycle instead of only the last ACCESS cycle.

## Module 8: Design project, FIFO flush

*2026-08-14 to 2026-08-18, "Use specification from FIFO Flush Specification.pdf and develop RTL"*

This is the capstone: trainees take a written specification, turn it into RTL, and prove it with a testbench. The specification PDF was not available when this manual was written, so the module gives the project method, a baseline FIFO to start from, and the questions to settle from the PDF. Replace the assumptions below with what the PDF says.

### Five-day plan

| Day | Date | Goal | Output |
| --- | --- | --- | --- |
| 1 | 2026-08-14 | Read the spec, list every requirement and ambiguity | Requirements table, assumptions list, block diagram |
| 2 | 2026-08-15 | Write RTL from the baseline | FIFO with flush that lints clean |
| 3 | 2026-08-16 | Directed tests | Testbench, waveforms for each scenario |
| 4 | 2026-08-17 | Random test with a reference model | Scoreboard, bug list and fixes |
| 5 | 2026-08-18 | Review and demo | Final RTL, test report, 10-minute walkthrough |

### Step 1: turn the specification into a requirements table

Give every sentence in the PDF that says what the design must do an ID (`R1`, `R2`, and so on), and write one test that proves it. Then list the questions the PDF does not answer. Settle at least these:

- What triggers a flush: a one-cycle pulse, a level, or a register write?
- How fast must it take effect: the same cycle, or after a stated number of cycles?
- What happens to `empty`, `full` and the occupancy count during and after a flush?
- What if a write or a read arrives in the same cycle as the flush?
- Does a flush drop all entries or only some?
- Is the FIFO single-clock or dual-clock, and in which domain does flush live?

### Step 2: baseline FIFO

The code below is a standard single-clock FIFO with one assumption for flush: it empties the FIFO in one cycle and ignores any write or read in that cycle. Change it to match the PDF.

```verilog
module sync_fifo #(parameter DW = 8, parameter AW = 4)
  (input clk, rst_n, flush, wr_en, rd_en, input [DW-1:0] din,
   output [DW-1:0] dout, output full, empty);
  reg [DW-1:0] mem [0:(1<<AW)-1];
  reg [AW:0]   wptr, rptr;          // one extra bit tells full from empty

  assign empty = (wptr == rptr);
  assign full  = (wptr[AW-1:0] == rptr[AW-1:0]) && (wptr[AW] != rptr[AW]);
  assign dout  = mem[rptr[AW-1:0]]; // first word falls through

  wire do_wr = wr_en && !full  && !flush;
  wire do_rd = rd_en && !empty && !flush;

  always @(posedge clk or negedge rst_n)
    if (!rst_n) begin wptr <= 0; rptr <= 0; end
    else if (flush) begin wptr <= 0; rptr <= 0; end
    else begin
      if (do_wr) begin mem[wptr[AW-1:0]] <= din; wptr <= wptr + 1'b1; end
      if (do_rd) rptr <= rptr + 1'b1;
    end
endmodule
```

The flush works by resetting both pointers, so stale data stays in `mem` but can never be read. Trainees should explain why that is safe and what would change if the spec required the memory to be cleared.

### Step 3: test plan

| Scenario | Check |
| --- | --- |
| Reset | `empty` high, `full` low |
| Fill to full, then write again | Extra write is ignored, `full` stays high |
| Drain to empty, then read again | Extra read is ignored |
| Flush when empty, half full, and full | `empty` high afterwards, no old data readable |
| Write in the same cycle as flush | Behaves as the spec says |
| Read in the same cycle as flush | Behaves as the spec says |
| Back-to-back flushes | No error, still empty |
| Fill, flush, refill | New data comes out in order, none from before the flush |
| Random traffic with random flushes | Output always matches the reference model |

For the random test, keep a reference model in the testbench: an array, a write index, a read index, and a flush that sets both to zero. On each accepted read, compare `dout` with the model.

```verilog
reg [7:0] ref_mem [0:1023];
integer   ref_wr = 0, ref_rd = 0;

always @(posedge clk) begin
  if (flush) begin ref_wr = 0; ref_rd = 0; end
  else begin
    if (rd_en && !empty) begin
      if (dout !== ref_mem[ref_rd]) $display("%0t MISMATCH got %h exp %h", $time, dout, ref_mem[ref_rd]);
      ref_rd = ref_rd + 1;
    end
    if (wr_en && !full) begin ref_mem[ref_wr] = din; ref_wr = ref_wr + 1; end
  end
end
```

The model uses blocking assignments on purpose: it is software in the testbench, not hardware. It also reuses the DUT's `full` and `empty`, so add a second check that the DUT's flags match `ref_wr - ref_rd` against 0 and the depth.

### Deliverables

- Requirements table with a test for each requirement.
- Assumptions list, each with the question it answers.
- RTL, testbench, and one waveform per scenario in the test plan.
- A one-page report: what failed first, how it was found, and how it was fixed.

### Checkpoint

- Why does the baseline use an extra pointer bit?
- What would break if `flush` were allowed to act in the same cycle as an accepted write?
- If the FIFO were dual-clock, which side should receive `flush`, and what extra circuit does it need (see Module 9)?

## Module 9: Pulse synchronizer and arbiter

*2026-08-19 to 2026-08-22, "Pulse Synchronizer (please refer Assignment 6)" and "Arbiter"*

Two classic interview and design topics. The pulse synchronizer moves an event from one clock domain to another without losing it or going metastable. The arbiter decides who gets a shared resource when several requesters ask at once.

### Concepts: clock-domain crossing

- **Metastability.** If a flop samples an input while it is changing, its output can hover between 0 and 1 for a while before settling. A two-flop synchronizer gives the first flop a full cycle to settle before the second flop samples it.
- **Why a plain pulse fails.** A one-cycle pulse in a fast clock can be shorter than the slow clock's period and be missed entirely. In the other direction, a long pulse can be counted twice.
- **Toggle method.** Convert each pulse to a level change (a toggle flop in the source domain), pass the level through a two-flop synchronizer, and turn each level change back into a pulse (an XOR of two consecutive samples in the destination domain).
- **Spacing rule.** Source pulses must be spaced far enough apart that the destination sees each toggle: at least three destination clock periods is a safe rule. If the source can pulse faster, use a request/acknowledge handshake instead.
- **Simulation limit.** RTL simulation does not model metastability. It shows that the logic is right, not that the circuit is safe; the safety comes from the structure and from checking it in review.

### Worked example: toggle-based pulse synchronizer

```verilog
module pulse_sync
  (input  clk_src, rst_src_n, pulse_in,
   input  clk_dst, rst_dst_n,
   output pulse_out);

  reg tog_src;                       // source domain: one toggle per pulse
  always @(posedge clk_src or negedge rst_src_n)
    if (!rst_src_n)    tog_src <= 1'b0;
    else if (pulse_in) tog_src <= ~tog_src;

  reg s1, s2, s3;                    // destination domain
  always @(posedge clk_dst or negedge rst_dst_n)
    if (!rst_dst_n) begin s1 <= 0; s2 <= 0; s3 <= 0; end
    else            begin s1 <= tog_src; s2 <= s1; s3 <= s2; end

  assign pulse_out = s2 ^ s3;        // one destination cycle per toggle
endmodule
```

`s1` and `s2` are the synchronizer. `s3` only exists to detect the change. Trainees must keep `s1` and `s2` together with no logic between them in a real layout.

### Worked example: testing it

Use two unrelated clock periods such as 10 ns and 13 ns, a random phase offset, and source pulses spaced at least eight source cycles apart. Count pulses in and pulses out, and compare the counts at the end.

```verilog
always @(posedge clk_src) if (pulse_in)  n_in  = n_in  + 1;
always @(posedge clk_dst) if (pulse_out) n_out = n_out + 1;
// after the run, wait a few clk_dst cycles, then:
if (n_in === n_out) $display("PASS %0d pulses", n_in);
else                $display("FAIL in=%0d out=%0d", n_in, n_out);
```

Then reduce the spacing until pulses are lost, and record the smallest spacing that still passes for each clock ratio. Repeat with a naive version (pulse through two flops and an edge detector) to show it fails when the destination clock is slower.

### Worked example: fixed-priority and round-robin arbiters

The arbiter takes a request vector and returns a one-hot grant. Two rules never change: no grant without a request, and never two grants at once.

```verilog
// Fixed priority: requester 0 wins. req & -req keeps only the lowest set bit.
module arb_fixed #(parameter N = 4)
  (input [N-1:0] req, output [N-1:0] grant);
  assign grant = req & (~req + 1'b1);
endmodule

// Round robin: the requester after the last winner has the highest priority.
module arb_rr #(parameter N = 4)
  (input clk, rst_n, input [N-1:0] req, output reg [N-1:0] grant);
  reg  [N-1:0] last;                                  // one-hot, last winner
  wire [N-1:0] hi   = req & ~((last << 1) - 1'b1);    // requests above the last winner
  wire [N-1:0] pick = (hi != 0) ? (hi & (~hi + 1'b1))
                                : (req & (~req + 1'b1));
  always @(posedge clk or negedge rst_n)
    if (!rst_n) begin grant <= 0; last <= {1'b1, {(N-1){1'b0}}}; end
    else begin
      grant <= pick;
      if (pick != 0) last <= pick;
    end
endmodule
```

With all four requests held high, `arb_rr` grants `0001`, `0010`, `0100`, `1000`, then repeats, one cycle after each decision. `arb_fixed` grants `0001` forever, so requester 3 starves. That contrast is the reason to use round-robin when fairness matters.

### Lab assignments

1. Write Verilog RTL for a pulse synchronizer that transfers a pulse generated in the source clock domain to the destination clock domain, avoiding metastability. Simulate and capture the waveform.
2. Write an arbiter: when multiple masters, requesters or agents try to access the same resource at the same time, it avoids conflict by granting access to the deserved agent. Build the fixed-priority version first, then round robin.
3. Extension: add a hold option so a requester that is granted keeps the grant until it drops its request, then check that no other requester gets a grant during that time.
4. Extension: connect `pulse_sync` to the FIFO from Module 8 so a `flush` request from another clock domain reaches it safely, and decide what to do with writes that are still in flight.

### Checkpoint

- What is the smallest spacing between source pulses that your synchronizer passes for a 10 ns source clock and a 13 ns destination clock?
- Why does the toggle method need the destination to see the level for at least one full clock?
- Under what request pattern does the fixed-priority arbiter starve a requester, and what does the round-robin version guarantee?

### Common mistakes

- Passing a pulse directly into a flop in another domain.
- Passing several related signals through separate synchronizers, so they arrive in different cycles.
- Using round robin without a hold for multi-cycle transfers, so a transfer is cut off by the next grant.

## Appendix

### Coding guidelines

- One module per file, and the file name matches the module name.
- Clocked blocks use `<=`; combinational blocks use `=`; never mix them in one block.
- Every combinational block gives every output a value on every path (default assignment at the top).
- Active-low async reset named `rst_n`; clock named `clk`; parameters in capitals.
- No `#` delays, `initial` blocks or `$display` in synthesizable RTL.
- State names and constants come from `localparam`, never from bare numbers.
- Run lint (`verilator --lint-only -Wall file.v`) before every simulation and fix warnings rather than silencing them.

### Debug checklist

1. Does the testbench print inputs with every failure?
2. Is anything `x` that should not be? Trace the first `x` back to its source.
3. Are inputs driven away from the clock edge the DUT samples on?
4. Does the waveform match the timing diagram you drew first?
5. Does a simpler reference model disagree with the DUT, and which one is wrong?
6. Does it still fail with a different random seed?

### Assessment rubric

| Area | Meets expectation |
| --- | --- |
| RTL quality | Lint clean, no inferred latches, parameterized where the assignment says so |
| Verification | Self-checking, prints PASS or FAIL, covers corner cases and reset |
| Evidence | Waveform for each assignment with the key signals labelled |
| Understanding | Can explain every line in a walkthrough and answer the checkpoint questions |
| Debugging | Report lists what failed, how it was found, how it was fixed |

### Trainer notes

- Every code sample in this manual was written without running a simulator, because none was available when it was built. Run each one in Icarus Verilog and Verilator before class and correct anything the tools reject. The UDP tables in Module 2 and the shared-edge behavior in Module 5 are the two places most likely to need a tweak.
- The Edinburgh Verilog course page could not be fetched (the site blocks automated access), so Module 6 follows standard Verilog-2001 topics rather than that course's chapter order. Open the page by hand and map its chapters onto the topic table in Module 6.
- The FIFO Flush Specification.pdf was not provided, so Module 8 uses stated assumptions. Replace them with the PDF's requirements.
- The course sheet repeats assignment 3 as assignment 4. This manual uses the second slot for a zero-delay vs unit-delay comparison; change that if the repeat was a typing error and the intended item is known.
- The reference slave `APB_Slave.v` is not APB-compliant. Module 7 uses that deliberately; tell trainees up front so they do not copy its style.

### Sources

- [AMBA 3 APB Protocol Specification v1.0 (ARM IHI 0024B)](https://web.eecs.umich.edu/~prabal/teaching/eecs373-f12/readings/ARM_AMBA3_APB.pdf): signals, transfers, states and error response used in Module 7.
- [APB\_Slave.v by maomran](https://github.com/maomran/APB-Slave/blob/master/APB_Slave.v): reference slave analyzed in Module 7.
- Course assignment sheets (three screenshots supplied by the requester): schedule and assignment text for Modules 1 to 9.
- Verilog language topics, FSM, CDC and arbitration material: standard textbook knowledge, not taken from a fetched page.
