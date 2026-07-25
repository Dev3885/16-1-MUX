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

## Notes

- The testbench currently applies test vectors and prints results via `$monitor`, but does not yet do automated pass/fail comparison against expected values — worth adding (similar in spirit to the `checking` task pattern used in the adder project) if you want it to flag mismatches automatically rather than checking the printed output by eye.
- This is a learning project — feedback and suggestions are welcome.
