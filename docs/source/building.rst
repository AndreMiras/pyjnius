.. _building:

Building PyJNIus
================

Building PyJNIus is necessary for development purposes, or if there is no 
pre-built binary for your particular platform. 

Building PyJNIus requires a  `Java Development Kit
<https://www.oracle.com/java/technologies/downloads/>`_  (JDK)
to be installed (NB: installing pre-built wheels requires only a JRE).

Apart from the JDK, the build requirements for each platform are as follows:

 - Linux: the `GNU Compiler Collection <https://gcc.gnu.org/>`_ (GCC), e.g. using 
   ``apt-get install build-essential`` on Debian-based distributions.
 - Windows: `Microsoft Visual C++ Build Tools <https://visualstudio.microsoft.com/downloads/>`_ 
   (the command-line tools subset of Visual Studio). 
   For more information or options, see Python's `Windows Compilers wiki
   <https://wiki.python.org/moin/WindowsCompilers>`_.
 - macOS: `Xcode command-line tools <https://mac.install.guide/commandlinetools/index.html>`_.

In all cases, after checking out PyJNIus from GitHub, it can be *built* and installed using::

     pip install .

This installs `Cython <https://cython.org/>`_ (as specified in the 
`pyproject.toml <https://pip.pypa.io/en/stable/reference/build-system/pyproject-toml/>`_) 
in a Python build environment. On all platforms, if PyJNIus cannot find your JDK, you can set 
the `JAVA_HOME` environment variable (this is often needed on Windows).

If you want to compile the PyJNIus extension within the directory for any development,
just type::

    make

You can run the test suite to make sure everything is running correctly::

    make tests

In these cases, you may need to have `Cython <https://pypi.org/project/Cython/>`_ 
and `pytest <https://pypi.org/project/pytest/>`_ installed in the current Python environment.

When building for Android, the host application's build system compiles the
Java bridge code into the app. The PyJNIus wheel itself contains no Java code
(see :ref:`android-wheel`).

For all other platforms, the Java bridge classes target Java 8 language
features, platform APIs, and class-file format. Builds with JDK 9 or newer use
``javac --release 8``; JDK 8 builds use ``-source 8 -target 8`` against that
JDK's APIs.

The Ant-based developer build requires Ant 1.9.8 or newer so both helper and
test-fixture compilation apply the corresponding release constraint.

Desktop wheel CI
----------------

For build-tool changes, include ``[build wheel]`` in the PR title before opening
the PR or pushing another commit to exercise the release-shaped builds and tests.
Editing the title alone does not trigger a new run.
The existing pytest suite runs against the installed wheel, with only Java test
fixtures compiled separately so the packaged helpers are used. Wheel-test
failures block tag releases. Java 8 compilation targeting complements this
installed-wheel runtime check.

The CPython 3.14 wheel jobs select these older-runtime references, using full
JDKs for fixture compilation:

.. list-table:: CPython 3.14 wheel runtime references
   :header-rows: 1

   * - Platform
     - Runtime JDK
   * - Windows x86 (win32)
     - Zulu 8
   * - Windows x64, Linux x86_64/aarch64, Intel macOS
     - Temurin 8
   * - Native Apple Silicon macOS
     - Temurin 11

The other main-matrix Python versions (3.11–3.13) use Java 17; Win32 runs only
the representative 3.14 job. Each job uses pip's compatible-wheel selection:
normally thin x86_64 on Intel macOS and universal2 on native ARM. Universal2's
Intel slice is not separately exercised.

Build compilers are independent of these runtime inputs and the Java 8 helper
target. Host builds use the ``build_java`` / ``build_distribution`` selectors in
``create.yml``, with a Zulu 17 override for Win32; Linux wheels compile with
container OpenJDK 11. Zulu also supplies the Win32 source-CI JDKs.

For applications, use a current maintained LTS suited to your platform and Java
dependencies: currently 25 on the available 64-bit reference platforms. Java 25
has source-CI coverage; wheel-runtime checks are the combinations above.
Application Java dependencies may require a higher minimum. Coverage remains
representative rather than an exhaustive Python/JVM/vendor/platform/artifact
guarantee; user-reported incompatibilities can receive targeted regressions.
