.. _command-line:

**************************************
Running Analyzer from the Command-Line
**************************************

Profile Analyzer can be run from the command line. This enables batch processing of logs, which is a convenient way to upload multiple logs for automatic processing. Note that it is not possible to analyze separate frames in this way.

When using batch processing, uploading and processing each log takes some time. It can range from several seconds to a couple of hours, depending on the size and detail of each log. If the specified stdout file is empty, wait a moment for contents to appear.

To open the Profile_analyzer in batch mode, run the following command.

::

    profile_analyzer [-c=\<path_to_config\>] [-b] [\<logs_to_open\>...]

* **-c=\<path_to_config\>** — define the path to a configuration file (analyzer_config.json in the current directory by default)

* **-bf** — launch subsystem analysis in batch mode (to receive results, you need to redirect stdout (> file_name)) (format csv: filename,avg_fps,min_fps[1],...,min_fps[N-1])

* **-bm** — launch memory usage analysis in batch mode (to receive results, you need to redirect stdout (> file_name))

* **\<logs_to_open\>** — the list of logs that will be opened at launch (in the GUI mode) or processed (in the batch mode)


Below is an example of a working .bat file that uses profile_analyzer:

::

    @echo off
    
    :: Syntax: test <replay_path> <log_path_without_extension>
    :: This will run replay 10 times and generate logs (log_path + _N.upl)
    :: After that analyzer will generate report (log_path + .txt)
    
    echo Playing replay %1:
    setlocal
    set UNIPROF_ENABLE=t
    set UNIPROF_AUTOSTART=t
    for /L %%i in (1,1,10) do call :run %1 %2_%%i.upl
    ..\game64\profile_analyzer.exe -c=..\game64\analyzer_config.json -bf %NSET% > %2.txt
    echo.
    echo Results (see also %2.txt):
    type %2.txt
    endlocal
    goto :eof
    
    :run
    echo - writing log %2...
    set UNIPROF_LOGNAME=%2
    wot_h %1
    set NSET=%NSET% %2
    goto :eof

An example of the resulting report file in -bf mode (subsystem analysis) is below:

::
    
    PC8_true_defense2.upl: average = 14.707ms (67.997 fps) (10287 frames)
        fps minimums = 2.640, 39.817, 47.591
 
    Total: average = 14.707ms (67.997 fps), std. dev = -1.#IO% (1 logs)
    Unassigned: average = 1.234ms, std. dev = -1.#IO%
    Shadows: average = 1.679ms, std. dev = -1.#IO%
    Scene: average = 3.222ms, std. dev = -1.#IO%
    Water: average = 0.427ms, std. dev = -1.#IO%
    Flora: average = 1.090ms, std. dev = -1.#IO%
    GUI: average = 1.209ms, std. dev = -1.#IO%
    Particles: average = 1.137ms, std. dev = -1.#IO%
    GPU Sync: average = 0.554ms, std. dev = -1.#IO%
    Scripts: average = 2.418ms, std. dev = -1.#IO%
    Sound: average = 0.551ms, std. dev = -1.#IO%
    Post-fx: average = 0.943ms, std. dev = -1.#IO%
    Decals: average = 0.241ms, std. dev = -1.#IO%

In the example above:

* The average tpf (in ms) and fps (equal to GUI mode at smoothing=0) values are indicated after the log name.
* The "fps minimums" line contains values of the minimum FPS recorded. The three values correspond to the smoothing values =0 sec, =2 sec, and =5 sec.
* The remainder of the file shows the names of subsystems and average times of operation for each of them (in ms).