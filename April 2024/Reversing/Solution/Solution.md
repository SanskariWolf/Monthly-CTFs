# CTF Challenge Solution: Architecture Astronaut

**Challenge Source:** MetaCTF April 2024 Flash CTF
**Challenge Goal:** Identify the CPU architecture the provided executable was compiled for, using only the provided Ghidra analysis output.

## Provided Information

We were given a snippet of Ghidra's decompilation output for a function within the target executable.

```c
/* WARNING: Globals starting with '_' overlap smaller symbols at the same address */
/* WARNING: Restarted to delay deadcode elimination for space: ram */
/* WARNING: Unknown calling convention: __rustcall -- yet parameter storage is locked */
/* DWARF original prototype: void __xtensa_lx_rt_main_trampoline(void) */

void __rustcall comped::comped::__xtensa_lx_rt_main_trampoline(void)

{
  &str msg;
  &dyn_core::fmt::Debug error;
  uint uVar1;
  dyn_core::fmt::Debug *pdVar2;
  usize (*pauVar3) [3];
  usize i;
  uint uVar4;
  <>::{vtable_type} *p<Var5;
  int iVar6;
  uint uVar7;
  undefined **ppuVar8;
  int iVar9;
  uint in_SCOMPARE1;
  undefined1 in_CCOUNT;
  undefined1 in_PRID;
  u64 uVar10;
  SetLoggerError e;
  u8 *in_stack_ffffff50;
  undefined4 in_stack_ffffff54;
  usize (*pauVar11) [3];
  dyn_core::fmt::Debug *pdVar12;
  undefined4 in_stack_ffffff5c;
  usize (*pauVar13) [3];
  uint uStack_a0;
  undefined4 uStack_9c;
  <>::{vtable_type} *p<Stack_90;
  Arguments AStack_68;
  char *pcStack_50;
  undefined4 uStack_4c;
  undefined4 uStack_48;
  char *pcStack_44;
  undefined4 uStack_40;
  undefined **ppuStack_3c;
  undefined4 uStack_38;
  <>::{vtable_type} *p<Stack_34;
  undefined4 uStack_30;
  undefined4 uStack_2c;
  <>::{vtable_type} <Stack_24;

  // ... (code omitted for brevity, relevant parts highlighted below) ...

  /* DWARF original prototype: void __xtensa_lx_rt_main_trampoline(void) */ // Clue 1

  // ...

  uVar7 = rsr(in_PRID); // Clue 2 (rsr instruction)
  // ...
  wsr((char)in_SCOMPARE1,0x100); // Clue 2 (wsr instruction)
  // ...
  memw(); // Clue 2 (memw instruction)

  // ...

  if (_ESP_HAL_DEVICE_PERIPHERALS == 0) { // Clue 3 (ESP_HAL reference)
    _ESP_HAL_DEVICE_PERIPHERALS = 1;
    // ...
    esp_hal::critical_section_impl::multicore::MULTICORE_LOCK = 0x100; // Clue 3
    // ...
    esp_hal::esp_hal::rtc_cntl::RtcClock::calibrate_internal(RtcCal8mD256,10); // Clue 3
    // ...
  }

  // ...

  // String references within the code (e.g., panic messages, log setup) point to build paths:
  // "/home/ubuntu/.rustup/toolchains/esp/lib/rustlib/..." // Clue 4 (esp toolchain)
  // "/home/ubuntu/.cargo/registry/src/index.crates.io-6f17d22bba15001f/log-0.4.21/src/lib.rs..."

  // ...
}
```

## Analysis and Solution

The goal is to identify the architecture without needing to run the binary or perform deep disassembly. We analyze the provided Ghidra output for specific clues:

1.  **Function Naming Convention:** The very first comment block and the function signature itself contain `__xtensa_lx_rt_main_trampoline`. The presence of `xtensa` is a very strong indicator of the target architecture being **Xtensa**.

2.  **Architecture-Specific Instructions/Intrinsics:** Ghidra's decompilation shows functions like `rsr` (Read Special Register) and `wsr` (Write Special Register), which are characteristic instructions for the Xtensa Instruction Set Architecture (ISA). The `memw()` (Memory Wait/Barrier) intrinsic is also commonly associated with Xtensa, especially in embedded contexts.

3.  **Hardware Abstraction Layer (HAL) References:** The code heavily references `esp_hal` (e.g., `_ESP_HAL_DEVICE_PERIPHERALS`, `esp_hal::critical_section_impl`, `esp_hal::esp_hal::rtc_cntl`). `esp-hal` is the Hardware Abstraction Layer crate used for programming Espressif Systems' microcontrollers (like the ESP32, ESP8266, ESP32-S series). These microcontrollers predominantly use Xtensa processor cores (specifically, variants like LX6 and LX7).

4.  **Build Environment Clues:** Although not fully shown in the snippet, comments within the analysis indicate embedded strings referencing build paths like `/home/ubuntu/.rustup/toolchains/esp/lib/rustlib/...`. The `esp` in the toolchain path signifies a toolchain specifically for Espressif targets, further confirming the link to Xtensa-based hardware.

## Conclusion

Combining the explicit function naming (`xtensa`), the presence of Xtensa-specific instructions (`rsr`, `wsr`, `memw`), the use of the Espressif HAL (`esp_hal`), and clues from the build environment (`/esp/`), it is clear that the executable was compiled for the **Xtensa** architecture.

## Flag

As per the challenge instructions, the flag is simply the name of the architecture.

**Flag:** `Xtensa`