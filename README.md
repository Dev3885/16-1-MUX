# 16:1 Multiplexer (Verilog)

A 16-to-1 multiplexer built hierarchically from smaller multiplexers, written and simulated in Verilog.

## Design structure

The 16:1 MUX isn't built directly — it's composed bottom-up from smaller building blocks:

```
mux2  (2:1 MUX)   — gate-level primitives (and/or)
  └── used 3x to build →  mux4  (4:1 MUX)
                              └── used 5x to build →  mux16  (16:1 MUX)
```

- **`mux2`** — the base case. A single 2:1 multiplexer built directly from `and`/`or` gate primitives rather than behavioral (`if`/`case`) code.
- **`mux4`** — a 4:1 MUX made of three `mux2` instances: two select between the low pair and high pair of inputs using `sel2`, and a third combines those two results using `sel1`.
- **`mux16`** — a 16:1 MUX made of five `mux4` instances: four handle 4-input groups of the 16-bit `data` bus using the lower two select bits (`S[1:0]`), and a fifth combines those four intermediate results using the upper two select bits (`S[3:2]`).

## Files

- **`rtl/2-1MUX.v`** — `mux2` module
- **`rtl/4-1MUX.v`** — `mux4` module
- **`rtl/16-1MUX.v`** — `mux16` module (top-level)
- **`sim/testbench.v`** — testbench that drives `data` and `S` through several combinations and monitors `Out`

## Ports

**`mux16`** (top-level):
| Port | Direction | Width | Description |
|---|---|---|---|
| `data` | input | `[15:0]` | 16 data inputs to select from |
| `S` | input | `[3:0]` | select lines (S[3:0] chooses one of 16 inputs) |
| `Out` | output | 1 bit | selected data bit |

## How to run

**Simulation (behavioral):**
1. Open the project in Vivado (or another simulator).
2. Make sure `sim/testbench.v` is set as the simulation top module.
3. Run behavioral simulation.
4. Check the console output (`$monitor`) to see `data`/`S`/`Out` for each applied test case.

**Synthesis:**
1. Keep only `rtl/2-1MUX.v`, `rtl/4-1MUX.v`, and `rtl/16-1MUX.v` in Design Sources (no testbench files).
2. Set `mux16` as the top module.
3. Run synthesis.

## Alternative implementation (commented out in source)

`16-1MUX.v` also contains two commented-out alternate versions, kept for comparison — worth understanding even though they're not the active implementation:

**Fully behavioral, no hierarchy at all:**
```verilog
module mux16 (input wire [15:0] data, input wire [3:0] S, output wire Out);
assign Out = data[S];
endmodule
```
Instead of building the MUX out of smaller MUXes, this uses Verilog's bit-select indexing directly: `S` picks which bit of `data` to route to `Out`. Functionally identical to the full gate-level hierarchy above, in a single line — but it hides the underlying structure a synthesis tool would still build, rather than expressing it explicitly. *(Note: the version as commented references a signal `sel` that doesn't exist in the port list — it should read `data[S]`, matching the declared port name.)*

**Behavioral 2:1 MUX for wide buses:**
```verilog
module mux64 (input wire [63:0] a, b, input wire sel, output wire [63:0] y);
assign y = sel ? b : a;
endmodule
```
A ternary-operator MUX selecting between two 64-bit buses. Since there are only two inputs to choose from, `sel` only needs to be 1 bit — no need for a select vector like `S[3:0]` in the 16:1 case.

**Why keep the hierarchical (`mux2`/`mux4`/`mux16`) version as the main implementation:** it mirrors how a 16:1 MUX would actually be built from discrete logic gates, and demonstrates structural/hierarchical design — the `assign Out = data[S];` one-liner is far more concise, but it lets Verilog's synthesis tool make all the structural decisions for you rather than practicing them yourself.

## Notes

- The testbench currently applies test vectors and prints results via `$monitor`, but does not yet do automated pass/fail comparison against expected values — worth adding (similar in spirit to the `checking` task pattern used in the adder project) if you want it to flag mismatches automatically rather than checking the printed output by eye.
- This is a learning project — feedback and suggestions are welcome.
