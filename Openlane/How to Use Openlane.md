
# Structure of a design directory

my_project/
	src/
		RTL files here
	config.json

## Config file

Config file is written in `JSON`, and contains several variables that provide basic information about the design.

```
{ 
"DESIGN_NAME": "counter", 
"VERILOG_FILES": [ "dir::src/counter.v" ], 
"CLOCK_PORT": "clk", 
"CLOCK_PERIOD": 10.0, 
"PDK": "sky130A" 
}
```

`DESIGN_NAME` is the name of the design
`VERILOG_FILES` is the files of the RTL
	use the `dir::` prefix to tell Openlane to look in a specific folder if the RTL is not found at the root
`CLOCK_PORT` is the name of the clock pin in the top level module
`CLOCK_PERIOD` is the target clock period specified in nanoseconds (1/period = frequency)
`PDK` tells Openlane which PDK to use