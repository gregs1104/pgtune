Features targeted for 1.0 release
=================================

Settings information
--------------------

* Validate against min/max - stub

  * What's really needed here is a clamp function that limits based
    on both the maximum setting possible as well as a tuning model
    maximum.  For example, the tuning model max for wal_buffers is
    16MB, while the tuning model max for shared_buffers on Windows
    is 512MB.  The system max matters too.  On a system with a large
    amount of RAM, the suggested value for work_mem might easily be set
    to bigger than its server maximum (2GB) even though there's no
    tuning model max intended for it.

* Handle exponential max values (random_page_cost max is 1.79769e+308)
* Make memory setting (-M) support all database memory input formats
* Show original value of tuned parameters in output
* Figure out how to deal with delimiters - stub

Limits on parameters
--------------------

* Limit shared_buffers on all platforms to 8GB
* Limit work_mem, maintenance_work_mem to 2GB (server max)
* Warn when total RAM <256MB (stub)
* Update 8.4 settings files, default_statistics_target should be 100
* Add 9.0 32-bit settings file
* Add 9.1, 9.2, 9.3 settings files

Model Improvements
------------------

* Current suggestions for work_mem are much too aggressive in
  many cases.  More reports of using too much memory than expected.
  Either more multi-node sorting may be going on, or footprint of
  other applications is larger then expected.
* Estimate shared memory used based on table in PostgreSQL docs
* Update model to something more sophisticated

  * Account for all RAM
  * Target using 1/4 for shared_buffers, up to 1/2 for work_mem, and
    typically an additional 1/2 of the OS buffer cache
  * If shared_buffers hits one of the limits, that will change how
    the rest of the tuning works out

Platform specific details
-------------------------

* Output sysctl.conf suggestion; see notes below.

  * Ideally based on estimated memory use, as described in section below
  * Initially just assuming a 50% of memory target for the memory block
    would work

* Additional program inputs

  * PostgreSQL version and platform, to select the right pg_settings file

  * Platform may be possible to detect; see notes below.

* Allow overriding the OS used when generating sysctl output
* Suggest size to reserve for application
* Look for postgresql.conf file using PGDATA

Documentation
-------------

* Describe all the types clearly, refer to docs in usage notes
* Suggest where the settings are still conservative
* Data warehouse:  more stats, turn off auto vacuum
* Does not account for application overhead, can reduce total mem if that's the case
* May not correctly parse existing values if you have manually increased either BLCKSZ or XLOG_BLCKSZ; should 
  still set values correctly though.
* Warn that minimal systems should just use what comes out of initdb

Changes to backport to stable release
-------------------------------------

* 8.4 settings file fixes
* constraint_exclusion change
* default_statistics_target detuning
* Nicer header

Implementation ideas
====================

This section mainly consists of ideas on how to build features planned
for the 1.0 release.

Estimating shared memory usage
------------------------------

http://www.postgresql.org/docs/current/static/kernel-resources.html

* Introduce a platform structure passed around everywhere that
  abstracts away OS, memory, version, PostgreSQL version, etc.
* Introduce an "info" option that will show all the information in there,
  including anything auto-detected.  This will make testing/debugging
  much easier.

Hint interface
--------------

One option for displaying suggestions is a hint interface that goes to
standard error.  Sample hints::

  # HINT:  Increase SHMMAX
  # HINT:  You won't be able to connect to this database over TCP/IP with your listen_address setting
  # HINT:  The value for max_connections is being reduced from $X to $Y
  # HINT:  Expected maximum memory use for this configuration:  $X (pretty printed)
  # HINT:  Autovacuum is off  
  # HINT:  If you aren't using partitions, you can improve query planning time by turning constraint_exclusion off
  # HINT:  Windows doesn't handle large numbers of connections
  # HINT:  Consider a conneciton pooler

Would be nice to make all the hints show up as comments in case they are
mixed together though.  That will make the SHMMAX example much less useful
though.  That may not matter given the ideas around automating sysctl
generation.

Python system information
-------------------------

* The platform bit width (32 or 64) should be detectable.
* System parameter SC_INT_MAX will tell us what kind of platform we're on
* The number of processors may be possible too.
* Bit width alternately available via platform information
* Look at the size of a pointer

Example system parameters::

  SC_INT_MAX 2147483647
  SC_NPROCESSORS_CONF 4

Python review suggestions
-------------------------

Line numbers here refer to an earlier version of the code now.

* from ctypes import * ( line 18 ) makes the block difficult to read and
  pollutes the namespace.

* The doc strings ( 59, 136, 206 ) don't follow standard conventions,
  described here http://www.python.org/dev/peps/pep-0257/.

* Functions also support doc strings ( 342, 351, etc. )

Future version ideas
====================

Reorganize with include files
-----------------------------

Provide a useful example of how to put the pgtune customization as something
included by the main postgresql.conf.  Starting in 9.3, this might be done
as a config directory instead.

V2.0 features
-------------

  Wizard to ask questions
  Real GUI

Improved compatibility features
-------------------------------

These are all considered lower priority than the other features outlined
here.  Compatibility with older/odd systems is hard to justify working on
relative to how much benefit it provides.

* Add 8.3, 8.2, 8.1 compatibility
* Set FSM parameters - needs an idea how big the database is
* Include an option to autodetect PG version.  This likely needs
  a series of sample postgresql.conf files from each version, to figure
  out which the input file is most like.
* Extend model to work properly on systems with smaller amounts of RAM aimed at a small number of users
* Set max_prepared_transactions
* List of parameters not to mess with (collate, archive_command) which
  may be needed for more advanced generation ideas

Tuning Free Space Map settings
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Only useful for adding PostgreSQL 8.3 and earlier versions.
The FSM stuff is not be necessary if targeting 8.4.  Values:

* web:     max_fsm_pages = DBsize / PageSize / 8
* oltp:    max_fsm_pages = DBsize / PageSize / 8
* Mixed:   max_fsm_pages = DBsize / PageSize / 8
* Desktop: max_fsm_pages = DBsize / PageSize / 8
* DW:      max_fsm_pages = DBsize / PageSize / 32

The DW case is different because they tend to insert and delete data
less frequently than the other types, leaving behind less free space
to be re-used.

Ideas for a config regeneration program
---------------------------------------

There are many settings in the postgresql.conf that are commented out.
This makes the file harder to navigate than it might be.  One idea for
improving this situation is to have pgtune remove lines that aren't
necessary.  A second is to support generating a configuration file
from scratch, based on templates supplies for each version.

A full configuration generator might support the following switches:

* -b , --basic — short conf file, listing only the 15-18 most commonly changed options
* -a , --advanced — conf file listing all 196+ options
* -t, --terse — conf file lists only category headings and actual settings, no comments
* -n, --normal — conf file has category and subcategory settings, with short, descriptive comments
* -v, --verbose — conf file lists full descriptions and recommendations in comments with each option
* -c "option = value" set specific option to specific value in the file
* -f "filename" — take options and values from file "filename".  This allows the program
  to handle the difficult settings manipulation part for a custom settings set suggested by
  a different tool.

The default would be "-b -n", with specific settings for shared_buffers. 
The current postgresql.conf is a lot more like a "-a -v" file.

The challenging part of generating a new file from scratch is getting
all of the locale and shared memory settings right, it would have to
duplicate much of the work that initdb handles to do that.  And in
the case where pgtune tried to remove the useless comments, it really
needs a sample postgresql.conf file from each version, to figure out
which lines are boilerplate from there and which are user comments.

Setup common idioms
-------------------

Several types of postgresql.conf changes happen as common sets of
changes that could be automated:

* Warning about listen_addresses if it's not set to '*'

  * Add an input parameter to allow setting it, too

* Configure logging for performance monitoring
* Adjust logging format for query analysis (pgfouine compatibility)
* Setup SSL
* Good syslog setup and practices
* Database managed log files with weekly rotation

Notes on workload types
-----------------------

The specific elements of a "DW" use-case aren't necessarily tied to
size.  They are:

* Data comes in in large batches rather than individual rows
* Small numbers of users
* Large complex queries

A database which is only 15GB might still show solid DW behavior, where
you want to keep max_connections to < 20 and even turn autovaccum off.

Internals information
=====================

Parsing Input Units
-------------------

This describes how input units are handled in the program.
It's based the logic used by the database in its GUC system.

parse_int is the internal routine there

kB MB and GB are the accepted units

Some parameters are "GUC_UNIT_MEMORY"; these are the ones this logic applies to
  These are ones where the unit name ends with kB

Raw integers are considered in kb unless they are blocksz or xlog_blcksz
ones.  A few constants do the conversions::

  #define KB_PER_MB (1024)
  #define KB_PER_GB (1024*1024)

* kB:  Divided by (unit size)/kb (typically =8) to get kB
* MB:  Multplied by KB_PER_MB , divided as above
* GB:  Multiplied by KB_PER_GB

There are also unit of time variables, don't care about those right now

This is the logic that maps the block size stuff into the units field::

                        case GUC_UNIT_KB:
                                values[2] = "kB";
                        case GUC_UNIT_BLOCKS:
                                snprintf(buf, sizeof(buf), "%dkB", BLCKSZ / 1024);
                        case GUC_UNIT_XBLOCKS:
                                snprintf(buf, sizeof(buf), "%dkB", XLOG_BLCKSZ / 1024);

So I don't have to worry about that; I can just use the unit size as kB

For booleans, on and off are the officially supported version of those values, but many others
are accepted too.
