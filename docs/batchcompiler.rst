*************************
Batch Compiler
*************************

.. contents::
   :local:
   :depth: 2

The Batch Compiler is a command-line front end for converting asset
source files into game-ready compiled assets. It can convert a single
source file or a folder of source files.

Configuration
=========================================

Setting Up Converters
-------------------------------------------------------

Before running the Batch Compiler, ensure that all of the converter
plugins you wish to use exist in a reachable folder (preferably in the
same folder as the Batch Compiler executable), and that they are listed
in the ``batch_compiler_plugins.txt`` file. If you are running the debug
version of the Batch Compiler (``batch_compiler_d.exe``), ensure that
you have the debug versions of the plugins you wish to load. It is safe
for both the debug and non-debug version of a converter plugin to exist
in the executable folder—only the appropriate version of the plugin will
be loaded.

Setting Up Asset Rules
-------------------------------------------------------

Batch Compiler uses asset rules to determine what files on disk are
assets, and how to convert them. For information on setting up asset
rules, see :doc:`Asset Rules </assetrules>`.

Running the Batch Compiler
=========================================

To run the Batch Compiler from the command line use the command:

::

    batch_compiler.exe target

To run the debug version of the Batch Compiler use:

::

    batch_compiler_d.exe target

For both commands, ``target`` is the source file or directory to
process. If the target specified is an asset file, the batch compiler
will process only that asset. If the target specified is a directory the
batch compiler will recursively search for and convert all assets under
that directory. The Batch Compiler supports both absolute and relative
paths. It currently does not support BigWorld Technology resource paths.

.. _command-line-options:

Command-Line Options
=========================================

The Batch Compiler can be run with a number of command-line options:

-  ``-intermediatePath path``
   where path specifies where to save intermediate files. By default,
   intermediate files are saved to the same location as their source
   files.
   
-  ``-outputPath path``
   where path specifies where to save output files. By default, output
   files are saved to the same location as their source files.
-  ``-cachePath path``
   where path specifies the directory location of the external shared
   file cache. By default no file cache is used.
-  ``-j numThreads``
   where numThreads specifies the number of threads to run the asset
   pipeline with. The more threads the asset pipeline is allowed to use
   the better it can utilize CPU cores, resulting in a faster execution
   time. By default, one thread is used.
-  ``-report reportFile``
   where reportFile is the file location where the Batch Compiler will
   save a report at the end of the conversion process. This file should
   have an html filename extension. If this option is omitted, the Batch
   Compiler will not generate a report.
-  ``-pluginConfig pluginsFile``
   where pluginsFile is a file containing the list of plugins to use in
   conversion. By default, a file called ``batch_compiler_plugins.txt``
   is used.

Examples:

::

    batch_compiler_d.exe D:\2_current\res\bigworld\ 
                         -report D:\2_current\bigworld_report.html -j 4

Runs the debug version of the Batch Compiler over the 
``res\bigworld``\ directory with 4 threads. Intermediate and output files
are saved in the same location as their source files, no file cache is
used, and a report is saved upon completion.

::

    batch_compiler.exe ..\..\..\res\fantasydemo\ 
                       -intermediatePath ..\..\..\intermediate\fantasydemo\ 
                       -outputPath ..\..\..\packed\fantasydemo\ 
                       -cachePath \\ASSETCACHE\assetcache 
                       -report ..\..\..\fantasydemo_report.html -j 8

Runs the release version of the Batch Compiler over the
``res\fantasydemo``\ directory with 8 threads. Intermediate and output
files are saved to custom locations, an external file cache is used, and
a report is saved on completion.

Output
=========================================


When the Batch Compiler is run, real-time progress is reported to the
console during the conversion process. The asset pipeline operates in
two stages—discovery and conversion.

During the discovery stage, the asset pipeline traverses directories and
files, searching for source assets to convert. The Batch Compiler
reports each directory it iterates through and the file name of every
source asset it encounters. At the end of this stage, the Batch Compiler
reports the number of source assets found as 'tasks' and the number of
directories and files it traversed.

In the conversion stage, the Batch Compiler reports the progress of each
conversion task in real time. As the asset pipeline can be run on
multiple threads, multiple tasks can be executed simultaneously, with
each task reporting its progress to the console. This can result in the
output of each task getting jumbled with the output of other tasks
running simultaneously. To counter this, each task is given an id which
is displayed at the start of each line of output associated with that
task.

As a task is being processed, if an error or warning is encountered
(triggered by the BWT warning macros), the error or warning will be
displayed to the console output with the id of the task in which the
error or warning was encountered. Similarly if an assert is triggered
during the processing of a task, the assert message and line number will
be displayed to the console output with the appropriate task id. Asserts
triggered during task processing are swallowed by the asset pipeline and
do not cause the front-end compiler to crash or bring up a critical
dialog.

At the end of processing each task, Batch Compiler displays the success
status for the conversion of that task. After processing the last task
it displays a summary of all the tasks. The summary shows the number of
tasks that succeeded and the number of tasks that failed. Of the tasks
that succeeded, it shows the number of those that were actually
converted, those whose dependencies were checked but were not in need of
conversion (up to date) and those that were skipped completely due to no
change to the source asset being detected since the last run of the
asset pipeline (skipped).

Report
=========================================

The Batch Compiler can be run with the -report option to produce an HTML
report on completion. The report is essentially a summary of the Batch
Compiler real time output, but structured in a way that is easier to
navigate.

The report contains lists of the tasks that were converted, specifying
which ones failed, had warnings, were converted, were up to date, or
were skipped completely. Each of these lists contains links to the
summaries of the individual tasks themselves. These summaries contain
the success status of the task, a log of any warnings, errors or asserts
that were encountered and a list of any output files that were generated
by the task.

The report also contains some information not available in the real-time
output of the Batch Compiler, including the duration of the entire
conversion process and the time taken to convert each task. Each task
summary will contain links to the subtasks that task was dependent on,
allowing you to track task errors or warnings through to subtasks.