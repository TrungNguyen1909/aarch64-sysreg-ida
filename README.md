# aarch64-sysreg-ida

## Overview
When reversing Operating Systems on ARM, it is quite common to see machine-specific-registers (MSR) being used.
However, IDA doesn't have a builtin database for those, and instead displays cryptic sequences:

For example:
```
__TEXT_EXEC:__text:FFFFFFF00812420C _start_first_cpu                        ; CODE XREF: __start↑j
__TEXT_EXEC:__text:FFFFFFF00812420C                 MSR             #0, c1, c0, #4
__TEXT_EXEC:__text:FFFFFFF008124210                 MSR             #6, #0xF
__TEXT_EXEC:__text:FFFFFFF008124214                 MOV             X20, X0
__TEXT_EXEC:__text:FFFFFFF008124218                 MOV             X21, #0
__TEXT_EXEC:__text:FFFFFFF00812421C                 ADRL            X0, _LowExceptionVectorBase
__TEXT_EXEC:__text:FFFFFFF008124224                 MSR             #0, c12, c0, #0, X0
```

Past solutions include [Brandon Azad's script to add comments](https://gist.github.com/bazad/42054285391c6e0dcd0ede4b5f969ad2) to these instructions.
However, it takes a while for these script to run and you will need to run it again upon marking new data as code.

This plugin attempts to solve this problem by hooking into functions that are responsible for displaying instructions in IDA.

The result is that these cryptic sequences are replaced with standard MSR names...

```
__TEXT_EXEC:__text:FFFFFFF00812420C                 EXPORT _start_first_cpu
__TEXT_EXEC:__text:FFFFFFF00812420C _start_first_cpu                        ; CODE XREF: __start↑j
__TEXT_EXEC:__text:FFFFFFF00812420C                 MSR             OSLAR_EL1, , ,
__TEXT_EXEC:__text:FFFFFFF008124210                 MSR             DAIFSet, #0xF
__TEXT_EXEC:__text:FFFFFFF008124214                 MOV             X20, X0
__TEXT_EXEC:__text:FFFFFFF008124218                 MOV             X21, #0
__TEXT_EXEC:__text:FFFFFFF00812421C                 ADRL            X0, _LowExceptionVectorBase
__TEXT_EXEC:__text:FFFFFFF008124224                 MSR             VBAR_EL1, X0, , ,
```

IDA caches these printing so the hook is generally only invoked once every session.
The performance overhead is generally unnoticable.

The plugin left the commas behind in order to avoid corrupting disassembler's data.
I haven't had a problem with doing that; however, I decided not to in order to avoid corruptions.

This plugin do supports SYS instructions as shown in this example:

```
__TEXT_EXEC:__text:FFFFFFF008124498                 MSR             MAIR_EL1, X0, , ,
__TEXT_EXEC:__text:FFFFFFF00812449C                 ISB
__TEXT_EXEC:__text:FFFFFFF0081244A0                 TLBI            VMALLE1, , ,
__TEXT_EXEC:__text:FFFFFFF0081244A4                 DSB             ISH
__TEXT_EXEC:__text:FFFFFFF0081244A8                 CBZ             X21, loc_FFFFFFF0081244BC
__TEXT_EXEC:__text:FFFFFFF0081244AC                 ADRL            X0, _cpu_ttep
__TEXT_EXEC:__text:FFFFFFF0081244B4                 LDR             X0, [X0]
__TEXT_EXEC:__text:FFFFFFF0081244B8                 MSR             TTBR1_EL1, X0, , ,
```

### MSR name database
The embedded database only includes standard ARMv8 MSRs; however, it could be extended by putting a register json database in the same directory

Do note that Apple SoC registers' names might varies between models.

## Installation

Download and put the `aarch64_sysreg.py` in the `plugins/` folder of IDA.

### Apple-specific registers

Download `apple_regs.json` from [Asahi Linux's m1n1 repo](https://github.com/AsahiLinux/m1n1/blob/main/tools/apple_regs.json)
and put it in the same folder with the Python script (`plugins/`).

## Disclaimer

This software comes with no warranty. It should work fine in normal circumstances.
However, in unfortunate cases (if exists), please do **NOT** blame the author for corrupted databases.
Please nicely file a bug report **AFTER** your anger is processed.

Examples are taken from XNU kernel.

## Contribution

Issues, PRs are welcomed.

## License

This repo is licensed under Mozilla Public License, v. 2.0.
