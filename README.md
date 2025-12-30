# CH32V003 CMake Template

Inspired by [ch32v_evt_makefile_gcc_project_template](https://github.com/cjacker/ch32v_evt_makefile_gcc_project_template).

This template enables development for the **CH32V003** platform using any IDE that supports the CMake build system.

## Setup

To get it running, you only need to configure the toolchain paths in two files:

### 1. CMakeLists.txt
Define the path to your compiler's `bin` folder:
```cmake
set(TOOLCHAIN_PATH "${CMAKE_CURRENT_SOURCE_DIR}/../toolchain/RISC-V Embedded GCC12/bin")

```

### 2. .clangd (LSP Support)

For full LSP support, update the include paths in `.clangd`:

```yaml
CompileFlags:
  Add: [
    "-I../toolchain/RISC-V Embedded GCC12/riscv-wch-elf/include",
    "-I../toolchain/RISC-V Embedded GCC12/lib/gcc/riscv-wch-elf/12.2.0/include"
  ]
  Remove: [-msave-restore, -msmall-data-limit=*, -march=*, -mabi=*]

```
## Hacky fix

This template replaces the compiled target with the `flash_and_monitor.sh` script. This script flashes the firmware using the [wlink](https://github.com/ch32-rs/wlink) utility.

### Dependencies

* **Python 3** (Required for the serial monitor).
* **wlink** (Required for flashing).
