
# CORE Python Scripting

* Table of Contents
{:toc}

## Overview

CORE can be used via the GUI or Python scripting. Writing your own Python scripts offers a rich programming environment with complete control over all aspects of the emulation. This chapter provides a brief introduction to scripting. Most of the documentation is available from sample scripts, or online via interactive Python.

The best starting point is the sample scripts that are included with CORE. If you have a CORE source tree, the example script files can be found under *core/daemon/examples/api/*. When CORE is installed from packages, the example script files will be in */usr/share/core/examples/api/* (or */usr/local/* prefix when installed from source.) For the most part, the example scripts are self-documenting; see the comments contained within the Python code.

The scripts should be run with root privileges because they create new network namespaces. In general, a CORE Python script does not connect to the CORE daemon, in fact the *core-daemon* is just another Python script that uses the CORE Python modules and exchanges messages with the GUI. To connect the GUI to your scripts, see the included sample scripts that allow for GUI connections.

Here are the basic elements of a CORE Python script:

```python
from core.emulator.coreemu import CoreEmu
from core.emulator.emudata import IpPrefixes
from core.enumerations import NodeTypes, EventTypes

# ip generator for example
prefixes = IpPrefixes(ip4_prefix="10.83.0.0/16")

# create emulator instance for creating sessions and utility methods
coreemu = CoreEmu()
session = coreemu.create_session()

# must be in configuration state for nodes to start, when using "node_add" below
session.set_state(EventTypes.CONFIGURATION_STATE)

# create switch network node
switch = session.add_node(_type=NodeTypes.SWITCH)

# create nodes
for _ in xrange(options.nodes):
    node = session.add_node()
    interface = prefixes.create_interface(node)
    session.add_link(node.objid, switch.objid, interface_one=interface)

# instantiate session
session.instantiate()

# shutdown session
coreemu.shutdown()
```

The above script creates a CORE session having two nodes connected with a hub. The first node pings the second node with 5 ping packets; the result is displayed on screen.

A good way to learn about the CORE Python modules is via interactive Python. Scripts can be run using *python -i*. Cut and paste the simple script above and you will have two nodes connected by a hub, with one node running a test ping to the other.

The CORE Python modules are documented with comments in the code. From an interactive Python shell, you can retrieve online help about the various classes and methods; for example *help(nodes.CoreNode)* or *help(Session)*.

**NOTE: The CORE daemon *core-daemon* manages a list of sessions and allows the GUI to connect and control sessions. Your Python script uses the same CORE modules but runs independently of the daemon. The daemon does not need to be running for your script to work.**

The session created by a Python script may be viewed in the GUI if certain steps are followed. The GUI has a *File Menu*, *Execute Python script...* option for running a script and automatically connecting to it. Once connected, normal GUI interaction is possible, such as moving and double-clicking nodes, activating Widgets, etc.

The script should have a line such as the following for running it from the GUI.

```python
if __name__ == "__main__" or __name__ == "__builtin__":
  main()
```

A script can add sessions to the core-daemon. A global *coreemu* variable is exposed to the script pointing to the *CoreEmu* object.
The example below has a fallback to a new CoreEmu object, in the case you would like to run the script standalone, outside of the core-daemon.

```python
coreemu = globals().get("coreemu", CoreEmu())
session = coreemu.create_session()
```

Finally, nodes and networks need to have their coordinates set to something, otherwise they will be grouped at the coordinates *<0, 0>*. First sketching the topology in the GUI and then using the *Export Python script* option may help here.

```python
switch.setposition(x=80,y=50)
```

A fully-worked example script that you can launch from the GUI is available in the examples directory.
