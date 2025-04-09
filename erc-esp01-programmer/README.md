# ESP01 Programmer
Everything is explained in the document "Explaining the Schematic.pdf". Other than that, an important thing is that while designing the PCB in Fusion 360, I could not figure out an option to connect the two ground planes (PGND and GND) without messing up a lot of connections. So, currently, the two grounds are floating with respect to each other (notice that in my schematic, I have not connected both of them).

Online sources said that you should only join the GND (bottom) and PGND (top) planes at one point ("star" connection) using a via. However, doing that makes Fusion 360 think that I have shorted both planes, and it connects the IC directly to the top PGND plane instead of routing it through the via to the bottom GND plane. I could not figure out a way to achieve this, so I have left the two planes disconnected (not good practice, but yeah whatever).

All of this is because I couldn't find a cheap buck-boost converter ready-made, so I made one myself :)

It was fun researching (although took a lot of effort) about this, but Fusion 360 is annoying (or maybe its my skill issue :p).