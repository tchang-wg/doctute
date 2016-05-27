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