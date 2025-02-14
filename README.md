[![Code style: black](https://img.shields.io/badge/code%20style-black-000000.svg)](https://github.com/ambv/black)
[![Build Status](https://travis-ci.org/pypa/bandersnatch.svg?branch=master)](https://travis-ci.org/pypa/bandersnatch)
[![codecov.io](https://codecov.io/github/pypa/bandersnatch/coverage.svg?branch=master)](https://codecov.io/github/codecov/codecov-python)
[![Documentation Status](https://readthedocs.org/projects/bandersnatch/badge/?version=latest)](http://bandersnatch.readthedocs.io/en/latest/?badge=latest)
[![Updates](https://pyup.io/repos/github/pypa/bandersnatch/shield.svg)](https://pyup.io/repos/github/pypa/bandersnatch/)
[![Downloads](https://pepy.tech/badge/bandersnatch)](https://pepy.tech/project/bandersnatch)

----

This is a PyPI mirror client according to `PEP 381`
http://www.python.org/dev/peps/pep-0381/.

## Installation

The following instructions will place the bandersnatch executable in a
virtualenv under `bandersnatch/bin/bandersnatch`.

- bandersnatch **requires** `>= Python 3.6.1`

### pip

This installs the latest stable, released version.

```
  $ python3.6 -m venv bandersnatch
  $ bandersnatch/bin/pip install bandersnatch
  $ bandersnatch/bin/bandersnatch --help
```

## Quickstart

* Run ``bandersnatch mirror`` - it will create an empty configuration file
  for you in ``/etc/bandersnatch.conf``.
* Review ``/etc/bandersnatch.conf`` and adapt to your needs.
* Run ``bandersnatch mirror`` again. It will populate your mirror with the
  current status of all PyPI packages.
  Current mirror package size can be seen here: https://pypi.org/stats/
* A ``blacklist`` or ``whitelist`` can be created to cut down your mirror size.
  Example blacklist generation tool: https://github.com/cooperlees/pypistats
* Run ``bandersnatch mirror`` regularly to update your mirror with any
  intermediate changes.

### Webserver

Configure your webserver to serve the ``web/`` sub-directory of the mirror.
For nginx it should look something like this::

```
    server {
        listen 127.0.0.1:80;
        server_name <mymirrorname>;
        root <path-to-mirror>/web;
        autoindex on;
        charset utf-8;
    }
```

* Note that it is a good idea to have your webserver publish the HTML index
  files correctly with UTF-8 as the charset. The index pages will work without
  it but if humans look at the pages the characters will end up looking funny.

* Make sure that the webserver uses UTF-8 to look up unicode path names. nginx
  gets this right by default - not sure about others.


### Cron jobs

You need to set up one cron job to run the mirror itself.

Here's a sample that you could place in `/etc/cron.d/bandersnatch`:

```
    LC_ALL=en_US.utf8
    */2 * * * * root bandersnatch mirror |& logger -t bandersnatch[mirror]
```

This assumes that you have a ``logger`` utility installed that will convert the
output of the commands to syslog entries.


### Maintenance

bandersnatch does not keep much local state in addition to the mirrored data.
In general you can just keep rerunning `bandersnatch mirror` to make it fix
errors.

If you want to force bandersnatch to check everything against the master PyPI::

* run `bandersnatch mirror --force-check` to move status files if they exist in your mirror directory in order get a full sync.

Be aware, that full syncs likely take hours depending on PyPIs performance and
your network latency and bandwidth.

### Operational notes

#### Case-sensitive filesystem needed

You need to run bandersnatch on a case-sensitive filesystem.

OS X natively does this OK even though the filesystem is not strictly
case-sensitive and bandersnatch will work fine when running on OS X. However,
tarring a bandersnatch data directory and moving it to, e.g. Linux with a
case-sensitive filesystem will lead to inconsistencies. You can fix those by
deleting the status files and have bandersnatch run a full check on your data.

#### Many sub-directories needed

The PyPI has a quite extensive list of packages that we need to maintain in a
flat directory. Filesystems with small limits on the number of sub-directories
per directory can run into a problem like this::

  2013-07-09 16:11:33,331 ERROR: Error syncing package: zweb@802449
  OSError: [Errno 31] Too many links: '../pypi/web/simple/zweb'

Specifically we recommend to avoid using ext3. Ext4 and newer does not have the
limitation of 32k sub-directories.

#### Client Compatibility

A bandersnatch static mirror is compatible only to the "static",  cacheable
parts of PyPI that are needed to support package installation. It does not
support more dynamic APIs of PyPI that maybe be used by various clients for
other purposes.

An example of an unsupported API is PyPI's XML-RPC interface, which is used
when running `pip search`.

### Contact

If you have questions or comments, please submit a bug report to
https://github.com/pypa/bandersnatch/issues/new

### Code of Conduct

Everyone interacting in the bandersnatch project's codebases, issue trackers,
chat rooms, and mailing lists is expected to follow the
[PyPA Code of Conduct](https://www.pypa.io/en/latest/code-of-conduct/).


### Kudos

This client is based on the original pep381client by *Martin v. Loewis*.

*Richard Jones* was very patient answering questions at PyCon 2013 and made the
protocol more reliable by implementing some PyPI enhancements.

*Christian Theune* for creating and maintaining `bandersnatch` for many years!
