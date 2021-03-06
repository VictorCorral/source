KBOX is a set of common, Linux console utilities ported to Android, in such a
way that the user is presented with a reasonably Linux-like command-line
environment. It works without rooting or any other warrany-busting strategies.
Packages available so far include heavyweights like Perl, Python 3, and gcc;
but many simpler utilties are also available. KBOX is designed to be installed
on top of an existing Android terminal emulator, using a relatively simple
command-line process. After installing the base system, further packages can be
downloaded and installed.
<p/>
This latest release of KBOX, version 4, offers the same utilities that were available
in version 3, but the build process has been completely redesigned, to separate
the source of the build from the source of the utilties themselves. People who have
wanted to contribute to the KBOX effort have frequently asked if they could have
the "source for KBOX"; until now, however, there has been no such thing. 
<p/>
The main end-user documentation for KBOX is here:
<p/>
http://kevinboone.net/kbox.html
<p/>
This document provides technical information for people who might want to get 
involved in developing the project.
<p/>
<h2>Structure of this archive</h2>

This archive consists of two main parts -- the directories <code>base</code> and 
<code>packages</code>. <code>base</code> contains the parts needed to build the base installer, whilst
"packages" consists of the build scripts for individual packages, one per directory.
Each directory, in either section, contains a script <code>build.sh</code>
that does the build,
based on settings in <code>env.sh</code>. This defines the important build parameters --
the location of the C compiler, where downloaded tarballs are to be cached,
where they are to be unpacked, etc. It should not be necessary, in practice,
to change anything outside of env.sh to build an existing package.
<p/>
The base installer section consists of three directories, each with its own
build script. The scripts must be run in the right order, so there is a 
"master" build script in <code>base/</code>. This won't build any packages -- 
at present these must be built individually, and there is a proper order
in which to build them (see the file doc/build_order.txt).

<h2>Prerequisites</h2>

To build the whole KBOX system requires a fairly substantial set of development 
tools, and it's difficult to predict exactly what they all are -- anything that
was installed on my workstation before I started work on KBOX, 
I probably wouldn't even know about.
See <code>doc/prerequisites.txt</code> for a list of dependencies that I'm sure
about.
<p/>
Most obviously you'll need C/C++ compilers and all the usual, associated tools
-- both for compiling code for the Android target, and for the build machine.
For the Android target, it's probably best to use the official Android NDK from
Google, as it has the (more or less) correct libraries and headers in place from the
start. At present, I am using release r14b. You can expect, sadly, to have to
change things in the code if a different version is used. It will be necessary
to create a "standalone toolchain" from the NDK for each Android architecture (ARM,
x86) to be supported.

<h2>The build process</h2>

Each build script carries out the same basic set of steps.
<p/>
1. Retrieves the source code from (ideally) a repository, and caches it in the
location indicated by $STAGING/tarballs. In a very few cases the source is 
unpacked from a tarball bundled into this archive. I have only done this where I can
no longer find a reliable on-line repository for the code.
<p/>
2. Unpacks the source into a directory under $STAGING, unless that directory already 
exists. Note that no checks are made at this stage beyond that the directory 
exists. The files in it might be completely mangled by previous unsuccessful build
attemps, but they won't be restored. To clean the build, you will need to delete
the directory in $STAGING. Note that everything in $STAGING is intended to be ephemeral --
it can all be recreated by running the build scripts. Any changes made in this area
during the course of making an application work must be converted into patches
that can be applied when the source is next unpacked.
<p/>
3. Patches the <code>configure</code> script, or similar, if there is one, and
if it is necessary. This is only necessary in a few cases, when the configuration
script won't run at all. 
<p/>
4. Runs <code>configure</code> (or similar) with the appropiate arguments for
cross-compiling.
<p/>
5. If necessary patches the Makefiles generated by <code>configure</code>. The
usual reason for needing to do this is because either (a) <code>configure</code>
has used information <i>from</i> the build system, rather than <i>about</i>
the target system (which it has no way of knowing); or (b) the Google
NDK headers and libraries don't match, so functions seem to be present when
they aren't.
<p/>
6. Patches the source code. This is nearly always necessary. Sometimes the
patches are minor, but sometimes they are almost applications in their
own right (Perl).
<p/>
7. Runs <code>make</code>, or equivalent, sometimes with specific 
environment settings.
<p/>
8. Runs <code>make install</code>, with environment settings to direct
the output to a directory called <code>image</code> in the build
directory.
<p/>
9. Builds an IPK package from the <code>image</code> directory. A
working directory called <code>out</code> is created during this process.
<p/>
In practice, some packages require significant departures from this process.
Python 3, for example, has to be built completely twice over, because the
loadable modules require different build settings from the main binary.
<p/>
10. Populates the Android NDK sysroot. This applies to libraries. Once build,
the libraries static objects and C/C++ headers are copied into the sysroot.
This way they are available to be used by packages built later.
Because of these dependencies, there is a recommended build order
(see doc/build_order.txt).
<p/>
11. Copies the final package to a location identified by <code>$DIST</code>.

<h2>Packages missing from this archive</h2>

Not everything available for the KBOX distribution can be built using these
scripts. Some packages were contributed in more-or-less complete form, using 
methods that only their contributors know. In some cases, they were build 
on an Android device -- this process has a lot to recommend it, but it doesn't
lend itself to centralizing in a repository.


<h2>Build nasties</h2>

The build scripts do make some attempt to stop if a particular step fails.
However, it isn't always possible to tell when it has failed, and you have
to be prepared for the fact that some scripts will carry on uselessly after
a failure. Worse, some parts of some builds actually fail, but we still get
a usable build (because, for example, we don't need all of it). In such 
cases the script will always carry on because it can't tell a "good" failure 
from a "bad" one.
<p/>
Some build scripts require patches to be applied to the source. In some cases,
these patches cannot be applied twice. So if a build fails, it may be fruitless
to run the build script again after fixing whatever the problem was -- the source
may no longer be patchable. In such cases, it is necessary to restart the whole
build by deleting the build directory in $STAGING.













