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

*SKY130_D2_SK1 - Chip Floor planning considerations*

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

<code>set $env(CLOCK_PERIOD) 15.000</code>

Finally, we can run the sythesis stage:

<code>run_sythesis</code>

### Steps to run floorplan using OpenLANE

Continuing from Day 1, sythesis was done. Next stage is floorplan.

Switches for configuration (including floorplan) can be found on <code>$OPENFLOW/configuration/Readme.md</code>:

![image](https://user-images.githubusercontent.com/5050761/113921350-3016b980-97e6-11eb-9b4f-e03a76b042e3.png)

As you move into the flow, it might be useful to modify parameters as needed. For example, during the placement stage, PT_TARGET_DENSITY might be widely spread for condition analysis, and then set closer to dense to test timing constraints during design closure.

From a design folder, these parameters can be applied to <code>./config.tcl</code>

<code>set ::env(FP_CORE_UTIL) 65</code>

TODO:
UNITS DISTANCE MICRONS 1000 ; (0.001 mm)
DIEAREA ( llx lly ) ( urx ury )

<code>magic -T ~/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.lef def read picorv32a.floorplan.def &</code>

*SKY130_D2_SK2 - Library Binding and Placement*

### Netlist binding and initial place design

Libraries contain:
  * w/h of cell
  * delay information of cell
  * required conditions of cell (venn conditions)
  * but also various shapes for same block

Optimize placement

Signal integrity - adding repeaters on the floorplan at expense of space
Data slew analysis

run_placement - runs course placement
