*****************
Profiler Settings
*****************

By default, the profiler is not compiled in consumer builds. It is set by the ENABLE_UNIPROF and FORCE_ENABLE_UNIPROF defines in ``cstdmf/config.hpp``. 

Moreover, there's a notion of **level** for most of the profiler events. The level limit is set during compilation. Lower-level events will not be written to the log: this avoids potential overheads that could occur if logging low-level events caused the compiler to generate excessive amounts of logging code. This way, the amount of logged information and profiling overhead can be varied for different tasks. Fine adjustment of profiler compilation time is performed in ``profiler/runtime/config.hpp``.

To view more detailed performance and memory usage logs, launch ``Wot_h_detail.exe`` (located in the same directory as the ``WorldOfTanks.exe`` file). Make sure that ``Wot_h_detail.exe`` is up to date, as it is prepared separately from the main .exe file.

.. _environment-variables:

Environment variables
=====================

The profiler's activities can be partially set through environment variables. To set these variables:

#. Open the **Control Panel**, and in the Large Icons view, click **System**.
#. In the System window's navigation menu, click **Advanced System Settings**.
#. Click the **Advanced** tab, and then click **Environment Variables**.
#. In the System Variables section, click **New** and add the following variables:

=======================   ===============================   =======================
Variable name             Variable values                   Description
=======================   ===============================   =======================
UNIPROF_ENABLE            true, false                       Indicates whether profiling is enabled. Disabled by default.
UNIPROF_MEMORY_ENABLE     true, false                       Indicates whether memory profiling is enabled. Disabled by default.
UNIPROF_LOGNAME           (location and name of log file)   Contains the name of the file to which the log will be written. If it ends with a slash (/), the name of the file will be generated automatically. For example a value of "profile_date/" would result in a log file called profile_date/[current time].upl). If a directory is not specified, the file will be created in the current directory.
UNIPROF_AUTOSTART         yes, no                           Defines whether main thread profiling begins once the application starts. Yes by default.
UNIPROF_INCLUDE_WORKERS   yes, no                           Defines whether secondary threads are profiled. Yes by default.
UNIPROF_INCLUDE_GPU       yes, no                           Defines whether the GPU will be profiled. Yes by default.
=======================   ===============================   =======================

Main thread profiling can be enabled or disabled during a profiling session by pressing ``F12``.

**Note:** The captured logs can become large, especially with detailed logging (one minute equals about 1.5 GB), so it is important to make sure there is enough space on the disk before running the profiler.