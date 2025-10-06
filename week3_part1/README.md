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

```
synth -top vsdbabysoc
```
```
dfflibmap -liberty /home/vishal/VSDBabySoC/src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```
```
abc -liberty /home/vishal/VSDBabySoC/src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

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

<img src="images/diagram.png" alt="My Diagram" width="900"/>





