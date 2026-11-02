

## Sky130 CMOS Inverter Design Using Xschem and Magic

A hands-on custom IC design project implementing a CMOS inverter using the **SkyWater SKY130A process design kit (PDK)**. The project documents the flow from schematic capture in Xschem to physical layout and verification in Magic, with SPICE netlisting and Netgen LVS as part of the intended workflow.



## Project objectives

- Create a CMOS inverter schematic using Sky130A NMOS and PMOS devices.
- Create and use a reusable inverter symbol.
- Build a testbench to exercise the inverter.
- Generate a SPICE netlist from Xschem.
- Create the physical layout in Magic using Sky130A technology.
- Extract a SPICE netlist from the physical layout.
- Check design-rule compliance (DRC).
- Compare the schematic and extracted layout netlists using Netgen LVS.

## Tools and technology

The project was developed in a Linux(Ubuntu) environment under WSL2.

| Tool / technology | Role | Environment observed |
|---|---|---|
| Ubuntu 22.04 on WSL2 | Linux development environment | Present |
| Xschem | Schematic capture and SPICE netlisting | Version 2.8.1 |
| Magic | Physical layout, DRC and extraction | Version 8.3.689 |
| Netgen | Layout-versus-schematic comparison | Version 1.5.326 |
| SkyWater SKY130A PDK | Device definitions, technology and design rules | Configured and loaded in Magic |
| ngspice | SPICE simulation | Availability and model setup not yet verified |

Tool versions describe the development environment; they are not installation instructions. The PDK is an external dependency and is not included in this repository.

## Design overview

A CMOS inverter uses one PMOS transistor as the pull-up device and one NMOS transistor as the pull-down device. Their gates share the input, their drains form the output, and their sources connect to the corresponding supply rails.

| Device terminal | Intended connection |
|---|---|
| PMOS gate | VIN |
| PMOS source | VDD |
| PMOS bulk | VDD |
| PMOS drain | VOUT |
| NMOS gate | VIN |
| NMOS source | VSS |
| NMOS bulk | VSS |
| NMOS drain | VOUT |

The intended top-level pins are **VDD**, **VIN**, **VSS**, and **VOUT**.

## Project structure

```text
sky130-cmos-inverter/
├── README.md
├── xschem/
│   ├── inverter.sch
│   ├── inverter.sym
│   └── testbench.sch
├── mag/
│   ├── inverter.mag
│   ├── inverter.ext
│   └── inverter.spice
├── simulation/
│   ├── testbench.spice
│   └── results/
└── docs/
    └── design-notes.md
```

## Design flow and current progress

### 1. Environment and tool availability

The work environment is Ubuntu 22.04 under WSL2. Xschem, Magic, and Netgen were installed and their versions were recorded. Magic successfully loaded the `sky130A` technology configuration.
#### 1. Verify the project directories
Open Ubuntu terminal and execute:
```bash 
whoami
pwd
cd ~/inverter18
ls -la
tree -L 2
```
If tree isn't installed: `sudo apt install tree`.
 #### 2. Configure the Sky130A PDK
 ``` bash
export PDK_ROOT=/home/desai2002/magic/netgen/xschem_git/release/open_pdks/sky130
export PDK=sky130A
export PDKPATH=$PDK_ROOT/$PDK
export XSCHEM_LIBRARY_PATH=$PDKPATH/libs.tech/xschem

echo $PDK_ROOT
echo $PDK
echo $XSCHEM_LIBRARY_PATH
```

#### 3. Verify the installed EDA tools
```bash
which xschem
xschem --version

which magic
magic --version

which ngspice
ngspice --version

which netgen-lvs
```
![](./Screenshots/Screenshot%202026-11-02%20132655.png)
![](./Screenshots/Screenshot%202026-11-02%20134719.png)
![](./Screenshots/Screenshot%202026-11-02%20134926.png)

### 2. Schematic capture in Xschem

#### 1. Configure the Sky130A PDK

Set the PDK environment variables:

```bash
export PDK_ROOT=/home/desai2002/magic/netgen/xschem_git/release/open_pdks/sky130
export PDK=sky130A
export PDKPATH=$PDK_ROOT/$PDK
export XSCHEM_LIBRARY_PATH=$PDKPATH/libs.tech/xschem
```
#### 2 Launch Xschem
```bash
cd ~/inverter18/xschem
xschem
```
#### 3 Create the CMOS Inverter Schematic
The CMOS inverter schematic was designed in Xschem using the Sky130A PDK transistor symbols.
Launch xschem with:
```bash
cd ~/inverter18/xschem
ls -lh
xschem
```
The design consists of:
- PMOS transistor: sky130_fd_pr__pfet_01v8
- NMOS transistor: sky130_fd_pr__nfet_01v8
- Input: VIN (../devices/ipin)
- Output: VOUT
- Power supply: VDD
- Ground: VSS
Circuit connections:
1. PMOS source and bulk connected to VDD.
2. NMOS source and bulk connected to VSS.
3. Both transistor gates connected to VIN.
4. Both transistor drains connected to VOUT.
An inverter schematic has been created. It uses Sky130 primitive PMOS and NMOS devices.
![Inverter Schematic in Xschem](./Screenshots/Screenshot%202026-10-28%20134722.png)

#### 4 Generate the SPICE Netlist
After completing the schematic, generate the SPICE netlist using Xschem's netlisting function.
The generated SPICE netlist is used for circuit simulation and subsequent Layout Versus Schematic (LVS) verification.

#### 5 Inverter symbol

A reusable symbol is part of the intended deliverables. It should expose VDD, VIN, VSS, and VOUT and be associated with the inverter schematic.
![Inverter Schematic in Xschem](./Screenshots/Screenshot%202026-11-02%20124629.png)

### 3. Testbench and simulation

A testbench should provide the supply rails and an input stimulus, then observe the output transition. Simulation results should be added only after the model library and simulator setup have been verified.

![Inverter Testbench & Simulation](./Screenshots/Screenshot%202026-10-28%20173144.png)


### 5. Physical layout in Magic

Magic was launched with the Sky130A technology configuration.
```bash
cd ~/inverter18/mag
ls -lh
magic -T $PDKPATH/libs.tech/magic/sky130A.tech
```
And and .spice was imported using 'readspice' command or 'import spice' option present in the magic. 
the layout in magic generated using the command  
```bash
readspice inverter18.spice
```
or using 'import spice' option to import the .spice file.
after importing and connecting the pins and pmos-nmos in the appropriate way, the layout looks as follows.
![Layout](./Screenshots/Screenshot%202026-10-31%20172926.png)

#### 1. Design-rule check (DRC)
DRC can be check with the following command inside the magic console:
``` bash
drc check
drc count
```
0 DRC errors were Found!.
![Inverter Testbench & Simulation](./Screenshots/Screenshot%202026-11-02%20141212.png)


#### 2. Extraction
To Extract the netlists
inside the magic consloe:
```bash
extract all
ext2spice lvs
ext2spice
```
A Magic extraction file (`inverter.ext`) and an extracted SPICE netlist (`inverter.spice`) were generated.

![Inverter Testbench & Simulation](./Screenshots/Screenshot%202026-11-01%20230923.png)



#### 3. Layout-versus-schematic check (LVS)

lvs can be performed useing the following command lines
```bash
cd ~/inverter18/mag

netgen-lvs -batch lvs \
"inverter.spice inverter" \
"inverter_sch.spice inverter" \
"$PDKPATH/libs.tech/netgen/sky130A_setup.tcl" \
lvs_final.log
```
![Inverter Testbench & Simulation](./Screenshots/Screenshot%202026-11-01%20230727.png)
![Inverter Testbench & Simulation](./Screenshots/Screenshot%202026-11-01%20230747.png)

### 6 Post-Layout Simulations:
#### 1. Extract parasitics in Magic
Launch Magic with your Sky130A technology file:
```bash 
export PDK_ROOT=/home/desai2002/magic/netgen/xschem_git/release/open_pdks/sky130
export PDK=sky130A
export PDKPATH=$PDK_ROOT/$PDK

cd ~/inverter18/mag

magic -T "$PDKPATH/libs.tech/magic/sky130A.tech" inverter.mag
```

At the Magic command prompt, execute:
```
extract all
extresist tolerance 10
extresist
ext2spice lvs
ext2spice cthresh 0
ext2spice extresist on
ext2spice
```
What these commands do:
- extract all — extracts layout connectivity and geometry.
- extresist — extracts interconnect resistance.
- ext2spice cthresh 0 — retains extracted parasitic capacitances, including small ones.
- ext2spice extresist on — includes extracted resistance in the SPICE output.
- ext2spice — generates the extracted SPICE netlist.
These are Magic's parasitic extraction controls.
![Inverter Testbench & Simulation](./Screenshots/Screenshot%202026-11-02%20154016.png)
![Inverter Testbench & Simulation](./Screenshots/Screenshot%202026-11-02%20154046.png)
![Inverter Testbench & Simulation](./Screenshots/Screenshot%202026-11-02%20154059.png)

#### 2. Protect the parasitic netlist
ext2spice overwrites inverter.spice each time, and that file now has parasitics. Copy it, and keep the LVS netlist separate in future:

```bash
cd ~/inverter18/mag
cp inverter.spice inverter_pex.spice
```
For the next LVS, regenerate a plain netlist in Magic first (ext2spice cthresh infinite, ext2spice extresist off, ext2spice lvs, ext2spice) so the R/C elements don’t end up in the LVS netlist.

#### 3. Post-layout testbench

The extracted subcircuit is .subckt inverter VDD VOUT VIN VSS, so instantiate the pins in that order.

```bash
cat > tb_post.spice << 'EOF'
* post-layout inverter testbench
.lib /home/desai2002/magic/netgen/xschem_git/release/open_pdks/sky130/sky130A/libs.tech/ngspice/sky130.lib.spice tt
.include /home/desai2002/inverter18/mag/inverter_pex.spice

VDD vdd 0 1.8
VIN vin 0 PULSE(0 1.8 1n 100p 100p 5n 10n)
XINV vdd vout vin 0 inverter
CL vout 0 10f

.temp 27
.control
tran 10p 30n
meas tran tpHL trig v(vin) val=0.9 rise=1 targ v(vout) val=0.9 fall=1
meas tran tpLH trig v(vin) val=0.9 fall=1 targ v(vout) val=0.9 rise=1
meas tran trise trig v(vout) val=0.18 rise=1 targ v(vout) val=1.62 rise=1
meas tran tfall trig v(vout) val=1.62 fall=1 targ v(vout) val=0.18 fall=1
plot v(vin) v(vout)
.endc
.end
EOF
ngspice tb_post.spice
```
Here VSS is node 0 (ground). The meas lines print propagation delays and 10-90% rise and fall times in the console. The plot opens through WSLg, as Magic’s windows do.
![Inverter Testbench & Simulation](./Screenshots/Screenshot%202026-11-02%20161727.png)
![Inverter Testbench & Simulation](./Screenshots/Screenshot%202026-11-02%20161743.png)
![Inverter Testbench & Simulation](./Screenshots/Screenshot%202026-11-02%20161759.png)

![Inverter Testbench & Simulation](./Screenshots/Screenshot%202026-11-02%20161759.png)

#### 5. Post-layout Simulation
![Inverter Testbench & Simulation](./Screenshots/Screenshot%202026-11-02%20164635.png)

##### The waveform

- vout is the inverse of vin in all three cycles (edges at 1, 11 and 21 ns, with the falling edges 5 ns later), and it settles cleanly at 0 V and 1.8 V.
- The small spikes at each edge are normal. vout pokes slightly above 1.8 V when the input rises and slightly below 0 V when it falls. That is gate-to-drain (Miller) coupling from the fast 100 ps input edge, and it disappears with a slower input or a heavier load.

The measurements

Measurement|Value
tpHL (output falling)|50 ps
tpLH (output rising)|94 ps
fall time (10–90%)|52 ps
rise time (10–90%)|141 ps

The rising edge is slower than the falling edge, as I mentioned earlier. Your pfet and nfet are both W = 1 µm, and a pfet of the same width drives less current. A rise-to-fall ratio near 2.7 is reasonable for sky130. The trigger times (1.05 ns and 6.15 ns) land exactly on the 0.9 V crossing of the input ramps, so the meas lines are set up correctly.

## Acknowledgements

- [SkyWater Open Source PDK](https://github.com/google/skywater-pdk)
- [Open_PDKs](https://github.com/RTimothyEdwards/open_pdks)
- [Xschem](https://github.com/StefanSchippers/xschem)
- [Magic VLSI](http://opencircuitdesign.com/magic/)
- [Netgen](http://opencircuitdesign.com/netgen/)

## License

