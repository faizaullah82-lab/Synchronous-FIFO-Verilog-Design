# Synchronous-FIFO-Verilog-Design
A parameterizable synchronous FIFO written in Verilog, with a directed testbench covering basic read/write operation, back-to-back interleaved access, and full/empty boundary conditions.

## Overview

This FIFO uses a dual-pointer architecture with an extra guard bit on both the read and write pointers. Instead of a separate counter to track occupancy, the FIFO compares the pointers directly: if they match exactly, the FIFO is empty, and if they match except for the guard bit, the FIFO is full. This is a common technique in synchronous FIFO design and avoids extra state that would otherwise need to be kept in sync with the read/write logic.

Key parameters:
- `FIFO_DEPTH`: number of entries (default 8)
- `DATA_WIDTH`: width of each entry in bits (default 32)

## Design details

- Write and read pointers are each one bit wider than needed to address the memory array. The extra bit acts as a wrap indicator, which is what makes the full/empty comparison work without a counter.
- Writes are gated on `cs && wr_en && !full`, reads are gated on `cs && rd_en && !empty`, so the design protects itself against overflow and underflow at the RTL level rather than relying on the testbench to avoid those cases.
- One assumption baked into the pointer logic: it assumes `FIFO_DEPTH` is a power of two. If it isn't, the pointer wraps at the next power of two rather than at the actual depth, which would cause incorrect full/empty behavior. Worth flagging if this were ever reused with a non-power-of-two depth.

## Verification approach

The testbench is directed rather than constrained-random, structured around three scenarios:

1. **Basic sequencing**: three back-to-back writes followed by three reads, checking that data comes out in the order it went in.
2. **Interleaved access**: a loop that writes and reads on alternating cycles across the full depth of the FIFO, exercising the pointers as they move through every address.
3. **Full and drain**: writes one more time than the FIFO can hold (9 writes into an 8-deep FIFO), which should cause the final write to be dropped by the `!full` guard, then drains the FIFO completely with reads. This is the test that actually exercises the full-detection logic rather than just the happy path.

Waveforms are dumped to VCD for viewing in EPWave or any standard waveform viewer.

## What this demonstrates

- RTL design fundamentals: pointer-based FIFO architecture, parameterization, synthesizable coding style
- Testbench construction: task-based stimulus, clock generation, self-checking via `$display` output
- Edge case thinking: explicitly testing the full boundary rather than only nominal read/write flow

## Possible extensions

- Wrap in a UVM environment as a lightweight companion to a larger UVM testbench project

## Running it

Available on EDA Playground: https://www.edaplayground.com/x/KLjV

To run locally with Icarus Verilog:
```
iverilog -o fifo_sim design.sv testbench.sv
vvp fifo_sim
```
