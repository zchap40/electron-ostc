Acorn Electron Open Source Turbo Card
-------------------------------------

The OSTC is a 'turbo card' that speeds up the Electron by partially mitigating 
the slow memory access that usually limits the 6502 CPU's performance. It acts 
similarly to ye olde turbo cards of days gone by, like the Slogger Turbo Driver, 
but is implemented using modern programmable logic and SRAM.


How it works
------------

Electrons are slow because of the ULA stealing RAM cycles to update the
display. When reading from the ROM the CPU works at 2MHz, but when accessing
RAM it runs at 1MHz in modes 4-6. In modes 0-3 the ULA stops the CPU clock
when drawing the display area, effectively throttling the 6502 down to 
around 0.5MHz.

Turbo cards are based on an interesting quirk; the ULA only needs to access
the upper 20K of system RAM for display purposes. The lower 12K is contended
between the ULA and CPU purely to simplify the Electron's design.

The trick is to stop the ULA from contending the lower 12K. We do this by
adding some new memory dedicated to the CPU, in this case a 32K SRAM chip.
All reads and writes to the 0-12K area are redirected to the SRAM chip - old
turbo cards handled the first 8K only to simplify the required logic, but here
we do all 12K. The 6502's databus is passed through the CPLD chip, which sets
it to high impedance when the SRAM is being accessed, effectively preventing
the Elk's on-board DRAM from causing bus contention.

That's great, but we're still not going any faster. The ULA is still throttling
the CPU clock, even when we're now accessing that lovely fast SRAM chip and not
the grotty contended motherboard DRAM.

The solution is to run the CPU's A15, A14 and RW lines through the CPLD. When
the CPU accesses the SRAM chip the CPLD pulls all of those lines high on the
ULA side, fooling it into thinking we are reading from the ROM - the the ULA
duly accelerates the CPU to 2MHz. The actual data read from the ROM never
reaches the CPU as the CPLD acts as a bus isolator at this point, cutting the CPU
and SRAM off from the motherboard.

One major difference between the older turbos and the OSTC is how 
disabling the board is handled. Other boards required a reboot after toggling
the board state. 

The OSTC never actually disables, the switch just returns the CPU to
normal speed. R/W operations are still processed by the SRAM chip, not DRAM.

This has the big advantage that SRAM contents is always up to date and turbo
speed can be toggled without a reboot. Downside is the very few games that try
to locate the display < 12K won't run.


Versions
--------

The OSTC was originally designed and developed by G. Colville (ramtop-retro),
and is available here as OSTC_v1.

I developed a more compact version of the PCB, with an almost identical circuit,
filed here as OSTC_v2. The Electron is not just an elegant looking machine from
the exterior, but in my opinion, is also of elegant design on the inside too.
I wanted a version of the OSTC that matched the aesthetic of the Electron's motherboard,
and that's what I hope OSTC_v2 delivers.

The turbo switching functionality of the original OSTC circuit is an emulated switch,
and does not allow the turbo card to be physically switched out of the motherboard.
This is why I removed the header for the external switch in OSTC_v2. I felt it wasn't
useful enough to take up board space. OSTC_v3 takes the idea for the OSTC further
by providing actual hardware switching functionality. It employs high-bandwidth analog
data switches to connect the 12 re-routed data lines from the motherboard to either the
OSTC circuit, or directly to the 6502 processor. This is controlled by the on-board
three-way switch, allowing the turbo card to be set to 1) turbo speed, 2) normal speed
(emulated), and 3) native speed (turbo card physically switched out). A two-pin micro
header allows an external switch to be connected to enable the 'native'
mode, where the turbo card is completely switched out of the hardware. Closing the
external switch will always override whatever setting is selected on the on-board switch
of the turbo card.

OSTC_v3 has the exact same PCB form factor as OSTC_v2. The only disadvantage is the
higher cost of construction.


Files
-----

 - PCB 		PCB layout in Diptrace format for OSTC_v1, and Autodesk Eagle format
		for OSTC_v2 and v3
 - Gerbers	Gerber and drill files for OSTC_v1, v2 and v3
 - Schematics	PDF schematic diagrams for OSTC_v2 and v3
 - ISE		Xilinx ISE project (verilog)
 - CPLD		Programming file for the CPLD


How to build it
---------------

The 'Gerbers' directory contains Zip files for each version that you can upload to any 
PCB fabrication service to have them produce OSTC PCBs for you. In Europe, aisler.net
is recommended. Please note that version 1 of the PCB is a 2-layer board,
while versions 2 & 3 are 4-layer. The extra cost for having a 4-layer board fabricated
is marginal at most of the cheap PCB houses.

The 'PCB' directory has the original PCB layout files in Diptrace format for version 1,
and Autodesk Eagle format for versions 2 & 3. Many PCB fabrication houses (including
Aisler and OSHPark) allow you to upload the original PCB design files (.dip for Diptrace
and .brd for Eagle) instead of Zipped gerber files, and this in fact usually allows the
boards to be rendered even more accurately by the fabrication house. It is recommended
to go this route if this option is available.

**OSTC_v1**

The components necessary for OSTC_v1 are:

REF			Component 					Part No.

C1, C2, C3		150pF 1206 size ceramic capacitor		399-8157-1-ND (Digikey)
C4, C5			22uF 1206 size tantalum capacitor		478-3865-1-ND (Digikey)
VR3v3			3.3v LM1117 voltage regulator			LM1117MP-3.3/NOPBCT-ND (Digikey)
R1			2Kohm 1206 resistor				RMCF1206JT2K00CT-ND (Digikey)
R2			4Kohm 1206 resistor				RNCP1206FTD4K02CT-ND (Digikey)
JTAG, SW1, DBG		0.1" pin headers, right-angle			547-3223 (RS Components)
6502			2x Low profile 40-way turned pin socket 	197-2726 (RS Components)
ElkMB			2x 20 way round pin headers (see below)		BBL-120-T-E (Toby Electronics)
XC9500XL		Xilinx XC9536XL 44-pin VQFP			122-1385-ND (Digikey)
SRAM			IS62C256AL-45ULI 28-pin SOP			706-1043-ND (Digikey)


OSTC_v1 has been designed to be fairly easy to build if you have some surface mount
soldering experience. A few points to note, however:

- Capacitor and resistor values are not super critical, but C4/5 should be tantalums of
at least 22uF or stability may be affected.
- I strongly recommend using turned pin type sockets, these are much more reliable than
the dual-wipe type, but they absolutely require the round-pin type headers listed above
and not the cheaper, more common, square pin type.
- The JTAG header should be right-angle, with the pins located over the top of the CPLD.
- DBG is the debug header, fitting this is not required

CPU speed is controlled by pins 1 and 2 of SW1. Shorting these pins slows the CPU to stock
speed, open is turbo speed.

JTAG header pinout is :

1 3.3v
2 GND
3 TCK
4 TDO
5 TDI
6 TMS

**OSTC_v2**

The components necessary for OSTC_v2 are:

REF	QTY	Component 					Part No.

X1	1	Molex 505567-0651 6-ckt Micro-Lock socket	538-505567-0651 (Mouser)
X2	2	Samtec BBL-120-G-E 20-way 0.1" terminal strip	200-BBL120GE (Mouser)
U1	1	Samtec ICO-640-SGG (socket for 6502)		200-ICO640SGG (Mouser)
U2	1	Xilinx XC9536XL-10CSG48I 48-BGA CPLD		217-C9536XL-10CSG48I (Mouser)
U3	1	ISSI IS62C256AL-45TLI 28-TSOP 8x32k SRAM	870-IS62C256AL-45TLI (Mouser)
U4	1	Texas LP5912Q3.3DRVRQ1 6-WSON 3.3V regulator	595-LP5912Q3.3DRVRQ1 (Mouser)
S1	1	NKK SS312SAH4 SPDT slide-switch			633-SS312SAH4 (Mouser)
C1, C2	2	10uF X7R ceramic capacitor, 1206 SMD		810-CGA5L1X7R1H106K6 (Mouser)
C3, C4	2	100nF X7R ceramic capacitor, 1206 SMD		810-CGA5L2X7R2A104K (Mouser)
R1	1	4K 0.25W resistor, 0805 SMD			660-RK73H2ATTD4021F (Mouser)
R2	1	2K 0.25W resistor, 0805 SMD			660-RK73H2ATTDD2001F (Mouser)	

Cable	1	Molex 45111-0606 Micro-Lock cable		538-45111-0606 (Mouser)
		(for programming CPLD)

**OSTC_v3**

The components necessary for OSTC_v3 are:

REF	QTY	Component 					Part No.

X1	1	Molex 505567-0651 6-ckt Micro-Lock socket	538-505567-0651 (Mouser)
X2	1	Molex 505567-0251 2-ckt Micro-Lock socket	538-505567-0251 (Mouser)
X3	2	Samtec BBL-120-G-E 20-way 0.1" terminal strip	200-BBL120GE (Mouser)
U1	1	Samtec ICO-640-SGG (socket for 6502)		200-ICO640SGG (Mouser)
U2	1	Xilinx XC9536XL-10CSG48I 48-BGA CPLD		217-C9536XL-10CSG48I (Mouser)
U3	1	ISSI IS62C256AL-45TLI 28-TSOP 8x32k SRAM	870-IS62C256AL-45TLI (Mouser)
U4	1	Texas LP5912Q3.3DRVRQ1 6-WSON 3.3V regulator	595-LP5912Q3.3DRVRQ1 (Mouser)
U5, U6	4	Maxim MAX4996ETG+ 24-TQFN analog switch 3x DPDT	700-MAX4996ETG (Mouser)
U7, U8	
S1	1	NKK SS314MAH4 SP3T slide-switch			633-SS314MAH4-R (Mouser)
C1, C2	2	10uF X7R ceramic capacitor, 1206 SMD		810-CGA5L1X7R1H106K6 (Mouser)
C3, C4	2	100nF X7R ceramic capacitor, 1206 SMD		810-CGA5L2X7R2A104K (Mouser)
R1, R2	1	4K 0.25W resistor, 0805 SMD			660-RK73H2ATTD4021F (Mouser)

Cable	1	Molex 45111-0606 Micro-Lock cable		538-45111-0606 (Mouser)
		(for programming CPLD)
Cable	1	Molex 45111-0206 Micro-Lock cable		538-45111-0206 (Mouser)
		(for connecting external switch)


How to fit it
-------------

Fitting the OSTC requires the Electron's 6502 CPU to be desoldered from the motherboard. It
is strongly recommended you use a proper vacuum desoldering station to do this, or find
someone who can do it for you, as the Elk's PCB is quite fragile. It is recommended to install
the following low-profile machined-pin socket on the Electron motherboard:

Component 					Part No.
Samtec ICA-640-SGG				200-ICA640SGG (Mouser)

Once the CPU is removed it should be installed into the socket on the OSTC marked '6502' (marked 'U1'
on OSTC_v2) with the orientation notch facing away from the JTAG header. Then, solder the recommended
40-pin socket into the CPU area (IC3) on the motherboard. Insert the OSTC into the socket, pushing
firmly and making sure it is correctly aligned.

If you want to externally control the CPU speed, a switch can be connected between pins 1 and 2 of
SW1 on OSTC_v1. On OSTC_v3, a Molex Micro-Lock cable assembly can be attached to the 2-pin 
Micro-Lock header to connect an external switch to control hardware switch-in/out of the turbo card.

A Rockwell R65C02 rated 2MHz or faster can be fitted in place of the original NMOS 6502, to
reduce heat build-up. WDC branded 65C02s will not work.


License
-------
Copyright for all the files in this repository copied from ramtop-retro/ostc remains with G. Colville, but
can be freely used for non-commercial purposes.

Copyright for all other files remains with A. Sadek, and may be used similarly.