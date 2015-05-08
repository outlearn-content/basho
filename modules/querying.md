

Now that we've had a taste of the CRUD interface for Riak, let's look
into a few ways to lay out and query our data: secondary indexes and key/value operations.


<!-- @section -->

## Configuration Changes

Before we experiment with these methods, we will have to change our Riak
instance's configuration a little bit.

To do this we will have to find Riak's `riak.conf` file, which can
usually be found at `/etc/riak/riak.conf`. If you are running
SmartOS it will be at `/opt/local/etc/riak/riak.conf`, and if you used
Homebrew to install Riak on OSX it will be at
`/usr/local/Cellar/riak/**VERSION**/libexec/etc/riak.conf`.

Open the `riak.conf` file in your favorite text editor.

### Using the LevelDB Backend for 2i

Search for the `storage_backend` setting and change it from `bitcask` to
`leveldb` (because only LevelDB supports secondary indexes, a
feature that we'll use in our examples in this tutorial).

Save your `riak.conf`, and restart riak by executing `riak stop`
followed by `riak start` from the command line. Run `riak ping` after
the restart to ensure that the node is up and running.

**Note**: If you are running a cluster instead of a single node, you
will have to make these changes on each node.
