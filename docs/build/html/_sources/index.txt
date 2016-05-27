##################
Universal Profiler
##################

.. toctree::
   :glob:
   :maxdepth: 2
   
   self
   Profiler Settings <profiler_settings>
   Profile Analyzer <profile_analyzer>
   Analyzer Background Mode <analyzer_background>
   Running Analyzer from the Command Line <analyzer_command_line>
   Settings Recommendations <settings_recommendations>
   Building the Profile Analyzer <building_analyzer>

************
Introduction
************

The Universal Profiler is a new profiling system that replaces multiple profiling utilities (including FPS Explorer, Resource Statistics, Dog Watch, WG profiler, and BW profiler). It profiles the World of Tanks client over a variety of indicators and measurements—in particular, performance and memory usage.

When running, it writes output to a log as a .upl file. Afterwards, the analyzer processes the log, extracts the data and presents it in a convenient format.