# Advanced-Physical-Design-using-OpenLANE-Sky130
## Day 1 - Inception of open-source EDA, OpenLANE and Sky130 PDK
### Theory
#### How to talk to computers
##### Introduction to QFN-48 Package, chip, pads, core, die and IPs
##### Introduction to RISC-V
##### From Software Applications to Hardware
#### SoC design and OpenLANE 
##### Introduction to all components of open-source digital asic design
##### Simplified RTL2GDS flow
##### Introduction to OpenLANE and Strive chipsets
##### Introduction to OpenLANE detailed ASIC design flow
### Lab
#### Get familiar to open-source EDA tools
##### OpenLANE Directory Structure in Detail
First part includes navigating terminal and the different tools installed on linux. This includes exploring open_pdks and openlane. Open_pdks ensures compatbility of open source tools with existing pdk's. It includes libraries with .ref and .tech extensions that include files related to the process node like timing and lef files as well as the tools utilized in the design of circuits for that node respectively. We will be primiarily using sky130_fd_sc_hd.
Openlane commands
```
cd OpenLane
make mount

$ ./flow.tcl -interactive // This allows each step to be looked at manually.

% package require openlane 0.9
% prep -design picorv32a
% run_synthesis
```
##### Design Preparation Step
```
cd OpenLane
cd designs
ls -al
cd picorv32a
ls -al
tree
```
![alt text](img/day1/design.png)
##### Review files after design prep and run synthesis
```
cd OpenLane/designs/picorv32a/runs/RUN_2023.01.25_06.32.21/tmp
cat merged.lef
cd ../results
cd ../reports
cd ..
cat config.tcl
cat cmds.log
```
This includes information about layers.
![alt text](img/day1/tmp.png)
![alt text](img/day1/config-tcl.png)
##### OpenLANE Project Git Link Description
[OpenLane repo](https://github.com/The-OpenROAD-Project/OpenLane)
```
git clone https://github.com/The-OpenROAD-Project/OpenLane.git
```
Repository contains more detailed information about the OpenLane flow.
This tools takes design files and the target pdk to generate GDSII files.
Interactive mode requires executing steps in order because of generated files function as dependencies for the next stage.
Follow installation steps to have functional setup.

[Fossi Dial Up](https://www.youtube.com/playlist?list=PLUg3wIOWD8yoZCg9XpFSgEgljx6MSdm9L)
##### Steps to characterize synthesis results
![alt text](img/day1/results-and-reports.png)
![alt text](img/day1/picorv32-v.png)
![alt text](img/day1/stat-rpt.png)
## Day 2 - Good floorplan vs bad floorplan and introduction to library cells
### Theory
#### Chip Floor planning considerations 
##### Utilization factor and aspect ratio
##### Concept of pre-placed cells
##### De-coupling capacitors
Learned about decoupling capacitors that help reduce reliance on chip power supply.
##### Power planning
Ground Bounce and Voltage Droop can exceed noise margin due to capacitors discharing for connections through variable bit buses. 
The solution is to provide multiple power supplies as a mesh grid for power rails.
##### Pin placement and logical cell placement blockage
Combine common pins for complete design similar to HDL languages (netlist).
Place pins on region between core and die which should be closest to their respective targets.
Logical cell placement blockage ensures proper placement and floor plan becomes ready for placement and routing.
### Lab
#### Chip Floor planning considerations 
Openlane commands
```
run_floorplan
```
Terminal commands
```
cd Openlane/configuration
vim README.md
vim floorplan.tcl
cd OpenLane/designs/picorv32a
cd OpenLane/designs/picorv32a/runs/<latest>
cd logs/floorplan
vim <4-io.log>
cd ../../
vim config.tcl
cd results/floorplan
vim .def
magic -T </path/to/tech/file> lef read ../../tmp/merged.lef def read </def/file> &
```
Magic commands - drawing
```
<Press s key while hovering over object>
<Press v key>
<Right click bottom left, Right click again for top right>
<Control+z zoom to box>
```
Magic commands - tcl console
```
what
```
##### Steps to run floorplan using OpenLANE
Different variables in configuration readme for different steps in workflow.
Tcl files has default configurations.
Config tcl files have priority from configuration to design folder to prepended sky130 files.
Potential conflicts with newer json files.
##### Review floorplan files and steps to view floorplan
Die area in .def file in results/floorplan set by distance variable
![alt text](img/day2/magic-command.png)
![alt text](img/day2/magic-gui.png)
##### Review floorplan layout in Magic
![alt text](img/day2/pin-what.png)
### Theory
#### Library Binding and Placement
##### Netlist binding and initial place design
1. Bind netlist with physical cells
Each block given proper width and height and can be customized in the library.
2. Placement
Must be placed near required pins for better timing.
##### Optimize placement using estimated wire-length and capacitance
This stage will estimate wire length and capacitance to incorporate repeaters to retain signal integrity
##### Final placement optimization
Wire placement can typically criss-cross each other and resolved using separate metal layers. 
##### Need for libraries and characterization
Logic synthesis, floorplanning, placement, clock timing synthesis (cts), routing, static timing analysis implemened with gates and cells which represent library.
### Lab
Openlane commands
```
run_placement
```
Linux terminal
```
cd results/placement
```
##### Congestion aware placement using RePlAce
![alt text](img/day2/magic-placement-command.png)
![alt text](img/day2/magic-placement-gui.png)
### Theory
#### Cell design and characterization flows
##### Inputs for cell design flow
Standard cells belong to libraries with different sizes and functionality as well as different threshold voltages. 
Cell design flow include inputs, design steps, and outputs.
Inputs are defined by PDKs, DRC/LVS, spice models, and library- and user-defined specs. 
##### Circuit design step
Specs cover things like cell height, supply voltage, metal layers, pin locations, and drawn gate length.
Design steps include circuit design, layout design, and characterization.
NMOS/PMOS network graph is a potential tool for transoforming circuit design to layout design.
Outputs include CDL file or circuit description language.
##### Layout design step
Art of layout - Euler's path and stick diagram
Output stage also includes GDSII, LEF, and extracted spice list (.cir)
GDSII is layout file, LEF is width and height of cells, and (.cir) includes parasitic capacitances. 
After these steps, characterization is done which will provide timing, noise, power .libs, function
##### Typical characterization flow
1. Read in models of NMOS/PMOS and tech files
2. Read extracted spice netlist
3. Define behavior of buffer
4. Read subcircuits 
5. Attach power supplies and gnd 
6. Apply stimuli
7. Provide necessary output capacitances
8. Provide necessary simulation command
Feed in these steps to GUNA software with a configuration file which generates resulting model for timing, power, and noise characterization.
#### General timing characterization parameters
##### Timing threshold definitions
There are different timing threshold definitions.
slew_low_rise_thr is about 20% and similarly for slew_high_rise_thr (as well as falling edge)
Take 50% of in_rise_thr and similarly for output. (as well as falling edge)
##### Propagation delay and transition time
Propagation delay is equal to the differenc between output and input thresholds for respective measurements.
Delay must be positive. Long wires can increase slew and produce negative delay.
## Day 3 - Design library cell using Magic Layout and ngspice characterization
### Lab
#### Labs for CMOS inverter ngspice simulations
##### IO placer revision
##### SPICE deck creation for CMOS inverter
##### SPICE simulation lab for CMOS inverter
##### Switching Threshold Vm
##### Static and dynamic simulation of CMOS inverter
##### Lab steps to git clone vsdstdcelldesign
#### Inception of Layout and CMOS fabrication process
##### Create Active regions
##### Formation of N-well and P-well
##### Formation of gate terminal
##### Lightly doped drain (LDD) formation
##### Source and drain formation
##### Local interconnect formation
##### Higher level metal formation
##### Lab introduction to Sky130 basic layers layout and LEF using inverter
##### Lab steps to create std cell layout and extract spice netlist
![alt text](img/day3/git-repo-and-command.png)
![alt text](img/day3/not-equal.png)
![alt text](img/day3/ngspice-plot.png)
![alt text](img/day3/magic-inverter.png)
![alt text](img/day3/spice.png)
#### Sky130 Tech File Labs
##### Lab steps to create final SPICE deck using Sky130 tech
Connect stimuli for simulation. Change scaling dimension
Magic commands
```
box // to see the proper dimensions for spice deck
```
![alt text](img/day3/modified-spice.png)
##### Lab steps to characterize inverter using sky130 model files
Ngspice commands
```
plot y vs time a
<increase C3 to 2fF and rerun>
```
![alt text](img/day3/20-80-values.png)
![alt text](img/day3/rise-time.png)
![alt text](img/day3/rise-delay-points.png)
![alt text](img/day3/rise-delay-value.png)
##### Lab introduction to Magic tool options and DRC rules
[Magic](http://opencircuitdesign.com/magic/index.html)
Tutorials and additional commands can be found above.
Technology files contain important information relating to the layout.
Reading Magic documentation is essential to utilizing every aspect of this tool.
##### Lab introduction to Sky130 pdk's and steps to download labs
[open_pdks](http://opencircuitdesign.com/open_pdks/)
Contains information about using Skywater's PDK with open source tools like magic such as an archive of drc tests for reference.
Look at Skywater's PDK documentation for further information about DRC rules.
Potential modification of tech file to change magic behavior.
##### Lab introduction to Magic and steps to load Sky130 tech-rules
Errors found in magic can be explained in the Skywater PDK documentation.
Consult physical design workshop repo for additional work using these EDA tools.
##### Lab exercise to fix poly.9 error in Sky130 tech-file
Changing tech file allows correct DRC rules to be displayed. Additional commands needed to populate modifications and for DRC engine to recognize errors.
##### Lab exercise to implement poly resistor spacing to diff and tap
Cross-referencing rules in tech file and Skywater documentation necessary for correct layout with no DRC violations.
##### Lab challenge exercise to describe DRC error as geometrical construct
##### Lab challenge to find missing or incorrect rules and fix them
Create a rule utilizing cif that finds errors by subtracting layers instead of implementing as an edge-based rule. Save tech file and load in magic as well as made only when drc full is run. Caution: some layers are not yet implemented in magic or are automatically generated.
## Day 4
#### Timing modelling using delay tables
##### Lab steps to convert grid info to track info
*Linux terminal commands:*
```
magic -d XR -T sky130A.tech sky130_inv.mag &
vim /path/to/pdk/sky130A/libs.tech/openlane/sky130_fd_sc_hd/tracks.info
```
*Magic commands*
```
<press g for grid-on>
help grid
grid 0.46um 0.34um 0.23um 0.17um
```
LEF file contains all the relevant information like pins and ports. Intersection of x and y axis indicates ports can be reached with routing tools.
![alt text](img/day4/tracks-info.png)
![alt text](img/day4/magic-grid.png)
##### Lab steps to convert magic layout to std cell LEF
*Magic commands*
```
edit -> text for converting label to port
port class <output/input/inout>
port use <signal/power/ground>
save sky130_vsdinv.mag
lef write
vim .lef
```
*Linux terminal*
```
magic -d XR -T sky130A.tech sky130_vsdinv.mag &
```
Standard cell required to be odd multiple of base quantity. Must convert required labels into ports.
![alt text](img/day4/label-text.png)
![alt text](img/day4/lef.png)
##### Introduction to timing libs and steps to include new cell in synthesis
*Linux terminal*
```
cd Openlane/designs/picorv32a/src/
cp vsdstdcelldesign/.lef ./
vim vsdstdcelldesign/libs/sky130_fd_sc_hd__typical.lib
cp vsdstdcelldesign/libs/sky130_fd_sc_hd__ ./
```
![alt text](img/day4/terminal.png)
*Openlane commands*
```
cd Openlane
make mount
./flow.tcl -interactive
package require openlane 0.9
prep -design picorv32a -tag <date> -overwrite
set lefs [glob $::env(DESIGN_DIR)/src/.lef]
add_lefs -src $lefs
run_synthesis
```
Must change lib files in config.tcl
![alt text](img/day4/success.png)
##### Introduction to delay tables
For expected behavior, a designed must verify the timing constraints are met based on load capacitances.
##### Delay table usage Part 1 
Delay tables are based on input slew and output load. Depends of CMOS sizing.
##### Delay table usage Part 2 
Total delay is calculated by summing up each level. Skew = 0 if identical and driving the same loads.
##### Lab steps to configure synthesis settings to fix slack and include vsdinv
*Linux terminal*
```
cd OpenLane/configuration
vim README.md
cd results/placement
magic -d XR -T /usr/local/share/pdk/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.nom.lef def read picorv32.def &
```
*OpenLane commands*
```
set ::env(SYNTH_STRATEGY) 1
echo $::env(SYNTH_STRATEGY)
echo $::env(SYNTH_BUFFERING)
echo $::env(SYNTH_SIZING)
set ::env(SYNTH_SIZING) 1
echo $::env(SYNTH_DRIVING_CELL)
run_synthesis
run_floorplan
run_placement
```
Chip area for module '\picorv32': 99853.267200
Look at configuration variables in configuration/README.md
Log files keep track of your changes.
![alt text](img/day4/merged-lef.png)
![alt text](img/day4/placement.png)
![alt text](img/day4/vsd-inv.png)
![alt text](img/day4/expanded.png)
#### Timing analysis with ideal clocks using openSTA 
##### Setup timing analysis and introduction to flip-flop setup time
From launch flop to capture flop is the max time delay between rising clock edge
Setup and hold times result from the propagation delay of transistors.
##### Introduction to clock jitter and uncertainty
Clock jitter results from variations in the production of the clock voltage perhaps from the PLL circuit.
This decreases the max time delay between rising clock edges for setup analysis.
##### Lab steps to configure OpenSTA for post-synth timing analysis
*Linux terminal*
```
cd vsdstdcelldesign/extras
vim sta.conf
vim my_base.sdc
cd OpenLane/scripts/
vim base.sdc
sta pre_sta.conf
```
![alt text](img/day4/pre-sta.png)
##### Lab steps to optimize synthesis to reduce setup violations
*Openlane commands*
```
set ::env(SYNTH_MAX_FANOUT) 4
run_synthesis
report_net -connections <netlist>
help replace_cell
replace_cell <first> <second>
report_checks -fields {net cap slew input_pins} -digits 4
```
Bad slew can propagate when chained.
##### Lab steps to do basic timing ECO
*Openlane commands*
```
report_checks -from <first> -to <second> -through
report_tns
report_wns
```
#### Clock tree synthesis TritonCTS and signal integrity
##### Clock tree routing and buffering using H-Tree algorithm
Producing 0 skew requires making distance signal travels equal.
Buffers help reduce the charge time of the load capacitances when placed equally.
##### Crosstalk and clock net shielding
Clock net shielding helps preserve the clock signal integrity from when wires are coupled and crosstalk
##### Lab steps to run CTS using TritonCTS
*Openlane commands*
```
help write verilog
write verilog /path/to/results/synthesis/picorv32a-synthesis.v
run_floorplan
run_placement
run_ctc
```
##### Lab steps to verify CTS runs
Linux terminal
```
cd OpenLane/scripts
vim cts.tcl
cd openroad
vim OpenLane/designs/picorv32a/src/typical.lib
```
Openlane commands
```
echo $::env(LIB_SYNTH_COMPLETE)
echo $::env(LIB_TYPICAL)
echo $::env(CURRENT_DEF)
echo $::env(SYNTH_MAX_TRAN)
echo $::env(CTS_MAX_CAP)
echo $::env(CTS_CLK_BUFFER_LIST)
echo $::env(CTS_ROOT_BUFFER)
```
#### Timing analysis with real clocks using openSTA 
##### Setup timing analysis using real clocks
Path through buffers to launch and capture flop is added to total delay for their respective values.
theta + delta1 > period + delta2 - setup - uncertainty_setup
Difference between these two delta is the skew
RHS is data required time. LHS is data arrival time. RHS - LHS is slack which must be positive to be accepted.
Setup time is delay for mux1 and hold time is delay for mux2.
hold: theta + delta1 > hold + delta2
##### Hold timing analysis using real clocks
hold: theta + delta1 > hold + delta2 + uncertainty_hold
RHS is data required time. LHS is data arrival time. LHS - RHS is slack which must be positive to be accepted.
Different loads attributed to potentially different buffers and wire capacitances.
##### Lab steps to analyze timing with real clocks using OpenSTA
type openroad into OpenLane
```
read_lef /openlane/designs/picorv32a/runs/RUN_2023.01.30_10.47.53/tmp/merged.nom.lef
read_def /openlane/designs/picorv32a/runs/RUN_2023.01.30_10.47.53/results/cts/picorv32.def
write_db pico_cts.db
read_db pico_cts.db
read_verilog /openlane/designs/picorv32a/runs/RUN_2023.01.30_10.47.53/results/synthesis/picorv32.v
read_liberty -max $::env(LIB_FASTEST)
read_liberty -min $::env(LIB_SLOWEST)
read_sdc /openlane/designs/picorv32a/src/my_base.sdc
set_propagated_clock [all_clocks]
report_checks -path_delay min_max -format full_clock_expanded -digits 4
```
##### Lab steps to execute OpenSTA with right timing libraries and CTS assignment
exit openroad and re-enter openroad
```
read_db pico_cts.db
read_verilog /openlane/designs/picorv32a/runs/RUN_2023.01.30_10.47.53/results/synthesis/picorv32.v
read_liberty $::env(LIB_SYNTH_COMPLETE)
link_design picorv32
read_sdc /openlane/designs/picorv32a/src/my_base.sdc
set_propagated_clock [all_clocks]
report_checks -path_delay min_max -format full_clock_expanded -digits 4
echo $::env(CTS_CLOCK_BUFFER_LIST)
lreplace $::env(CTS_CLOCK_BUFFER_LIST) 0 0
set ::env(CTS_CLOCK_BUFFER_LIST) [lreplace $::env(CTS_CLOCK_BUFFER_LIST) 0 0]
```
##### Lab steps to observe impact of bigger CTS buffers on setup and hold timing
```
set ::env(CURRENT_DEF) /openlane/designs/picorv32a/runs/RUN_2023.01.30_10.47.53/results/placement/picorv32.def
run_cts
openroad
read_lef /openlane/designs/picorv32a/runs/RUN_2023.01.30_10.47.53/tmp/merged.nom.lef
read_def /openlane/designs/picorv32a/runs/RUN_2023.01.30_10.47.53/results/cts/picorv32.def
write_db pico_cts1.db
read_db pico_cts1.db
read_verilog /openlane/designs/picorv32a/runs/RUN_2023.01.30_10.47.53/results/synthesis/picorv32.v
read_liberty $::env(LIB_SYNTH_COMPLETE)
link_design picorv32
read_sdc /openlane/designs/picorv32a/src/my_base.sdc
set_propagated_clock [all_clocks]
report_checks -path_delay min_max -format full_clock_expanded -digits 4
report_clock_skew -hold
report_clock_skew -setup
set ::env(CTS_CLOCK_BUFFER_LIST) [linsert $::env(CTS_CLOCK_BUFFER_LIST) <index1> <index2>]
```
## Day 5 - Final steps for RTL2GDS using tritonRoute and openSTA
#### Routing and design rule check (DRC)
##### Introduction to Maze Routing - Lee's algorithm
Connects source and target. 4-connected neighbors are incremented.
##### Lee's Algorithm conclusion
Physical elements will prevent some paths from being taken. Less bends are better
##### Design Rule Check 
There is a minimum wire width due to resolution of masks and correct wire pitch will reduce drc errors.
Proper wire spacing ensures that there are no short circuits.
Using more metal layers solves these potential drc errors.
There are also minimum via width based on wire width rules as well as via spacing to ensure correct results
#### Power Distribution Network and routing
##### Lab steps to build power distribution network
*Openlane commands*
```
gen_pdn
```
Standard cells must be multiples of 2.72 for power rails.
##### Lab steps from power straps to std cell power
Straps help provide power and ground for standard and macro cells.
##### Basics of global and detail routing and configure TritonRoute
```
run_routing
echo $::env(ROUTING_STRATEGY)
```
Routing has a fast route and a detail route
#### TritonRoute Features
##### TritonRoute feature 1 - Honors pre-processed route guides
Performs mixed integer linear programming to obtain feasible solution to routing
Preprocessing involves initial route, splitting, merging, bridging, and preprocessed guides.
Metal1 is typically vertical and Metal2 is horizontal as well as proper unit width
Two guides are connected if same layer and touching or vertically connected when overlapping.
Unconnected terminals should be overlapped with routing guides
##### TritonRoute Feature2 & 3 - Inter-guide connectivity and intra- & inter-layer routing
Each layer should have parallel routing
##### TritonRoute method to handle connectivity
Inputs include lef, def, and preprocessed route guides
Output will be detailed routing solutipn with optimized wire-length and via count
Constraints are route guide honoring, connectivity constraints and design rules
Access points and Access point clusters enable proper connectivity found at the intersections of lower and upper layers
##### Routing topology algorithm and final files list post-route
```
Algorithm 1 Optimization of Routing Topology
for all i = 1 to n-1 do
    for all j = i+1 to n do
        cost_i_j <- dist(APC_i, APC_j)
    end for
end for
T <- MST(APCs, COSTs)
Return e_i_j in set of T
```
Any violations can be found in reports/routing/tritonRoute.drc or in log files
Check tmp/routing for .guide files
Use provide SPEF_EXTRACTOR python utility to extract parasitic capacitances
python3 main.py <lef file> <def file>
Creates a spef file
![alt text](img/day5/routing.png)
