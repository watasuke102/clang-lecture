---
title: "ELF"
date: 2022-06-07T17:30:18+09:00
draft: true
weight: 10
---

## 前提知識

- Linux における「コマンド」という概念の存在

## 導入

Linux を扱う上で、コマンドの存在は欠かせません。では、この「コマンド」の正体とは何でしょうか？ `cat` コマンドを例に取り、 `type` コマンドと `file` コマンドを用いて確かめてみましょう。

`type` コマンドは、特定のコマンドに関する基本的な情報を閲覧できるコマンドです。引数として `cat` を渡してみましょう。

```
$ type cat
cat is /usr/sbin/cat
```

環境に左右されるとは思いますが、おおよそこのような出力が行われます。 `/usr/sbin/cat` というファイルのようです。

`file` コマンドを用いて、このファイルの種類を見てみましょう。

```
$ file /usr/sbin/cat
/usr/sbin/cat: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=cab75842a80c1ad598a461d119a9745209c97501, for GNU/Linux 4.4.0, stripped
```

`executable` （実行可能な） が示す通り、Linux におけるコマンドの正体は **実行形式ファイル** でした。

一言で「実行形式ファイル」といっても、いろいろな種類があります。画像を保存するのに `jpg` や `png` など様々なフォーマットがあるように、実行形式ファイルにも多くのフォーマットが存在します。

Windows においては、拡張子が `.exe` であるものが実行形式ファイルとして扱われ、そのフォーマットは `PE (Portable Executable)` と呼ばれるフォーマットです。本当は Windows の実行形式についても書いたほうが良いんだろうな～とは思いますが、全くと言っていいほど知識がないので程々にしておきます。知りたい方は、Windows SDK に含まれる[winnt.h](https://docs.microsoft.com/en-us/windows/win32/api/winnt/)などが参考になるらしいので、よければどうぞ。

さて、Linux の話に戻りましょう。コマンドの実体が実行形式ファイルであることはわかりましたが、`file` コマンドは更に多くの情報を提供しています。

注目していただきたいのは `ELF` という文字列です。このページのタイトル等から察しているかもしれませんが、Linux における実行形式ファイルのフォーマットは `ELF` です。

Linux 上で動作するプログラムを書く際、ELF について知っておくことは大切です。このページでは、ELF というフォーマットについて解説していきます。

## ELF とは？

ELF は `Executable and Linkable Format` から取られたもので、名前の通り Executable である時もあれば Linkable な時もあります。Executable・実行可能である時はすなわちそのファイルが実行形式であるということです。そして、Linkable・リンク可能である時というのは、例えばオブジェクトファイルなどがこれにあたります。これについては別のページで触れるため、ひとまず ELF には実行可能とリンク可能の 2 種類あるんだな～という認識をぼんやり持っておいてください。

## ELF の構成要素

ELF フォーマットは、大きく分けて 3 つの要素に分けることが出来ます。**ELF ヘッダ** ・ **セクションヘッダテーブル** ・ **プログラムヘッダテーブル** です。それぞれ順番に見ていきましょう。

### ELF ヘッダ

必ずファイルの先頭に位置します。ELF ファイルに関する基本的な情報が含まれています。

こういうのは実際に見てみるのが一番です。 ELF に関する情報を取得する際は `readelf` コマンドを使います。このコマンドに `-h` オプションを付与することで、ELF ヘッダを閲覧可能です。これを用いて、`cat` コマンド（というかファイル）の ELF ヘッダを覗いてみましょう。

```
$ readelf -h /usr/sbin/cat
ELF Header:
  Magic:   7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00
  Class:                             ELF64
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0
  Type:                              DYN (Position-Independent Executable file)
  Machine:                           Advanced Micro Devices X86-64
  Version:                           0x1
  Entry point address:               0x3350
  Start of program headers:          64 (bytes into file)
  Start of section headers:          33152 (bytes into file)
  Flags:                             0x0
  Size of this header:               64 (bytes)
  Size of program headers:           56 (bytes)
  Number of program headers:         13
  Size of section headers:           64 (bytes)
  Number of section headers:         26
  Section header string table index: 25
```

さきほど、ELF には実行可能とリンク可能の 2 種類あるという話をしました。この 2 つを区別する判断材料となるのも ELF ヘッダです。この出力結果にある `Type` という項目で見分けることが可能です。以下に例を示します：

- `DYN (Position-Independent Executable file)`：動的リンクによって動作する実行ファイル
- `EXEC (Executable file)`：静的リンクが行われた実行ファイル
- `REL (Relocatable file)`：再配置可能ファイル（リンク可能）

その他には、そのファイルが ELF であることを示す **マジックナンバー** (`.ELF`、0x7f 0x45 0x4c 0x46...) と、対応している OS・アーキテクチャ・エンディアンなどの情報が含まれています。

### セクションヘッダテーブル

**セクション** に関する情報が格納されます。ELF ファイルの最小単位とも言えるこのセクションは、主にリンカのための情報になります。セクションの例を以下に示します：

- `.text`：実行可能な機械語
- `.data`：初期値を持つグローバルもしくは静的な変数
- `.rodata`：文字列リテラルのような、読み取り専用（ReadOnly）なデータ

とりあえず、ELF に格納したいデータを種類ごとにまとめたようなものだと解釈して良いと思います。

セクションヘッダテーブルは、 `readelf` に `-S` オプションを付与することで閲覧可能です。おなじみ `cat` で試しましょう。

```
$ readelf -S /usr/sbin/cat
There are 26 section headers, starting at offset 0x8180:

Section Headers:
  [Nr] Name              Type             Address           Offset
       Size              EntSize          Flags  Link  Info  Align
  [ 0]                   NULL             0000000000000000  00000000
       0000000000000000  0000000000000000           0     0     0
  [ 1] .interp           PROGBITS         0000000000000318  00000318
       000000000000001c  0000000000000000   A       0     0     1
  [ 2] .note.gnu.pr[...] NOTE             0000000000000338  00000338
       0000000000000050  0000000000000000   A       0     0     8
  [ 3] .note.gnu.bu[...] NOTE             0000000000000388  00000388
       0000000000000024  0000000000000000   A       0     0     4
  [ 4] .note.ABI-tag     NOTE             00000000000003ac  000003ac
       0000000000000020  0000000000000000   A       0     0     4
  [ 5] .gnu.hash         GNU_HASH         00000000000003d0  000003d0
       000000000000001c  0000000000000000   A       6     0     8
  [ 6] .dynsym           DYNSYM           00000000000003f0  000003f0
       0000000000000618  0000000000000018   A       7     1     8
  [ 7] .dynstr           STRTAB           0000000000000a08  00000a08
       000000000000030b  0000000000000000   A       0     0     1
  [ 8] .gnu.version      VERSYM           0000000000000d14  00000d14
       0000000000000082  0000000000000002   A       6     0     2
  [ 9] .gnu.version_r    VERNEED          0000000000000d98  00000d98
       0000000000000090  0000000000000000   A       7     1     8
  [10] .rela.dyn         RELA             0000000000000e28  00000e28
       0000000000000750  0000000000000018   A       6     0     8
  [11] .init             PROGBITS         0000000000002000  00002000
       000000000000001b  0000000000000000  AX       0     0     4
  [12] .text             PROGBITS         0000000000002020  00002020
       0000000000003873  0000000000000000  AX       0     0     16
  [13] .fini             PROGBITS         0000000000005894  00005894
       000000000000000d  0000000000000000  AX       0     0     4
  [14] .rodata           PROGBITS         0000000000006000  00006000
       0000000000000daf  0000000000000000   A       0     0     32
  [15] .eh_frame_hdr     PROGBITS         0000000000006db0  00006db0
       00000000000000a4  0000000000000000   A       0     0     4
  [16] .eh_frame         PROGBITS         0000000000006e58  00006e58
       0000000000000418  0000000000000000   A       0     0     8
  [17] .init_array       INIT_ARRAY       0000000000008af0  00007af0
       0000000000000008  0000000000000008  WA       0     0     8
  [18] .fini_array       FINI_ARRAY       0000000000008af8  00007af8
       0000000000000008  0000000000000008  WA       0     0     8
  [19] .data.rel.ro      PROGBITS         0000000000008b00  00007b00
       0000000000000140  0000000000000000  WA       0     0     32
  [20] .dynamic          DYNAMIC          0000000000008c40  00007c40
       00000000000001b0  0000000000000010  WA       7     0     8
  [21] .got              PROGBITS         0000000000008df0  00007df0
       0000000000000208  0000000000000008  WA       0     0     8
  [22] .data             PROGBITS         0000000000009000  00008000
       0000000000000068  0000000000000000  WA       0     0     16
  [23] .bss              NOBITS           0000000000009080  00008068
       0000000000000140  0000000000000000  WA       0     0     32
  [24] .comment          PROGBITS         0000000000000000  00008068
       0000000000000012  0000000000000001  MS       0     0     1
  [25] .shstrtab         STRTAB           0000000000000000  0000807a
       0000000000000100  0000000000000000           0     0     1
Key to Flags:
  W (write), A (alloc), X (execute), M (merge), S (strings), I (info),
  L (link order), O (extra OS processing required), G (group), T (TLS),
  C (compressed), x (unknown), o (OS specific), E (exclude),
  D (mbind), l (large), p (processor specific)
```

先ほど例に挙げた `.text` や `.rodata` などが含まれていることがわかります。

ちなみに、このセクションの中身を見る場合は `readelf -x <セクション名>` で表示できます。`.rodata` の内容を表示すると、そのプログラムで表示される文字列を覗くことが出来ます。長いので貼りませんが、例えば `cat` だと、`--help` オプションで表示されるヘルプテキストなどを見ることが出来ます。

### プログラムヘッダテーブル

**セグメント** に関する情報が格納されます。これは主に実行時に必要なもので、セクションを役割・性質ごとにまとめたものです。「実行時に必要」という性質上、リンク可能ファイルには含まれません。

例えば `LOAD` というセグメントは「実行時にメモリに読み込む必要がある」という役割のセクションがまとまったものです。他にも、動的リンク情報に関するセクションがまとまった `DYNAMIC` セクションなどがあります。

そして、「性質」というのは、各セクションに対する **読み込み可能** ・ **書き込み可能** ・ **実行可能** といったフラグです。Linux におけるファイルのパーミッションみたいなものだと思ってください。

前述のセクションを例に挙げてみましょう。実行可能な機械語が格納される `.text` セクションは、プログラムを書き換えられると困るので、書き込み可能であってほしくないですね。ということで、このセクションが含まれるセグメントは **読み込み可能・実行可能** であることが期待されます。

また、`.rodata` はその名の通り読み込みだけ可能であってほしいです。グローバルもしくは静的な変数が置かれる `.data` は、読み込みと書き込みが可能であってほしいですね。

プログラムヘッダテーブルは、 `readelf` に `-l` オプション付与することで閲覧可能です。おなじみ `cat` のセクションを見てみましょう。

```
$ readelf -l /usr/sbin/cat

Elf file type is DYN (Position-Independent Executable file)
Entry point 0x3350
There are 13 program headers, starting at offset 64

Program Headers:
  Type           Offset             VirtAddr           PhysAddr
                 FileSiz            MemSiz              Flags  Align
  PHDR           0x0000000000000040 0x0000000000000040 0x0000000000000040
                 0x00000000000002d8 0x00000000000002d8  R      0x8
  INTERP         0x0000000000000318 0x0000000000000318 0x0000000000000318
                 0x000000000000001c 0x000000000000001c  R      0x1
      [Requesting program interpreter: /lib64/ld-linux-x86-64.so.2]
  LOAD           0x0000000000000000 0x0000000000000000 0x0000000000000000
                 0x0000000000001578 0x0000000000001578  R      0x1000
  LOAD           0x0000000000002000 0x0000000000002000 0x0000000000002000
                 0x00000000000038a1 0x00000000000038a1  R E    0x1000
  LOAD           0x0000000000006000 0x0000000000006000 0x0000000000006000
                 0x0000000000001270 0x0000000000001270  R      0x1000
  LOAD           0x0000000000007af0 0x0000000000008af0 0x0000000000008af0
                 0x0000000000000578 0x00000000000006d0  RW     0x1000
  DYNAMIC        0x0000000000007c40 0x0000000000008c40 0x0000000000008c40
                 0x00000000000001b0 0x00000000000001b0  RW     0x8
  NOTE           0x0000000000000338 0x0000000000000338 0x0000000000000338
                 0x0000000000000050 0x0000000000000050  R      0x8
  NOTE           0x0000000000000388 0x0000000000000388 0x0000000000000388
                 0x0000000000000044 0x0000000000000044  R      0x4
  GNU_PROPERTY   0x0000000000000338 0x0000000000000338 0x0000000000000338
                 0x0000000000000050 0x0000000000000050  R      0x8
  GNU_EH_FRAME   0x0000000000006db0 0x0000000000006db0 0x0000000000006db0
                 0x00000000000000a4 0x00000000000000a4  R      0x4
  GNU_STACK      0x0000000000000000 0x0000000000000000 0x0000000000000000
                 0x0000000000000000 0x0000000000000000  RW     0x10
  GNU_RELRO      0x0000000000007af0 0x0000000000008af0 0x0000000000008af0
                 0x0000000000000510 0x0000000000000510  R      0x1

 Section to Segment mapping:
  Segment Sections...
   00
   01     .interp
   02     .interp .note.gnu.property .note.gnu.build-id .note.ABI-tag .gnu.hash .dynsym .dynstr .gnu.version .gnu.version_r .rela.dyn
   03     .init .text .fini
   04     .rodata .eh_frame_hdr .eh_frame
   05     .init_array .fini_array .data.rel.ro .dynamic .got .data .bss
   06     .dynamic
   07     .note.gnu.property
   08     .note.gnu.build-id .note.ABI-tag
   09     .note.gnu.property
   10     .eh_frame_hdr
   11
   12     .init_array .fini_array .data.rel.ro .dynamic .got
```

まずプログラムヘッダの一覧が表示され、次にそのセグメントに含まれるセクションの一覧が表示されています。例えば、プログラムヘッダ一覧の 2 番目にある `INTERP` セグメントは R（Readable・読み込み可能）というフラグが設定されています。また、下にあるセクション一覧の 2 番目（0 から始まっているので、上から 2 番目のセグメントに対応するセクション一覧は 01 であることに注意してください）を見ると、 `.interp` セクションが含まれていることがわかります。

先ほど例に挙げたセクションを見てみましょう。 `.text` セクションは 3 番目のセグメントにあるようです。0 から始まってるので、プログラムヘッダ一覧の上から 3+1 番目にあるセグメントが `.text` を含んでいるということになります。これに相当するセグメントは `LOAD` で、フラグは RE（Executable：実行可能）です。

他のセクションも見てみます。 `.rodata` セクションは 4 番目のセグメントで、これは `LOAD` セグメントに対応します。フラグは R のみです。`.data` セクションは 5 番目で、これも `LOAD` セグメントです。フラグは RW（Writable：書き込み可能）ですね。

## 参考資料

- Linux man page of elf  
  `man elf` コマンドで閲覧できます（Arch Linux であれば `man-pages` パッケージを入れましょう）。
- elf.h  
  `/usr/include/elf.h` などにあります。
- [Executable and Linkable Format - Wikipedia]  
  なかなか分かりやすい図とざっくりした解説があります。日本語版は今のところあんまり書かれてないです。

正直に言うと、これらの情報があればこのページは必要ないです（？？）。

ELF について詳しくなりたければ、man を 256 回くらい読めば良いと思います。具体的にどの値がどのパラメータと対応しているのかを知りたければ、適宜 `elf.h` を参照しましょう。
