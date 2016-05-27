************************
Settings Recommendations
************************

* By default, only functions marked as Important get to the log in the hybrid build. There are relatively few of these functions, which is why the overhead stays low (the FPS rates difference between the replay with disabled profiling and profiling of the Important level should be within confidence bounds). Together with that, the time taken by all subsystems can be defined. This is enough to detect and perform the initial analysis of productivity issues.
* Information on consumed memory is collected on the Detail level. When the information is being collected, a noticeable drop in productivity (10–20%) and in volume of created logs (1 Gb/sec or higher) may occur. To improve productivity, it is advisable to create a RAM-disk and write the log there.