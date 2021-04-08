# apd_openlane_sky130
Advanced Physical Design using OpenLANE/Sky130 - Repository by Nuno Antunes (nuno.antunes@low-end.net)

## Day 1 Notes:

hd high density
fd foundry
sc standard cell

ss slow slow
ff fast fast
tt typical typical

XXXC temp
lvXX voltage

![Screenshot 2021-04-06 232619](https://user-images.githubusercontent.com/5050761/113910182-d3f96880-97d8-11eb-8fd5-4d6ded688c84.png)

![Screenshot 2021-04-06 234230](https://user-images.githubusercontent.com/5050761/113910161-d065e180-97d8-11eb-8144-524d8691ef7d.png)

## Day 2 Notes:

FF = Flip Flops/Lathes/rEGISTERS

A1, I1 = Standard Cells (AND/OR/NOT)

Dimensions are based on the standard cells/flip flops size.

Core, where logic is placed
And Die, the semiconductor material where the core is fabricated.

### Core utilization
  * Utilization Factor = Area occupied by Netlist/Total area of the Core.
    * Ideally 0.5/0.6 Utilization factor is used.
  * Aspect Ratio = Height/Width

### Preplaced Cells
  * Reusable combinational logic "Black Boxes" as IP's or modules.
    * Only input/output are defined externally.
  * Example: Memory, Clock gating cell, Comparator, Mux.
  * Arrangement of Preplaced Cells/IP's in a chip is referred as Floorplanning
  * Pre-defined locations, before automated placement-and-routing is performed
  * Locations are dependent on design

### Decoupling capacitors
  During a logic state change an increased demand on current behavior happens. Resistance in a non-idea circuit means there are multiple voltage drops betwen the supply and logic circuit.
  * Noise Margin : voltages should be inside a logic margin value (NM_l or NM_h) do be detected as 0 or 1, respectively. Voltage drops can affect the result for the logic outcome (undefined region).
  Decoupling capacitors are placed next to the preplaced cells to prevent the voltage drops during transition. 

### Power planning
  After local communication, power supply communication is required. For driver/load pairs, the communication link between two devices needs to be taken into account - this means a disturbance that results in a voltage/ground bounces.
  
  This means that multiple Vdd/Vss pairs might be required to ensure adequate supply is delivered.
  
  * This is often defined as the mesh, with Vdd and Vss on horizontal or vertical lines.

### Pin placement
  Taking into account the inputs, outputs and preplaced cells, the netlist is defined (via VHDL/Verilog). 
  
  Normally input and output pins are placed at opposite sides of the core.
  
  Pin placement also depends on where the logic blocks are placed - this requires a full understanding of the design.
  
  The communication ("handshake") between frontend team (that defined the network connectivity) and backend team (that defines the pin placement) is also critical.
  
  Clock ports are bigger in size, as the clock drives the flip flops and require more current/less resistance.
  
### Logical Cell Placement Blockage
  The final step is blocking the areas for logical cells - Logical Cell Placement Blockage.
  
  This ensures automatic placement and routing will not place cells in the blocked areas.
  
### Preparation file notes
Required commands for openLAnE flow (interactive):

<code>./flow.tcl -interactive</code>

Inside flow, add the required package:

<code>package require openlane 0.9</code>

For preparation, it is possible to define a single "tag" that will be the folder used for the project run (without tag, folder name will be date/time based). 

<code>prep -design picorv32a -tag trial_run1</code>

You can view configuration variables using the echo command.

<code>echo $env(CLOCK_PERIOD) 15.000</code>

In order to create further changes from the prep, add <code<-overwrite</code> to the command.

<code>prep -design picorv32a -tag trial_run1</code>

This allows to apply any changes - everthing will be erased. An option to further change configuration during the run is using the <code>set</code> command:

<code>set ::env(CLOCK_PERIOD) 15.000</code>

Finally, we can run the sythesis stage:

<code>run_sythesis</code>

### Steps to run floorplan using OpenLANE

Continuing from Day 1, sythesis was done. Next stage is floorplan.

Switches for configuration (including floorplan) can be found on <code>$OPENFLOW/configuration/Readme.md</code>:

![image](https://user-images.githubusercontent.com/5050761/113921350-3016b980-97e6-11eb-9b4f-e03a76b042e3.png)

As you move into the flow, it might be useful to modify parameters as needed. For example, during the placement stage, PT_TARGET_DENSITY might be widely spread for condition analysis, and then set closer to dense to test timing constraints during design closure.

From a design folder, these parameters can also be applied to <code>./config.tcl</code>, for example:

<code>set ::env(FP_CORE_UTIL) 65</code>

For now, we are adding some configuration values to the design - FP_CORE_UTIL to 65, FP_:

![Screenshot 2021-04-07 213023](https://user-images.githubusercontent.com/5050761/114052492-61e35b00-988e-11eb-8207-d992caa54ee1.png)

In OpenLANE flow interactive, assuming the synthesis was completed in the same way as day1, we can start the placement:

<code>run_floorplan</code>

TODO:
UNITS DISTANCE MICRONS 1000 ; (0.001 mm)
DIEAREA ( llx lly ) ( urx ury )

Rrom the result/floorplan foldder, it is possible to run magic to visually inspect the floorplan result:

<code>magic -T ~/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.lef def read picorv32a.floorplan.def &</code>

![image](https://user-images.githubusercontent.com/5050761/114075375-c52cb780-98a5-11eb-8460-50b1102a0f0f.png)

### Netlist binding and initial place design

Libraries contain multiple variants of standard cells (logic gates, buffers, inverters, dff, latch, IGC) including:
  * w/h of cell
  * delay information of cell
  * required conditions of cell (venn conditions)
  * different threhold voltage (hVt, lVt) for the cell
  * but also various shapes for same block

During the placement step in openLANE, the standard cells are placed in the die, attempting to preserve signal integrity based on the input/output pins, and adding buffers between elements if distance becomes an issue.

<code>run_placement</code>

This results in a legal placement
![image](https://user-images.githubusercontent.com/5050761/114076170-a8dd4a80-98a6-11eb-848e-16307160b7cc.png)

And it is also possible to use magic to view the cell placement.

<code>magic -T ~/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.lef def read picorv32a.placement.def &</code>

![image](https://user-images.githubusercontent.com/5050761/114077114-bcd57c00-98a7-11eb-829b-e62024ba7965.png)

## Cell design and Characterization flows

With the standard cells placed, we should look into the standard cell design itself.

In a library a standard cell is designed by using the several elements, that we divide in three sections:
  * Inputs
    * Process design kits (PDKs) : DRC&LVS rules, SPICE models, library and user-defined specs -> normally provided by foundry.
      * Example DRV&LVS input parameters: poly width, extension over active, poly to active spacing
      * Example Spice input parameters: values for dynamic behavior (Threshold voltage, linear region, saturation region equations) 
      * Example library and user-defined parameters: cell-height, supply voltage, pin location, drawn gate-length
  * Cell design
    * Circuit design
      * Taking into account the transistor requirements (eg. current and switching threshold) values determine requirements for the circuit.
    * Layout design
      * From the circuit design, extract paths for pmos and nmos networks
      * Obtain [Euler's path](https://en.wikipedia.org/wiki/Eulerian_path) for both
      * Draw stick diagram
      * Convert into layout
    * Characterization
      * The step that helps getting timing, noise, power characteristics <- **We will follow deeper into this**.
  * Outputs
    * CDL (Circuit Description Language)
    * GDSII
    * LEF
    * Extracted spice netlist (.cir)

** Characterization flow 

  1. Read Models (NMOS/PMOS) (and tech file)
  2. Read extracted Spice netlist
  3. Recognize behavior of the cell
  4. Read the subcircuits of the cell
  5. Attach the necessary power sources
  6. Apply the stimulus
  7. Provide the necessary output capacitance(s)
  8. Provide the necessary simulation commands

All of this information is fed to the characterization program GUNA, generating timing, noise and power .libs.

** Timing characterization

Timing threshold definitions:

  * `slew_low_rise_thr` - low V threshold for slew rise (typ 20% of waveform)
  * `slew_high_rise_thr` - high V threshold for slew rise (typ 80% of waveform)
  * `slew_low_fall_thr` - low V threshold for slew fall (typ 20% of waveform)
  * `slew_high_fall_thr` - high V threshold for slew fall (typ 80% of waveform)
  * `in_rise_thr` - input waveform rise delay (typ 50% of waveform)
  * `in_fall_thr` - input waveform fall delay (typ 50% of waveform)
  * `out_rise_thr` - output waveform rise delay (typ 50% of waveform)
  * `out_fall_thr` - output waveform fall delay (typ 50% of waveform)

*** Delay calculations:

Buffer
`time(out_rise_thr) - time(in_rise_thr)`

Inverter
`time(out_fall_thr) - time(in_rise_thr)`

Choosing threshold point (ensuring in___xxx___thr is earlier than out___xxx_thr) is critical for characterization. This can be even more challenging if the circuit isn't optimized.

*** Transition time

`time(slew_high_fall_thr) - time(slew_low_fall_thr)`
`time(slew_low_rise_thr) - time(slew_high_rise_thr)`

*** Output current waveform

We will see this at a later stage.

* Day 3

** Labs for CMOS inverter ngspice simulations

We will be using a initial cell design from GitHub to save time for lab. 

*** IO placer revision

This is a quick test of the ability to change parameters during the openLANE flow. In this case an alternative floor IO placer (from 1 - random equidistant, to 2). From our loaded previous design, we perform:

`set ::env(FP_IO_MODE) 2`

And we then do `run_floorplan` again. The IO pin organization changes accordingly:
![image](https://user-images.githubusercontent.com/5050761/114092777-90772b00-98ba-11eb-8a06-1d21eb88da5d.png)

*** SPICE deck creation for CMOS inverter

Component connectivity for this circuit (netlist):
![image](https://user-images.githubusercontent.com/5050761/114093428-62deb180-98bb-11eb-89fc-cc333462591a.png)

  * Vin connects to M1 and M2 gate
  * PMOS (M1) source connects to Vdd
  * NMOS (M2) source connects to Vss

Component values:

  * cload (load capacitor) is set to 10fF (we will calculate this value later)
  * PMOS wl = 0.375u/0.25u
  * NMOS wl = 0.375u/0.25u

(ideally PMOS should be 2x, 3x wider)

  Vin = 2.5V (related to channel site - 0.25 -> 2.5V)
  Vdd = 2.5V
  Vss -> common to supplies.
  
  Identify nodes in SPICE netlist:
  ![image](https://user-images.githubusercontent.com/5050761/114094352-80604b00-98bc-11eb-81e2-d1b92c15ee1d.png)
