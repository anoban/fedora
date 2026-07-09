## ___Getting started with `micromamba` on Fedora 44___
--------------------

[`micromamba`](https://mamba.readthedocs.io/en/latest/installation/micromamba-installation.html) is a fully statically linked, standalone `Python` environment manager, which also manages the native dependencies needed by the `Python` environment.

This is the most bare bones system compared to `mamba`, `miniconda` and `conda`. And the best part is it is a `C++` codebase that does not need a `Python` interpretor to run (unlike `conda` & `mamba`). Hence it is small, fast and self contained.

Like `conda` based systems, `micromamba` does not require projects to contain the virtual environments within their directory structure. So, multiple projects can share a single `Python` virtual environment.

-----------------------

1. To install `micromamba` on x86_64 linux, follow the instructions on the [official website](https://mamba.readthedocs.io/en/latest/installation/micromamba-installation.html) or run:
`curl -Ls https://micro.mamba.pm/api/micromamba/linux-64/latest | tar -xvj bin/micromamba`.

2. Next, we can make persistent configurations to `micromamba` by modifying the default shell's `rc` file. This is accomplished by running `$ micromamba shell init -s <shell name> -r <MAMBA_ROOT_ENVIRONMENT>`. Where the `-s` argument specifies the shell of choice (e.g. `bash`) and `MAMBA_ROOT_ENVIRONMENT` specifies where the virtual environment will be stored e.g. `/home/usrname/mamba`.

3. This will add a few lines to your shell's `rc` file. An example `.bashrc` is shown below:

    ```bash
    # >>> mamba initialize >>>
    # !! Contents within this block are managed by 'micromamba shell init' !!
    export MAMBA_EXE='/usr/bin/micromamba';
    export MAMBA_ROOT_PREFIX='/home/usrname/mamba';
    __mamba_setup="$("$MAMBA_EXE" shell hook --shell bash --root-prefix "$MAMBA_ROOT_PREFIX" 2> /dev/null)"
    if [ $? -eq 0 ]; then
        eval "$__mamba_setup"
    else
        alias micromamba="$MAMBA_EXE"  # Fallback on help from micromamba activate
    fi
    unset __mamba_setup
    # <<< mamba initialize <<<

    ```

4. Avoid using directories that need superuser privileges for read/write operations, as this can cause errors down the line if `micromamba` is invoked without superuser privileges. Also as a precaution, avoid using `sudo` with `micromamba`.

5. `micromamba` uses a unix style directory to lay out the virtual environments. In Unix-like platforms, installing a piece of software consists of placing files in subdirectories of an `installation prefix` e.g. if the prefix is `/usr` executables will go inside `/usr/bin`, headers will go inside `/usr/include`, static and dynamic libraries will go inside `/usr/lib` and `/usr/lib64` etc. [Documentation](https://mamba.readthedocs.io/en/latest/user_guide/concepts.html).

6. The top most directory of this layout is the `ROOT_PREFIX`. A prefix is a fully self-contained and portable installation. To disambiguate with `root prefix`, prefix is often referred to as `target prefix`. An `environment` is just another name for a `prefix`. `root prefix` can be imagined as equivalent to `conda`'s `base` environment.

7. The directory structure of the `ROOT_PREFIX` looks something like this:
    ```
    ├── bin
    ├── conda-meta
    ├── etc
    ├── fonts
    ├── include
    ├── lib
    ├── libexec
    ├── pkgs
    ├── PySide6
    ├── sbin
    ├── share
    ├── shiboken6
    ├── shiboken6_generator
    ├── ssl
    ├── var
    └── x86_64-conda-linux-gnu
    ```

8. You can’t create a `base` environment because it’s already part of the `root prefix` structure. Directly install in base instead.

9. For environment creation, activation, package installation and package channel configuration refer to the [documentation](https://mamba.readthedocs.io/en/latest/installation/micromamba-installation.html).
