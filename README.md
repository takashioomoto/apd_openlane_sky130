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

SKY130_D2_SK1 - Chip Floor planning considerations

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
  
  
  
  
