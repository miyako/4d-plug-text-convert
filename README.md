![version](https://img.shields.io/badge/version-18%2B-EB8E5F)
![platform](https://img.shields.io/static/v1?label=platform&message=mac-intel%20|%20mac-arm%20|%20win-64&color=blue)
[![license](https://img.shields.io/github/license/miyako/4d-plugin-text-convert)](LICENSE)
![downloads](https://img.shields.io/github/downloads/miyako/4d-plugin-text-convert/total)

# 4d-plugin-text-convert

A plugin to perform various text convert operartions that are not currently support in 4D

About
---
The native 4D commands **CONVERT FROM TEXT** and **Convert to text** supports various encodings, as listed [here](http://doc.4d.com/4Dv14R4/4D/14-R4/CONVERT-FROM-TEXT.301-1708428.en.html), but some encodings such as EUC-JP and ISO-2022-JP-1 are not supported.

The plugin provides interface to the ICU (mac:68.2, windows:61.1) and LIBICONV (1.14) libraries, as well as the platform native APIs that perform similar tasks.

In addition, there are commands to guess the possible encoding of a string, perform Unicode [Transform](http://userguide.icu-project.org/transforms/general) or apply [Normalization](http://userguide.icu-project.org/transforms/normalization).

### Examples

* Get the list of applicable encodings

```4d
ARRAY LONGINT($codes;0)
ARRAY TEXT($names;0)

CP GET ENCODING LIST ($codes;$names)
  //147 on Win7 (JA) and 91 on OS X 10.10
```

```4d
ARRAY TEXT($names;0)

ICU GET ENCODING LIST ($names)
  //231 on both Windows 7 and OS X 10.10
```

```4d
ARRAY TEXT($names;0)

ICONV GET ENCODING LIST ($names)
  //393 on Windows 7 and 345 on OS X 10.10
```

* Convert to EUC-JP and back

```4d
$err:=CP Convert from text ("あいうえお";$euc;51932)
$err:=CP Convert to text ($euc;$unicode;51932)
```

```4d
$err:=ICU Convert from text ("あいうえお";"euc-jp-2007";$euc)
$err:=ICU Convert to text ($euc;"euc-jp-2007";$unicode)
```

```4d
TEXT TO BLOB("あいうえお";$utf8;UTF8 text without length)
$err:=ICONV Convert ("utf-8";"euc-jp";$utf8;$euc)
$err:=ICONV Convert ("euc-jp";"utf-8";$euc;$utf8)
$original:=BLOB to text($utf8;UTF8 text without length)
```

JIS 
---

Various wayst to encode JIS (not to be confused with ISO-2022-JP).

```4d
$test1:=JIS Convert from text ("ｱ";JIS7_ESC)
$result:=JIS Convert to text ($test1;JIS7_ESC)
ALERT($result)

$test2:=JIS Convert from text ("ｱ";JIS7_SO_SI)
$result:=JIS Convert to text ($test1;JIS7_SO_SI)
ALERT($result)

$test3:=JIS Convert from text ("ｱ";JIS8)
$result:=JIS Convert to text ($test1;JIS8)
ALERT($result)

$test4:=JIS Convert from text ("漢字ｶﾀｶﾅabc漢字abcｶﾀｶﾅ";JIS8)
$result:=JIS Convert to text ($test4;JIS8)
ALERT($result)
```

ICU
---

* Transliteration

```4d
$src:="ﾊﾝｶｸゼンカクひらがな"
$id:="[:East Asian Width=Halfwidth:];Katakana-Hiragana"
$rule:=""
$error:=ICU Transform text ($id;$rule;ICU Transform Forward;$src;$dst)
  //はんかくゼンカクひらがな
```

```4d
$src:="Kaloudis"
$id:="Latin-Cyrillic"
$rule:=""
$error:=ICU Transform text ($id;$rule;ICU Transform Forward;$src;$dst)
  //Калоудис
```

```4d
$src:="株式会社４Ｄジャパン渋谷区道玄坂１ー１０ー２"
$id:="[:^Katakana:];Fullwidth-Halfwidth"
$rule:=""
  //株式会社4Dジャパン渋谷区道玄坂1ｰ10ｰ2
```

```4d
$src:="カキクケコ"
$id:="Katakana-Hiragana"
$rule:=""
$error:=ICU Transform text ($id;$rule;ICU Transform Forward;$src;$dst)
  //かきくけこ
```

```4d
$src:="それは12345ですか"
$id:=""
$rule:="[:^Number:] > ; # 数字だけを抜き取るぞ"
$error:=ICU Transform text ($id;$rule;ICU Transform Forward;$src;$dst)
  //12345
```

```4d
$src:="abcde"
$id:=""
$rule:="a > A; # aをAに変換するだけだ"
$error:=ICU Transform text ($id;$rule;ICU Transform Forward;$src;$dst)
  //Abcde
```

```4d
$src:="トランスフォームデスネ"
$id:="-Latin"
$rule:=""
$error:=ICU Transform text ($id;$rule;ICU Transform Forward;$src;$dst)
  //toransufōmudesune
```

* Normalization

```4d
$error:=ICU Normalize text ("ががが";ICU Normalize NFD;$normalized)
$length:=Length($normalized)
  //6; because NFD splits "が" to letters and combining accent characters
```

* Detect

```4d
$err:=ICU Convert from text ("あいうえお";"euc-jp-2007";$euc)
$err:=ICU Get good encodings ($euc;$encodings;$languages;$confidences)
```

```4d
$err:=CP Convert from text ("あいうえお";$euc;51932)
$err:=CP Get good encodings ($euc;$codepages)
```
