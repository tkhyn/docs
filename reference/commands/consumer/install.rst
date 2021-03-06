.. _conan_install_command:

conan install
=============

.. code-block:: bash

    $ conan install [-h] [-g GENERATOR] [-if INSTALL_FOLDER] [-m [MANIFESTS]]
                    [-mi [MANIFESTS_INTERACTIVE]] [-v [VERIFY]]
                    [--no-imports] [-u] [-pr PROFILE] [-r REMOTE]
                    [-o OPTIONS] [-s SETTINGS] [-e ENV]
                    [-b [BUILD [BUILD ...]]]
                    path

Installs the requirements specified in a conanfile (.py or .txt). If any
requirement is not found in the local cache it will retrieve the recipe from a
remote, looking for it sequentially in the available configured remotes. When
the recipes have been downloaded it will try to download a binary package
matching the specified settings, only from the remote from which the recipe
was retrieved. If no binary package is found you can build the package from
sources using the '--build' option. When the package is installed, Conan will
write the files for the specified generators. It can also be used to install a
concrete recipe/package specifying a reference in the "path" parameter.

.. code-block:: bash

    positional arguments:
      path                  path to a folder containing a recipe (conanfile.py or
                            conanfile.txt) or to a recipe file. e.g.,
                            ./my_project/conanfile.txt. It could also be a
                            reference

    optional arguments:
      -h, --help            show this help message and exit
      -g GENERATOR, --generator GENERATOR
                            Generators to use
      -if INSTALL_FOLDER, --install-folder INSTALL_FOLDER
                            Use this directory as the directory where to put the
                            generatorfiles, conaninfo/conanbuildinfo.txt etc.
      -m [MANIFESTS], --manifests [MANIFESTS]
                            Install dependencies manifests in folder for later
                            verify. Default folder is .conan_manifests, but can be
                            changed
      -mi [MANIFESTS_INTERACTIVE], --manifests-interactive [MANIFESTS_INTERACTIVE]
                            Install dependencies manifests in folder for later
                            verify, asking user for confirmation. Default folder
                            is .conan_manifests, but can be changed
      -v [VERIFY], --verify [VERIFY]
                            Verify dependencies manifests against stored ones
      --no-imports          Install specified packages but avoid running imports
      -u, --update          check updates exist from upstream remotes
      -pr PROFILE, --profile PROFILE
                            Apply the specified profile to the install command
      -r REMOTE, --remote REMOTE
                            look in the specified remote server
      -o OPTIONS, --options OPTIONS
                            Define options values, e.g., -o Pkg:with_qt=true
      -s SETTINGS, --settings SETTINGS
                            Settings to build the package, overwriting the
                            defaults. e.g., -s compiler=gcc
      -e ENV, --env ENV     Environment variables that will be set during the
                            package build, -e CXX=/usr/bin/clang++
      -b [BUILD [BUILD ...]], --build [BUILD [BUILD ...]]
                            Optional, use it to choose if you want to build from
                            sources: --build Build all from sources, do not use
                            binary packages. --build=never Never build, use binary
                            packages or fail if a binary package is not found.
                            --build=missing Build from code if a binary package is
                            not found. --build=outdated Build from code if the
                            binary is not built with the current recipe or when
                            missing binary package. --build=[pattern] Build always
                            these packages from source, but never build the
                            others. Allows multiple --build parameters. "pattern"
                            is a fnmatch file pattern of a package name. Default
                            behavior: If you dont specify anything, it will be
                            similar to --build=never, but package recipes can
                            override it and decide to build with "build_policy"


:command:`conan install` executes methods of a *conanfile.py* in the following order:

1. ``config_options()``
2. ``configure()``
3. ``requirements()``
4. ``package_id()``
5. ``package_info()``

Note this describes the process of installing a pre-built binary package and not building with :command:`--build`.

**Examples**

- Install a package requirement from a ``conanfile.txt``, saved in your current directory with one
  option and setting (other settings will be defaulted as defined in
  ``<userhome>/.conan/profiles/default``):

  .. code-block:: bash

      $ conan install . -o PkgName:use_debug_mode=on -s compiler=clang

  .. note::

      You have to take into account that **settings** are cached as defaults in the
      **conaninfo.txt** file, so you don't have to type them again and again in the
      **conan install** or **conan create** commands.

      However, the default **options** are defined in your **conanfile**.
      If you want to change the default options across all your **conan install** commands, change
      them in the **conanfile**. When you change the **options** on the command line, they are only
      changed for one shot. Next time, **conan install** will take the **conanfile** options as
      default values, if you don't specify them again in the command line.

- Install the **OpenCV/2.4.10@lasote/testing** reference with its default options and default
  settings from ``<userhome>/.conan/profiles/default``:

  .. code-block:: bash

      $ conan install opencv/2.4.10@lasote/testing

- Install the **OpenCV/2.4.10@lasote/testing** reference updating the recipe and the binary package
  if new upstream versions are available:

  .. code-block:: bash

      $ conan install opencv/2.4.10@lasote/testing --update

.. _buildoptions:

build options
-------------

Both the conan **install** and **create** commands have options to specify whether conan should try
to build things or not:

* :command:`--build=never`: This is the default option. It is not necessary to write it explicitly.
  Conan will not try to build packages when the requested configuration does not match, in which
  case it will throw an error.
* :command:`--build=missing`: Conan will try to build from source, all packages of which the
  requested configuration was not found on any of the active remotes.
* :command:`--build=outdated`: Conan will try to build from code if the binary is not built with the
  current recipe or when missing binary package.
* :command:`--build=[pattern]`: A fnmatch file pattern of a package name. e.j ``zl*`` will match
  ``zlib`` package. Conan will force the build of the packages, the name of which matches the given
  **pattern**. Several patterns can be specified, chaining multiple options,
  e.g. :command:`--build=pattern1 --build=pattern2`.
* :command:`--build`: Always build everything from source. Produces a clean re-build of all packages
  and transitively dependent packages

env variables
-------------

With the :command:`-e` parameters you can define:

- Global environment variables (:command:`-e SOME_VAR="SOME_VALUE"`). These variables will be defined
  before the `build` step in all the packages and will be cleaned after the `build` execution.
- Specific package environment variables (:command:`-e zlib:SOME_VAR="SOME_VALUE"`). These variables will
  be defined only in the specified packages (e.g. zlib).

You can specify this variables not only for your direct ``requires`` but for any package in the
dependency graph.

If you want to define an environment variable but you want to append the variables declared in your
requirements you can use the [] syntax:

.. code-block:: bash

    $ conan install . -e PYTHONPATH=[/other/path]

This way the first entry in the PYTHONPATH variable will be :command:`/other/path` but the PYTHONPATH values
declared in the requirements of the project will be appended at the end using the system path
separator.

settings
--------

With the :command:`-s` parameters you can define:

- Global settings (:command:`-s compiler="Visual Studio"`). Will apply to all the requires.
- Specific package settings (:command:`-s zlib:compiler="MinGW"`). Those settings will be applied only to
  the specified packages.

You can specify custom settings not only for your direct ``requires`` but for any package in the
dependency graph.

options
-------

With the :command:`-o` parameters you can only define specific package options.

.. code-block:: bash

    $ conan install . -o zlib:shared=True
    $ conan install . -o zlib:shared=True -o bzip2:option=132
    # you can also apply the same options to many packages with wildcards:
    $ conan install . -o *:shared=True

.. note::

    You can use :ref:`profiles <profiles>` files to create predefined sets of **settings**,
    **options** and **environment variables**.
