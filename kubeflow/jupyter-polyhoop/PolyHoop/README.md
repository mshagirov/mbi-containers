# PolyHoop

Authors:
Dr. Roman Vetter (vetterro@ethz.ch)
Steve V. M. Runser (steve.runser@bsse.ethz.ch)
Prof. Dr. Dagmar Iber (dagmar.iber@bsse.ethz.ch)


## Contents

* polyhoop.cpp - Complete source code in C++11 (includes model parameters)
* ensemble.off - Sample input file, a single circle with unit radius
* README.md - This readme file
* LICENSE - BSD 3-clause license file


## Compilation

PolyHoop requires a compiler compatible with the C++11 standard. For multithreading support, the OpenMP 3.1 specification or newer is additionally required.
We tested it using GCC v4.8.2 - 12.2.0, Clang/LLVM v3.6.0 - 15.0.7, and ICC v14.0.1 - 19.1.0.
To compile PolyHoop, run

g++ -fopenmp -O3 -o polyhoop polyhoop.cpp

or an equivalent command. It can be compiled for serial execution by omitting the OpenMP compile option.


## Execution

To run a simulation, set the desired parameters in polyhoop.cpp (lines 13-43), compile it, and execute the binary by typing

OMP_NUM_THREADS=8 ./polyhoop

or similar. If PolyHoop is compiled with OpenMP support and the number of threads is not specified, the selection of a suitable number of threads is delegated to OpenMP.


## Input

PolyHoop reads in a mandatory input file named ensemble.off from the current working directory, specifying the initial configuration in GeomView's Object File Format (OFF).
Since OFF specifies vertex coordinates in three dimensions x,y,z, but PolyHoop uses only two (x and y), the z coordinate is used to indicate the phase of the polygon containing the vertex.
z is expected to be an integer that, if odd, denotes phase 1, and if even, -1.
The input file only specifies vertex positions; velocities are initialized to zero.
The creation of such input files representing suitable initial starting conditions for different applications is left to the user.
The first Nr polygons in the input file are treated as rigid obstacles. Nr can be set in the source file.


## Output & Visualization

PolyHoop writes the simulated configurations at specified time intervals to the current working directory, naming them frame\%06d.vtp.
With each output frame, a single-line status report is printed to the command line.
The output files are in the VTK ASCII polygon format.
For each polygon in the ensemble, they store the area, perimeter and coordination number.
The simulation output can be visualized and animated by opening the VTK file series in ParaView (https://paraview.org).
For non-convex polygons, the *Extract Surface* and *Triangulate* filters need to be applied to ensure correct visual representation.
Similar to the input file, the phase phi of each polygon is stored in the unused z coordinate of the output VTK files, as a binary flag z=(phi+1)/2
.

## License

All source code is released under the 3-Clause BSD license (see LICENSE for details).
