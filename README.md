# 6502マシン語ゲームプログラミング

昨年（2018年）末、会社から少し長めの年末年始休暇を頂いたものの特に用事もなかったので、暇つぶしにファミコンゲームの開発方法を独学で習得して、[COSMIC SHOOTER](https://github.com/suzukiplan/stg-for-nes)と[BATTLE MARINE](https://github.com/suzukiplan/battle-marine-fc)という２本のゲームを開発しました。

ファミコンのハードウェアは既に解析し尽くされていて、Web上に解説情報が多く存在しますが、リファレンスとして有用なものや断片的なものは多くある反面、本のように網羅的且つ体系的に解説されたものが見当たりませんでした。そういった情報を発信するのであれば、Webよりも本の方が適しています。しかし、テーマ的に出版が難しい内容なので、残念ながら本はありません。（同人誌ならあるかもしれませんが）

そこで、私が実際にファミコンゲームを開発する過程で得られた知見を元に、そのプログラミング技法を可能な限り網羅的且つ体系的に解説を試みることにしました。本書を一通り読むことで、プログラマなら誰でもファミコン用ゲームを開発できるようになることを目指します。

> ただし、あまりにも初歩的な内容を細かく説明すると文書量が膨大になって読み難くなってしまうため、目安として情報処理技術者試験のLv2（基本情報処理技術者試験）合格程度の知識レベルがあることを前提に解説します。

## Agenda

本書は、「基礎編」「実践編」「リファレンス編」の3編構造になっています。

最低限「基礎編」「実践編」だけ学習すれば、「ファミコンのプログラミングができるプログラマ」になれます。「リファレンス編」の内容は全部覚える必要が無く、プログラミング時に必要に応じて調べれば良い内容ですが、リファレンス編の内容を全部覚えきることができれば、「ファミコンの基本ハードウェア仕様を概ね全て把握しているプログラマ」にクラスチェンジすることができます。

読者は、まずは「基礎編」「実践編」の内容をじっくり学習し、「リファレンス編」の内容は流し読みするか、実際にプログラムを書きながら必要に応じて参照することで、効率的にファミコン・プログラミングの技術を身につけることができます。

### 基礎編

- (0) 数値表記
- (1) ファミコンの仕組み
- (2) 開発環境を準備
- (3) ファミコンのプログラムでできること

### 実践編

- (4) プログラミング・マニュアル

### リファレンス編

- (5) メモリマップ（主記憶）
- (6) Mapper
- (7) PPU
- (8) メモリマップ（VRAM）
- (9) スプライト
- (10) APU
- (11) コントローラ

### 本編　(WIP)

- (12) マシン語ゲームプログラミング

# 基礎編

## (0) 数値表記

本書では、以下の数値表記を用います。

- `%1010101` : 2進数（先頭に `%` を記載）
- `12345678` : 10進数
- `$FFFF` : 16進数（先頭に `$` を記載）

> 全般的に [cc65](https://cc65.github.io/) の書式仕様に倣います。

## (1) ファミコンの仕組み

ファミコン（ファミリーコンピュータ）に限らず、パソコンやスマホなどのコンピュータ全般は大まかに、

- 入力装置
- 処理装置（演算装置+制御装置）
- 記憶装置 (主記憶装置, 補助記憶装置)
- 出力装置

の４つの装置で構成されています。

> 情報処理のテキストによっては処理装置を演算装置と制御装置に分類しているかもしれませんが、ここではそれらを纏めて処理装置と呼びます。

ファミコンの場合基本的には、

- 入力装置 ＝ コントローラ
- 処理装置 ＝ 本体（CPU/PPU/APU）
- 主記憶装置 ＝ 本体（メモリ）
- 補助記憶装置 ＝ カセット（ROM）
- 出力装置 ＝ テレビ/スピーカ

となっています。

ファミコンにカセットを挿入して電源を投入すると、カセット内のデータ（とプログラム）がメモリ（$0000〜$FFFF）の所定の番地に読み込まれて、$8000番地からプログラムが動き始めます。そして、プログラムが動いた（処理した）結果の映像と音声がテレビとスピーカに出力されます。

## (2) 開発環境を準備

### プログラマ向け

PCはWindows、Mac、Linuxの何れでも問題なく開発できます。
開発に最低限必要なツールは以下の3つです。

- アセンブラ: [cc65](https://cc65.github.io/)
- テキストエディタ: [Visual Studio Code](https://code.visualstudio.com/download)推奨
- ファミコンエミュレータ: [Mesen](https://www.mesen.ca/)推奨

テキストエディタはソースコードを書くためのツールですが、左側にファイルのエクスプローラ、右側にテキストエディタという構成が使いやすいので、[Visual Studio Code](https://code.visualstudio.com/download)がオススメです。
誰か6502のマシン語コード入力に特化した良い感じのプラグイン作ってください。

統合開発環境は無いので、アセンブルやリンクはTerminalかMS-DOSプロンプト上で行ってください。（私はMakefileを作ってGNU makeでそれらの操作を行っていますが、この点はお好みでご自由にして頂ければ良いと思います）

動作確認にはファミコンエミュレータを使うのが便利です。
ファミコンエミュレータは沢山ありますが、プログラマにとって必要な全ての機能を備えている最強のエミュレータは [Mesen](https://www.mesen.ca/)です。
メモリダンプはもちろん、ブレークポイント設定、ステップ実行、逆ステップ実行（これがかなり凄い）などデバッグに必要な全ての機能が備わっています。

もちろん、最終的なテストはファミコンの白ROM（フラッシュROM）に焼いて実機で行うのが望ましいです。

### デザイナ向け

ファミコンのグラフィックの作成をグラフィックデザイナに依頼するのは極めて難儀なことです。本当なら、デザイナも本書を呼んでファミコンのハードウェア制約を理解した上でグラフィックの描画や画面デザインを行うことが望ましいです。ただ、そうなるとデザイナ参入障壁が極めて高くなってしまい、我々プログラマが描いたショボイグラフィックだけでゲームを作らなければなりません。

[YY-CHR](http://www.geocities.jp/yy_6502/yychr/0yychr.html)というグラフィック・エディタを用いれば、デザイナでもファミコンの制約に則ったグラフィックを（半強制的に）描画することができます。

ただし、[YY-CHR](http://www.geocities.jp/yy_6502/yychr/0yychr.html)はかなりクセの強いツールで、デザイナにとってグラフィック・エディタとは手足みたいなものなので「YY-CHRで描いてください」と依頼すると「は？」と言われてしまうかもしれません。

そこで、プログラマはデザイナに以下のように依頼してみると良いかもしれません。

```
- グラフィックは3色以下の16x16（BG）または8x8（スプライト）の単位で描く
- 128x128ピクセルのサイズで2枚（BG+スプライト）納品
```

ただし、ファミコンのハードウェア制約で上記仕様だと実機では再現できいケースが（割と多く）あるので、その点は実機での動作を見ながらプログラマとデザイナで都度相談しながら作るしかありません。

## (3) ファミコンのプログラムでできること

ファミコンには、モステクノロジー社製のMOS6502というCPUをリコー社がカスタマイズしたRP2A03というCPUが搭載されています。RP2A03は6502から一部機能（デシマルモードと呼ばれる機能）を削減した上で、AY-3-8910をベースにカスタマイズしたと思しき音源モジュール（APU; Audio Porcessing Unit）をオンチップで搭載しています。つまり、「ファミコンのプログラムでできること≒6502のプログラムでできること」となります。

6502のプログラムができることは、大まかに以下の５つのことです。

- メモリの特定番地からデータをレジスタへ読み込む（load, pull）
- メモリの特定番地へレジスタのデータを記憶する（store, push）
- レジスタ間での値の入れ替え（transfer）
- 演算（足し算, 引き算, 各種論理演算, 比較）
- 分岐（branch, jump）

上記とファミコンが提供するハードウェア機能を利用してゲームの開発を行います。

次章で具体的なプログラミング方法について解説します。

# 実践編

## (4) プログラミング・マニュアル

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

- `LDA` メモリの値（または即値）をaへload
- `STA` aの値をメモリへstore
- `LDX` メモリの値（または即値）をxへload
- `STX` xの値をメモリへstore
- `LDY` メモリの値（または即値）をyへload
- `STY` yの値をメモリへstore

ゲーム内で変数を扱う場合、WRAM（ユーザプログラムで自由に読み書きできる領域）の番地毎に役割を決めておき格納することになります。
オールマシン語（数値の羅列のみ）でプログラミングする場合、番地と変数の意味を対応付けて覚えておく必要がありますが、[cc65](https://cc65.github.io/)であれば、全てのメモリ番地にラベルを付けることができるので、ラベルで変数を管理できます。

> 6502のプログラミングでは、原則全ての変数をグローバル変数で管理することになるイメージでOKです。ローカル変数を持つこともできますが、6502にはスタック領域が256バイトしかないので、ローカル変数を使おうものなら、一瞬でスタックオーバーフローしてしまいます。

load にはレジスタに読み込むメモリの指定方式として 即値（immediate）, アドレス（zero page / absolute / indirect）による指定方式があります。storeはアドレス指定のみできます。


#### immediate (load)

[cc65](https://cc65.github.io/) の場合、数値の前に `#` を付けることで即値表現になります。

```
;   assembly          C形式
    LDA #123        ; a = 123;
    LDA #$AB        ; a = 0xAB;
    LDA #%10101010  ; a = 0b10101010;
    LDX #123        ; x = 123;
    LDX #$AB        ; x = 0xAB;
    LDX #%10101010  ; x = 0b10101010;
    LDY #123        ; y = 123;
    LDY #$AB        ; y = 0xAB;
    LDY #%10101010  ; y = 0b10101010;
```

#### address (load / store)

##### zero page

6502は、メモリの $0000〜$00FF 番地の256バイトの領域（ゼロページ）へのアクセス速度が、他のアドレス（$0100〜$FFFF 番地）よりも1サイクル速くフェッチできる分高速な特性があります。

```
;   assembly          C形式
    ; load
    LDA $00         ; a = memory[0x0000];
    LDA $10, x      ; a = memory[0x0010 + x];
    LDX $20         ; x = memory[0x0020];
    LDX $30, y      ; x = memory[0x0030 + y];
    LDY $40         ; y = memory[0x0040];
    LDY $50, x      ; y = memory[0x0050 + x];
    ; store
    STA $00         ; memory[0x0000] = a;
    STA $10, x      ; memory[0x0010 + x] = a;
    STX $20         ; memory[0x0020] = x;
    STX $30, y      ; memory[0x0030 + y] = x;
    STY $40         ; memory[0x0040] = y;
    STY $50, x      ; memory[0x0050 + x] = y;
```

上記例の $00, $20, $40 のようにアドレス値をそのまま指定する方式だけでなく、$10, $30, $50 のようにインデックス（x/y）を添え字として指定することも可能です。

ただし、aの添字にyを使った場合、absolute扱いになります。

##### absolute

ゼロページとはアクセス先アドレスが異なるだけで基本的には同じですが、先述のようにaの添字としてyを使うことができる点が異なります（以下の例の $0320 への load / store）。

```
;   assembly          C形式
    ; load
    LDA $0300       ; a = memory[0x0300];
    LDA $0310, x    ; a = memory[0x0310 + x];
    LDA $0320, y    ; a = memory[0x0320 + y];
    LDX $0330       ; x = memory[0x0330];
    LDX $0340, y    ; x = memory[0x0340 + y];
    LDY $0350       ; y = memory[0x0350];
    LDY $0360, x    ; y = memory[0x0360 + x];
    ; store
    STA $0300       ; memory[0x0300] = a;
    STA $0310, x    ; memory[0x0310 + x] = a;
    STA $0320, y    ; memory[0x0310 + y] = a;
    STX $0330       ; memory[0x0330] = x;
    STX $0340, y    ; memory[0x0340 + y] = x;
    STY $0350       ; memory[0x0350] = y;
    STY $0360, x    ; memory[0x0360 + x] = y;
```

> __cycle penalty:__ absoluteでインデックス（x/y）を加算した結果、ページ（アドレス上位8bit）の値が即値と異なる場合（例えば `LDA $HHLL, x` で `$LL + x >= $100` の時）、一部命令の実行サイクルが1Hz長くなる現象（サイクルペナルティ）が発生します。
>
> - cycle penaltyの影響を受ける命令:
>   - ADC, AND, CMP, EOR, LDA, LDX, LDY, ORA, SBC
> - cycle penaltyの影響を受けない命令:
>   - ASL, DEC, INC, LSR, ROL, ROR, STA

##### indirect

特殊な load / store のアドレス指定方式として indirect と呼ばれるものがあります。
indirectはアキュームレータ（a）のみ対応しており、インデックス（x/y）と組み合わせて使用しますが、xとyで用途が異なります。

```
    LDA ($00, x)   ; ex1
    STA ($10, x)   ; ex2
    LDA ($20), y   ; ex3
    STA ($30), y   ; ex4
```

indirectとは、特定のゼロページアドレスに格納されている値（リトルエンディアンの2バイト）が示す番地に格納された値を load / store する方法です。

ex1をCスタイルで書くと以下のようになります。

```c
    // ex1: LDA ($00, x) in C
    unsigned short addr;
    addr = memory[0x00 + x];
    addr += memory[0x00 + x + 1] * 256;
    a = memory[addr]; // ex2なら: memory[addr] = a;
```

また、ex3をCスタイルで書くと以下のようになります。

```c
    // ex3: LDA ($20), y in C
    unsigned short addr;
    addr = memory[0x20];
    addr += memory[0x20 + 1] * 256;
    a = memory[addr + y]; // ex4なら: memory[addr + y] = a;
```

> indirectの用途は、C言語の関数ポインタ配列による分岐処理（switch/case文よりも高速なインデックスによる条件分岐）に近いイメージだと考えられます。

> __cycle penalty:__ indirect yでもabsolute x/yと同様にサイクルペナルティが発生する可能性があります。

### レジスタ間の値の転送（transfer）

aとx、aとyは相互に値を転送することができます。

- `TAX` aをxへ転送（x = a）
- `TXA` xをaへ転送（a = x）
- `TAY` aをyへ転送（y = a）
- `TYA` yをaへ転送（a = y）

転送方向がどっちだったのかを忘れてしまいバグを作り込むことが割とよくあるため、ニーモニックを正式名称で覚えることをオススメします。

- `TAX` = Transfer a to x (aをxへ転送)
- `TXA` = Transfer x to a (xをaへ転送)
- `TAY` = Transfer a to y (aをyへ転送)
- `TYA` = Transfer y to a (yをaへ転送)

また、あまり使わないと思いますが、xとsの間で値を転送することもできます。

- `TXS` = Transfer x to s (xをsへ転送)
- `TSX` = Transfer s to x (sをxへ転送)

> 私の経験上、リセット割り込み時にスタックポインタを初期化したい時にのみ使っています。8bitコンピュータのスタックは256バイトのリングバッファなので、敢えてsを初期化する必要もないかもしれませんが、例えばスタック領域の後半部分をワークエリアとして使いたい時などは初期化が必要になるかと思われます。

### スタック（push / pull）

6502では、メモリの $0100〜$01FF番地の256バイトの領域をスタックとして使います。

16bit以上のCPUでのプログラミングに慣れていると、スタックがたったの256バイトしか無いと何もできないのではないかと思われるかもしれません。確かにC言語で関数呼び出しをすればスタックはすぐに尽きてしまいます。しかし、フルアセンブリ言語でコードを組む場合、スタックが必要な場面がかなり限られるので256バイトで十分に賄えます。

> C言語で関数呼び出しをする場合、呼び出し規約にもよりますが最低限、呼び出し元で戻りアドレス（6502の場合は2バイト）と引数の情報、更に呼び出し先でローカル変数の情報をスタックに格納します。

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

PHA, PLA, PHP, PLP でスタックを明示的に利用するケースは割と稀かもしれませんが, スタックは次のケースで暗示的に利用されます。

- `JSR` : サブルーチンへジャンプする時, 戻りアドレス（2byte）がスタックに push される
- `RTS` : サブルーチンから復帰する時, スタックから戻りアドレス（2byte）が pull される
- NMI/IRQ発生した時, P（1byte）と戻りアドレス（2byte）が push される
- `RTI` : 割り込みから復帰する時, P（1byte）と戻りアドレス（2byte）が pull される

ユーザプログラムでスタックを利用するケースの大半は `JSR` と `RTS` になると思われます。

> サブルーチンのみ利用した場合最大128階層まで潜ることができ、129階層潜ると最初にJSRした処理に復帰できなくなります。
> 意図して129階層も潜るような変態プログラムを作る人は居ないと思いますが、再帰的に潜った結果129階層バグにエンカウントすることなら、稀にあるかもしれません。

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

更に、掛け算（MUL）、割り算（DIV）、剰余算（MOD）といった演算命令はそもそもありません。
それらの演算をしたい場合は、論理演算（ASL, LSR, AND）で代用します。

> CPUで掛け算（MUL）や除算（DIV）がサポートされるようになったのは、16bitコンピュータの頃からです。ただし、16bitコンピュータの頃の掛け算や割り算はものすごく遅かったので、ゲームの場合は論理演算、事務処理の場合はMUL/DIVといった形での使い分けをしていました。現在のコンピュータであれば論理演算と四則演算の速度は同じぐらいですが、FPUが進化した結果、浮動小数点数演算もかなり高速になりました。ちなみに、JavaScriptだと（ランタイムの実装にも依存しますが）数値型を基本的には浮動小数点数で管理しているエンジンが多いため、論理演算を行おうとすると整数変換が間に挟まる関係で「論理演算の方が遅い」という結果になったりします。

### インデックス (x/y)

インデックス（x/y）は、メモリを配列的にアクセスする時の添字としてよく使う8bitレジスタです。

xとyはほぼ同じですが、xの方がやや優遇されています。具体的には、演算処理でゼロページアクセスでサイクル数短縮したい場面ではxしか使えず、yだと非ゼロページと同等の方法でアクセスしてしまうので、ゼロページの性能面でのメリットが無くなってしまいます。そのため、通常はxを用いるようにして、インデックスを2つ使いたい場面に限ってyを用いる形でプログラミングするのが望ましいです。

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

上記コードで　`ADC #3` の結果は, `a = a + 3` ではなく `a = a + 3 + 1(carry)` となります。この例で示したケースであれば、キャリーが確実に立っていることが明確なので問題無いかもしれませんが、キャリーが立っているかが不明確な場合は `CLC` 命令を実行してキャリーをクリアしてから `ADC` を実行するのが無難です。

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

> __cycle penalty:__ absoluteで加算したブランチ先のアドレスのページ（アドレス上位8bit）の値が、ブランチ元アドレスのページと異なる場合、命令の実行サイクルが1Hz長くなる現象（サイクルペナルティ）が発生します。

### 比較演算とステータス（p）の関係

比較演算（CMP, CPX, CPY）は他の演算命令と比べると少しだけ特殊です。

比較演算は、演算対象のレジスタ（a/x/y）と演算対象値を比較した結果を見てステータス（p）を設定します。

以下に具体例を示します。

```
CMP #32 ; レジスタ（a）と32（即値）を比較
```

- a >= 32 なら carry がセット（BCSで分岐できる）
- a < 32 なら carry がクリア（BCCで分岐できる）
- a == 32 なら zero がセット（BEQで分岐できる）
- a != 32 なら zero がクリア（BNEで分岐できる）

> 余談になりますが、何故上記のようなルールになっているのかを理解するため、6502の演算回路の内部構造に少しだけ触れておきます。
> 比較命令は内部的に減算と同じ回路で作られています。
> ただし、減算命令（SBC）と違い減算結果のレジスタへの格納を省略しています。
> 上記例で示した `CMP #32` の場合 `SEC(carryをset)した状態で a - 32` の演算を行い、aが32以上なら結果が負数にならないのでcarryのセットが保たれ、32未満なら結果が負数になるのでcarryがクリアされます。そして、aが32と同値なら結果は0なのでゼロがセットされ、aが32でなければ結果は非0なのでゼロがクリアされます。
> _（「少し特殊」なのは、その他の演算と違い「結果の格納」をskipしている点と、比較は常にunsigned扱いなのでオーバーフローに影響しない点です）_

### ジャンプ（無条件分岐とサブルーチン）

6502には、ブランチ命令（条件分岐）以外にジャンプ命令（無条件分岐）もあります。

- `JMP` 特定番地へジャンプ
- `JSR` 特定番地のサブルーチンへジャンプ
- `RTS` サブルーチンから `JSR` の次命令位置へ復帰
- `RTI` 割り込みからの復帰 (解説省略)

> 無条件分岐の飛び先アドレスは相対番地（absolute）だけでなく絶対番地を指定することもできます。相対番地は飛び先アドレスを8bitの相対値で指定するのに対して、絶対番地は16bitで指定するため、フェッチするバイト数の関係で相対番地で飛んだ方が高速に実行できます。そのため、アセンブラは飛び先ラベルと現在番地のアドレスを見て、相対番地が使える場合は相対番地で飛び、飛べない場合は絶対番地で飛ぶようにアセンブルします。

### 割り込み

本書では割り込みの解説を割愛します。

> NMIやIRQはとりあえずゲームを作ってみる段階であれば不要で、実際 [COSMIC SHOOTER](https://github.com/suzukiplan/stg-for-nes) や [BATTLE MARINE](https://github.com/suzukiplan/battle-marine-fc) では（リセット割り込みの対応処理以外）割り込みを使っていません。
> ただし、割り込みを使いこなすことで出来るようになる幅がかなり広がるのも事実です。
> まずは、割り込み無しでゲームを試作してみて、それから習得するのが良いと思います。

### I/O制御

6502には「マリオを画面に描画」みたいな命令は存在しません。

ファミコンに限らずコンピュータ全般では、I/Oポートへの入出力（I/O制御）でハードウェアと通信することで、そのハードウェアが提供する機能を実行します。ファミコンの場合、コントローラからの入力受付、PPU（映像処理装置）への映像出力指示、APU（音声処理装置）への音声出力指示をI/O制御で行います。

つまり、「マリオを画面に描画」を行いたい場合は、PPUへそういう指示を出します。

6502以外のCPU（Z80や8086など）にはI/O制御命令が存在しますが、6502では特定の主記憶（メモリ）番地へアクセスすることでI/O制御を行うことができます。

「メモリの何処の番地がどのような用途で使えるか」という情報のことを「メモリマップ」と呼び、プログラマはメモリマップ（やI/Oマップ）を参照しながら、対象ハードウェアを操作するプログラムを作成することになります。

# リファレンス編

## (5) メモリマップ（主記憶）

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

標準Mapper（Mapper0）のROMカセットの場合、バンク切り替えを行うことができないため、プログラムのROM領域は常に固定ですが、MMC3などの拡張Mapperを使うことでバンク切り替えにより動的に（LOWまたはHIGHの）プログラムROMの内容を異なるプログラムROMに置き換えることができます。

## (6) Mapper

ファミコンは、カセットに特殊なチップを積むことでハードウェアを拡張することができます。初期の頃のゲームと後期の頃のゲームで表現力に大幅な違いがあるのはこのためです。本書では、特殊なチップを積んでいない標準構成（Mapper0）に絞って解説します。

Mapper0では、プログラムが最大32KB（16KB×2）、キャラクタデータが最大8KB（4KB×2）しか使うことができません。つまり、Mapper0のROMカセットにはたったの40KB（モノによっては24KB）のデータしか入っていません。

初期の頃のファミコンゲームは、40KBという現代のゲームのグラフィック1枚にも満たないデータの中にプログラム、グラフィック、音楽といった全てのデータが入っていたと考えると驚異的です。

> 初代スーパーマリオブラザーズはMapper0で作られているので、 __スーパーマリオブラザーズぐらいの規模感のゲームなら40KB以内で作ることができる__ といえます。なので、本書では標準構成（Mapper0）に絞った解説で十分だと考えています。

## (7) PPU

ファミコンはCPUとは別に画像処理を行うPPU（Picture Processing Unit）を搭載しています。
PPUの機能は、背景画像（BG）の描画とスプライトの描画の２つです。
そして、主記憶とは別にVRAM（Video RAM）とスプライトRAM（OAM; Object Attribute Memory）呼ばれる2つのメモリ空間があります。

PPUへのアクセスは、主記憶の $2000~$2007番地（と$4013番地）への load / store により実現します。

### $2000 (Basic settings / store only)

> __PPUの基本設定__ を行う書き込み（出力）専用のI/Oポート

usage:

```
    LDA #%VPHBSINN
    STA $2000
```

- `V:` vBlank発生を割り込みで検出 `(0: off, 1: on)`
- `P:` PPU type `(0: master, 1: slave)`
- `H:` スプライトのサイズ `(0: 8x8, 1: 8x16)`
- `B:` BGのキャラクタテーブル番号 `(0: $0000, 1: $1000)`
- `S:` スプライトのキャラクタテーブル番号 `(0: $0000, 1: $1000)`
- `I:` VRAM入出力時のアドレス変化値 `(0: +1, 1: +32)`
- `NN:` メインスクリーン `(00: $2000, 01: $2400, 10: $2800, 11: $2C00)`

### $2001 (Mask settings / store only)

> __画面表示の設定__ を行う書き込み（出力）専用のI/Oポート

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

### $2002 (Drawing status / load only)

> __画面描画の状態取得__ を行う読み取り（入力）専用のI/Oポート

usage:

```
    STA $2002 ; A = #%VSN.....
```

- `V:` vBlankの発生状態 `(0: 描画中, 1: vBlank中)`
- `S:` 0番スプライトの描画 `(0: 未検出, 1: 検出)`
- `N:` 描画ラインのスプライト描画上限 `(0: 8以下, 1: 9以上)`

ブラウン管テレビは、走査線（scan line）と呼ばれる仕組みで 一般的には1/60秒の周期（60Hz）で上から下に書けて1行づつ線（スキャンライン）を描画しています。更に細かく補足すると、スキャンラインは左から順番にピクセル単位で描画を行っています。

例えば、横4px縦3pxの画面であれば、下図のように描画されます。

<img src="images/scan-line-4x3.png"/>

また、ファミコンの場合、画面の下の方に見えないラインが存在しており、この見えないラインを描画している期間のことを vBlank と呼びます。

下図に横4px縦3pxの画面に1行のvBlankがある場合の例を示します。

<img src="images/scan-line-4x3-1blank.png"/>

ファミコンは、スキャンライン242本（各256px）、vBlankが20本という画面構成になっています。そして、スキャンライン描画中（vBlank描画期間外）にPPUメモリ（VRAM+OAM）の更新を行うと描画内容に乱れが生じる仕様なので、vBlank描画タイミングをチェックするために V を参照します。

<img src="images/scan-line-nes.png" width=50%/>

ゲームのメインループ処理は、走査線の描画と合わせて、1ループを1/60秒で実行するように同期（垂直同期）処理を入れて実装することになるため、スキャンライン描画期間に行う処理ブロック（PPUにアクセスせず主記憶のみ更新する「主記憶更新ブロック」）と、vBlank描画の期間中に行う「PPU更新ブロック」の2ブロックに分割されることになります。

なお、S（0番スプライトの描画タイミング）をチェックすることでメインループの処理を3ブロックに分割しているソフトが多くあります。実際、スーパーマリオブラザーズのメインループも3ブロックに分割して実装している筈で、そのことは画面の構成を見れば一目瞭然です。この点については $2005 (Window) で詳しく解説します。

> 厳密には、スキャンラインの1行づつの描画（CPUクロック数 = 113+2/3Hz）が終わった後に次の垂直同期での表示内容を描画する方式であれば、スキャンラインの描画期間であっても描画内容が乱れません。ファミコンは、昨今のコンピュータと違ってOS等が搭載されていない厳密なリアルタイムシステムなので、実際命令の実行クロックを計算して描画処理を行う高度な変態プログラム（イギリスのレア社が開発したBattle Toadsなど）が存在しました。

### $2003~$2004 (Sprite)

> __スプライトRAM (OAM)__ への入出力を行うI/Oポート

$2003 への store でスプライトRAM (OAM) のアクセス先アドレスを設定して $2004 への load / store でOAMへの入出力を行いますが __このI/Oポートは実用上の理由でほぼ使いません。__

> スプライトに関しては「(9) スプライト」でより詳しい解説を行うので、ここでの解説は省略します。

### $2005 (Window position / store only)

> __BGのスクリーン表示位置（Window）__ の設定を行います。

> NesDev.comやその他のサイトでは、$2005のことを「スクロール」と呼んでいますが、Window (画面の表示範囲) と覚えた方がわかり易いかと思います。（実現できる機能はスクロールで間違いありませんが）

usage:

```
    LDA X座標
    STA $2005
    LDA Y座標
    STA $2005
```

$2005は上記のように2回連続で store を行う必要があり、1回目でX座標、2回目でY座標を設定します。

### Window

ファミコンの画面サイズは256x240pxで、この中にBGとスプライトが合成表示されます。

BGは、8x8の矩形を1単位（キャラクタ）として、キャラクタをタイル状に敷き詰めて表示する仕様なので __1画面 = 32x30マス = 256x240px__ となっています。

<img src="images/bg-tiles.png" width=50%>

> ただし、上端8px、下端8px、左端8px、右端8px（図面上の灰色の■の部分）はブラウン管だと表示されないので、実際の __有効可視範囲は 240x224px（30x28マス）__ です。更に、上下左右16pxには重要な情報を表示してはならないという不文律がありました。つまり、有効可視範囲内であっても画面端は見切れる可能性があります。そのため、__安全可視範囲は更に狭い 224x208px (28x26マス)__ となっています。（気の利いたエミュレータは有効化可視範囲で表示する仕様のものが多いですが、256x240px全て表示してしまっているエミュレータも存在していて、256x240pxで表示しているエミュレータは画面端にノイズ情報が表示されてしまっています）

BGの画面（SCREEN）は4つあり、スクリーン・レイアウトは下図のようになります。

<img src="images/screen-usage.png" width=50%>

> 標準MAPPERの場合、図面でも示していますが画面4つ中2つはミラーになります。そのため、4画面全てを自由に描画できる訳ではありません。水平ミラーか垂直ミラーのどちらにするかは iNESヘッダーの4byte目の第6bit（※実機カセットの場合は配線）で設定するので、プログラムで動的に変更することができません。

つまり、内部座標系としては 512x480（64x60マス）の領域があり、$2005で指定した任意座標の256x240px矩形が画面への表示領域（Window）となります。

<img src="images/window-basic.png" width=50%>

### Split window (parallax scrolling)

$2002 (Drawing status) で、スプライト番号0の描画タイミングを検出できますが、この仕様を応用して、

- スプライト0描画前のWindow位置
- スプライト0描画後のWindow位置

に異なる座標を（$2005で）指定することで、Windowを上下に分割することができます。

<img src="images/window-advance.png" width=50%>

そして、このWindow分割のテクニックを利用することで、「Window2のみスクロールさせる」という部分的なスクロール（ラスタスクロール; parallax scrolling）が実現できます。

> なお、Window分割は走査線の描画タイミングを利用したものなので、Windowの上下分割はできますが左右分割はできません。ファミコンのゲームで画面の上か下にスコア表示領域を設けているものが多く、画面の右か左にスコア表示領域を設けているものがあまりないのはこのためです。（ギャラガや拙作のCOSMIC SHOOTERでは画面を左右分割していますが、それらはWindow分割ではなく単純に1枚のWindowで処理しています）

### $2006~$2007 (VRAM access)

> __VRAM__ への入出力を行うI/Oポート

usage:

```
    LDA VRAMアドレス上位1byte
    STA $2006
    LDA VRAMアドレス下位1byte
    STA $2006
    LDA #1
    STA $2007 ; 指定アドレスに1を書き込む
```

- $2006 へ2回 store を行うことでVRAMのアクセス先番地を設定
- $2007 へ store で値を書き込む
- $2007 から load で値を読み込む
- $2007 へのアクセスの都度、アクセス先番地が +1 または +32 される
  - 加算される値は $2000 (Basic settings) の I の値に依存

VRAMへキャラクタを横書きする時は、$2006のstore 1セット(2回)だけで行い、I=0で$2007へ連続storeすれば良いことになります。

> 縦書き（I=1）の使い所はあまり無いかもしれません（少なくとも私は必要な場面にまだ遭遇していません）

## (8) メモリマップ（VRAM）

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
|$3F00-$3F0F|パレットテーブル0（BG）|4色パレット×4つ|
|$3F10-$3F1F|パレットテーブル1（スプライト）|4色パレット×4つ|
|$3F20-$3FFF|未使用|パレットのミラー|

### パターンテーブル ($0000-$1FFF)

パターンテーブルは、ROMから読み込まれるキャラクタデータが格納されている領域で、4096バイト（$1000バイト）のテーブル2つで構成されています。そして、1テーブルあたり、8x8ピクセル単位 の 2bitカラー のキャラクタデータ（8×2=16バイト）が 256個（16×256=4096バイト）格納されています。

下図に、1キャラクタのデータレイアウトを示します。

<img src="images/pattern-table.png" />

### ネームテーブル ($2000-$2FFF)

Windowの解説で先述した通り、ファミコンのBGは32x30マスの画面4つで構成されおり、ネームテーブルが各画面の実際の描画パターンになります。

1つのネームテーブルは、960byteのタイル領域と64byteの属性領域の2つの領域に分かれています。

- タイル領域: 1マス毎にキャラクタデータ（0〜255）の配置パターンを格納
  - Size = 32 × 30 = 960 byte
- 属性領域: 2x2マス毎にキャラクタデータに割り当てるパレット番号（0〜3）を格納
  - Size = 64 byte

タイル領域の仕様は明確でわかり易いのですが、属性領域は（VRAMの搭載メモリサイズを節約している関係で）やや分かり難い仕様なので、詳細に解説します。

属性領域は、1バイトにつき 4x4マス (32x32px) 分のパレット番号を持っています。

属性領域1バイトが管理する区画のイメージを下図に示します。

<img src="images/attr-area.png"/>

1バイト（8bit）で持つことが出来る2bitのパレット番号のデータ数の上限は4つ（8÷2）までしかないため、パレット番号の情報は 2x2マス (16x16px) 単位で持ちます。

属性領域 1byte のビットレイアウトは次のようになっています。

- bit 7〜6 : 左上2x2マスのパレット番号（%00, %01, %10, %11）
- bit 5〜4 : 右上2x2マスのパレット番号（%00, %01, %10, %11）
- bit 3〜2 : 左下2x2マスのパレット番号（%00, %01, %10, %11）
- bit 1〜0 : 右下2x2マスのパレット番号（%00, %01, %10, %11）

### パレットテーブル（$3F00〜$3F1F）

- BG用、スプライト用にそれぞれ パレットを4つづつ 定義します
- 1つのパレットは、4色の色番号を指定した4byteの配列です
  - 各パレットの %00色目 は常に透明色になります
  - 例外的にスプライトパレット0番の %00色目 ($3F10) は画面全体の背景色になります
- 色番号は $00〜$3F (下図) の中から任意の1色を選択します

<img src="images/palette.png"/>

- 64個の色番号がありますが、重複色が存在するため実際は52色です
- `0D` `2D` `3D` を赤字で記載していますが、これらの色を使用すると実機の種類やエミュレータの種類によって出力結果が異なるため、使用しない方が良いです

> そもそもファミコンの色はデジタルRGB出力ではないため、アナログテレビでなければ完璧な色の再現ができないらしいです。（この辺は私の専門外なので、詳しいことはよく分かりません...）

## (9) スプライト

### メモリマップ（OAM）

- スプライトは、スプライトRAM（OAM）へデータを入力することで表示します
- スプライトは、8x8px または　8x16px サイズのキャラクタで、最大64個同時に表示することができます
- OAMは、スプライト1個につき4バイトの情報を持つ256（4×64）バイトの構造体配列です

```c
struct Sprite {
    UINT8 y;            // スプライトの標示座標（Y）
    UINT8 tile;         // キャラクタ番号
    UINT8 attribute;    // 属性
    UINT8 x;            // スプライトの標示座標（X）
};

struct Sprite OAM[64];
```

tile:
- 8x8であればキャラクタテーブルのタイル番号（0〜255）を指定
- 8x16の場合: `%tttttttp`
  - t: タイル番号×2 (右隣の奇数タイルが下側に表示)
  - p: パターンテーブル（0: $0000, 1: $1000）

> 何故8x16というモードがあるのに、16x8や16x16が存在しないのか。それは、ファミコンの画面表示はアナログテレビの走査線の仕様に則って設計されていることに由来すると考えられます。1本の走査線は横方向にドットを描画していき、ファミコンはその中でBGとスプライトの合成表示を行っているため、水平方向のハードリミットが垂直方向と比べて低いことが分かります。そのため、垂直方向であれば画面の上から下まで64個のスプライトを（少し重ねて）並べても問題無く表示できるのに対して、水平方向は8個までしか並べることができない制約があります。

attribute:
- bit 7: 上下反転標示
- bit 6: 左右反転表示
- bit 5: 0ならBGの前面、1ならBGの背面に標示
- bit 4〜2: 未使用
- bit 1〜0: パレット番号

### OAMへのアクセス

OAMへのアクセスには次の2種類の方法があります。

1. I/Oポート（$2003〜$2004）によるアクセス
2. WRAMからのDMA転送

OAMへのアクセスはvBlank期間中に行わなければなりませんが、64個全てのスプライト情報をI/Oポートで入出力するのは無理があります。そこで、WRAMの内容（256バイト）を一回のI/Oポート入力でOAMへ転送するDMA転送機能を用いてアクセスします。

DMA転送をする場合、予め転送先OAMアドレスに0を設定しておき、

```
    LDA #$00
    STA $2003
```

メインループ内でWRAMのページ番号を $4014 に store します。

```
    LDA #$03
    STA $4014
```

上記のコードでは WRAM　の 3ページ（$0300）の内容がOAMに転送されます。

> __ページ番号:__ 6502のメモリ空間は $0000〜$FFFF までありますが、上位8bitのことを一般的にページ番号と呼びます。例えば、$0000〜$00FF であれば　0ページ となります。

なお、DMA転送には513〜514程度のCPUサイクルを要します。

### 水平方向の表示上限

スプライトは、水平方向に最大8個までしか表示できない制約があります。

水平方向に9個以上のスプライトを表示した場合、スプライトにチラつきが生じますが、その瞬間フレームを見てみると8個しか表示されていない（9個目以降は描画が省略されている）ことが分かります。

## (10) APU

ファミコンのAPUには、次の波形の音声を出力する機能があります。

1. 矩形波（2系統）
2. 三角波
3. ノイズ
4. DPCM

APUへのI/Oは、主記憶の $4000〜$4015 に対する store により行います。

### $4000〜$4003 (矩形波CH1)

```
    ; エンベロープ（波形パターン）の設定
    LDA #%DDLCVVVV
    STA $4000
```

- `D` : デューティ比（00: 87.5%, 01: 75%, 10: 50%, 11: 25%）
- `L` : 再生時間カウンタ（0: 無効, 1: 有効）
- `C` : 音量の固定化（0: 可変, 1: 固定）
- `V` : 音量（0〜15）

> __波形とデューティ比の解説:__ 矩形波（Square）は0と1のみ（16biなら1と-1のみ）で作る非常にシンプルな波形です。0の期間（周期）が220回（Hz）で、1の期間が220回で構成する波形（440Hz）を再生すれば、通常の矩形波のラの音が鳴ります。そして、波形の周期を倍（880Hz）にすれば1オクターブ低いラが鳴り、周期を半分（220Hz）にすれば1オクターブ高いラが鳴ります。そして、ここまで解説した0と1をちょうど半分のタイミングで区切った状態（通常の波形）の波形がデューティ比50%となります。0と1の周期をそれぞれ110Hz、330Hz（つまり、デューティ比25%）にすると、ピッチ（音程）はラですが、音色が変わります。デューティ比25%の矩形波の波形はノコギリ波（Saw）に近いため、どちらかといえばノコギリ波に近い音色になります。デューティ比25%と75%は、単音で聴くと音色に変化はありません。しかし、複数チャンネルで同時に発音する（つまり、合成波形にする）と微妙に結果の音響に変化が生じます。

```
    ; スウィープ（周波数の変化）の設定
    LDA #%EPPPSSSS
    STA $4001
```

- `E` : スウィープ（0: 無効, 1: 有効）
- `P` : 期間（0〜7）
- `S` : シフト値（-8〜7）

> __ファミコンの音源種別:__ ファミコンの音源（RP2A03）の種別をカテゴライズする場合、このスウィープ機能が存在する関係でFM音源の一種に分類されます。「ファミコンはPSG音源だ」と主張する人も居るかもしれませんが、広義の意味で全てのチップチューン音源はPSG（Programmable Sound Generater; プログラム可能な音源）ですので、それもまた間違いではありません。

```
    ; タイマー下位8bitの設定
    LDA #%TTTTTTTT
    STA $4002
    ; タイマー上位3bitと再生する長さの設定
    LDA #%LLLLLTTT
    STA $4003
```

- `T` : 周期値（0〜2047）
- `L` : 再生する長さ（0〜31）

音声の周波数（音程） `f` は `T` の値から次の式で計算できます。

```
f = CPU / (16 * (T + 1))
T = (CPU / (16 * f)) - 1
CPU = 1.789773MHz
```

### $4004〜$4007 (矩形波CH2)

設定できる値は $4000〜$4003 (矩形波CH1) と同じなので解説を省略します。

### $4008〜$400B (三角波)

```
    ; リニアカウンタの設定
    LDA #%CRRRRRRR
    STA $4008
    ; タイマー下位8bitの設定
    LDA #%TTTTTTTT
    STA $400A
    ; タイマー上位3bitと再生する長さの設定
    LDA #%LLLLLTTT
    STA $400B
```

- `C` : リニアカウンタの有効化（0: 無効, 1: 有効）
- `R` : リニアカウンタ値 (0〜127)
- `T` : 周期値（0〜2047）
- `L` : 再生する長さ（0〜31）

音声の周波数（音程） `f` は矩形波と同様 `T` から求まりますが、矩形波よりも1オクターブ低く（つまり、矩形波の周波数÷2で）発生されます。

### $400C〜$400F (ノイズ)

```
    ; エンベロープ（波形パターン）の設定
    LDA #%--LCVVVV
    STA $400C
```

- `L` : 再生時間カウンタ（0: 無効, 1: 有効）
- `C` : 音量の固定化（0: 可変, 1: 固定）
- `V` : 音量（0〜15）

```
    ; タイマー設定
    LDA #%T---PPPP
    STA $400E
    ; 再生する長さの設定
    LDA #%LLLLL---
    STA $400F
```

- `T` : タイマー設定（0: 無効, 1: 有効）
- `P` : 周期値パターン (0〜15)
- `L` : 再生する長さ（0〜31）

なお、 `P` の設定値により求まる周期値は NTSC（日本のアナログテレビで採用されていた規格)）と PAL（ヨーロッパで普及したアナログカラーテレビの規格）で微妙に異なります。

|$400E|NTSC|PAL|
|---|---|---|
|%10000000|4|4|
|%10000001|8|8|
|%10000010|16|14|
|%10000011|32|30|
|%10000100|64|60|
|%10000101|96|88|
|%10000110|128|118|
|%10000111|160|148|
|%10001000|202|188|
|%10001001|254|236|
|%10001010|380|354|
|%10001011|508|472|
|%10001100|762|708|
|%10001101|1016|944|
|%10001110|2034|1890|
|%10001111|4068|3778|

### $4010〜$4015 (DPCM)

すみません、まだ試していないので解説省略します。

以下URLで詳しく解説されているので、使いたい方はそちらを参照してください。

[https://wiki.nesdev.com/w/index.php/APU_DMC](https://wiki.nesdev.com/w/index.php/APU_DMC)

## (11) コントローラ

ファミコンには標準で2プレイヤ分のジョイパッド（コントローラ）があり、読み取り準備（$4016番地へのstore）後に、$4016番地(1P側) or $4017番地(2P側) の load により入力状態を取得できます。

なお、日本の初代ファミコン、NEWファミコン（AV仕様のファミコン）と海外のNESでは若干の仕様差がある点に注意が必要です。

- 初代ファミコン: 2P側のみマイク入力がありselect/startが無い
- NES: 1P/2P同じ仕様
- NEWファミコン: NES仕様に準拠

つまり、2Pを扱う場合は動作対象のエミュレータが「初代ファミコン・エミュレータ」なのか「NESエミュレータ」or「NEWファミコン」なのかを意識して実装しなければなりません。現代のゲームプログラマが2Pを対象にしたゲームをデザインする場合、ファミコンとNESの共通項の範囲で扱うのが妥当だと考えられます。

### usage

```
    ; 読み取りのための前準備
    LDA #$01
    STA $4016
    LDA #$00
    STA $4016

    LDA $4016
    AND #$01
    BEQ end_push_a
    ; 1PのAボタンを押した時の処理
end_push_a:

    LDA $4016
    AND #$01
    BEQ end_push_b
    ; 1PのBボタンを押した時の処理
end_push_b:

    LDA $4016
    AND #$01
    BEQ end_push_select
    ; 1PのSELECTボタンを押した時の処理
end_push_select:

    LDA $4016
    AND #$01
    BEQ end_push_start
    ; 1PのSTARTボタンを押した時の処理
end_push_start:

    LDA $4016
    AND #$01
    BEQ end_push_up
    ; 1Pの上ボタンを押した時の処理
end_push_up:

    LDA $4016
    AND #$01
    BEQ end_push_down
    ; 1Pの下ボタンを押した時の処理
end_push_down:

    LDA $4016
    AND #$01
    BEQ end_push_left
    ; 1Pの左ボタンを押した時の処理
end_push_left:

    LDA $4016
    AND #$01
    BEQ end_push_right
    ; 1Pの右ボタンを押した時の処理
end_push_right:
```

コントローラの状態を読み取りを始める前に、$4016のLSB (最下位bit) に1/0の順でstoreを行います。

> 1をstoreするとボタン読み取り位置が先頭に戻され、0をstoreすると読み取りの都度、ボタン位置がシフトしていく形になります。

その後、$4016をloadするとLSBに A, B, SELECT, START, UP, DOWN, LEFT, RIGHT の順番で1Pのボタンの入力状態が返ります。（2Pのコントローラの状態を知りたい場合は$4017をloadします）

# 本編

## (12) マシン語ゲームプログラミング

本書のここまでの内容で、ファミコンのゲームを作るため技術的な基礎（実践編）とハードウェア知識（リファレンス編）が整いました。ここから先は、簡単なシューティングゲームの実装を通じて、ファミコンでゲームを作る上の実践的なテクニックを紹介する「本編」を執筆する予定でした。

が、ザックリと流れだけ記してお茶を濁しておくことにします。

- §0 描画ツールで描いたグラフィックをCHR形式に変換
- §1 スプライトで16x16のキャラクタを表示する
- §2 表示したキャラクタをコントローラで動かす
- §3 キャラクタからショットを撃てるようにする
- §4 BGの描画
- §5 敵キャラクタの実装
- §6 敵キャラクタを破壊できるようにする（当たり判定）
- §7 スコアの実装
- §8 ゲームオーバー
- §9 タイトル画面
- §10 効果音を入れる

> 上記は COSMIC SHOOTER を実装時のコーディングの流れに概ね沿っていると思われるので、[COSMIC SHOOTER の commitログ](https://github.com/suzukiplan/stg-for-nes/commits/master)を古いのものから順に眺めて頂いても良いかもしれません。

# あとがき

可能な限り、（Mapper0に限って）網羅的＆体系的な文書を書こうと思ったのですが、結果的に「割り込み」と「DPCM」の解説は省きました。何故なら、私がまだ試していないので。ですが、それらを使うための予備知識は既に（NesDev wikiで）確認済みなので、機会を見て使ってみて、その結果この文書をアップデートするかもしれません。

平成も終わろうとしている2019年初頭、何故今さら1983年に発売されたゲーム機のプログラミングを解説しているのか。それは、「そこに山があるから」みたいな曖昧で単純な理由ではなく、もっと明確な事情があります。

それは、ファミコンでゲームを作れるようになれば、プラットフォームや機種依存の呪縛から逃れることができるためです。

ただし、それだけの理由であれば、世の中には既に有用なゲームエンジンが数多く存在するので、敢えてファミコンを選択する人は居ないでしょう。実際、ファミコンをひとつのゲームエンジンと見做した時、他と比べてそのハードリミットはあまりにも低すぎるので。

しかし、ゲームを作る上での必要十分な機能が備わっていて、事実スーパーマリオブラザーズみたいなゲームであれば作ることができます（しかも、たったの40KBで）。また、エミュレータも豊富に存在しており、中にはJavaScriptで作られたものもあるのでブラウザで動かすこともできてしまいます。この点（ポータビリティの高さ）においては間違いなく最強のゲームエンジンです。

もちろん、仮にハードリミットが許容できたとしても、フルアセンブリ言語で組まなければならないなどの高いハードルがあるのも事実です。

それでも尚、ファミコンでゲームを作ることに挑戦したい奇特な方々も多く居るだろうと願っており、本書がそれらの方々を後押しをできれば幸いです。

# 参考サイト

- [Nesdev Wiki](https://wiki.nesdev.com/w/index.php/Nesdev_Wiki)
- [NES研究室](http://hp.vector.co.jp/authors/VA042397/nes/index.html)
- [魔法使いの森（ファミコンの画面について）](https://www.wizforest.com/OldGood/ntsc/famicom.html)

# 引用商標

- 任天堂は、日本またはその他地域における任天堂株式会社の登録商標です。
- ファミコン及びファミリーコンピュータは、日本またはその他地域における任天堂株式会社の登録商標です。
- スーパーマリオブラザーズは、日本またはその他地域における任天堂株式会社の登録商標です。
- ギャラガは、日本またはその他地域における株式会社バンダイナムコエンターテインメントの登録商標です。
- Battle Toadsは、米国またはその他地域におけるMicrosoft社の登録商標です。
