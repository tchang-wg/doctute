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
rules, see :doc:`Asset Rules </assetrules>`.

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
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

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