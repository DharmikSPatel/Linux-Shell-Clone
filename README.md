# mysh: Linux Command Line Implementation

This is a custom implementation of the Linux Command Line called **mysh**.

## Table of Contents
* [Makefile Commands](#makefile-commands)
* [Overview](#overview)
* [Important Notes](#important-notes)
* [Implementation Details](#implementation-details)
    * [job.c](#jobc)
    * [builtins.c](#builtinsc)
    * [mysh.c](#myshc)
* [Tests](#tests)
    * [Test Interactive Mode](#test-interactive-mode)
    * [Test Batch Mode](#test-batch-mode)

---

## Makefile Commands

| Command | Description |
| :--- | :--- |
| `make mysh` | Creates the executable file `mysh`. Run it using `./mysh`. |
| `make testJobMaking` | Tests the job creation logic. See the Interactive Test section. |
| `make batchmodetest` | Compiles dependencies and runs `test.sh` in batch mode. |
| `make removeTXTFiles` | Cleans up `.txt` files generated during batch mode testing. |
| `make clean` | Removes all `.o`, `mysh` executable, and `.dSYM` files. |

> **Note on Naming:** If `make` fails, ensure the file is named exactly `Makefile` or `MakeFile` as per your system's case sensitivity.
>
> **Debugging:** Edit the Makefile and change line 2 to `DEBUG = -D DEBUG=1` to enable verbose debug information. Use the `-B` flag with any command to force a rebuild.

---

## Overview

* **Interactive Mode:** Used to run one command at a time via user input.
* **Batch Mode:** Used to run a sequence of commands from a file until the `exit` command or EOF (End of File) is reached.

---

## Important Notes

* **Buffered Commands (e.g., `less`):** On ilab, scrolling past the EOF in `less` may cause random characters to be printed to `mysh` after quitting. This occurs because the characters are sent to the tty while `mysh` is still in `exec()`.
* **Quotes:** `mysh` does not handle quotes; they are treated as literal characters.
* **Escaped Spaces:** `mysh` does not support spaces in file/directory names (e.g., `\ `).
* **Conditionals:** * In a **pipe**, the exit status depends on the **last** command in the pipe.
    * In a **single command**, it depends on that command's status.
* **Exiting:** Any call to `exit` (with or without arguments/pipes) will close `mysh`. However, the string "exit" used as a standard argument (e.g., `echo exit`) will not trigger a shutdown.
* **Whitespace:** All whitespace is ignored, including gaps between pipe symbols. `|ls|echo|` is treated the same as `|  ls  |  echo  |`.
* **Constraints:** Only single pipes are supported (no multi-pipe chains). Tab completion and up-arrow history are not implemented.

---

## Implementation Details

### job.