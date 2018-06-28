古い RICOH のレーザープリンタ NX760 を macOS High Sierra (10.13) から使用する方法
==============================================================================

## はじめに

ネットワークに接続された古い RICOH NX760 レーザープリンタを macOS  から使用する為には，プリンタ本体に Postscript オプションを装着する必要がある．
本文書では，カジュアルな印刷を目的として，高価な Postscript オプションを使用せずに以下のソフトウェアにより解決する方法の一例を示す．

- Foomatic-RIP v4.0.6
- Ghostscript v9.22
- Matt Broughton’s script Yosemite_foomatic_paths.command
- 修正改変した Ricoh-RPDL_IV_Laser_Printer-rpdl.ppd

### 仕組みの概要（著者の想像）

macOS 標準の 印刷ジョブ は PDF 形式であるが，Foomatic で横取りし PPD に記述された gs 呼び出しによりプリンタの解釈できる RPDL 形式に変換してから，プリンタへ送信するものと思われる．
また最近の macOS のセキュリティ強化に対応するために，スクリプトの実行と，  ``/etc/cups/cups-files.conf`` ファイルの修正が必要なようである．


## 注意喚起

- この文書を実施前に通読し，十分に内容を理解する必要がある．もし，理解出来ない点や不明な点が有る場合は，この文書に示す手順を実施してはならない．
- この文書は，実施例であり，実施内容や結果について読者の目的を達成することを保証するものではありません．
- この文書の内容は，macOS 等の設定変更やプリンタへのデータ送信を含んでおり，コンピュータやネットワークおよびプリンタ等を使用不可能とするリスクを内在している．
- この文書の内容に誤りが無いことを，著者は保証できない．
- この文書の手順を実施することや，いかなる実施結果になったとしても，それらに対する責任は全て実施者にあり，著者は責任を負わないものとする．
- 今回対象の macOS は macOS High Sierra (10.13) であるが，限定するものではない．他のバージョンに適用出来る場合もあるし，適用できない場合もある．


## 謝辞

この文書の内容は，以下の記事とほぼ同一である．これらの著者には非常に感謝する．
- [HPIJS on High Sierra (legacy printers)](https://discussions.apple.com/thread/8245788)
- [Getting OS X 10.11 El Capitan printing to a color Ricoh MP C2503](http://flying-geek.blogspot.com/2016/05/getting-os-x-1011-el-capitan-printing.html) 


## 手順

1. プリンタをネットワークに接続しプリンタの電源を入れる．
1. 必要なソフトウェアのダウンロードを行う．
    - Foomatic-RIP v4.0.6
      - [OpenPrinting : MacOSX : foomatic](https://wiki.linuxfoundation.org/openprinting/macosx/foomatic) から入手できる．
        - Foomatic-RIP for Mac OS X 10.3.x (Panther) \- Mac OS X 10.9.x (Mavericks) の所．
      - なお，当該ページの gplgs-8.71.dmg は，最近の macOS では *使用出来ない* ．
    - Ghostscript v9.22
      - [Richard Koch’s site at the University of Oregon](http://pages.uoregon.edu/koch/) から入手できる．
    - 改変前のPPDファイル ``Ricoh-RPDL_IV_Laser_Printer-rpdl.ppd`` を [Ricoh RPDL IV Laser Printer	
 ](http://www.openprinting.org/printer/Ricoh/Ricoh-RPDL_IV_Laser_Printer) からダウンロードする．
      - 以下のプリンタ追加手順で使用する．
    - ``yosemite_foomatic_paths.command`` を [Yosemite Postscript driver sandbox lifting script](https://gist.github.com/coffeesam/578d7b6beef0fbda975a) から ダウンロードする．

2. 以下のソフトウェアのインストールを行う．
    - Foomatic-RIP v4.0.6
    - Ghostscript v9.22

3. PPD ファイルを修正改変する．
    - まず，36行目付近の以下2行について，数値 ``100`` と ``0`` の修正と，2行目冒頭 ``*`` の次に ``%`` の追加をする．
      - 修正前
      ```
      *cupsFilter:	"application/vnd.cups-postscript 100 foomatic-rip"
      *cupsFilter:	"application/vnd.cups-pdf 0 foomatic-rip"
      ```

      - 修正後

      ```
      *cupsFilter:	"application/vnd.cups-postscript 0 foomatic-rip"
      *%cupsFilter:	"application/vnd.cups-pdf 100 foomatic-rip"
      ```

    - 次に，216行目付近のデフォルト解像度を ``600x600dpi`` に修正をする．
      - 修正前
      ```
      *DefaultResolution: 400x400dpi
      ```

      - 修正後

      ```
      *DefaultResolution: 600x600dpi
      ```

    - 修正改変した PPD ファイルに適当な名前を付けて保存する(例： ``Ricoh-NX760-RPDL_IV_Laser_Printer-rpdl.ppd`` )．

4. macOSの アップルメニュー >「システム環境設定」>「プリンタとスキャナ」からプリンタを追加する．なお，既に使用するプリンタが追加済の場合は，一度削除してから再度追加する．
    - プリンタリストの追加（+ボタン）をクリックし，「IP」をクリックして，以下の情報を入力する．入力後，「追加」ボタンを押下する．
      - アドレス：プリンタのIPアドレス(例：192.168.0.212) 
      - プロトコル：LPD (Line Printer Daemon)
      - キュー：空欄でOK
      - 名前：デフォルト(例：192.168.0.212)でOK
      - 場所：空欄でOK
      - ドライバ：「その他...」を選択し，前項の修正改変したPPDファイル(``Ricoh-NX760-RPDL_IV_Laser_Printer-rpdl.ppd``)を指定する．

5. macOSのターミナル(Terminal)を起動し，``yosemite_foomatic_paths.command`` を実行する．
    - 実行時に管理者パスワードを要求される場合がある．ターミナル内で ``Password:`` と表示される場合もある．
    - スクリプトが *対象となるプリンタ* を特定した際には yes と回答してください．
      - 無関係なプリンタの場合は， no と回答してください．
    
    ```
    The printer queue _192_168_0_212 should be modified.
    Do you want to continue? (y,n) y
    ```

6. "root" ユーザーとして ``/etc/cups/cups-files.conf`` ファイルに文字列 Sandboxing Relaxed を追加する．
   - ターミナル(Terminal)において以下を実行する．
    
    ```
    $ sudo -i
    # echo "Sandboxing Relaxed" >> /etc/cups/cups-files.conf
    ```

7. 引き続きターミナル(Terminal)において，CUPSを再起動し，ターミナルを閉じる．

    ```
    # launchctl stop org.cups.cupsd
    # launchctl start org.cups.cupsd
    # exit
    ```

## おわりに

印刷時に「用紙サイズに合わせる」などの設定をしないと，はみ出る可能性があるため要注意である．
