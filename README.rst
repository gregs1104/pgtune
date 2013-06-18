Introduction
============

pgtune takes the wimpy default postgresql.conf and expands the database 
server to be as powerful as the hardware it's being deployed on.

Installation/Usage
==================

Source installation
-------------------

There is no need to build/compile pgtune, it is a Python script.
Extracting the tarball to a convenient location is sufficient.
Note that you will need the multiple
pg_settings-<version>_<architecture> files included with the
program too, pgtune can't work without those.

RPM Installation
----------------

The RPM package installs:

 * The pgtune binary under/usr/bin
 * Documents in /usr/share/doc/pgtune-$version
 * Setting files in /usr/share/pgtune

Using pgtune
============

pgtune works by taking an existing postgresql.conf file as an input,
making changes to it based on the amount of RAM in your server and
suggested workload, and output a new file.

Here's a sample usage::

  pgtune -i $PGDATA/postgresql.conf -o $PGDATA/postgresql.conf.pgtune

pgtune --help will give you additional usage information.  These
are the current parameters:

 * -i or --input-config : Specifies the current postgresql.conf file.

 * -o or --output-config : Specifies the file name for the new 
   postgresql.conf file.

 * -M or --memory: Use this parameter to specify total system memory. If 
   not specified, pgtune will attempt to detect memory size.

 * -T or --type : Specifies database type. Valid options are:
   DW, OLTP, Web, Mixed, Desktop

 * -P or --platform : Specifies platform, defaults to the platform running
   the program.  Valid options are Windows, Linux, and Darwin (Mac OS X).

 * -c or --connections: Specifies number of maximum connections expected.
   If not specified, it depends on database type.

 * -D or --debug : Enables debugging mode. 

 * -S or --settings: Directory where settings data files are located at.
   Defaults to the directory where the script is being run from.  The
   RPM package includes a patch to use the correct location these
   files were installed into.

Todo
====

A TODO list is included in the tarball.  There are also some TODO
items marked in the source code itself.

Bugs
====

There aren't any known bugs, besides the cleanup areas mentioned
in the source code with TODO tags.  These shouldn't impact use of
the program.  If you find a bug, there is a tracker on the pgfoundry
page for pgtune listed below.

Documentation
=============

The documentation README.rst for pgtune is in ReST markup.  Tools
that operate on ReST can be used to make versions of it formatted
for other purposes, such as rst2html to make a HTML version.

Contact
=======

 * The project is hosted at http://pgfoundry.org/projects/pgtune
 * Initial commits are done in the git reportiory at
   http://git.postgresql.org/git/pgtune.git and
   http://github.com/gregs1104/pgtune

If you have any hints, changes or improvements, please contact:

 * Greg Smith greg@2ndQuadrant.com

License
=======

pgtune is licensed under a standard 3-clause BSD license.

Copyright (c) 2009-2013, Gregory Smith
All rights reserved.

Redistribution and use in source and binary forms, with or without 
modification, are permitted provided that the following conditions are 
met:

  * Redistributions of source code must retain the above copyright 
    notice, this list of conditions and the following disclaimer.
  * Redistributions in binary form must reproduce the above copyright 
    notice, this list of conditions and the following disclaimer in 
    the documentation and/or other materials provided with the 
    distribution.
  * Neither the name of the author nor the names of contributors may 
    be used to endorse or promote products derived from this 
    software without specific prior written permission.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS 
IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED 
TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A 
PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT 
HOLDER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, 
SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT
LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, 
DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY 
THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
(INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE 
OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

