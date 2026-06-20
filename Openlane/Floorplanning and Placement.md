First step in PD is establishing boundaries:
	Die area vs Core area
		- Die area = absolute outer boundary of the physical silicon slice
		- Core area = sits inside die area and is the zone where the standard cells are placed
		- The buffer area between the die area and core area is usually for I/O pads and rings
	Core utilization (`PL_TARGET_DENSITY`) determines how densely packed the standard cells will be
		Ex. a target density of 0.6 means 60% of the core area will be filled with cells, and 40% will be empty space
			- If you set utilization very high (>0.8-0.9) your chip will be very small but routing will likely fail because there isn't enough physical space between cells to run the interconnecting metal wires
Power distribution network (PDN) generation:
	No robust PDN = severe IR drops (voltage drops due to wire resistance) which can cause transistors to run slowly/fail entirely