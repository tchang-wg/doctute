##############
Asset Pipeline
##############

.. toctree::
   :glob:
   :maxdepth: 2
   
   self
   Batch Compiler <batchcompiler>
   TestCompiler <testcompiler>
   JIT Compiler <jitcompiler>
   Utility Classes <utilityclasses>
   Extending the Asset Pipeline <extendingassetpipeline>
   Asset Rules <assetrules>

************
Introduction
************

Overview of the Asset Pipeline
==============================

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
:ref:`writing-compilers` for more information.

The current version of the asset pipeline batch compiler converts assets
faster than res\_packer, but only supports the conversion of ``.bmp``,
``.tga``, ``.jpg``, ``.png``, ``.texformat`` and ``.dds`` files. It
compiles ``fx`` and ``fxh`` files into ``fxo`` files. It does not
support ``model``, ``cdata``, or ``chunk`` yet.

The back end of the asset pipeline is managed by a DLL plugin system
that defines what converters and conversion rules are loaded by the
framework to handle the asset processing.

Source Assets
=============

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
==========

Converters contain the logic to convert source asset files into compiled
assets. Texture and effects converters are supplied with the Asset
Pipeline package. You can also write your own converters. See :doc:`Extending
the Asset Pipeline </extendingassetpipeline>` for more information.

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
==================

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
===============

By default, the asset pipeline outputs intermediate and output files to
the locations specified by "-intermediatePath" and "-outputPath" in
``game/res/bigworld/resources.xml``. This makes it easier to compress
and package output files for distribution with the final game. It also
allows you to keep your source directory clean from intermediate and
output files, which is desirable when working with file versioning
software. If "-outputPath" is not specified, then the output files will
be saved in the same location as their corresponding source input file.

Performance
===========

When running the asset pipeline over the same source assets multiple
times, second iterations of the asset pipeline only reconvert source
assets that have been changed. This is to ensure that the asset pipeline
can be run repeatedly over large sets of data with minimal overhead per
iteration.

To further improve performance for multiple users running the asset
pipeline on different machines, a file cache can be specified (see
:ref:`command-line-options`). When a
file cache is used, all intermediate and output files produced by the
asset pipeline are copied to an external shared location that can be
queried on subsequent runs of the pipeline.
