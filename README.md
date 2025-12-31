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

### job.c
Represents the **Job** Abstract Data Type.
* **`execPath`**: 
    * Builtins: The name of the builtin.
    * Non-builtins: Full path located via `/usr/local/bin`, `/usr/bin`, or `/bin`.
    * If the input contains a `/`, it is treated as a direct path.
* **`args`**: Array of strings expanded using wildcards (`*`). Wildcards match zero or more characters and must appear at the end of a path.
* **Redirection**: Stores paths for `inputReDirectPath` (`<`) and `outputReDirectPath` (`>`).

### builtins.c
Handles core shell commands:
* **`cd`**: Changes working directory via `chdir()`. Expects exactly one argument.
* **`which`**: Locates the path of a program. Checks builtins first, then system paths.
* **`pwd`**: Prints current working directory using `getcwd()`.

### mysh.c
The main program launcher.
* Manages `mysh_errno` to track exit statuses for `then`/`else` logic.
* **Execution Flow**:
    1.  Trim whitespace and split commands by pipes.
    2.  Create `Job` objects for each sub-command.
    3.  **Single Job**: If builtin, runs in-process with `dup2` redirection. If non-builtin, uses `fork()`, `dup2`, and `exec()`.
    4.  **Piped Jobs**: Sets up communication using `pipe()` and `dup2()`.

---

## Tests

### Test Interactive Mode
Used to verify that `mysh` parses input, expansions, and redirections correctly.

**Example Test Cases:**
1.  `echo man name > txt.txt`: Tests output redirection and bare name expansion.
2.  `cat < txt.txt`: Tests input redirection.
3.  `ls *.c`: Tests wildcard expansion for files in the current directory.
4.  `ls testFolder2/*.txt > txt.txt`: Tests wildcards within a specific path.

### Test Batch Mode
Run this using `make batchmodetest`. This suite covers:
* **Wildcards**: Including scenarios with no matching files (returns original string).
* **Redirection**: Input, output, and combined (e.g., `./sum < in > out`).
* **Pipes**: Mixing system functions, builtins, and redirection within pipes.
* **Conditionals**: Verifying that `then` and `else` blocks execute correctly based on the success of the preceding command.