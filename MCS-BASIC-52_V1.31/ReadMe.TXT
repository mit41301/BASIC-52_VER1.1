## BASIC-52_ASEM1.3
;*****************************************************************************
;*                                                                           *
;*                    MCS-BASIC-52 V1.31 Source Listing                      *
;*                           12/1986 till 11/2001                            *
;*       The original source code of V1.1 (BASIC.SRC and FP52.SRC) by        *
;*            Intel Corporation, Embedded Controller Operations              *
;*                             is public donain                              *
;*                                                                           *
;*****************************************************************************
;
;*****************************************************************************
;* General alterations made by D. Wulf, 12/1999.                             *
;* e-mail: Detlef.Wulf@onlinehome.de                                         *
;*****************************************************************************
;
;  The following general alterations are made to the original source code:
;
;  - The original source code had 2 files BASIC.SRC and FP52.SRC those have
;    been incorporated into this file for easy of assembly.
;
;  - All absolute and relativ jumps and calls without labels were provided
;    with labels.
;
;  - All machine code in the original source, coded in databytes are replaced
;    by the menomics.
;
;  - One routine in the source was different to the ROM code and is replaced
;    by the ROM code.
;
;  - Some "ORG" statements between BASIC and floating point code are remarked
;    out.
;
;  - To get room for new code the "ego message" had to be disabled.
;    (Remarked with "Sorry")
;
;  - To get more room for new code the "FPROG" command had to be disabled.
;    (Remarked with "get room")
;
;*****************************************************************************
;* Bugfixes for MCS-52-BASIC from D. Karmann, 8/1993.                        *
;* e-mail: dankarmann@lucent.com                                             *
;*****************************************************************************
;
;  - Corrected Intel bug to allow BASIC autoboot EPROM at 8000H with user
;    command extensions to work.
;    (Remarked as Karmann 1)
;
;  - Corrected Intel bug to that discarded the 'F' in any variable ending in
;    F, FP, FPR and FPRO and followed by a space.
;    (Remarked as Karmann 2)
;
;*****************************************************************************
;* Bugfix and performance for MCS-52-BASIC from                              *
;* D. Mudric and Z. Stojsavljevic descipt in                                 *
;* Elektor Electronics magazine german issue 3/1992.                         *
;*****************************************************************************
;
;  - Modifications to the unprocess a BASIC line routine.
;    (Remarked as Elektor 1)
;
;  - Modifications to the floating point subtraction routine.
;    (Remarked as Elektor 2)
;
;  - HEX to BIN performance improvements.
;    (Remarked as Elektor 3)
;
; The same article describes a fix for the multiplication underflow bug, but
; the fixes did not work.
;
; The multiplicaton underflow bug is now (V1.31) really fixed by D. Wulf!
;    (Remarked as Wulf 1)
;
;*****************************************************************************
;* Change UV-EPROM to EEPROM programming from R. Skowronek, 4/1996           *
;* e-mail: r.skowronek@kfa-juelich.de                                        *
;*****************************************************************************
;
; This altered section of code writes the ram resident Basic program to
; EEPROM just like the ROM resident Basic interpreter writes to UV-EPROMs.
; The EEPROM is connected just like a RAM, i.e. it uses /wr on pin 27
; and gets it's adresses from the real address lines, i.e. the only
; difference from the normal setup is the use of the /wr line instead of
; P1.4, which supplies the program pulse for UV-EPROMs. Now MCS-BASIC-52
; can be located in externally ROM and is non the less able to programm
; EEPROMs!
; (Remarked as Skowronek)
;
; The original code from R. Skowronek didn't support the "PGM" statement
; this feature is added by D. Wulf.
; Memory is now limited to 32K bytes RAM, because memory tests above it
; would change the EEPROM.
;
;*****************************************************************************
;* Change timer 0 from 13 bit to 16 bit counter mode to use XTAL up to 78MHz *
;* from D. Wulf 1/2000                                                       *
;*****************************************************************************
;
; The max. value for XTAL is now 78627473 Hz, for use BASIC-52 with
; Dallas 80C320 high speed / low power microcontroller (33 MHz).
; The defaut crystal value is still 11059200 Hz. You can set it with
; XTAL or patch the souce code at
;
;	17F1H = 11
;	17F0H = 05
;	17EFH = 92
;	17EEH = 00
;
; with a new crystal value.
; (Remarket as Wulf 2)
;
;*****************************************************************************
;* New baudrate detection from D. Wulf 1/2000                                *
;*****************************************************************************
;
; The new baudrate detection uses timer 2 for time measurement in state of
; the code loop timing. So the Dallas 80C320 and other controllers can be
; used. Also at higher clock speeds the baudrate will detect automaticly.
; (Remarked as Wulf 3)
;
;*****************************************************************************
;* New processor type detection from D. Wulf 2/2000                          *
;*****************************************************************************
;
; A new reset routine detects the processor type. So BASIC-52 V1.3 can be
; used with the following controllers:
;
; 8032, 87C52#, Dallas 80C320, 80515*#, 80517*#, 80517A#, 80528, 80535*,
; 80537*, 80575 or similars.
;
; - On processor types marked with the "*" only two different autodetect
;   baudrates, depending on the crystal are possible.
; - The processor types marked with the "#" have internal ROM, so BASIC-52
;   V1.3 can be located there, because it is still only 8K bytes long!
;
; (Remarked as Wulf 4)
;
;*****************************************************************************
;* OPBYTE 43H for POP from H.-J. Boehling 1/2000                             *
;* e-mail: H-Boehling@gmx.de                                                 *
;*****************************************************************************
;
; A feature of BASIC-52 is the ability to add up to 16 custom keywords
; representing commands or instructions that you define with assembler
; routines. For using system routines in your assembler code there are
; operation bytes (for more information see Intels "MCS BASIC-52 MANUAL").
; In the original souce code is no OPCODE to put a value from argument
; stack and store in a variable.
; With BASIC-52 V1.3 you can use OPBYTE 43H which does the same than the
; "POP" statement.
; (Remarked as Boehling 1)
;
;*****************************************************************************
;* Reset millisecond counter on "TIME=" from H.-J. Boehling 2/2000           *
;*****************************************************************************
;
; The command "TIME=0" now zeros the millisecond register so that TIME
; returns with zero.
; (Remarked as Boehling 2)
;
;*****************************************************************************
;* New command "ERASE" by H.-J. Boehling 2/2000                              *
;*****************************************************************************
;
; To erase an EEPROM (fill 16K byte up to 8000H with 0FFH) the new command
; "ERASE" is implemented. It takes 2 min. and 45 sec. to erase the 16K bytes!
; (Remarked as Boehling 3)
;
;*****************************************************************************
;* Correct "ASC(x)" bug by D. Wulf 2/2000                                    *
;*****************************************************************************
;
; BASIC-51 V1.1 gives erroneous results for the "ASC(x)" funktion if "x" is
; one of the following signs : *, +, -, /, <, =, > or ?.
; BASIC-51 V1.3 returns the correct values.
; (Remarked as Wulf 5)
;
;*****************************************************************************
;*****************************************************************************
; To indicate the new version the start message is changed from
; *MCS-51(tm) BASIC V1.1* to
; *MCS-BASIC-52 V1.31*
;
; H.-J. Boehling, D. Wulf 11/26/2001
;*****************************************************************************
