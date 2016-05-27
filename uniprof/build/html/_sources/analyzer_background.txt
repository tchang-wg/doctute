**********************************************************************************
Analyzer Background Mode (receiving statistics on several playbacks of one replay)
**********************************************************************************

Tests typically require result processing of several playbacks of the same replay in the automatic mode. To perform this operation, the user should run the analyzer in the background mode (see :ref:`command-line`), having indicated paths to the logs for processing. The average frametime and time of subsystem work are calculated for each log, and then these values are averaged for all logs.

To automatically replay and analyze the results, it is more convenient to use a .bat file.