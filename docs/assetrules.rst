******************
Asset Rules
******************

.. contents::
   :local:
   :depth: 2

The asset pipeline uses conversion rules to determine which files on
disk are assets, and how to convert them.

Conversion rules can be created in C++ and registered through a plugin
system (for more information, see :ref:`registering-converters-and-conversion-rules`).

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
