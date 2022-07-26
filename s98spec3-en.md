# S98V3 file format

  - Japanese version 2006-05-15
  - English version 2022-07-26

S98 is a music format that stores instructions sent to OPNA/OPN chips.

### HEADER FORMAT

ofs |size/endian| name | description 
----|-----------|------|------------
0000|3BYTE|MAGIC|`S98`
0003|1BYTE|FORMAT VERSION|`3`
0004|DWORD(LE)|TIMER INFO|sync numerator. If value is 0, default time is 10.
0008|DWORD(LE)|TIMER INFO2|sync denominator. If value is 0, default time is 1000.
000C|DWORD(LE)|COMPRESSING|The value is always 0.
0010|DWORD(LE)|FILE OFFSET TO TAG|If value is 0, no title exists.
0014|DWORD(LE)|DUMP DATA OFFSET| File offset of the data dump
0018|DWORD(LE)|LOOP POINT OFFSET| File offset of the data dump loop point
001C|DWORD(LE)|DEVICE COUNT|The default value is 0, meaning OPNA, the clock is 7987200Hz
0020|DWORD(LE)|DEVICE INFO|× DEVICE COUNT

  - The COMPRESSING flag introduced in V1 has remained unused so it was discarded.
  - DEVICE COUNT must be no larger than 64.
  - For values above 1, DEVICE INFO immediately follows starting at 0x20.
  - For compatibility with S98V1, a DEVICE COUNT of 0 results in a single OPNA1 device.


### DEVICE INFO

ofs |size/endian| name | description
----|-----------|------|------------
0000|DWORD(LE)|DEVICE TYPE|See below.<br>YM2149 means a non-AY-3-8910-compatible clock mode.
0004|DWORD(LE)|CLOCK(Hz)|The external oscillator clock.
0008|DWORD(LE)|PANNING|See below.
000C-000F|RESERVED|


### DEVICE TYPE

code|meaning
-|-
0|NONE
1|PSG(YM2149)
2|OPN(YM2203)
3|OPN2(YM2612)
4|OPNA(YM2608)
5|OPM(YM2151)
6|OPLL(YM2413)
7|OPL(YM3526)
8|OPL2(YM3812)
9|OPL3(YMF262)
15|PSG(AY-3-8910)
16|DCSG(SN76489)

  * When it is NONE, any existing data is ignored.


### PANNING

  - For mono devices only. (PSG/OPN/OPLL/OPL/OPL2/DCSG)
  - Intended for the use where panning is implemented using two or more chips.
  - The two bits represent L/R channels; mute when the bit is set.

(PSG)
bit|meaning
-|-
0|ch1 L
1|ch1 R
2|ch2 L
3|ch2 R
4|ch3 L
5|ch4 R

(OPN)
Same as PSG until bit 5.

bit|meaning
-|-
6|FM L
7|FM R

(OPLL/OPL/DCSG)

bit|meaning
-|-
0|L
1|R

### TAG

 The structure mostly conforms to the PSF format tag:
 
  * Attached at the end of file.
  * Tags are written as tagname=value.
  * Tag names can be half-width or full-width.
  * The line breaks are 0x0a.
  * Finishing the tag sequence with 0x00 is recommended.

 The parts that are different from PSF:
 
  * The tag ID is "[S98]".
  * Character encoding may be either multibyte (Shift_JIS for Japanese locale) or UTF-8.
  * If a BOM(EF BB BF) is present immediately after the tag ID, it will be UTF-8, multibyte otherwise.

 Refer to the following sample:

`[S98]`\
"`title=Opening`" `0x0a`\
"`artist=Yuzo Koshiro`" `0x0a`\
"`game=Sorcerian`" `0x0a`\
"`year=1987`" `0x0a`\
"`genre=game`" `0x0a`\
"`comment=This is sample data.`" `0x0a`\
"`copyright=Nihon Falcom`" `0x0a`\
"`s98by=foo`" `0x0a`\
"`system=PC-8801`" `0x0a`

 The tag names may be anything you want, but the above tags are defined as basic tags.

[DUMP DATA FORMAT]
raw data | meaning
-|-
00 aa dd | DEVICE1(normal)
01 aa dd | DEVICE1(extend)
02 aa dd | DEVICE2(normal)
03 aa dd | DEVICE2(extend)
... |
FF       | 1SYNC
FE `vv`  | nSYNC
FD       | END/LOOP

  - The elapsed time in `1SYNC` is the value of `TIMER INFO`/`TIMER INFO2`(sec) from the header.
  - The `DEVICE1`, `DEVICE2` etc. correspond to the order defined in `DEVICE INFO`, with two commands provided per device: `normal` and `extend`.

The difference in the usage of `normal`/`extend` is as follows:

|device|usage
|-|-
| PSG/OPN/OPM/OPLL/OPL/OPL2 | strictly `normal`
| OPNA/OPN2/OPL3 | `normal`/`extend`
| DCSG | strictly `normal`
|    | Specified in register 0, and GG extension defines register 1.

 `vv` is specified by `FE` command, and is a variable-length little-endian value, with the 7th bit as the continuation flag.

 Consider the following code.

    int getvv(byte *p)
    {
        int s = 0, n = 0;
        p--;
        do
        {
            n |= (*(++p) & 0x7f) << s;
            s += 7;
        }
        while (*p & 0x80);
        return n + 2;
    }


## CONTACTS

For any improvement suggestions or considerations, please email me or use the 2ch boards I may be looking at.


## HISTORY

2006/05/16
  - FE command parameter explanation added.
  - DEVICE COUNT being 0 case explanation added.
  - DEVICE TYPE being NONE behaviour explanation added.
  - AY-3-8910 added to DEVICE TYPE.
  - Peculiarities of CLOCK parameter being YM2149 described.
  - Description of elapsed time in SYNC command added.

2006-01-13
  - DEVICE TYPE : Typo fixed (YM2610->YM2612)

2005-10-08
  - TAG   : Basic tag names explained.

2005-08-05
  - DEVICE: OPL2 and OPL3 defined
    - Reserved space for 118 board and SB16(98) in case
  - TAG: Clarified the encodings further.

2005-07-18
  - TAG : Description of encodings added.
  - PAN : Detailed description of bits added.
