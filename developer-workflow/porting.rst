================
Platform support
================

Python depends on the underlying *platform*: a term that encompasses things
like the operating system, processor architecture, lower-level libraries such
as ``libc``, ``zlib`` or ``libffi``.

The core team maintains Python for platforms listed in :pep:`11`.
Only these are officially tested and supported by “python.org”.

However, CPython still serves as a hub for third-party projects that provide
their own support for other platforms.


Guidelines for CPython core devs and triagers
=============================================

Our official policy is in :pep:`11`:

   Support for unsupported platforms may be partial within the code base, such as
   from active development around new platform support or accidentally.

   Code changes to platforms not listed in the [PEP 11] tiers may be rejected
   or removed from the code base without a deprecation process if they
   cause a maintenance burden or obstruct general improvements.

The PEP is intentionally rather vague. This guide supplements it, informally,
with the following best practices.

Code that only targets unsupported platforms should be kept minimal
and generic.
Pragmatically, platform-specific logic is better left to a project that
can test it.

However, we don't unnecessarily remove working code. Some valid reasons
to remove something are:

- We have some indication that it doesn't work as intended, and the fix isn't
  trivial.
- We're transferring platform-specific code to a third-party project,
  with that project's cooperation.

Pull requests that only fix unsupported platforms, if accepted, are
generally not backported to maintenance branches.

However, *tests* that should also pass on supported platforms
generally are accepted and backported, even if the test is redundant
or trivial on the supported platforms.
(Failing tests can alert maintainers of *other* affected platforms, and test
comments should link to an issue or discussion wher fixes can be coordinated.)

Where reasonable, we welcome changes to better support relevant standards
(mainly, C and POSIX), even when all supported platforms share a particular
implementation detail that could simplify CPython code.

.. note::

   For example: on all supported platforms, :mod:`errno` codes are positive,
   but POSIX doesn't require that, and some platforms use negative values.
   While we cannot test on such platforms, Python should not rely on
   ``errno`` being positive, and we generally accept related
   improvements or fixes.
   Relevant tests -- but not necessarily the fixes themselves -- should be
   backported to bugfix branches.

Issues filed for unsupported platforms should get the :gh-label:`OS-unsupported`
label, and may be closed (as "not planned") at maintainer discretion.
Applying the label will put the issue in a `dedicated GitHub project <https://github.com/orgs/python/projects/27>`_,
where it can later be grouped by platform.

When labeling or closing the issue, or adding platform metadata in the project,
please make sure any relevant :ref:`platform-experts <experts>` are mentioned.


.. _unsupported-platform-projects:

Guidelines for third-party ports and redistributors
===================================================

If you support Python on some platform: thank you!

While supporting can mean as little as helping others with their own builds,
this section is mainly meant for projects that patch, build and
*distribute* Python on "their" platform.
This can be a variant of a CPython-supported platform: for example, Debian
maintains a variant of the tier-1-supported “Linux on ``x86_64``”.

Since we don't backport fixes for unsupported platforms, your project
should have a means to track and apply any necessary patches.
This can be a fork of the CPython repository on GitHub, for example.

If your project either welcomes public contributions *or* provides an
official build of Python on a proprietary platform, you are
welcome to list the project and your username in the table at
:ref:`platform-experts`.


.. _porting:

Porting to a new platform
=========================

The first step is to familiarize yourself with the development toolchain on
the platform in question, notably the C compiler. Make sure you can compile and
run a hello-world program using the target compiler.

Next, learn how to compile and run the Python interpreter on a platform to
which it has already been ported; preferably Unix, but Windows will
do, too. The build process for Python, in particular the ``Makefile`` in the
source distribution, will give you a hint on which files to compile
for Python.  Not all source files are relevant: some are platform-specific,
and others are only used in emergencies (for example, ``getopt.c``).

It is not recommended to start porting Python without at least a medium-level
understanding of your target platform; how it is generally used, how to
write platform-specific apps, and so on. Also, some Python knowledge is required, or
you will be unable to verify that your port is working correctly.

You will need a ``pyconfig.h`` file tailored for your platform.  You can
start with ``pyconfig.h.in``, read the comments, and turn on definitions that
apply to your platform.  Also, you will need a ``config.c`` file, which lists
the built-in modules you support.  Again, starting with
``Modules/config.c.in`` is recommended.

Finally, you will run into some things that are not supported on your
target platform.  Forget about the ``posix`` module in the beginning. You can
simply comment it out of the ``config.c`` file.

Keep working on it until you get a ``>>>`` prompt.  You may have to disable the
importing of ``site.py`` by passing the ``-S`` option. When you have a prompt,
bang on it until it executes very simple Python statements.

At some point you will want to use the ``os`` module; this is the time to start
thinking about what to do with the ``posix`` module.  It is okay to simply
comment out functions in the ``posix`` module that cause problems; the
remaining ones will be quite useful.

Before you are done, it is highly recommended to run the Python regression test
suite, as described in :ref:`runtests`.
