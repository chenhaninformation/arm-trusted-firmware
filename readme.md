ATF for ESPRESSObin
===================

This branch of ATF is for ESPRESSObin board and it is based on
[Marvell ATF][Marvell ATF] branch
[atf-v1.3-armada-17.10][atf-v1.3-armada-17.10].

We only provide the build steps and some notes. You can find documentation at
both the mainline of [ATF][ATF] and most recent release of
[Marvell ATF][Marvell ATF].

******

What is ATF (General)
---------------------

ARM TrustZone Technology is a hardware design execution environment isolation,
the SoC can running in Trusted Zone or None-Trusted Zone independently.

ATF aka ARM-Trusted-Firmware, is an open source software implementation of
ARM Trusted Zone Technology, it will running as an independent system.

The None-Trusted Zone (like Linux/U-boot) act as client, the Truste Zone
(like ATF/OpenTEE) act as server. Client can send a request to the server
side, and the scheduler will handle the CPU to the server and server handle
CPU to client agine when sending the resposes back to the client.

e.g. Request a fingerprint verification on a smart phone.

ARM TrustZone is security sensitive, it also design to do some critical
initialization in most of the SoC vender.

For more information, please refer to [ATF][ATF] repository on github.

ATF on Marvell SoC
------------------

Marvell's ATF porting require specifying the DDR memory layout in order to
initialize the DDR and set the DDR rate (800MHz or 1000MHz etc.) at booting
stage. The DDR layout file is not included in this repository, but in
[a3700-utils-ch][A3700-utils-marvell-ch] repository.

[a3700-utils-ch][A3700-utils-marvell-ch] repository is just the fork of
[a3700-utils][A3700-utils-marvell] in order to add our own patches. And the
branch [A3700\_utils-armada-17.10-ch-dev][A3700_utils-armada-17.10-ch-dev] of
[a3700-utils-ch][A3700-utils-marvell-ch] is based on
[a3700-utils][A3700-utils-marvell-ch] branch 
[A3700\_utils-armada-17.10][A3700_utils-armada-17.10] as well.

You need download [a3700-utils-ch][A3700-utils-marvell-ch] branch
[A3700\_utils-armada-17.10-ch-dev][A3700_utils-armada-17.10-ch-dev] to against
this branch of ATF in order to build ATF for ESPRESSObin board.

Note
====

ATF Boot Stage
--------------

ATF design with multiple boot stage (BL1, BL2, BL3), BL3 (aka BootLoad Stage
3) also design multiple stage. Overall boot sequece are shown in follow:

BL1 --> BL2 --> BL31 --> BL32 --> BL33

ATF already take care of BL1 to BL32, the BL33 is the final payload of
None-Trusted bootloader provide by user, common use U-boot as payload.

From U-boot POV, the ATF is the wrapper layer for U-boot to make the U-boot
to be loaded and running by ATF.

Memory topology
---------------

ESPRESSObin board can mount two memory chip, current support DDR3/DDR4
512MB/1GB memory chip. Possible memory chip layout are shown in table below

|DDR\_TOPOLOGY|DDR type|Chip Size|Chip Select|Memory Size|
|:-----------:|:------:|:-------:|:---------:|:---------:|
|     0       |  DDR3  |  512MB  |     1     |   512MB   |
|     1       |  DDR4  |  512MB  |     1     |   512MB   |
|     2       |  DDR3  |  512MB  |     2     |    1GB    |
|     3       |  DDR4  |   1GB   |     2     |    2GB    |
|     4       |  DDR3  |   1GB   |     1     |    1GB    |
|     5       |  DDR4  |   1GB   |     1     |    1GB    |
|     6(3)    |  DDR4  |   1GB   |     2     |    2GB    |
|     7       |  DDR3  |   1GB   |     2     |    2GB    |
|     8       |  DDR4  |  512MB  |     2     |    1GB    |

ATF is responsable for initialize the DDR memory, you must specifing the memory
topology type above when compile the ATF.

Build Step
==========

There is two types of variables need to be noted, one is the environment
variable BL33 and the other variables of Makefile specify when run "make"
command.

Environment variable BL33: export BL33=/path/to/u-boot.bin

Makefile variables: make var1=value1 var2=value2
Supported Makefile variables can be find [here][build.txt].

Here is a build example from official site after export BL33 variable from
envirionment variables:
```
make DEBUG=1 USE_COHERENT_MEM=0 LOG_LEVEL=20 SECURE=0 \
     CLOCKSPRESET=CPU_1000_DDR_800 DDR_TOPOLOGY=2 BOOTDEV=SPINOR PARTNUM=0 \
     WTP=../a3700-utils/ PLAT=a3700 all fip
```

* DEBUG: Debug enable? default disable.(=1 enable, =0 disable)
* USE\_COHERENT\_MEM: It should be set to 0, see [build.txt][build.txt]
* LOG\_LEVEL: Output log level, see [build.txt][build.txt]
* SECURE: Typo, shold be MARVELL\_SECURE\_BOOT=0, see PR#1731 of [ATF][ATF]
* CLOCKSPRESET: CPU/DDR frequency, see [build.txt][build.txt]
* DDR\_TOPOLOGY: Index of DDR topology in a3700-utils-marvell-ch, we already
list it above
* BOOTDEV: Boot device that store ATF image, see [build.txt][build.txt]
* WTP: /path/to/a3700-utils
* PLAT: Platform, must be a3700
* all & fip: Build output image

More information can be find [here][Build From Source - Bootloader].

TODO
====

******

*Copyright (C) 2018, Hunan ChenHan Information Technology Co., Ltd. All rights reserved.*

[ATF]: https://github.com/ARM-software/arm-trusted-firmware "ARM Trusted Firmware"

[Marvell ATF]: https://github.com/MarvellEmbeddedProcessors/atf-marvell "Marvell ATF"
[atf-v1.3-armada-17.10]: https://github.com/MarvellEmbeddedProcessors/atf-marvell/tree/atf-v1.3-armada-17.10 "atf-v1.3-armada-17.10"

[A3700-utils-marvell-ch]: https://github.com/chenhaninformation/A3700-utils-marvell "A3700-utils-marvell-ch"
[A3700_utils-armada-17.10-ch-dev]: https://github.com/chenhaninformation/A3700-utils-marvell/tree/A3700_utils-armada-17.10-ch-dev

[A3700-utils-marvell]: https://github.com/MarvellEmbeddedProcessors/A3700-utils-marvell "A3700-utils-marvell"
[A3700_utils-armada-17.10]: https://github.com/MarvellEmbeddedProcessors/A3700-utils-marvell/tree/A3700_utils-armada-17.10

[build.txt]: ./docs/marvell/build.txt#L40

[Build From Source - Bootloader]: http://wiki.espressobin.net/tiki-index.php?page=Build+From+Source+-+Bootloader "Build From Source - Bootloader"
