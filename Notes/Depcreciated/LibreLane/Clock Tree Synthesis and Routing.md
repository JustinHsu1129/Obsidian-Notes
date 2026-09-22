Clock tree insertion with TritonCTS. Since DFFs are in different places they receive the clock signal at different times which results in skew-which can benefit/hurt timing margins
- Paths are typically balanced using an H-tree topology for timing, physical distance, and capacitive load
- Buffers are inserted to clean up clock edge, maintain rise/fall times, and boost signal strength on long lines since longer wires = more RC delay

Global routing vs Detailed routing:
	- Global routing = coarse grid superimposed over entire chip layout to carve into small zones called G-cells
		- For each G-cell the routing algorithm decides whether a wire will travel through and which metal layers are used
		- Congestion monitored to ensure routing resources aren't crammed together
	- Detailed routing = wires are placed and made sure to follow PDK design rules (min spacing between wires, min wire width, etc.)
		- Uses iterative optimization strategy