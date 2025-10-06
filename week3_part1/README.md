# Introduction
GLS (Gate level simulation) is performed after synthesis, where the RTL design is transformed into a gate-level netlist consisting of logic gates and flip-flops. GLS verifies that the synthesized design maintains the intended functionality, checks for timing violations, and ensures the circuit will behave correctly when implemented in hardware. It provides a more accurate representation of real hardware behavior compared to RTL simulation and is an essential step in the digital design verification flow.

Now lets look at GLS waveform of Babysoc after synthesis.


Note: we have to use yosys tool to convert RTL design to gate level netlist, hence all the steps down must be done using yosys.

```
read_verilog /home/vishal/VSDBabySoCC/src/module/vsdbabysoc.v
read_verilog -I /home/vishal/VSDBabySoC/src/include /home/vishal/VSDBabySoC/src/module/clk_gate.v
read_verilog -I /home/vishal/VSDBabySoC/src/include /home/vishal/VSDBabySoC/output/compiled_tlv/rvmyth.v
```
```
read_liberty -lib /home/vishal/VSDBabySoC/src/lib/avsddac.lib
read_liberty -lib /home/vishal/VSDBabySoC/src/lib/avsdpll.lib
read_liberty -lib /home/vishal/VSDBabySoC/src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib

```
Reading verilog and liberty files is the first step to creating gate level netlist.

```
synth -top vsdbabysoc
```

It tells Yosys which module is the design’s entry point for synthesis; Yosys will elaborate the hierarchy starting at vsdbabysoc, resolve parameters/generates, flatten as configured, and then map the reachable logic to the target cell library to produce a gate-level netlist.

```
dfflibmap -liberty /home/vishal/VSDBabySoC/src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```
This command maps all flip-flops (DFFs) in your synthesized design to the actual standard-cell flip-flops from the specified Sky130 Liberty library.

```
abc -liberty /home/vishal/VSDBabySoC/src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```
This command invokes the ABC logic optimization and technology mapping tool (integrated via Yosys) to map your design’s combinational logic gates to the real standard cells provided by the specified Liberty file.


```
stat
```
This shows all the statistics of the vsdbabysoc.
```
write_verilog -noattr /home/vishal/VSDBabySoC/output/synth/vsdbabysoc.synth.v
```
This will write the gate level netlist for the RTL design created.

```
iverilog -o /home/vishal/VSDBabySoC/output/post_synth_sim/post_synth_sim.out -DPOST_SYNTH_SIM -DFUNCTIONAL -DUNIT_DELAY=#1 -I /home/vishal/VSDBabySoC/output/synth -I /home/vishal/VSDBabySoC/src/include -I /home/vishal/VSDBabySoC/src/module /home/vishal/VSDBabySoC/src/module/testbench.v
```
This saves the output in post_synth_sim in the output folder.

```
cd /home/vishal/VSDBabySoC/output/post_synth_sim
./post_synth_sim.out
```
This creates the vcd file finally.

```
gtkwave post_synth_sim.vcd

```

Waveform generated:

<img src="../Screenshots-20251006T162544Z-1-001/Screenshots/Screenshot from 2025-10-06 20-01-51.png" alt="My Diagram" width="900"/>

**Explanation:** The waveform shown in the screenshot is a GTKWave view of the post-synthesis gate-level simulation (GLS) for the VSDBbaysoc testbench. The displayed signals, such as CLK, reset, and OUT (with D[9:0]), indicate that the synthesized design is functionally correct at the gate level—producing expected outputs for given test inputs after synthesis. The correct toggling of the OUT signal, aligned with the clock and reset, confirms that functional behavior is preserved through the full synthesis and netlist mapping flow. This demonstrates that the gate-level netlist matches RTL intent and works as intended under simulated conditions.





