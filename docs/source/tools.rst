Command Line Tools
==================

The package installs two scripts: one for starting the catalog server and
another is the client for accessing local or remote catalogs.

Intake Server
-------------

The server takes one or more catalog files as input and makes them available on
port 5000 by default. You can view the data sources by opening your web browser
to `<http://localhost:5000>`_.

You can see the full description of the server command with:

::

  >>> intake-server --help
  usage: intake-server [-h] [-p PORT] [--sys-exit-on-sigterm] FILE [FILE ...]

  Intake Catalog Server

  positional arguments:
    FILE                    Name of catalog YAML file

    optional arguments:
      -h, --help            show this help message and exit
      -p PORT, --port PORT  port number for server to listen on
      --sys-exit-on-sigterm internal flag used during unit testing to ensure
                            .coverage file is written

To start the server with a local catalog file, use the following:

::

  >>> intake-server intake/catalog/tests/catalog1.yml
  Creating catalog from:
    - intake/catalog/tests/catalog1.yml
  catalog_args ['intake/catalog/tests/catalog1.yml']
  Entries: entry1,entry1_part,use_example1
  Listening on port 5000

You can use the catalog client (defined below) using:

::

  >>> intake list http://localhost:5000
  entry1
  entry1_part
  use_example1

Intake Client
-------------

While the Intake data sources will typically be accessed through the Python
API, you can use the client to verify a catalog file.

Unlike the server command, the client has several subcommands to access a
catalog. You can see the list of available subcommands with:

::

  >>> intake --help
  usage: intake {list,describe,exists,get,discover} ...

We go into further detail in the following sections.

List
''''

This subcommand lists the names of all available catalog entries. This is
useful since other subcommands require these names.

If you wish to see the details about each catalog entry, use the ``--full`` flag.
This is equivalent to running the ``intake describe`` subcommand for all catalog
entries.

::

  >>> intake list --help
  usage: intake list [-h] [--full] URI

  positional arguments:
    URI         Catalog URI

  optional arguments:
    -h, --help  show this help message and exit
    --full

::

  >>> intake list intake/catalog/tests/catalog1.yml
  entry1
  entry1_part
  use_example1
  >>> intake list --full intake/catalog/tests/catalog1.yml
  [entry1] container=dataframe
  [entry1] description=entry1 full
  [entry1] direct_access=forbid
  [entry1] user_parameters=[]
  [entry1_part] container=dataframe
  [entry1_part] description=entry1 part
  [entry1_part] direct_access=allow
  [entry1_part] user_parameters=[{'default': '1', 'allowed': ['1', '2'], 'type': u'str', 'name': u'part', 'description': u'part of filename'}]
  [use_example1] container=dataframe
  [use_example1] description=example1 source plugin
  [use_example1] direct_access=forbid
  [use_example1] user_parameters=[]


Describe
''''''''

Given the name of a catalog entry, this subcommand lists the details of the
respective catalog entry.

::

  >>> intake describe --help
  usage: intake describe [-h] URI NAME

  positional arguments:
    URI         Catalog URI
    NAME        Catalog name

  optional arguments:
    -h, --help  show this help message and exit

::

  >>> intake describe intake/catalog/tests/catalog1.yml entry1
  [entry1] container=dataframe
  [entry1] description=entry1 full
  [entry1] direct_access=forbid
  [entry1] user_parameters=[]


Discover
''''''''

Given the name of a catalog entry, this subcommand returns a key-value
description of the data source. The exact details are subject to change.

::

  >>> intake discover --help
  usage: intake discover [-h] URI NAME

  positional arguments:
    URI         Catalog URI
    NAME        Catalog name

  optional arguments:
    -h, --help  show this help message and exit

::

  >>> intake discover intake/catalog/tests/catalog1.yml entry1
  {'npartitions': 2, 'dtype': dtype([('name', 'O'), ('score', '<f8'), ('rank', '<i8')]), 'shape': (None,), 'datashape':None, 'metadata': {'foo': 'bar', 'bar': [1, 2, 3]}}


Exists
''''''

Given the name of a catalog entry, this subcommand returns whether or not the
respective catalog entry is valid.

::

  >>> intake exists --help
  usage: intake exists [-h] URI NAME

  positional arguments:
    URI         Catalog URI
    NAME        Catalog name

  optional arguments:
    -h, --help  show this help message and exit

::

  >>> intake exists intake/catalog/tests/catalog1.yml entry1
  True
  >>> intake exists intake/catalog/tests/catalog1.yml entry2
  False


Get
'''

Given the name of a catalog entry, this subcommand outputs the entire data
source to standard output.

::

  >>> intake get --help
  usage: intake get [-h] URI NAME

  positional arguments:
    URI         Catalog URI
    NAME        Catalog name

  optional arguments:
    -h, --help  show this help message and exit

::

  >>> intake get intake/catalog/tests/catalog1.yml entry1
         name  score  rank
  0    Alice1  100.5     1
  1      Bob1   50.3     2
  2  Charlie1   25.0     3
  3      Eve1   25.0     3
  4    Alice2  100.5     1
  5      Bob2   50.3     2
  6  Charlie2   25.0     3
  7      Eve2   25.0     3