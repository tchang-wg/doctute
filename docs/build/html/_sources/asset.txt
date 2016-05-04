##################
Asset Pipeline
##################

.. toctree::
   :maxdepth: 3

-  `Introduction`_
-  `Batch Compiler`_
-  `TestCompiler`_
-  `JIT Compiler`_
-  `Utility Classes`_
-  `Extending the Asset Pipeline`_
-  `Asset Rules`_

*****************
Introduction
*****************

Overview of the Asset Pipeline
=========================================

The asset pipeline is a framework for discovering and converting source
assets to game-ready compiled assets. When fully implemented it will
unify and replace multiple resource management solutions in the current
BigWorld Technology code base, such as the res\_packer,
offline\_processor, asset\_processor, and in-game conversion, with a
single solution. This solution will make it possible to reuse any asset
conversion code you write. For example, conversion code written for
batch compilation could also be reused for JIT (Just-In-Time)
compilation.

There are currently three supported compilers: the Batch Compiler, JIT
Compiler, and Test Compiler. You can also write your own compilers—see
`Writing Compilers`_ for more
information.

The current version of the asset pipeline batch compiler converts assets
faster than res\_packer, but only supports the conversion of ``.bmp``,
``.tga``, ``.jpg``, ``.png``, ``.texformat`` and ``.dds`` files. It
compiles ``fx`` and ``fxh`` files into ``fxo`` files. It does not
support ``model``, ``cdata``, or ``chunk`` yet.

The back end of the asset pipeline is managed by a DLL plugin system
that defines what converters and conversion rules are loaded by the
framework to handle the asset processing.

Source Assets
=========================================

The asset pipeline takes source asset files as inputs and produces
game-ready compiled assets as outputs. The files it treats as source
assets are defined by the converter plugins, and are unknown to the
compiler front ends. Typical source assets include non dds texture files
(for example, ``bmp``, ``tga`` and ``png`` files), and uncompiled shader
programs (``fx`` and ``fxh``). The output files are also determined by
the converter plugins and are unknown to the compiler front ends with
examples such as dds texture files (``dds``), and compiled shader
programs (``fxo``).

Converters
=========================================

Converters contain the logic to convert source asset files into compiled
assets. Texture and effects converters are supplied with the Asset
Pipeline package. You can also write your own converters. See `Extending
the Asset Pipeline`_ for
more information.

For a converter to be loaded by the asset pipeline, its DLL executable
needs to be placed in an appropriate location (usually in the same
directory as the compiler) and it needs to be added to the plugins list
file for that compiler (for example, ``batch_compiler_plugins.txt`` for
``batch_compiler.exe``). You only need to specify the relative path (if
any) and the base name of the plugin DLL, as the compiler will determine
and load the appropriate DLL. For example, specify "spaceconverter" for
the compiler to load the ``spaceconverter.dll`` plugin (or
``spaceconverter_d.dll`` plugin in debug mode).

Intermediate files
=========================================

As well as producing compiled assets, the asset pipeline outputs a
number of intermediate files. The most important of these intermediate
files are the dependency files (deps). The asset pipeline will generate
one of these files for every source asset it discovers. It stores
important information such as the hash of the source asset at the time
of conversion, the converter that was used to convert the source asset,
and the hash of any files that were required for the conversion process.
These dependency files need to persist between runs of the asset
pipeline, as they are used to determine whether a source asset needs to
be reconverted. If a dependency file for a source asset is missing, a
source asset is assumed to be out of date and in need of conversion.

Converters can also output intermediate files during the conversion
process. These intermediate files are functionally the same as ordinary
output files. However, while output files are used by the game engine,
intermediate files are not. By default, the intermediate files are saved
in the location specified by "-intermediatePath" in
``game/res/bigworld/resources.xml``. If a location is not specified,
they will be saved in the same location as their corresponding source
input file.

Output Location
=========================================

By default, the asset pipeline outputs intermediate and output files to
the locations specified by "-intermediatePath" and "-outputPath" in
``game/res/bigworld/resources.xml``. This makes it easier to compress
and package output files for distribution with the final game. It also
allows you to keep your source directory clean from intermediate and
output files, which is desirable when working with file versioning
software. If "-outputPath" is not specified, then the output files will
be saved in the same location as their corresponding source input file.

Performance
=========================================

When running the asset pipeline over the same source assets multiple
times, second iterations of the asset pipeline only reconvert source
assets that have been changed. This is to ensure that the asset pipeline
can be run repeatedly over large sets of data with minimal overhead per
iteration.

To further improve performance for multiple users running the asset
pipeline on different machines, a file cache can be specified (see
`Command-Line Options`_). When a
file cache is used, all intermediate and output files produced by the
asset pipeline are copied to an external shared location that can be
queried on subsequent runs of the pipeline.

*************************
Batch Compiler
*************************

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
rules, see `Asset Rules`_.

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

*************************
TestCompiler
*************************

The TestCompiler is a specialised front end to the asset pipeline
designed to facilitate the development of unit tests for new converters
and conversion rules. The intended users of the TestCompiler are
developers who wish to create converter plugins for the asset pipeline.
For information on using the TestCompiler, see `Writing Unit
Tests`_.

*************************
JIT Compiler
*************************

Overview
=========================================

The JIT Compiler (Just-In-Time Compiler) is an asset server that can be
connected to multiple asset clients. It converts assets to game-ready
compiled assets as soon as they are edited and saved to disk.

BWClient, World Editor, Model Editor, NavGen, and Particle Editor are
all asset clients. You can also create your own.

The JIT Compiler is designed to run in the background on a developer’s
machine. On startup, it scans all resource directories looking for
assets to convert and queues them for conversion. After scanning, it
monitors those directories for file changes, and queues any asset that
has changed for reconversion.

To ensure that multiple compilers aren’t attempting to operate on the
same assets, only one asset server can be run per resource path.

Configuration
=========================================

Setting the Default Asset Server
-------------------------------------------------------

When an asset client starts it will attempt to connect to an executing
asset server. If there aren’t any asset servers running, the client will
launch the default asset server defined in
``game/res/bigworld/resources.xml``. To specify the default asset
server, set the ``assetServer`` tag in ``resources.xml``.

Setting Up Converters
-------------------------------------------------------

Before running the JIT Compiler, ensure that all of the converter
plugins you wish to use exist in a reachable folder (preferably in the
same folder as the JIT Compiler executable), and that they are listed in
the ``jit_compiler_plugins.txt`` file.

Setting Up Asset Rules
-------------------------------------------------------

The JIT Compiler uses asset rules to determine what files on disk are
assets, and how to convert them. For information on setting up asset
rules, see `Asset Rules`_.

Asset Client Mode
-------------------------------------------------------

Asset clients can only communicate with asset servers when running in
debug or hybrid mode. In consumer release it is assumed that all assets
are already converted and ready for use. You can disable asset client
communication in debug and hybrid mode by running the asset client (for
example, bw\_client, world\_editor or model\_editor) with the command
line argument “-noConversion”.

JIT Compiler Options Dialog
-------------------------------------------------------

To open the JIT Compiler Options dialog:

1. Open the JIT Compiler dialog (click the JIT Compiler icon in the
   Windows notification area).
2. Select **File** > **Options**.

The following options are available:

-  **Balloon Notifications of warnings and errors** — Enables or disables the appearance of the balloon notifications that
   occur when a task completes with a warning or error.
-  **Threads** — Sets the number of threads used to process assets. For optimal
   performance we recommend matching the number of threads to the number
   of cores on your local machine.
-  **Output Path** — Specifies where to save converted resources. By default, the param
   "-outputPath" specified in ``game/res/bigworld/resources.xml`` is
   used. If no output path is specified, the converted resources will be
   saved in the same location as their corresponding source input files.
-  **Intermediate Path** — Specifies where to save asset pipeline intermediate files. By
   default, the param "-intermediatePath" specified in
   ``game/res/bigworld/resources.xml`` is used, if not specified, then
   the intermediate files will be saved in the same location as their
   corresponding source input file.
-  **Cache Path** — The path to the asset cache. Typically, this cache is on a network
   machine and is shared by all users. If no path is specified, the JIT
   compiler will not use a cache (even if you select the "Enable reading
   from the cache" and "Enable writing to the cache" options).
-  **Enable reading from the cache** — If this option is selected and a required asset cannot be found on
   your local machine, the JIT compiler will attempt to copy it from the
   cache instead of building it.
-  **Enable writing to the cache** — Enables writing converted files to the cache.
-  **Force Rebuild of Assets** — Forces every asset to be rebuilt even if it is found on your local
   machine or in the cache.

**Note:** If you modify the Output Path or Intermediate Path, you will
need to restart the JIT compiler in order for your changes to be
applied.

Viewing the Current Configuration
-------------------------------------------------------

To view information about the current configuration, select **Help** >
**Current Config**.

The following configuration information is displayed:

-  Directories used by the JIT Compiler (for example, the cache path)
-  The options specified in the Options dialog
-  The plugins that are loaded

This information can be copied and pasted into a bug report if needed.

Using the JIT Compiler
=========================================

Starting the Compiler
-------------------------------------------------------

The JIT Compiler automatically starts running in the background when an
asset client needs to convert assets. You can also start it manually by
running the JIT Compiler executable
(``game\bin\tools\\asset_pipeline\\jit_compiler.exe``).

The JIT Compiler dialog
-------------------------------------------------------

To open the JIT Compiler dialog, click the JIT Compiler icon in the
Windows notification area.

The main dialog of the JIT Compiler has the followings sections.

**Requested Assets** — shows tasks that have been requested by a client.

**Current Assets** — shows tasks that are currently being processed.

**Completed Assets** — shows completed tasks.

**Sorting Modes** — you can select one of the following **Sort** modes to sort the list of
completed tasks:

-  Completed — Sort by order of completion.
-  File Name — Sort by filename in alphabetical order.
-  Converter — Sort by the converter used to compile the asset.
-  Result — Sort by order of importance (tasks with errors first, then
   warnings, and then the rest).

Filtering Buttons
^^^^^^^^^^^^^^^^^
The three coloured buttons above the Completed Tasks list can be
toggled on or off to filter the list as follows:

-  **Blue **— Show successfully completed tasks (tasks with no errors or
   warnings).
-  **Yellow **— Show tasks with warnings but no errors.
-  **Red **— Show tasks with errors.
-  **Blue and Yellow **— Show tasks with no errors.
-  **Yellow and Red **— Show tasks with warnings and/or errors.
-  **Blue, Yellow and Red **— Show all tasks.

By default the yellow (warnings) and red (errors) options are enabled.

To display the conversion log for an asset
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Double-click the asset (or right-click it and select **Details**).

To open an asset in its default editing application
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Right-click the asset and select **Open**.

To navigate to the location of an asset in Windows
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Right-click it and select **Open Folder**.

JIT Compiler Status Icon
^^^^^^^^^^^^^^^^^^^^^^^^^^

When the JIT Compiler is running, the JIT Compiler icon appears in the
Windows notification area. The colour of the icon indicates the status
of the compiler, as follows:

-  **Red **— An asset failed to build.
-  **Yellow **— All assets built successfully but warnings were
   detected.
-  **Blue **— No warnings were detected.

Exiting the JIT Compiler
^^^^^^^^^^^^^^^^^^^^^^^^^^

To close the JIT Compiler and leave it running in the background, simply
close the JIT Compiler dialog.

To terminate the JIT Compiler, do either of the following:

-  Right-click the JIT Compiler icon and select **Quit**, or
-  Select **File** > **Quit**.

**Note:** if an application needs to convert assets it will
automatically start the JIT Compiler.

*************************
Utility Classes
*************************

The asset\_pipeline module provides a number of utility classes for
creating asset servers and asset clients, and handling connections
between them.

AssetServer
=========================================

An asset server is an application that can accept requests to convert
assets from multiple connections. To implement an asset server, derive
from the AssetServer class. The base class implementation of the
AssetServer handles creating a Windows Named Pipe, and listening for
connections from multiple clients. It also ensures that no two asset
servers are operating over the same resource paths, system wide.

When creating an asset server, you must implement functionality to
handle requests for assets. To do so, override the following function on
AssetServer:

::

    virtual void onAssetRequested( const StringRef & asset )

The asset parameter will contain the resource-relative path of an asset
that has been requested by a connected client.

Once your asset server has finished processing the request, you will
need to call the following function on AssetServer:

::

    void broadcastAsset( const StringRef & asset )

This function communicates to all connected clients that the asset
server has finished processing the requested asset.

AssetClient
-------------------------------------------------------

An asset client is an application that can connect to an asset server
and request assets for conversion. When an asset server receives an
asset request it will push the required asset to the front of its
conversion queue.

To implement an asset client, create an instance of the AssetClient
class within your application. The AssetClient class automatically
handles connecting to an asset server. If an executing asset server
cannot be found, the AssetClient class tries to start the default asset
server. The default asset server is defined in
``game/res/bigworld/resources.xml`` under the tag ``<assetServer>``.

To request an asset from an asset client, call the following function on
AssetClient:

::

    void requestAsset( const StringRef & asset, bool wait )

The asset parameter specifies the resource-relative path of the asset to
process. If the wait parameter is set to true, this function will only
return once the connected asset server has finished processing the
asset, otherwise the function will return immediately. In both cases,
the asset client cannot assume that a requested asset has been
successfully processed. It is possible for an asset server to receive a
request for an asset that it cannot process, or that contains an error.
As such, it is the responsibility of the asset client to verify the
contents of an asset.

AssetLock
-------------------------------------------------------

The implementation of the asset server provided with BWT is known as the
JIT Compiler. The JIT Compiler automatically detects and processes
changes to assets on disk. This can sometimes lead to undesirable
behaviour where the JIT Compiler starts processing an asset that is
still being edited. To overcome this issue, use the AssetLock class.

The AssetLock class can be used to prevent the JIT Compiler from
processing any assets or responding to file modifications. It does not
stop the JIT Compiler from scanning the disk, searching for assets that
have been modified. Assets modified while the JIT Compiler is locked
will be processed when the lock is released.

To lock the JIT Compiler, simply create an instance of the AssetLock
class. The constructor of the AssetLock will only return once the JIT
Compiler has been successfully locked. AssetLock instances can be
created across multiple threads and processes. After all instances of
the AssetLock are destroyed, or all processes with active AssetLocks are
terminated, the lock on the JIT Compiler will be released.

****************************
Extending the Asset Pipeline
****************************

Overview
=========================================

The asset pipeline is designed to be flexible and extendable. To extend
the asset pipeline, developers can create new converters and conversion
rules for the back end, or new compilers for the front end.

Converters and conversion rules are standalone modules of code that can
be registered with the asset pipeline. Conversion rules define what
files can be converted through the asset pipeline, whereas converters
contain the actual conversion logic. Converters are also required to
inform the asset pipeline of any dependency a source asset may have,
after which the asset pipeline will ensure that these dependencies are
up to date and built.

Compilers are executable modules that provide a front-end interface to
the asset pipeline. There are three compilers provided with BWT: the
Batch Compiler, JIT Compiler, and Test Compiler. You can design
additional compilers to suit your needs (for example, compilers
providing graphical interfaces or progress feedback).

Modules
=========================================

The asset pipeline itself is divided into four library modules—the
compiler, conversion, dependency, and discovery modules. When writing
your own extensions for the asset pipeline, you should not need to
modify these libraries and should treat the entire asset pipeline as a
black box. The following descriptions are intended purely as a summary
of how the asset pipeline operates.

Compiler
-------------------------------------------------------

The compiler module contains the front-end compiler interface and the
default base implementation of the compiler (asset\_compiler). All
user-defined compilers must inherit from the asset\_compiler base class
and not the compiler interface. This has been enforced by making the
constructor of the compiler interface private, and only available to the
asset\_compiler base class. The asset\_compiler provides a number
callback functions that can be overloaded to monitor the progress of the
asset pipeline during discovery and conversion phases.

Conversion
-------------------------------------------------------

The conversion module is responsible for processing conversion tasks,
checking dependency hashes and invoking the appropriate converters. It
contains the converter interface which all user-defined asset converters
must derive from, the conversion\_task (a data struct that is used to
track the status of an asset during conversion), the task\_processor (a
class which processes a queue of tasks on multiple threads, resolving
any interdependencies), and the content\_addressable\_cache (a static
class that acts an extension to the task\_processor to allow for faster
task processing).

Dependency
-------------------------------------------------------

The dependency module is a simple data library that contains the
different dependency structures that the asset pipeline knows how to
handle. It contains the dependency base, the converter\_dependency,
converter\_params\_dependency, intermediate\_file\_dependency,
output\_file\_dependency, directory\_dependency, and
source\_file\_dependency implementations and the dependency\_list. As
simple data structures, the only functionality the dependency classes
provide are serialising to and from disk and generating dependency
hashes. The dependency\_list class is a slightly more complicated data
structure that maintains a list of primary and secondary dependencies
associated with a source asset and a list of output files generated by
the source asset. The dependency\_list structure represents the
``.deps`` files that are generated by the asset pipeline during
conversion.

Discovery
-------------------------------------------------------

The discovery module is responsible for the task collection that occurs
during the discovery phase of the asset pipeline. It contains the
task\_finder class, which iterates directories and files on disk,
searching for potential source assets eligible for conversion, and the
conversion\_rule interface, the interface all user-defined conversion
rules must implement. Conversion rules are used by the task\_finder to
determine which files on disk can be converted.

The Conversion Process
=========================================
   
A conversion task is created for every asset that the asset pipeline
attempts to convert. A conversion task contains:

-  the file name of the asset being converted, 
-  the id and version of the converter to be used, 
-  any parameters to be passed to the converter, 
-  the status of the conversion's progress, and 
-  links to any dependent conversion tasks for the asset.

To ensure minimal effort is spent processing tasks, the asset pipeline
first attempts to determine if an asset needs to be converted at all. To
do this, it attempts to deduce if an asset, any of its dependencies, or
any of its outputs have changed since the last time the asset was
processed. To track this, the asset pipeline uses **deps** files.

When an asset is processed, its deps file is queried from disk. If no
deps file exists, a primary deps file is created. A primary deps file
contains all the primary dependencies of a source asset. The primary
dependencies of an asset are the asset source file itself, and the
converter id, converter version and converter parameters required to
convert the asset. If a deps file is found on disk, the stored primary
dependencies are checked to see if they are up to date. If the primary
dependencies are up not up to date, any secondary dependencies stored in
the deps file are considered invalid and need to be regenerated. To
regenerate the secondary dependencies, the converter for the asset is
invoked via the converter function ``createDependencies``.

Once the secondary dependencies have been generated, the asset pipeline
must then verify that these dependencies are also up to date. As
secondary dependencies can include dependencies on compiled assets, the
asset pipeline may be required to trigger subtasks at this point. The
asset pipeline has two modes of operation for accomplishing this:
recursive conversion and non-recursive conversion.

Recursive Conversion
-------------------------------------------------------

This conversion mode blocks the current thread processing a task, and
recursively triggers conversion tasks of any sub dependencies. This has
the benefit of simplicity and can be easily debugged, as the entire
hierarchy of dependencies that triggered the conversion of a task can be
viewed in the debug callstack. Complications do still arise with this
method when multiple tasks being processed on separate threads have
dependencies on the same subtask. In this case, the first thread to
request the common subtask will trigger the conversion of the subtask
while any subsequent threads will be required to block and wait for the
first thread to finish. The drawback to this method is that when using
large root tasks which trigger multiple subtasks, all of these subtasks
must be processed on the same thread as the root parent task. For
example, if a task to convert a space triggers multiple subtasks for
models, textures and shaders, all of those subtasks need to be processed
on the thread used to convert the space. This reduces the benefit of the
parallel conversion capabilities that the asset pipeline provides.

Non-Recursive Conversion
-------------------------------------------------------

To maximise the parallel-conversion capabilities of the asset pipeline,
a non-recursive conversion mode is provided. In this mode, whenever a
parent task is determined to have a subtask that must be converted, the
parent task is paused and pushed back into the asset pipeline task
queue. The asset pipeline goes through a task management process to
ensure that the next time the parent task is processed, all of its
subtasks, and any further recursive subtasks have been processed to
completion. The asset pipeline also detects and protects against cyclic
dependencies between assets. This mode of operation is the more
efficient of the two modes of operation, but in this mode dependencies
between assets can be harder to debug.

After all the secondary dependencies of a task have been processed, the
hashes of the dependencies are checked against the assets deps file. If
the dependencies match, the deps file is then checked for the expected
outputs of the asset. If the expected outputs match those on disk the
task is considered up to date and conversion is skipped. If the
secondary dependencies do not match, or the expected outputs of the
asset do not match, the converter for the asset is invoked to convert
the asset via the converter function convert.

This conversion process is repeated for every conversion task discovered
by the asset pipeline.

Writing Converters and Conversion Rules
=========================================

The asset pipeline operates in two phases: the discovery phase and the
conversion phase. During the discovery phase, the asset pipeline
searches for files on disk that it can convert and creates conversion
tasks for them. During the conversion phase, the asset pipeline does the
actual asset conversion. Conversion rules are primarily used during the
discovery phase and converters during the conversion phase. In most
cases, when designing a new converter, you will need to create a
converter and a conversion rule; however, there isn't a one-to-one link
between converters and conversion rules and the two are independent of
each other.

Converters
-------------------------------------------------------

To create a new converter, you must derive from the Converter interface
and implement the following:

Constructor
^^^^^^^^^^^^^

::

    // Constructor
    // \param params command line parameters for initialising the converter
    Converter( const BW::string& params )

The constructor for your converter must take a ``params`` string
parameter. Your constructor should defer to the base class constructor
which will store the ``params`` string in a member variable ``params_``.
The ``params`` parameter can contain any optional parameters of your
choosing and is passed to your constructor from the asset pipeline via
the conversion rules (see `Conversion Rules`_).

createDependencies
^^^^^^^^^^^^^^^^^

::

    // builds the dependency list for a source file.
    // \param sourcefile the name of the source file on disk.
    // \param dependencies the dependency list to generate.
    // \return true if the dependency list was successfully generated
    virtual bool createDependencies( const BW::string& sourcefile,
                                     const Compiler & compiler,
                                     DependencyList & dependencies )

This function must be overloaded to inform the asset pipeline of the
dependencies of the source file you wish to convert. The sourcefile
parameter contains the absolute path of the asset to convert. The
compiler parameter contains a reference to the front-end compiler that
has triggered this conversion. The dependencies parameters contains the
dependency list structure that must be filled out with the source
asset's dependencies.

convert
^^^^^^^^^^^^^^^^^

::

    // convert a source file.
    // \param sourcefile the name of the source file on disk.
    // \param convertedFiles a list of filenames that were converted from the source file.
    // \return true if the source file was successfully converted. 
    virtual bool convert( const BW::string& sourcefile,
                          const Compiler & compiler,
                          BW::vector< BW::string > & intermediateFiles,
                          BW::vector< BW::string > & outputFiles )

This function must be overloaded to perform the actual conversion of
your source asset. The sourcefile parameter contains the absolute path
of the asset to convert. The compiler parameter contains a reference to
the front-end compiler that has triggered this conversion. The
intermediateFiles parameter contains a vector of strings to which you
must append the path of any intermediate file produced by your
conversion. The outputFiles parameter contains a vector of strings to
which you must append the path of any output file produced by your
conversion.

Absolute vs Relative Paths
-------------------------------------------------------

Internally the asset pipeline operates on absolute paths to avoid errors
with files with the same file name existing in multiple resource
directories. However, when adding files to the dependency list in the
createDependencies function, file names must first be converted to
relative paths. This is necessary as these file names get saved to disk
and shared via a file cache between multiple users, who may have
different resource path setups. Conversely, when pushing file names back
into the intermediateFiles and outputFiles outputs of the convert
function, these file names need to be absolute paths. The reason for
this is that the asset pipeline is required to hash the contents of
these files to store in the generated deps file, and by using absolute
paths we can ensure that the correct file is hashed.

There is, however, an important consideration that needs to be made when
converting between relative and absolute paths. The asset pipeline can
be run with custom intermediate and output directories. When resolving
an intermediate or output file to an absolute path, this path needs to
be resolved to the appropriate directory. As the intermediate and output
directories are unknown to the converters until run time, the compiler
interface provides a number of convenience functions for converting
between absolute and relative paths.

-  ``virtual bool      resolveRelativePath( BW::string & path ) const``
   ****
   Converts the path to a relative path.
-  ``virtual bool      resolveSourcePath( BW::string & path ) const``
   Converts the path to an absolute source path.
-  ``virtual bool      resolveIntermediatePath( BW::string & path ) const``
   ****
   Converts the path to an absolute intermediate path.
-  ``virtual bool      resolveOutputPath( BW::string & path ) const``
   ****
   Converts the path to an absolute output path.

In summary, before adding a file to a dependency list, call
``resolveRelativePath``. Before adding a file to the intermediate files
list, call ``resolveIntermediatePath``. Before adding a file to the
output files list, call ``resolveOutputPath``. Also note, it is the
responsibility of the converter to save any files it outputs to the
appropriate directories. It is good practice to work with absolute paths
during conversion where possible.

Dependency List
-------------------------------------------------------

When pushing back dependencies in the createDependencies function, there
are a number of options. Assets are currently allowed to have the
following types of dependencies: source file dependencies, intermediate
file dependencies, output file dependencies, directory dependencies,
converter dependencies, and converter parameter dependencies.

-  **Source file dependencies** are when an asset file depends on
   another raw source asset. This dependency indicates that the file it
   depends on does not need any sort of processing by the asset
   pipeline.
-  **Intermediate file dependencies** and **output file dependencies**
   occur when an asset file depends on another compiled asset. These
   types of dependencies tell the asset pipeline that a subtask has to
   be initiated to convert a source asset into a compiled format before
   the conversion of the current asset can take place. The only
   difference between intermediate and output file dependencies is the
   location where the compiled asset is expected to reside.
-  **Directory dependencies** allow an asset to recursively depend on a
   directory of assets. Directory dependencies can use a regex pattern
   to filter all files in a directory into a smaller subset of files.
-  **Converter dependencies** allow an asset to depend on a certain
   version of a converter, and **converter parameter dependencies**
   allow an asset to depend on the parameter string that is passed to
   the converter's constructor. These last two dependency types are only
   used internally by the asset pipeline system.

All dependencies can be marked as critical or non critical. A critical
dependency is a dependency that must exist for the parent asset to be
converted. If an error is encountered in a critical dependency, it will
automatically fail the parent asset. A converter should ideally be
designed to have as few critical dependencies as possible.

The DependencyList class provides the following functions for adding
dependencies:

``void addPrimarySourceFileDependency( const BW::string & filename )``

Adds a primary source file dependency. The filename parameter
specifies the name of the file to depend on.
  
``void addPrimaryConverterDependency( size_t converterId, size_t converterVersion )``

Adds a primary converter dependency. The converterId parameter
specifies the id of the converter to depend on, converterVersion
specifies the version of the converter to depend on.

``void addPrimaryConverterParamsDependency( const BW::string & converterParams )``

Adds a primary converter params dependency. The convertParams
parameter specifies the parameter string to depend on.

``void addSecondarySourceFileDependency( const BW::string & filename, bool critical )``

Adds a secondary source file dependency. The filename parameter
specifies the name of the file to depend on, and critical specifies
whether this is a critical dependency.

``void addSecondaryIntermediateFileDependency( const BW::string & filename, bool critical )``

Adds a secondary intermediate file dependency. The filename parameter
specifies the name of the compiled file to depend on, and critical
specifies whether this is a critical dependency.

``void addSecondaryOutputFileDependency( const BW::string & filename, bool critical )``

Adds a secondary output file dependency. The filename parameter
specifies the name of the compiled file to depend on, and critical
specifies whether this is a critical dependency.

``void addSecondaryDirectoryDependency( const BW::string & directory,``
``const BW::string & pattern, bool recursive, bool critical )``

Adds a secondary directory dependency. The pattern parameter
specifies a regex pattern to match files within a directory,
specified by the directory parameter, to depend on. Recursive
specifies whether the pattern should be applied recursively to the
directory and critical specifies whether this is a critical
dependency.

Errors and Exceptions
-------------------------------------------------------

The asset pipeline is designed to be run as an unattended process.
Because of this we need to prevent asserts from halting the conversion
process. As such all calls into the converters are wrapped in try/catch
blocks. When an assert is fired, the asset pipeline swallows the assert
and throws an exception. The exception is then caught by the task
processor and the currently processing task is failed. The assert
information is added to the error log of the task and the asset pipeline
is able to continue on. Additionally, the asset pipeline handles the
ERROR\_MSG macro. When this macro is triggered within a converter, the
currently processing task is marked with an error. Processing is allowed
to continue on the task, but on completion the task is set as failed and
the message added to the error log of the task.

Whilst processing a task, if your converter encounters an error and
wishes to fail the current task without using an assert or an error
message, simply return false from the createDependencies or convert
function of your converter. This will set the current task as failed. If
a task fails during the createDependencies function, the asset pipeline
will not attempt to call the convert function.

Conversion Rules
-------------------------------------------------------

To create a conversion rule you must implement the ConversionRule
interface.

A conversion rule takes a single boolean argument in its constructor:

::

    ConversionRule( bool bRoot )

The bRoot argument indicates whether a conversion rule is a root rule or
a non-root rule. Root rules are rules used during the discovery phase of
the asset pipeline to match files on disk to conversion tasks. Non-root
rules are used to resolve tasks for sub dependencies of other tasks.

For example, if we had a rule to match source texture files to texture
conversion tasks and we flagged this rule as a root rule, the discovery
phase of the asset pipeline would identify tasks for every source
texture on disk. However, if we were to flag this rule as a non-root
rule, the discovery phase of the asset pipeline would not find any
texture conversion tasks. During the conversion phase of the asset
pipeline, if another task is processed and found to have a dependency on
a compiled texture, the non-root texture conversion rule is queried to
create the texture conversion task. In this way, by carefully selecting
which conversion rules are flagged as root rules, we can ensure that
only assets referenced by root assets are compiled and packaged to the
output directory.

For a conversion rule to match a source asset file name to a conversion
task, the following function must be overloaded:

::

    /* returns true and populates a conversion task if the rule can match the input filename. */
    virtual bool createTask( const BW::StringRef& sourceFile, ConversionTask& task )

This function is invoked by the asset pipeline whenever it tries to
determine how to build a source asset. If your conversion rule is able
to handle the asset passed in by the parameter sourceFile, you must
initialise the task structure and return true. The task structure
requires that you set the id, version, and parameters of the converter
that will handle the conversion of this asset.

If your rule is a non-root rule (it is intended to be invoked when
processing the dependencies of other assets), you must also overload the
following function:

::

    /* returns true if the rule can match the output filename. */
    virtual bool getSourceFile( const BW::StringRef& file, BW::string& sourcefile ) const

This function is invoked by the asset pipeline whenever it tries to
determine what source file is used to create a compiled dependency file.
In this case, the parameter file will contain the file name of the
compiled file the asset pipeline is trying to compile. If your
conversion rule knows what source asset file is used to convert into
this file, it should fill in the sourcefile parameter and return true.

Writing Compilers
=========================================

The asset pipeline supports the creation of custom front-end interfaces.
These interfaces enable users to interact with the asset conversion
process. In the asset pipeline framework, these front-end interfaces are
known as compilers.

Asset Compiler
-------------------------------------------------------

Any custom front-end compiler you write must inherit from the
AssetCompiler base class. The AssetCompiler class provides a number of
overloadable callback functions that are called by the asset pipeline
during different stages of the discovery and conversion phases. In most
cases where a compiler callback has been overloaded, it is important
that the base class implementation of the overloaded function is called
at some point during your own callback handling code. In some cases, not
doing so can cause the asset pipeline to fail.

Below are some of the more useful callbacks that you can overload.

Discovery Callbacks
-------------------------------------------------------

The following callbacks are invoked during the discovery phase:

-  ``virtual bool      shouldIterateFile( const BW::StringRef & file )``
-  ``virtual bool      shouldIterateDirectory( const BW::StringRef & directory )``

These callbacks are invoked whenever the task finder attempts to iterate
a file or directory whilst searching for potential source assets. By
returning false from either of these functions, you can tell the asset
pipeline to ignore certain files or directories.

Task Callbacks
-------------------------------------------------------

The following callbacks are invoked during the conversion phase:

-  ``virtual void      onTaskStarted( ConversionTask & conversionTask )``
-  ``virtual void      onTaskResumed( ConversionTask & conversionTask )``
-  ``virtual void      onTaskSuspended( ConversionTask & conversionTask )``
-  ``virtual void onTaskCompleted( ConversionTask & conversionTask )``

These callbacks are invoked by the task processor whilst processing
conversion tasks. They provide a mechanism for notifying the compiler of
the tasks that are currently executing. If you overload these callbacks,
it is extremely important to call the base implementation, as the
AssetCompiler uses these callbacks to manage the processing of dependent
tasks and catching cyclic dependencies.

Converter Callbacks
-------------------------------------------------------

These callbacks are invoked by the task processor prior to and after the
createDependencies function on a converter is invoked and prior to and
after the convert function on a converter is invoked. Once again, if you
overload these callbacks, make sure you call the base implementation.

-  ``virtual void      onPreCreateDependencies( ConversionTask & conversionTask )``
-  ``virtual void      onPostCreateDependencies( ConversionTask & conversionTask )``
-  ``virtual void      onPreConvert( ConversionTask & conversionTask )``
-  ``virtual void      onPostConvert( ConversionTask & conversionTask )``

Conversion Callback
-------------------------------------------------------

This callback is invoked for every intermediate and output file that is
generated by the asset pipeline. Note, if an output file is not
generated during a run of the asset pipeline, due to files being up to
date, this callback will not be invoked.

-  ``virtual void      onOutputGenerated( const BW::string & filename )``

Cache Callbacks
-------------------------------------------------------

These callbacks are invoked whenever the asset pipeline attempts to read
or write from the shared file cache. If a cache read or write is
successful, the onCacheRead and onCacheWrite callbacks will be invoked.
If a cache read or write is not successful, due to network or disk
errors or a file not existing in the cache, the onCacheReadMiss and
onCacheWriteMiss callbacks are invoked.

-  ``virtual void onCacheRead( const BW::string & filename )``
-  ``virtual void onCacheReadMiss( const BW::string & filename )``
-  ``virtual void onCacheWrite( const BW::string & filename )``
-  ``virtual void onCacheWriteMiss( const BW::string & filename )``

Message Callbacks
-------------------------------------------------------

These callbacks are invoked whenever a BigWorld Technology message macro
is used. The intention of these callbacks is to enable the compilers to
filter message spam from BigWorld Technology systems and choose which
messages to display, and how to display them to the user. These
callbacks are also integral to the operation of the asset pipeline, so
if you choose to overload these functions, make sure you call the base
implementations.

.. code:: python

    virtual bool handleMessage( DebugMessagePriority componentPriority, DebugMessagePriority messagePriority, const BW::string & category, DebugMessageSource messageSource, const char * format, va_list argPtr )

.. code:: python
    
    virtual void handleCritical( const char * msg )

Registering Converters and Conversion Rules
======================================================

One of the responsibilities of the compiler front end is to manage the
converters and conversion rules that are used by the asset pipeline
framework. The AssetCompiler base class provides two functions to this
end:

::

    virtual void registerConversionRule( ConversionRule& conversionRule );
    virtual void registerConverter( ConverterInfo& converterInfo );

Registering conversion rules is a straight forward process. An instance
of the conversion rule must be created and then passed to the
registerConversionRule function. Registering converters is a little more
complicated. To register a converter, an instance of the following
structure must be passed to the registerConverter function.

::

    /// struct containing information about a converter in the asset pipeline.
    struct ConverterInfo
    {
    public:
        /// the display name of the converter.
        BW::string name_;
        /// the id of the converter. Must be unique to each type of converter.
        size_t typeId_;
        /// the current version of the converter.
        size_t version_;
        /// converter flags
        enum
        {
            THREAD_SAFE         = 1 << 0, // can the converter be run on multiple threads.
            CACHE_DEPENDENCIES  = 1 << 1, // should dependencies be read and written to the cache.
            CACHE_CONVERSION    = 1 << 2, // should conversion be read and written to the cache.
            DEFAULT_FLAGS       = THREAD_SAFE | CACHE_DEPENDENCIES | CACHE_CONVERSION
        }                   flags_;
        /// function pointer for creating an instance of the converter.
        ConverterCreator    creator_;
    };

This structure contains all the necessary information for the asset
pipeline to instantiate converters to process tasks. The reason we do
not simply register an instance of the converter on the compiler, as we
do the conversion rules, is because the asset pipeline can process tasks
on multiple threads, meaning more than one converter of the same type
may be required at any one time. Allowing the asset pipeline to create
converters on the fly avoids the need for converters to manage thread
local storage or thread locks around member variables.

A standard practice for managing converter info structures is to define
the following on your converter class definition:

::

    /// MyConverter.hpp
    static size_t getTypeId() { return s_TypeId; }
    static size_t getVersion() { return s_Version; }
    static const char * getTypeName() { return "MyConverter"; }
    static Converter * createConverter( const BW::string& params ) { return new MyConverter( params ); }
    static const size_t s_TypeId;
    static const size_t s_Version;
    /// MyConverter.cpp
    const size_t MyConverter::s_TypeId = hash_string( MyConverter::getTypeName(), strlen( MyConverter::getTypeName()));
    const size_t MyConverter::s_Version = 1;

Defining your converter info then becomes:

::

    ConverterInfo myConverterInfo;
    myConverterInfo.name_ = MyConverter::getTypeName();
    myConverterInfo.typeId_ = MyConverter::getTypeId();
    myConverterInfo.version_ = MyConverter::getVersion();
    myConverterInfo.flags_ = myConverterFlags;
    myConverterInfo.creator_ = MyConverter::createConverter;

A convenience macro exists to allow you to then replace this with:

::

    ConverterInfo myConverterInfo;
    INIT_CONVERTER_INFO( myConverterInfo, ConverterInfo , myConverterFlags);

Initiating a Build
-------------------------------------------------------

Once all converters and conversion rules have been registered, a
compiler can initiate a build. Compilers have access to both the
discovery phase of the asset pipeline, through the taskFinder\_ member,
and the conversion phase of the asset pipeline, through the
taskProcessor\_ member.

To initiate a search for all source assets that match any of the
registered root rules, call the following:

::

    taskFinder_.findTasks( path );

where path is the directory or file to search.

To get the specific task for a source asset that matched either a root
rule or a non-root rule, call the following:

::

    ConversionTask & task = taskFinder_.getTask( file );
    if (task.converterId_ != ConversionTask::s_unknownId)
    {
        queueTask( task );
    }

In this case, it is necessary to manually queue the conversion task so
that it will be processed during the conversion phase.

To initiate the conversion phase, simply call:

::

    taskProcessor_.processTasks();

Writing Plugins
=========================================

The asset pipeline was designed with a DLL plugin architecture in mind.
Although a compiler can be implemented without using the plugin
framework, there are a number of benefits to using plugins. By using a
plugin framework, converter DLLs can be built once and distributed
between different compiler front ends. Also, developing a new converter
plugin does not require a rebuild of all the different compiler front
ends that may exist. This prevents changes to converter plugins,
introducing bugs into the compiler executable.

Plugin Loader
-------------------------------------------------------

To allow a compiler to make use of the asset pipeline plugin system, a
compiler must inherit from the PluginLoader class.

The PluginLoader class provides a number of functions for loading and
unloading converter DLLs.

::

    void initPlugins();
    void finiPlugins();
    HMODULE loadPlugin( LPCWSTR plugin );
    bool unloadPlugin( HMODULE plugin );


-  ``initPlugins`` reads a file named "<executable>\_plugins.txt" and
   loads the appropriate (debug or hybrid) plugin DLLs specified within.
-  ``finiPlugins`` unloads all currently loaded DLLs.
-  ``loadPlugin`` loads the plugin with file name plugin.
-  ``unloadPlugin`` unloads a specific loaded plugin by its HMODULE.

When using these functions to load converter plugins, it is the
responsibility of the converter plugin code to register conversion rules
and converters with your compiler.

Converter Plugin
-------------------------------------------------------

To create a converter plugin there are two functions that your DLL must
expose. When scanning for potential plugins to load, the plugin loader
will search for these functions to determine whether your plugin can be
loaded. These functions are defined differently on debug and hybrid
builds. This allows debug and hybrid configurations of your DLLs to
reside side by side in a file directory whilst still ensuring that the
plugin loader can, and will, only load the appropriate configuration
version of your DLL. To facilitate exposing the correct dll functions
based on configuration, use the macros PLUGIN\_INIT\_FUNC and
PLUGIN\_FINI\_FUNC.

The following is an example of how to write a converter plugin:

::

    BW_BEGIN_NAMESPACE
    DECLARE_APP_DATA( "MyConverter", true )
    MyConversionRule myConversionRule;
    ConverterInfo myConverterInfo;

    PLUGIN_INIT_FUNC
    {

        Compiler * compiler = dynamic_cast< Compiler * >( &pluginLoader );
        if (compiler == NULL)
        {
            return false;
        }

        // Init any systems required by your converter
        INIT_CONVERTER_INFO( myConverterInfo, MyConverter, DEFAULT_FLAGS );
        compiler->registerConversionRule( myConversionRule );
        compiler->registerConverter( myConverterInfo );
        return true;
    }

    PLUGIN_FINI_FUNC
    {       
        // Fini any systems that were started by your converter
        return true;
    }

    BW_END_NAMESPACE

The ``DECLARE_APP_DATA`` macro is a convenience macro to set up a number
of global constant values that allow the core BigWorld Technology
systems to be run in a plugin. The first parameter of this macro is a
unique identifier for your plugin, and the second is a boolean to
indicate that the current module is being compiled as a plugin.

The ``PLUGIN_INIT_FUNC`` macro automatically generates an exposed
function that gets called when your plugin is loaded. This function
takes one argument: pluginLoader. As shown in the example above, you can
verify the module attempting to load your plugin is actually a compiler
by dynamically casting the pluginLoader argument to a Compiler object.
If the cast fails, the function should return false.

The ``PLUGIN_FINI_FUNC`` macro automatically generates an exposed
function that gets called when your plugin is unloaded.

Writing Unit Tests
=========================================

The final step in writing a converter plugin for the asset pipeline is
to write unit tests. To make this as easy as possible, a custom
front-end compiler is provided. The TestCompiler provides a number of
convenience functions that can be used to easily test conversion rules
and converters.

To test a conversion rule, use the following function:

::

    // Test that a conversion rule can be found for a source file
    bool testConversionRule( const StringRef & sourceFile, 
                             const StringRef & outputFile, 
                             const StringRef & converterName, 
                             const StringRef & converterParams );

This function takes a source asset file name, the expected output file,
the expected name of the converter that would convert this asset, and
the expected conversion parameters for this asset. If a source asset is
expected to compile to more than one output file, this function should
be called for each of the individual output file names.

There are several steps involved to test a converter.

First, the test needs to be set up. To set up a test we must provide the
TestCompiler with the source files to convert. Optionally, we can also
provide copies of the output files we expect to be produced by the
conversion process. The following functions are provided for setting up
a converter test:

::

    // Set the directory where the source files for this test can be found
    bool setSourceFileDirectory( const StringRef & sourceFileDirectory );
    // Set the directory where the output files for this test can be found
    bool setOutputFileDirectory( const StringRef & outputFileDirectory );
    // Add a file from the source file directory that should be used in this test

    bool addSourceFile( const StringRef & sourceFile );
    // Add a file from the output file directory that should be built in this test

    bool addOutputFile( const StringRef & outputFile );

The ``setSourceFileDirectory`` function tells the TestCompiler where to
find the source assets to use for the test. The
``setOutputFileDirectory`` functiion tells the TestCompiler where to
find the output files to verify the conversion process against. The
``addSourceFile`` and ``addOutputFile`` functions tell the TestCompiler
which files in the source directory and output directory to use for the
test.

After the test has been set up, it can be run with one of the following
functions:

::

    // Trigger a test build for a single file
    bool testBuildFile( const StringRef & file );
    
    // Trigger a test build for a directory
    bool testBuildDirectory( const StringRef & directory );


The testBuildFile function tests building a single file. The
testBuildDirectory function tests building an entire directory of files.
The input to these functions should be the source asset or directory to
build relative to the source file directory. Normally you will want to
call testBuildDirectory with an empty directory parameter to build every
source asset in the source file directory.

When the TestCompiler finishes building your assets it will
automatically verify that the outputs produced by the asset pipeline
exactly match the copies of the expected outputs that you provided when
setting up the test. If you did not provide any expected outputs to the
test setup then the TestCompiler will assume the conversion process
completed successfully. The TestCompiler then provides the following
functions to further verify the results of the conversion process:

::

    // Get the number of tasks that were processed by this test
    long getTaskCount() const { return taskCount_; }
    
    // Get the number of tasks that were failed by this test
    long getTaskFailedCount() const { return taskFailedCount_; }
    
    // Returns if this test encountered a cyclic error
    bool hasCyclicError() const { return hasCyclicError_; }
    // Returns if this test encountered a dependency error
    bool hasDependencyError() const { return hasDependencyError_; }
    // Returns if this test encountered a conversion error
    bool hasConversionError() const { return hasConversionError_; }


-  ``getTaskCount`` allows you to verify how many tasks were actually
   processed.
-  ``getTaskFailedCount`` allows you to verify how many of these tasks
   failed, as you may wish to test certain situations in which your
   converter should fail.
-  ``hasCyclicError`` will tell you if a cyclic dependency error occured
   during conversion.
-  ``hasDependencyError`` will tell you if any task reported an error
   during the createDependencies function of your converter.
-  ``hasConversionError`` will tell you if any task reported an error
   during the convert function of your converter.

After the TestCompiler finishes its testing, it will automatically clean
up any changes it may have made on disk and return.

******************
Asset Rules
******************


The asset pipeline uses conversion rules to determine which files on
disk are assets, and how to convert them.

Conversion rules can be created in C++ and registered through a plugin
system (for more information, see `Registering Converters and Conversion
Rules`_.

Alternatively, conversion rules can be created through
``asset_rules.xml`` configuration files. An ``asset_rules.xml`` file can
contain any number of source rules and destination rules.

Source Rules
=========================================

A source rule contains a regex to match source assets to converters. For
example:

.. code-block:: xml

    <rule pattern="^texture_details\.xml$">
        <root>true</root>
        <converter>HierarchicalConfigConverter</converter>
        <converterParams>-f texture_details.xml -o system/data/texture_detail_levels.xml</converterParams>
    </rule>


This rule matches the source asset “texture\_details.xml” to the
HierarchicalConfigConverter with the parameters “-f texture\_details.xml
-o system/data/texture\_detail\_levels.xml” and specifies that the asset
is a root asset.

Destination Rules
=========================================

A destination rule contains a regex to match built assets to their
source files, through a regex replace pattern. For example:

.. code-block:: xml

    <rule pattern=".*(\\.fxo)$">
        <sourcePattern>(\\.[0-1]+)?(\\.fxo)$</sourcePattern>
        <sourceFormat>.fx</sourceFormat>
    </rule>

This rule matches a file such as “myshader.01.fxo” and uses
regex\_replace( “>(\\\\.[0-1]+)?(\\\\.fxo)$”, “.fx” ) to determine the
source file, “myshader.fx”.

Location of asset\_rules.xml
=========================================

You can place ``asset_rules.xml`` files in any resource folder. The
folder hierarchy is used to define custom rules or overrides for
subfolders. For example, consider if the following rule existed in an
``asset_rules.xml`` file in the ``bigworld/shaders`` folder.

.. code-block:: xml

    <rule pattern=".*(\\.fx)$">
        <root>true</root>
        <converter>EffectConverter</converter>
    </rule>


Any file with the ``.fx`` suffix in any folder under
``bigworld/shaders`` would be considered a root asset and would use the
EffectConverter. Suppose we then created an ``asset_rules.xml`` file
containing the following rule, in the ``bigworld/shaders/custom``
folder:

.. code-block:: xml

    <rule pattern=".*(\\.fx)$">
        <root>false</root>
    </rule>


Any file with ``.fx`` suffix in any folder under
``bigworld/shaders/custom`` would now be considered a non-root asset.
However, as these assets would still be in a subfolder of
``bigworld/shaders``, they would still be using the EffectConverter as
defined by the rule in ``bigworld/shaders``.

Excluding Files from Conversion
=========================================

Source rules can also use a “noConversion” tag to indicate that certain
files should not be converted. For example if the following rule was
added to the ``asset_rules.xml`` file in the ``bigworld/shaders``
folder, any file ending in “do\_not\_convert.fx” would be ignored by the
asset pipeline:

.. code-block:: xml

    <rule pattern=".*(do_not_convert\\.fx)$">
        < noCoversion >false</noCoversion>
    </rule>


