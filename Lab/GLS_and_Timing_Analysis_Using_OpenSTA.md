# Yosys Synthesis Tutorial for BabySoC

## Overview

This guide demonstrates the synthesis process of the BabySoC project using the Yosys Open SYnthesis Suite. It covers command usage, the correct handling of file paths, and provides solutions for common synthesis errors.

***

## Directory Structure

Make sure the following project structure is set up:

```
~/Desktop/labs/Week2/VLSI/VSDBabySoC/
├── src/
│   ├── include/           # Verilog include files
│   ├── module/            # Verilog modules (vsdbabysoc.v, clkgate.v, rvmyth.v, etc)
│   └── lib/               # Liberty files (sky130fdschdtt025C1v80.lib, avsddac.lib, avsdpll.lib)
```
All paths provided in commands below match those seen in actual flows.

***

## Step-by-Step Synthesis Flow

### 1. Launch Yosys

Start Yosys from your working directory:

```sh
cd ~/Desktop/labs/Week2/VLSI/VSDBabySoC/src
yosys
```

***

### 2. Load Liberty Cell Libraries

Load the technology cell library (example for Sky130):

```yosys
read_liberty -lib /home/vsdtapeout/Desktop/labs/Week_1/Day_1/VLSI/sky130RTLDesignAndSynthesisWorkshop/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_liberty -lib /home/vsdtapeout/Desktop/labs/Week_2/VLSI/VSDBabySoC/src/lib/avsddac.lib /home/vsdtapeout/Desktop/labs/Week_2/VLSI/VSDBabySoC/src/lib/avsdpll.lib
```
<img width="822" height="610" alt="image" src="https://github.com/user-attachments/assets/09e61736-eae1-402f-879f-b65297e6f0e1" />

If the library contains definitions, you should see "Imported XXX cell types from liberty file".

***

### 3. Read Verilog Source Files

Indicate SV (SystemVerilog) support and all include/module paths. Avoid typographical errors in commands or filenames:

```yosys
read_verilog -sv -I /home/vsdtapeout/Desktop/labs/Week_2/VLSI/VSDBabySoC/src/include/ -I /home/vsdtapeout/Desktop/labs/Week_2/VLSI/VSDBabySoc/src/module/ /home/vsdtapeout/Desktop/labs/Week_2/VLSI/VSDBabySoC/src/module/vsdbabysoc.v /home/vsdtapeout/Desktop/labs/Week_2/VLSI/VSDBabySoC/src/module/clk_gate.v /home/vsdtapeout/Desktop/labs/Week_2/VLSI/VSDBabySoC/src/module/rvmyth.v
```
<img width="823" height="609" alt="image" src="https://github.com/user-attachments/assets/22ea9913-9278-4f4b-893d-068893e53331" />

The above paths must exist; adjust them as per your actual directory. If successful, you'll see "Successfully finished Verilog frontend".

***

### 4. Specify the Top Module

Set the top-level module for synthesis (usually `vsdbabysoc`):

```yosys
synth -top vsdbabysoc
```
<img width="827" height="789" alt="image" src="https://github.com/user-attachments/assets/ac6fbf92-d994-4b22-b39b-270372b875fa" />

This will trigger module hierarchy and process netlist conversion with messages such as "Analyzing design hierarchy... Top module Used module..." and "Creating decoders for process...".

***

### 5. Technology Mapping

Map RTL to gates using ABC and techmap:

```yosys
# Automatic, as part of 'synth'
```
Expect log reports with extracted gate counts and optimized modules (e.g., "Extracted 4089 gates and 5280 wires to a netlist network with 1188 inputs and 216 outputs").

***

### 6. Run Design Checks

```yosys
check
```
<img width="603" height="212" alt="image" src="https://github.com/user-attachments/assets/4a70ce88-d118-40e6-a099-60826a35abfb" />

This checks for obvious structural problems in your synthesized design.

***

### 7. Write the Synthesized Netlist

Export your final synthesized Verilog netlist:

```yosys
write_verilog vsdbabysoc_synth_net.v
abc -liberty /home/vsdtapeout/Desktop/labs/Week_2/VLSI/VSDBabySoC/src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib 

```
Netlist files are typically written in the working directory.

***

### 8. Visualize the Synthesized Design

For visual inspection, use:

```yosys
show vsdbabysoc
```
<img width="1050" height="310" alt="image" src="https://github.com/user-attachments/assets/d6b5d48d-8936-47f4-8fd6-ebf0c63b0881" />
If multiple modules are present, specify only one module due to Graphviz format limitations ("ERROR: For formats different than 'ps' or 'dot' only one module must be selected"). By default, results are stored in `/home/vsdtapeout/.yosys_show.dot`.

***

## Common Errors & Solutions

| Error Message | Cause | Solution |
|---------------|-------|----------|
| `ERROR Missing function on output GCLK of cell ...` | Liberty file lacks function definition for some output | Fix .lib file or use correct version |
| `ERROR syntax error, unexpected TOKID ...` | File is not valid Verilog/SystemVerilog | Check syntax using a simulator (Icarus Verilog recommended) |
| `ERROR Cant open input file ... for reading No such file or directory` | File path typo, or file missing | Double-check directory and filenames |
| `ERROR Re-definition of module ...` | Module loaded twice | Ensure each file is loaded only once |
| `ERROR Cant open include file ...` | Include file missing or path wrong | Verify/include directory and file names |
| `ERROR Command syntax error No filename given.` | Syntax error in Yosys command | Follow documented syntax, use help |
| `WARNING: Font family 'Times-Roman' is not available, using 'Ubuntu Sans 11'` | Graphviz font not installed | Usually not fatal, just an info message |

***

## Tips for Successful Synthesis

- Validate Verilog modules in a simulator before Yosys synthesis to catch subtle syntax issues (Yosys parser is not very descriptive for errors).
- Always use absolute paths for scripts to avoid directory confusion.
- Organize modules, includes, and libraries separately for clarity.
- Read the log output for warnings about memory/register transformations or dead code removal.
- Use `check` for sanity of final netlist before downstream tools.

***

# Post-Synthesis of VSDBabySoC

***

## 1. Launch Yosys

```bash
yosys
```

Start the Yosys synthesis tool. This opens the Yosys interactive shell where you can enter synthesis commands.

<img width="814" height="605" alt="image" src="https://github.com/user-attachments/assets/fd430366-995c-463a-a9fe-f9f8cb1ea421" />

***

## 2. Read the Main Verilog Module

```bash
read_verilog /home/vsdtapeout/Desktop/labs/Week_2/VLSI/VSDBabySoC/src/module/vsdbabysoc.v
```

This command loads the main Verilog source file of your design into Yosys. It parses the RTL code and converts it into Yosys's internal representation.

<img width="822" height="237" alt="image" src="https://github.com/user-attachments/assets/5e82492f-1a75-41d7-838a-afda9357fe8a" />

***

## 3. Read Additional Verilog Modules with Include Paths

```bash
read_verilog -I /home/vsdtapeout/Desktop/labs/Week_2/VLSI/VSDBabySoC/src/include/ /home/vsdtapeout/Desktop/labs/Week_2/VLSI/VSDBabySoC/src/module/rvmyth.v

read_verilog -I /home/vsdtapeout/Desktop/labs/Week_2/VLSI/VSDBabySoC/src/include/ /home/vsdtapeout/Desktop/labs/Week_2/VLSI/VSDBabySoC/src/module/clk_gate.v
```

These commands load additional Verilog modules required by your design. The `-I` option specifies the directory for included files (like headers or macros). This ensures all dependencies are resolved.

<img width="820" height="615" alt="image" src="https://github.com/user-attachments/assets/9292c8a5-3f46-4a7a-9c36-08f2fe40d4b4" />


***

## 4. Load Liberty Cell Libraries

```bash
read_liberty -lib /home/vsdtapeout/Desktop/labs/Week_2/VLSI/VSDBabySoC/src/lib/avsdpll.lib /home/vsdtapeout/Desktop/labs/Week_2/VLSI/VSDBabySoC/src/lib/avsddac.lib /home/vsdtapeout/Desktop/labs/Week_2/VLSI/VSDBabySoC/src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

This command loads the standard cell libraries in Liberty format. These libraries contain timing, power, and area information for the cells used in synthesis and technology mapping.

<img width="818" height="613" alt="image" src="https://github.com/user-attachments/assets/0813d972-0245-4872-a203-04fae198b871" />

***

## 5. Synthesize the Design with Top Module

```bash
synth -top vsdbabysoc
```

This runs the generic synthesis process on the design, targeting the specified top module `vsdbabysoc`. It performs RTL elaboration, optimization, and prepares the design for mapping to standard cells.

<img width="817" height="616" alt="image" src="https://github.com/user-attachments/assets/8403593b-d50a-4c5f-b4d9-010010020fa6" />

***

## 6. Map Flip-Flops to Standard Cells

```bash
dfflibmap -liberty /home/vsdtapeout/Desktop/labs/Week_2/VLSI/VSDBabySoC/src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

This step maps all flip-flops in the design to the flip-flop cells defined in the Liberty library. It ensures the flip-flops are technology-specific and compatible with the target process.

<img width="815" height="612" alt="image" src="https://github.com/user-attachments/assets/c960d730-95f5-4a1b-822f-c9841b9d9a94" />

***

## 7. Technology Mapping and Optimization with ABC

```bash
abc -liberty /home/vsdtapeout/Desktop/labs/Week_2/VLSI/VSDBabySoC/src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib -script +strash;scorr;ifraig;retime{D};strash;dch,-f;map,-M,1{D}
```

This command invokes the ABC tool integrated in Yosys for advanced logic optimization and technology mapping. The script applies a sequence of transformations:

- `+strash`: Structural hashing to reduce logic.
- `scorr`: Structural correlation.
- `ifraig`: Iterative functional reduction.
- `retime{D}`: Retiming to optimize registers.
- `dch,-f`: Don't care optimization.
- `map,-M,1{D}`: Map logic to standard cells with specific constraints.

This step improves area, timing, and power characteristics.

<img width="820" height="615" alt="image" src="https://github.com/user-attachments/assets/4792df63-fb6b-4a06-b789-42c76f4cd0b9" />

***

## 8. Flatten the Hierarchy

```bash
flatten
```

This command removes all module hierarchy, producing a flat netlist. Flattening simplifies the design for downstream tools like place-and-route but can make debugging harder.

<img width="462" height="163" alt="image" src="https://github.com/user-attachments/assets/ad77fe72-c1ee-427b-9d9e-e8c552fff8a6" />


***

## 9. Set Undefined Signals to Zero

```bash
setundef -zero
```

This replaces any undefined signals in the design with constant zero. This avoids simulation or synthesis issues caused by unknown values.

<img width="760" height="88" alt="image" src="https://github.com/user-attachments/assets/c8cfe015-6bcc-419d-a86c-35254b75f0c7" />

***

## 10. Clean Unused Wires and Cells

```bash
clean -purge
```

This removes any unused wires, cells, or logic from the design, reducing netlist size and complexity.

<img width="491" height="75" alt="image" src="https://github.com/user-attachments/assets/b4f0874c-6830-482f-83e3-30e2175f01b6" />


***

## 11. Rename Nets and Cells for Readability

```bash
rename -enumerate
```

This renames nets and cells with simple enumerated names (e.g., `net1`, `cell2`) to make the netlist easier to read and analyze.


***

## 12. Print Synthesis Statistics

```bash
stat
```

This command prints statistics about the synthesized design, such as the number of wires, cells, and memory elements. It helps verify the synthesis results.

<img width="390" height="274" alt="image" src="https://github.com/user-attachments/assets/a6371a64-9fd4-438e-b725-bf4b1c387029" />

***

## 13. Write the Synthesized Verilog Netlist

```bash
write_verilog -noattr /home/vsdtapeout/Desktop/labs/Week_2/VLSI/VSDBabySoC/src/vsdbabysoc.synth.v
```

This exports the final synthesized netlist to a Verilog file without any synthesis attributes (`-noattr`). This netlist is technology-mapped and optimized, ready for place-and-route or further analysis.

<img width="809" height="194" alt="image" src="https://github.com/user-attachments/assets/0f499bf1-e1b7-4c5d-a283-fecc76555dd7" />

***

# Post Synthesis Simulation and Waveforms

## Compile the Testbench

Before running the iverilog command, copy the necessary standard cell and primitive models: These files must be present in the same directory as the testbench (src/module) to resolve all module references during compilation.

To ensure that the synthesized Verilog file (vsdbabysoc.synth.v) is available in the src/module directory for further processing or simulation, you can copy it from the output directory to the src/module directory. Here is the step to do that:

Run the following iverilog command to compile the testbench:

```bash
iverilog -o /home/vsdtapeout/Desktop/labs/Week_2/VLSI/VSDBabySoC/src/output/post_synth_sim.out -DPOST_SYNTH_SIM -DFUNCTIONAL -DUNIT_DELAY=#1 -I /home/vsdtapeout/Desktop/labs/Week_2/VLSI/VSDBabySoC/src/include/ -I /home/vsdtapeout/Desktop/labs/Week_2/VLSI/VSDBabySoC/src/module/ /home/vsdtapeout/Desktop/labs/Week_2/VLSI/VSDBabySoC/src/module/testbench.v
```

| **Option / Argument**                                                      | **Purpose / Description**                                                            |
| -------------------------------------------------------------------------- | ------------------------------------------------------------------------------------ |
| `iverilog`                                                                 | Icarus Verilog compiler used to compile Verilog files into a simulation executable.  |
| `-o /home/vsdtapeout/Desktop/labs/Week_2/VLSI/VSDBabySoC/output/post_synth_sim.out` | Specifies the output binary file for simulation.                                     |
| `-DPOST_SYNTH_SIM`                                                         | Defines the macro `POST_SYNTH_SIM` (used in testbench to switch simulation modes).   |
| `-DFUNCTIONAL`                                                             | Defines `FUNCTIONAL` to use behavioral models instead of detailed gate-level timing. |
| `-DUNIT_DELAY=#1`                                                          | Assigns a unit delay of `#1` to all gates for post-synthesis simulation.             |
| `-I /home/vsdtapeout/Desktop/labs/Week_2/VLSI/VSDBabySoC/src/include`                              | Adds the `include` directory to the search path for `\`include\` directives.         |
| `-I /home/vsdtapeout/Desktop/labs/Week_2/VLSI/VSDBabySoC/src/module`                               | Adds the `module` directory to the include path for additional module references.    |
| `/home/vsdtapeout/Desktop/labs/Week_2/VLSI/VSDBabySoC/src/module/testbench.v`                      | Specifies the testbench file as the top-level design for simulation.                 |

#### ❗Note - You may encounter this error:
```bash
$ iverilog -o /home/vsdtapeout/Desktop/labs/Week_2/VLSI/VSDBabySoC/output/post_synth_sim/post_synth_sim.out -DPOST_SYNTH_SIM -DFUNCTIONAL -DUNIT_DELAY=#1 -I /home/vsdtapeout/Desktop/labs/Week_2/VLSI/VSDBabySoC/src/include -I /home/vsdtapeout/Desktop/labs/Week_2/VLSI/VSDBabySoC/src/module /home/vsdtapeout/Desktop/labs/Week_2/VLSI/VSDBabySoC/src/module/testbench.v
/home/vsdtapeout/Desktop/labs/Week_2/VLSI/VSDBabySoC/src/module/sky130_fd_sc_hd.v:74452: syntax error
I give up.
```
_To resolve this : Update the syntax in the file sky130_fd_sc_hd.v at or around line 74452._

###### Change:
```bash
`endif SKY130_FD_SC_HD__LPFLOW_BLEEDER_FUNCTIONAL_V
```
###### To:
```bash
`endif // SKY130_FD_SC_HD__LPFLOW_BLEEDER_FUNCTIONAL_V
```
### **Step 2: Navigate to the Post-Synthesis Simulation Output Directory**
```bash
cd output/
```
---
### **Step 3: Run the Simulation**

```bash
./post_synth_sim.out
```
<img width="808" height="291" alt="image" src="https://github.com/user-attachments/assets/a8056627-e4c5-4c6f-8f2a-4bea01418cb9" />
---

### **Step 4: View the Waveforms in GTKWave**

```bash
gtkwave post_synth_sim.vcd
```
---

<img width="1051" height="580" alt="image" src="https://github.com/user-attachments/assets/9976a3b2-b160-4be5-bb9c-c7b31ca23f61" />

## Comparing Pre-Synthesis and Post-Synthesis Output

To ensure that the synthesis process did not alter the original design behavior, the output from the pre-synthesis simulation was compared with the post-synthesis simulation.

Both simulations were run using GTKWave, and the resulting waveforms were observed.

<img width="1051" height="562" alt="image" src="https://github.com/user-attachments/assets/1aefb89d-70b5-4ad8-81b2-4fd186aa5e9d" />

<img width="1051" height="580" alt="image" src="https://github.com/user-attachments/assets/9976a3b2-b160-4be5-bb9c-c7b31ca23f61" />

✅ _The outputs match exactly, confirming that the functionality is preserved across the synthesis flow._
