# 6502マシン語ゲームプログラミング

本書は、[COSMIC SHOOTER](https://github.com/suzukiplan/stg-for-nes)と[BATTLE MARINE](https://github.com/suzukiplan/battle-marine-fc)の開発で得られた知見をベースに、ファミコンでのゲーム開発方法をなるべく誰でもできるように解説することを目標に書かれたマニュアル（兼、私自身の備忘録）です。

## Agenda

- (1) ファミコンの仕組み
- (2) 開発環境を準備
- (3) ファミコンのプログラムでできること
- (4) コーディング・マニュアル
- (5) メモリマップ（主記憶）
- (6) Mapper
- (7) PPU
- (8) メモリマップ（VRAM）
- (9) スプライト
- (10) APU

## (1) ファミコンの仕組み

ファミコン（ファミリーコンピュータ）に限らず、パソコンやスマホなどのコンピュータ全般は大まかに、

- 入力装置
- 処理装置（演算装置+制御装置）
- 記憶装置 (主記憶装置, 補助記憶装置)
- 出力装置

の４つの装置で構成されています。情報処理のテキストによっては処理装置を演算装置と制御装置に分類しているかもしれませんが、ここではそれらを纏めて処理装置と呼びます。

ファミコンの場合、入力装置＝コントローラ、処理装置＝本体（CPU/PPU/APU）、主記憶装置＝本体（メモリ）、補助記憶装置＝カセット（ROM）、出力装置＝テレビ/スピーカとなっています。

ファミコンにカセットを挿入して電源を投入すると、カセット内のデータ（とプログラム）がメモリの所定の番地に読み込まれて、所定の位置（$8000番地）からプログラムが動き始めます。そして、プログラムが動いた（処理した）結果の映像と音声がテレビとスピーカに出力されます。

## (2) 開発環境を準備

PCはWindows、Mac、Linuxの何れでも問題なく開発できます。
開発に最低限必要なツールは以下の3つです。

- アセンブラ: [cc65](https://cc65.github.io/)
- テキストエディタ: [Visual Studio Code](https://code.visualstudio.com/download)推奨
- ファミコンエミュレータ: [VirtualNES](http://virtuanes.s1.xrea.com/)推奨

テキストエディタはソースコードを書くためのツールですが、左側にファイルのエクスプローラ、右側にテキストエディタという構成が使いやすいので、[Visual Studio Code](https://code.visualstudio.com/download)がオススメです。
誰か6502のマシン語コード入力に特化した良い感じのプラグイン作ってください。

ファミコンエミュレータは何でも良いのですが、WRAMをダンプできる機能があるとデバッグが捗るのでVirtualNESのチート支援機能が役立ちます。私の開発環境はMacなので、普段は[Nestopia](http://nestopia.sourceforge.net/)でテストして、どうしてもメモリダンプを見たい場面ではWineでVirtualNESを使っています。

> _イチイチWineで動かすのが面倒なので、私の方で開発者向けの機能が揃った独自エミュレータをCocoaAppで作ろうかと思っています。_

## (3) ファミコンのプログラムでできること

ファミコンには、モステクノロジー社製のMOS6502というCPUをリコー社がカスタマイズしたRP2A03というCPUが搭載されています。RP2A03は6502から一部機能（デシマルモードと呼ばれる機能）を削減した上で、AY-3-8910をベースにカスタマイズしたと思しき音源モジュール（APU; Audio Porcessing Unit）をオンチップで搭載しています。つまり、「ファミコンのプログラムでできること≒6502のプログラムでできること」となります。

6502のプログラムができることは、大まかに以下の５つのことです。

- メモリの特定番地からデータをレジスタへ読み込む（load, pull）
- メモリの特定番地へレジスタのデータを記憶する（store, push）
- レジスタ間での値の入れ替え（transfer）
- 演算（足し算, 引き算, 各種論理演算, 比較）
- 分岐（branch, jump）

次章で具体的なコーディング方法について解説します。

## (4) コーディング・マニュアル

本章では、ファミコンのゲームを開発するために必要な最低限のプログラミング方法を解説します。若干ボリュームのある内容かもしれませんが、本章の内容を読んで理解さえできれば、ニーモニック表とメモリマップを眺めながらファミコンのプログラムを作れるようになるので頑張ってください。（可能な限り誰でも分かるような記述を心掛けたつもりですが、果たしてどうなんだろうか？）

### レジスタ

レジスタとは、CPU内部の高速な記憶装置のことで、6502には以下の6種類があります。

- a（アキュームレータ）
- x（インデックス）
- y（インデックス）
- p（ステータス）
- s（スタックポインタ）
- pc（プログラムカウンタ）

pc以外のレジスタは全て8bitで、pcのみ16bitです。

### メモリからの読み込みと記憶（load / store）

レジスタへメモリの値を読み込むことを load といい、レジスタの値をメモリへ記憶することを store といいます。そして、アキュームレータ（a）とインデックス（x/y）レジスタが load / store を直接行えるレジスタとなっています。

- `LDA` メモリの値をaへload
- `STA` aの値をメモリへstore
- `LDX` メモリの値をaへload
- `STX` aの値をメモリへstore
- `LDY` メモリの値をaへload
- `STY` aの値をメモリへstore

### レジスタ間の値の転送（transfer）

aとx、aとyは相互に値を転送することができます。

- `TAX` aをxへ転送（x = a）
- `TXA` xをaへ転送（a = x）
- `TAY` aをyへ転送（y = a）
- `TYA` yをaへ転送（a = y）

結構よく使うのですが、転送方向がどっちだったのかを忘れてしまいバグを作り込むことが割とよくあります。なので、ニーモニックを正式名称で覚えることをオススメします。

- `TAX` = Transfer a to x (aをxへ転送)
- `TXA` = Transfer x to a (xをaへ転送)
- `TAY` = Transfer a to y (aをyへ転送)
- `TYA` = Transfer y to a (yをaへ転送)

### スタック（push / pull）

6502では、メモリの $0100〜$01FF番地の256バイトの領域をスタックとして使います。

16bit以上のCPUでのプログラミングに慣れていると、スタックがたったの256バイトしか無いと何もできないのではないかと思われるかもしれません。確かにC言語で関数呼び出しをすればスタックはすぐに尽きてしまいます。しかし、フルアセンブリ言語でコードを組む場合、スタックが必要な場面がかなり限られるので256バイトで十分に賄えます。

スタックへのpush（積み込み）とpull（取り出し※popと同じ意味）はアキュームレータ（a）とステータス（p）のみ対応しています。

- `PHA` aをスタックへpush
- `PLA` スタックからaへpull(pop)
- `PHP` pをスタックへpush
- `PLP` スタックからpへpull(pop)

レジスタの一時記憶をしたい場合にaをpush、演算結果のステータスを一時記憶したい場合にpをpushします。

x/yを一時記憶したい場合、以下の例のようにa経由でpush/pullします。

```
    ; assembly        C形式
    TXA             ; a = x
    PHA             ; stack[s++] = a

    ~~~ xを用いた処理を実行 ~~~

    PLA             ; a = stack[--s];
    TAX             ; x = a
```

### アキュームレータ (a)

アキュームレータ（a）は、演算することに適した8bitレジスタで、足し算、引き算、論理演算、比較といった演算命令（下記）を実行することができます。

- `ADC` 足し算 (Add with carry)
- `SBC` 引き算 (Sub with carry)
- `ORA` 論理和
- `AND` 論理積
- `EOR` 排他的論理和
- `ASL` 左シフト
- `LSR` 右シフト
- `ROL` 左ローテート
- `ROR` 右ローテート
- `BIT` ビット比較
- `CMP` 比較

注意点として、インクリメント（1加算）とデクリメント（1減算）はできません。
また、キャリーを用いない加算と減算も無い点に注意する必要があります。

### インデックス (x/y)

インデックス（x/y）は、メモリを配列的にアクセスする時の添字としてよく使う8bitレジスタです。

x/yは両方ともほぼ同機能ですが、xの方がやや優遇されています。具体的には、演算処理でゼロページアクセスでサイクル数短縮したい場面ではxしか使えず、yだと非ゼロページと同等の方法でアクセスしてしまうので、ゼロページの性能面でのメリットが無くなってしまいます。

実行できる演算命令は、インクリメント、デクリメント、比較のみです。

- `INX` Xをインクリメント
- `DEX` Xをデクリメント
- `CPX` Xを比較
- `INY` Yをインクリメント
- `DEY` Yをデクリメント
- `CPY` Yを比較

インデックスの使い所を示す例として、メモリの $0300〜$030F に 255 をセットする例を示します。

```
    ; assembly        C形式
    ldx #0          ; x = 0
    lda #255        ; a = 255
loop:
    sta $0300, x    ; memory[$0300 + x] = a(255)
    inx             ; x = x + 1
    cpx #16
    bcc loop        ; if (x < 16) goto loop
```

### ステータス (p) と条件分岐（ブランチ）

ステータス（p）は条件分岐（branch）の判定に用いる8bitレジスタで、各bit毎に以下のような意味を持っています。

|bit位置|意味|補足|
|---|---|---|
|0|キャリー|演算の結果aがキャリー（桁あふれ）した場合にセット|
|1|ゼロフラグ|loadまたは演算の結果a/x/yがゼロになった場合にセット|
|2|IRQ禁止|セット時 = IRQ禁止(SEI), リセット時 = IRQ許可(CLI)|
|3|デシマルモード|ファミコンでは未使用|
|4|ブレークモード|BRK発生時にセット, IRQ発生時にクリア|
|5|予約|常にセットされている|
|6|オーバーフロー|演算の結果aがオーバーフローした場合にセット|
|7|ネガティブ|演算の結果aが負数になった場合にセット|

> __キャリー（桁あふれ）とオーバーフローの違い:__ やや判り難いですが補足しておきます。8bit数値はunsignedなら0〜255, signedなら-128〜127の範囲の数値を表現できます。unsignedでの桁あふれ(ex: `255+1=0` など) なら キャリー, signedでの期待値と異なる結果(ex: `64+64=-128` など) なら オーバーフロー になります。

キャリーについては、分岐（branch）以外にも足し算（ADC; add with carry）と引き算（SBC; sub with carry）にも影響する点に注意が必要です。この特性は、16bit以上の数値型を扱う場合には便利ですが、8bit数値で扱う場合しばしばバグの温床になります。

以下にキャリーが加算されるパターンのわかり易い例を示します。

```
    CMP #10
    BCC other_proc
    ADC #3 ; この場合carryがセットされている
```

上記コードで　`ADC #3` の結果は, `a = a + 3` ではなく `a = a + 3 + 1(carry)` となります。この例で示したケースであれば、キャリーが確実に立っているので問題無いかもしれませんが、キャリーが立っているか不明確なケースでは `CLC` 命令を実行してキャリーをクリアしてから `ADC` を実行するのが無難です。

ステータス（p）に影響する条件分岐（ブランチ）命令には次のものがあります。

- `BEQ` ゼロフラグが 1(set) であればジャンプ
- `BNE` ゼロフラグが 0(clear) であればジャンプ
- `BCS` キャリーが 1(set) であればジャンプ
- `BCC` キャリーが 0(clear) であればジャンプ
- `BVS` オーバーフローが 1(set) であればジャンプ
- `BVC` オーバーフローが 0(clear) であればジャンプ
- `BMI` ネガティブが 1(set) であればジャンプ
- `BPL` ネガティブが 0(clear) であればジャンプ

ブランチ命令の注意点として、飛び先アドレスの指定方法が相対指定（absolute）で、8bitの範囲（-128〜127 ※0を除く）でなければならない点に気をつける必要があります。

(ex1: ブランチできない例)

```
    BCS label

    ~ 128バイト以上の命令がある ~

label:
```

(ex2: ブランチできない例)

```
label:

    ~ 129バイト以上の命令がある ~

    BCS label
```

### ジャンプ（無条件分岐とサブルーチン）

6502には、ブランチ命令（条件分岐）以外にジャンプ命令（無条件分岐）の命令もあります。

- `JMP` 特定番地へジャンプ
- `JSR` 特定番地のサブルーチンへジャンプ
- `RTI` サブルーチンから `JSR` の次命令位置へ復帰

### 割り込み

本書では割り込みの解説を割愛しています。

> NMIやIRQはとりあえずゲームを作ってみる段階であれば不要で、実際 [COSMIC SHOOTER](https://github.com/suzukiplan/stg-for-nes) や [BATTLE MARINE](https://github.com/suzukiplan/battle-marine-fc) では（リセット割り込みの対応処理以外）割り込みを使っていません。
> ただし、割り込みを使いこなすことで出来るようになる幅がかなり広がるのも事実です。
> まずは、割り込み無しでゲームを試作してみて、それから習得するのが良いと思います。

### I/O制御

6502には「マリオを画面に描画」みたいな命令は存在しません。

ファミコンに限らずコンピュータ全般では、I/Oポートの制御（I/O制御）を行うことで外部のハードウェアとの入出力を行い、そのハードウェアが提供する機能を実行します。ファミコンの場合、コントローラからの入力受付、PPU（映像処理装置）への映像出力指示、APU（音声処理装置）への音声出力指示などをI/O制御により行います。

つまり、「マリオを画面に描画」を行いたい場合は、PPUへそういう指示を出すことになります。

6502以外のCPU（Z80や8086など）にはI/O制御命令が存在しますが、6502では特定の主記憶（メモリ）番地へアクセスすることでI/O制御を行うことができます。

「メモリの何処の番地がどのような用途で使えるか」という情報のことを「メモリマップ」と呼びます。

## (5) 主記憶のメモリマップ

ファミコンのプログラムからアクセスできる主記憶装置の番地は $0000〜$FFFF までの 65536バイト（64KB）の範囲で、そのメモリマップは下表のようになっています。

|番地|用途|補足|
|---|---|---|
|$0000-$00FF|WRAM (zero page)|高速にアクセス可能なWRAM|
|$0100-$01FF|スタック|6502共通仕様|
|$0200-$07FF|WRAM (2〜7 page)||
|$0800-$1FFF|未使用|$0000-$07FFのミラー|
|$2000-$2007|I/Oポート (PPU)||
|$2008-$3FFF|未使用|$2000-$2007のミラー|
|$4000-$401F|I/Oポート (APU, etc)||
|$4020-$5FFF|拡張RAM|Mapper0では使用できない|
|$6000-$7FFF|バッテリーバックアップRAM|Mapper0では使用できない|
|$8000-$BFFF|プログラムROM LOW|ROMから読み込まれる|
|$C000-$FFFF|プログラムROM HIGH|ROMから読み込まれる|

標準Mapper（Mapper0）のROMカセットの場合、バンク切り替えを行うことができないため、プログラムのROM領域は常に固定ですが、MMC3などの拡張Mapperを使うことでバング切り替えにより動的に（LOWまたはHIGHの）プログラムROMの内容を異なるプログラムROMに置き換えることができます。

## (6) Mapper

ファミコンは、カセットに特殊なチップを詰むことでハードウェアを拡張することができます。初期の頃のゲームと後期の頃のゲームで表現力に大幅な違いがあるのはこのためです。本書では、特殊なチップを詰んでいない標準構成（Mapper0）に絞って解説します。

Mapper0では、プログラムが最大32KB（16KB×2）、キャラクタデータが最大8KB（4KB×2）しか使うことができません。つまり、Mapper0のROMカセットにはたったの40KB（モノによっては24KB）のデータしか入っていません。

初期の頃のファミコンゲームは、40KBという現代のゲームのグラフィック1枚にも満たないデータの中にプログラム、グラフィック、音楽といった全てのデータが入っていたと考えると驚異的です。

> 初代スーパーマリオブラザーズはMapper0で作られているので、 __スーパーマリオブラザーズぐらいの規模感のゲームなら40KB以内で作ることができる__ といえます。なので、本書では標準構成（Mapper0）に絞った解説で十分だと考えています。

## (7) PPU

ファミコンはCPUとは別に画像処理を行うPPU（Picture Processing Unit）を搭載しています。
PPUの主な機能は、背景画像（BG）の描画とスプライトの描画の２つです。
PPUには、主記憶とは別にVRAM（Video RAM）とスプライトRAM呼ばれるメモリ空間があります。

PPUへのアクセスは、主記憶の $2000~$2007番地（と$4013番地）への load / store により実現します。

### $2000 (Basic settings / store only)

__PPUの基本設定__ を行う書き込み（出力）専用のI/Oポート

usage:

```
    LDA #%VPHBSINN
    STA $2000
```

- `NN:` メインスクリーン `(00: $2000, 01: $2400, 10: $2800, 11: $2C00)`
- `I:` VRAM入出力時のアドレス変化値 `(0: +1, 1: +32)`
- `S:` スプライトのキャラクタテーブル番号 `(0: $0000, 1: $1000)`
- `B:` BGのキャラクタテーブル番号 `(0: $0000, 1: $1000)`
- `H:` スプライトのサイズ `(0: 8x8, 1: 8x16)`
- `P:` PPU type `(0: master, 1: slave)`
- `V:` vBlank発生を割り込みで検出 `(0: off, 1: on)`

### $2001 (Mask settings / store only)

__画面表示の設定__ を行う書き込み（出力）専用のI/Oポート

usage:

```
    LDA #%BGRSBMmC
    STA $2001
```

- `B:` 青を強調表示 `(0: off, 1: on)`
- `G:` 緑を強調表示 `(0: off, 1: on)`
- `R:` 赤を強調表示 `(0: off, 1: on)`
- `S:` スプライト表示 `(0: off, 1: on)`
- `B:` BG表示 `(0: off, 1: on)`
- `M:` 左端8x8のスプライト表示 `(0: off, 1: on)`
- `m:` 左端8x8のBG表示 `(0: off, 1: on)`
- `C:` モノクロ表示 `(0: color, 1: mono)`

### $2002 (V-SYNC status / load only)

__画面描画（垂直同期）の状態取得__ を行う読み取り（入力）専用のI/Oポート

usage:

```
    STA $2002 ; A = #%VSN.....
```

- `V:` vBlankの発生状態 `(0: 描画中, 1: vBlank中)`
- `S:` 0番スプライトの描画 `(0: 未検出, 1: 検出)`
- `N:` 描画ラインのスプライト描画上限 `(0: 8以下, 1: 9以上)`

ブラウン管テレビは、垂直同期（V-SYNC）と呼ばれる仕組みで 一般的には1/60秒の周期（60Hz）で上から下に書けて行ラインを描画しています（更に細かくいうと行ラインは左から順番にピクセル単位で描画を行っています）。

例えば、横4px縦3pxの画面であれば、下記に順番でピクセル（点）表示されます。

```
0123 (line1)
4567 (line2)
89AB (line3)
```

ファミコンの場合、画面の下の方に見えないラインが存在しており、この見えないラインを描画している期間のことを vBlank と呼びます。上記の例で、4行目にvBlankがある場合は、以下のようになります。

```
0123 (visible line1)
4567 (visible line2)
89AB (visible line3)
CDEF (hidden line4 = vBlank)
```

ファミコンの場合、上記の例が横256px、縦240行に拡大され、更に16行のvBlank期間が存在すると考えれば良い。

vBlankの期間内にVRAMの更新を行うと、描画内容が強かに乱れます。そのため、$2002 をチェックしてvBlankが発生している期間内にVRAMの更新を完了させなければなりません。

> S (0番スプライトの描画) の説明は省略します（後で詳しく解説します）が、これは非常に有効な機能です。この仕組みを用いることでBGの画面表示を上下に2分割したりラスタスクロール（部分スクロール）を実現することができます。

### $2003~$2004 (Sprite)

__スプライトRAM (OAM)__ への入出力を行うI/Oポート

$2003 への store でスプライトRAM (OAM) のアクセス先アドレスを設定して $2004 への load / store でOAMへの入出力を行いますが __このI/Oポートは実用上の理由でほぼ使いません。__

> スプライトに関しては「(9) スプライト」でより詳しい解説を行うので、ここでの解説は省略します。

### $2005 (Window position / store * 2 only)

__BGのスクリーン表示位置（Window）__ の設定を行います。

> NesDev.comやその他のサイトでは、$2005のことを「スクロール」と呼んでいますが、Window (画面の表示範囲) と覚えた方がわかり易いかと思います。（実現できる機能はスクロールで間違いありませんが）

usage:

```
    LDA X座標
    STA $2002
    LDA Y座標
    STA $2002
```

$2005は上記のように2回連続で store を行う必要があり、1回目でX座標、2回目でY座標を設定します。

### Window

ファミコンの画面サイズは256x240pxで、この中にBGとスプライトが合成表示されます。

BGは、8x8の矩形を1単位（キャラクタ）として、キャラクタをタイル状に敷き詰めて表示する仕様なので __1画面 = 32x30マス = 256x240px__ となっています。

<img src="images/bg-tiles.png" width=50%>

> ただし、上端8px、下端8px、左端8px、右端8px（図面上の灰色の■の部分）はブラウン管だと表示されないので、実際の __有効可視範囲は 240x224px（30x28マス）__ です。

BGの画面（SCREEN）は4つあり、スクリーン・レイアウトは下図のようになります。

<img src="images/screen-usage.png" width=50%>

> 標準MAPPERの場合、図面でも示していますが画面4つ中2つはミラーになります。そのため、4画面全てを自由に描画できる訳ではありません。水平ミラーか垂直ミラーのどちらにするかは iNESヘッダーの4byte目の第6bitで指定します。

つまり、内部座標系としては 512x480（64x60マス）の領域があり、$2005で指定した任意座標の256x240px矩形が画面への表示領域（Window）となります。

<img src="images/window-basic.png" width=50%>

$2002 (V-SYNC status) で、スプライト番号0の描画タイミングを検出できるので、この仕様を用いることで、「スプライト0描画前のWindow位置」と「スプライト0描画後のWindow位置」を$2005で指定することで、Windowを上下に分割することが可能です。

<img src="images/window-advance.png" width=50%>

そして、Window分割のテクニックを応用することで、「Window2のみスクロールさせる」という部分的なスクロール（ラスタスクロール）を実現できます。

> なお、Window分割はVSYNCが上から下へ流れるタイミングを応用したものなので、Windowの上下分割はできますが左右分割はできません。ファミコンのゲームで画面の上か下にスコア表示領域を設けているものが多く、画面の右か左にスコア表示領域を設けているものがあまりないのはこのためです。（ギャラガや拙作のCOSMIC SHOOTERでは画面を左右分割していますが、それらはWindow分割ではなく単純に1枚のWindowで処理しています）

### $2006

### $2007

## (8) VRAMのメモリマップ

VRAMには $0000〜$3FFF までの16384バイト（16KB）のメモリ空間があり、メモリマップは下表のようになっています。

|番地|用途|補足|
|---|---|---|
|$0000-$0FFF|パターンテーブル0|ROMから読み込まれる|
|$1000-$1FFF|パターンテーブル1|ROMから読み込まれる|
|$2000-$23FF|ネームテーブル0|画面0のBG配置パターン|
|$2400-$27FF|ネームテーブル1|画面1のBG配置パターン|
|$2800-$2BFF|ネームテーブル2|画面2のBG配置パターン|
|$2C00-$2FFF|ネームテーブル3|画面3のBG配置パターン|
|$3000-$3EFF|未使用|ネームテーブルのミラー|
|$3F00-$3F0F|パレット0|4色パレット×4つ|
|$3F10-$3F1F|パレット1|4色パレット×4つ|
|$3F20-$3FFF|未使用|パレットのミラー|


### パターンテーブル

TODO: あとで書く

### BGスクリーン

TODO: あとで書く

### ネームテーブル

TODO: あとで書く

### パレット

TODO: あとで書く

## (9) スプライト

スプライト用の256バイトのメモリ空間は、全64個のスプライトの情報構造体（各4バイト）で構成されています。

```c
struct Sprite {
    UINT8 y;
    UINT8 tile;
    UINT8 attribute;
    UINT8 x;
};
```

TODO: あとで書く

## (10) APU

TODO: あとで書く

## 制約と誓約

ゲームや音楽には NEW OLD SCHOOL という宗派があって、それは一般的ではないもののもの凄い信仰心により支えられています。昨今のレトロゲーム・ブームとは別の括りかもしれません。彼等は、新しいハードウェア環境であっても、意図的に旧来のハードウェア的な制約を誓約として課してゲームや音楽を創っています。
誓約というのは例えば、現代ならスプライトに色数制限はなくて24bitカラーの全色を使うことができますが、あえて16色以下しか使わないようにするといったものです。色彩的な観点では使用できる同系色を3色以下とした方がバランスが良いので、そういった誓約は意味が無いようでいて意味があることも稀によくありますが、大半は明確は意味はありません。
FM音源の時代に活躍したコンポーザーが、MIDI全盛以降に影を潜める例が数えればキリが無い程あることからも分かるように、制約環境下の方がパフォーマンスを発揮できるクリエイターが存在します。彼等は制約が創作の構成要素になっていたため、制約が失われたことでクリエイターとしての輝きを失います。NEW OLD SCHOOL とは、そういった技術の進化により行き場を失ったクリエイターの墓場なのかもしれません。

