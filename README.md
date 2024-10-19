# fdimageid

Floppy Disk Image File Identifier - フロッピーディスクイメージファイルの情報を表示する

## 概要

フロッピーディスクのイメージファイルには様々なフォーマットがあります(「ディスクのフォーマット」にも様々な種類があって、どちらも「フォーマット」と呼ばれるのでややこしい)。しかし、元になる「フロッピーディスク」というモノがあり、それを仮想化したものだという点はみな同じなので、相互に変換できるし、自分の望むフォーマットで統一することもできます。

…そう思っていたことが、自分にもありました…。

実際のところ、そうはいかなくて、D88をXDFに変換したらゲームが動かなくなったとか、同じDIMでも他の特定のフォーマットに変換できるものとできないものがあったりとかで、いろいろうまくいかないし、しかも、なんでうまくいかないのか、よく判らないんですよね。

そのあたりをうまいこと知ることができるツールが欲しいと思って、これを作りました。

本ツールの主な用途は、フロッピーディスクイメージファイルの情報を報告することです。報告する情報は、(「XDF」「D88」などの)ファイルのフォーマット情報や、(「DOS 1.23MB」「2HD Disk BASIC」などの)ディスクのフォーマット情報がまず挙げられますが、その他にも、フロッピーディスクにはプロテクトの黒歴史があり、特定のエラーが生じるようになっていたり、変態フォーマットだったり、そういったことも表現できるファイルフォーマットもありますので、そういう普通じゃない情報があれば報告するようにしています。XDF(べた)のファイルフォーマットが一番シンプルですので、そこで表現できない情報はひととおり報告しようとしています。

## 使い方

    Usage: fdimageid [options] file...

### オプション

オプションをつけないのが普通です。この場合、与えられたファイルの情報を標準出力に出力します。

以下のオプションは、アドホックに作者のニーズのために作ったものを残してあるだけであり、将来削除される可能性があります。

* `--convert-xdf`  
  XDF形式への変換を行います。入力ファイルは「2HD 1.23MB」などの、いわゆる一般的なジオメトリでなければなりません(そうでない場合は、その旨を表示して終了します)。出力ファイルのファイル名は、入力となったファイル名に`.xdf`を付加したものになります。(つまり、`….dim.xdf`のようになります)

* `--truncate-9`  
  各トラックの9セクタ目を切り捨てて、8セクタ/トラックのファイルとして出力します。入力ファイルはXDF形式またはD88形式の、8セクタ/トラックの2DDのイメージファイルでなければなりません(そうでない場合は「形式不明」となります)。出力ファイルのファイル名は、入力となったファイル名にもう一つ`.xdf`または`.d88`を付加したものになります。(つまり、`….xdf.xdf`のようになります)  
  このオプションを作った目的は、NP2kaiに8セクタ/トラックの2DDイメージを食わせた時に、書き込みが行われるとなぜか9セクタ/トラックで書き出されてしまうため(私の環境や使い方に誤りがあったのかもしれません)、そのようなファイルを8セクタ/トラックに補正するためです。  
  本オプションは「8セクタ/トラックでフォーマットされているのに、9個目の**不要な**セクタデータを持つ」イメージファイルを補正するためのものです。もともと9セクタ/トラックでフォーマットされているイメージファイルを本オプションで8セクタ/トラックに変換すると、データロストが生じますのでご注意ください。

* `--force`  
  危険な操作を実行します。  
  `--convert-xdf`と併用した場合、一般的なジオメトリでない場合でも構わずXDF形式として出力します。しかし、そのようなデータはもはやXDF形式と呼んでいいのか微妙で、おそらく多くのエミュレータもこれをフロッピーディスクイメージファイルとは認識しないでしょう。

## 対応フォーマット

### べたイメージ(XDF形式)

* べたデータ
* セクタのデータのみを順番に格納したフォーマットです。CHRNのC>H>Rの昇順で格納されています。ディスクのフォーマットの種類は、ファイルサイズから判断します。ファイルサイズが同じになる、「2D 320KB」と「1DD 320KB」など、複数の候補がある場合は、その旨が表示されます。2HDのDisk BASICのディスクの場合は、最初のトラックの128×26バイトの後に、以降のトラックの256×26バイトのデータが続くことになります。
* X68000では、拡張子`.xdf`が主に使われます。その他、T98では`.tfd`、VirtualPCでは`.vfd`、MSXでは`.dsk`、その他にも`.dup` `.flp` `.hdb` `.hdm` `.2hd` `.2dd` `.2d` `.1d`など、多くの拡張子が使われます。そんな状況なので、「○○形式」のような一般的な呼び方がなく、「べたイメージ」で一般に通用している感じでしょうか。本ツールでは便宜上、「XDF形式」と呼びます。
* 参考にした資料 : なし

### T98-Next NFD r0形式

* ヘッダ(68112バイト)+べたデータ
* T98-NextネイティブのFDイメージです。拡張子は`.nfd`です。r0形式とr1形式とが存在するようです(r1形式については次項)。インターリーブフォーマットやBIOSステータス等を表現できます。常に最大数の164×26セクタ分のヘッダ情報を持つので、ファイルが少しだけ大きくなる傾向があるようです。  
  ヘッダ部には、コメント・ライトプロテクト・ヘッド数の情報が格納されています。
* 参考にした資料 : [【1】](https://www.pc98.org/project/doc/nfdr0.txt)

### T98-Next NFD r1形式

* ヘッダ(全体(960バイト)+セクタ+特殊読み込み)+データ
* T98-NextネイティブのFDイメージです。r0形式とr1形式とがあるうちのr1のほうです。拡張子は`.nfd`で、r0形式と同じですので、r0かr1かはファイルの中身を見ないと判りません。インターリーブフォーマットやBIOSステータス等に加え、セクタのデータが読むたびに異なるようなディスクも表現できます。また特殊読み込みにも対応しており、Read Diagnostic等の特殊な読み取りの結果を格納することもできるようです。  
  ヘッダ部には、コメント・ライトプロテクト・ヘッド数の情報が格納されています。本ツールは追加情報ヘッダには未対応です。
* 参考にした資料 : [1](https://www.pc98.org/project/doc/nfdr1.txt)

### Anex86 FDI形式

* ヘッダ(通常4096バイト)+べたデータ
* Anex86ネイティブのFDイメージです。拡張子は`.fdi`です。ヘッダ部にCHRNの情報を持っているので、例えば2DDの、DOSとDisk BASICのフォーマットの違い(総サイズは同じでセクタ長・セクタ数が異なる)を表現することができます。ヘッダ部に持っているので、トラック毎に異なるセクタ長を使うことや、インターリーブフォーマットは表現できません。  
  ヘッダ部には、FDのメディアタイプと、CHRNの情報が格納されています。
* 参考にした資料 : [【1】](https://www.pc98.org/project/doc/hdi.html)

### D88形式

* ディスクヘッダ+セクタ毎にヘッダ+データ
* 多くのエミュレータで古くから使われているフォーマットです。拡張子は`.d88`のほか、`.88d`も使われているようです。また、どの機種用のイメージファイルなのかを識別する目的で、`.d98` `.d77` `.d68`等も使われています。セクタ毎にCHRNの情報を持っており、インターリーブフォーマットや、その他変態フォーマットを表現できます。  
  ヘッダ部には、FDのメディアタイプ・ライトプロテクト・コメントの情報が格納されています。  
  NP2kaiでは、1.44MBのメディアを表現するために独自の拡張を行っているようです。これを検出した際は、その旨を表示します。
* この形式は1つのファイルに複数のディスクイメージを格納できますが、本ツールでは対応していません。
* 参考にした資料 : [【1】](https://www.pc98.org/project/doc/d88.html) [【2】](https://github.com/waitingmoon/quasi88/blob/develop/document/FORMAT.TXT)

### X68000 DIFC.X DIM形式

* ヘッダ(256バイト)+べたデータ
* X68000でよく使われるフォーマットです。拡張子は`.dim`です。NP2kaiもこのフォーマットに対応していますが、2HD 1.23MBのイメージファイルを与えて何か書き込むと、なぜか170トラック1.36MBのファイルになってしまうのは、私の環境や使い方に誤りがあるのでしょうか…。  
  ディスクのフォーマットの種類は、ファイル冒頭のMedia byteで判別され、既定外のCHRNを持つようなディスクは表現できません。仕様上は2HD Disk BASICのイメージも扱えるようですが、本ツールでは対応していません。
* 参考にした資料 : [【1】](https://stdkmd.net/xeij/source/FDMedia.htm) [【2】](https://www.pc98.org/project/doc/dim.html)
* 【2】の冒頭には「DCU/DCP File Format」と書かれていますが誤りで、本文はDIM形式の説明のようです。
* 【2】の資料では、「Sector present bytesは160バイトで、その後に固定値0x00が10バイト続く」とされていますが、Sector present bytesが170バイト(オーバートラック対応)あると考えるべきと思われます。
* 【2】のMedia Byteについての説明で、0x00,0x01は共に2HS 9sec/trk 1440kと書かれていますが、正しくは0x00は2HD 8sec/trk 1.23MB(0x01が2HS 9sec/trk 1.44MB)と思われます。同じく、0x03として説明されている2HQ 18sec/track 1440kは正しくは0x09で、0x03は正しくは2HDE 9sec/trk 1.44MBだと思われます。

### DCP/DCU形式

* ヘッダ(162バイト)+べたデータ
* PC-98のディスクコピーユーティリティ、[DCP](https://www.vector.co.jp/soft/dos/util/se000478.html)と[DCU](https://www.vector.co.jp/soft/dos/util/se015189.html)でイメージファイルを扱う時に使われるフォーマットです。そもそも高速多機能なディスクコピーツールとしての用途がメインのツールですが、イメージファイルを扱うこともでき、その際の拡張子はそれぞれ`.dcp`と`.dcu`(ツール名と同じ)です。標準的なフォーマットのディスクを高速にコピーするためのツールなので、変態フォーマットには対応していません。トラック数も160が最大です。仕様上は2HD Disk BASICのイメージも扱えるようですが、本ツールでは対応していません。  
  公開時期からすると、DCUの方が後発のようです。DCUの作者がDCPの仕様を意識したのか判りませんが、イメージファイルの仕様は酷似しており、しかし微妙なところで異なります。イメージファイルの構成としては、162バイトの小さなヘッダに、フォーマットの種類と、トラック毎の保存のするしない(トラックマップ)が格納されており、その直後から(「保存する」の)トラックのデータが続きます。但し、トラックマップの中の最終の「保存する」だけは、これ以上トラックが無いことのマークであり、ファイルにはそのトラックのデータは保存されて**いません**。また、トラックマップに関係なく「全トラックを保存」することもでき、その場合はトラックマップ上「保存しない」のトラックであってもデータが保存されています。全トラックが保存されているかどうかはヘッダからは判別できず、ファイル長から判断することになります。なんでこんな仕様にしたんだろう…。
* 参考にした資料 : [【1】](https://www.pc98.org/project/doc/dcp.html) [【2】](https://melog.info/archives/2004/09/30/94#comment-27)
* 【1】の資料では「Disk Image Pro」や「Disk Copy Pro」というユーティリティ由来のフォーマットであるとの記述がありますが、そんなツール聞いたことないです。
* 【1】のDCP/DCU Headerのレイアウトには、誤りとおぼしき点があります。All cylinders stored flagを別に数えるとすると、0001hからのTrack mapは160バイトなので、0001h ~ 00A**0**hを占め、続くAll cylinders stored flagはオフセット00A**1**hに、Disk imageはオフセット00A**2**hからとなるのが正しいと思われます。
* 【1】で説明されている、All cylinders stored flagですが、説明が誤っているように思われます。そもそもそのようなフラグはなく、上で書いたような仕様ではないかと思われます。

### Virtual98 FDD形式

* 固定長ヘッダ(50172バイト)+可変長データ
* Virtual98で使われていたフォーマットです。拡張子は`.fdd`です。特殊読み込み情報を格納することができます。最大160×26個のセクタを格納でき、ヘッダは大きめになりますが、セクタ内の全バイトが同一値の場合はその旨を記録してデータそのものの格納はしないということができるなど、なかなか気の利いたフォーマットだと思います。  
  ヘッダ部には、コメント・ライトプロテクトの情報が格納されています。  
* 参考にした資料 : [【1】](https://www.pc98.org/project/doc/fdd.html)

### ERCACHE ERCVFD形式

* ヘッダ(64バイト)+セクタ毎に(ヘッダ(10バイト)+データ)
* ディスクキャッシュソフトERCACHEの仮想FDコントローラERCVFDで使われるフォーマットです。仕様について公開されているものが見当たらず、ソースしか頼るものがありません。本ツールでは、この形式であることの識別のみができます。
* 参考にした資料 : [【1】](https://www.vector.co.jp/soft/dos/game/se023028.html)

### DIP形式

* ヘッダ(256バイト)+べたデータ(1261568バイト)
* 仕様の資料があったので実装しましたが、何のツールのフォーマットなのか、どういうときに使われるのか不明です。このフォーマットであることを識別するマジックワードも、実は何か意味があるような数字だし…。  
  256バイトのヘッダ部は、マジックワードとコメントのみで、メディアやフォーマットの種類を示す情報はありません。従ってディスクのフォーマットも決め打ちで、2HD 1.23MB固定のようです。ということは日本発のフォーマットなのでしょうけど、これに関する情報が見つかりません。
* 参考にした資料 : [【1】](https://www.pc98.org/project/doc/dip.html)

### SL9821 SLF形式

* フォーマットヘッダ(16バイト)+トラックヘッダ(5120バイト)+データ
* PC9821のエミュレータSL9821およびラズパイFDCのpiFDCで扱われるフォーマットです。あまり普及しているものでは無いようですが、仕様が公開されていたので実装してみました。このフォーマットでは、セクタのデータのほか、GAPやCRC等の情報も持つことができ、トラックの生のデータに近い情報を表現できます。また、FM/MFMデコード前のビットデータを持つこともできるようですが、本ツールでは対応していません。
* 参考にした資料 : [【1】](https://www.satotomi.com/pifdc/pifdc_slf.html) [【2】](https://www.satotomi.com/sl9821/sl9821.html)

## 対応していないフォーマット

### [MAHALITO形式](https://www.vector.co.jp/soft/dos/util/se000604.html) (未対応)

### [HxC Floppy Emulator HFE形式](https://hxc2001.com/floppy_drive_emulator/HFE-file-format.html) (未対応)

### [SuperCard Pro Flux Disk Image SCP形式](https://www.cbmstuff.com/downloads/scp/scp_image_specs.txt) [【2】](https://github.com/keirf/disk-utilities/tree/master/scp) (未対応)

## チェックしている項目の一覧

執筆中

## サポート

GitHub: [o-p-a/fdimageid](https://github.com/o-p-a/fdimageid)

* 質問や感想や意見は[Discussions](https://github.com/o-p-a/fdimageid/discussions)へ、要望や不具合報告は[Issue](https://github.com/o-p-a/fdimageid/issues)へ。

## 今後の改善候補

* ファイルフォーマットの変換機能  
  べたイメージ(XDF)以外への変換

* ディスクデータの比較機能  
  ファイルフォーマットが異なるイメージ同士を比較する機能ってニーズありますかね?

* ブランクディスク作成機能  
  空のメディアを作成する機能がないエミュレータのために。

* HDDのイメージファイルへの対応  
  これについては、そもそも困っていないため、今のところ対応する必要性を感じていません。フォーマットは何種類かありますが、プロテクトをかけたり変態フォーマットをしたりされることはほぼ無いため、いくつかの既にある変換ツールで対応可能と考えています。

* オーバートラックを切り捨てるオプション  
  `--truncate-overtrack`みたいな。
