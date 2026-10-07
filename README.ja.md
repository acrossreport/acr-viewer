# ACR Viewer

[English](README.md) | 日本語 | [Français](README.fr.md)

ACR Viewer は、システムトレイに常駐し、監視フォルダや HTTP で受け取った ACR(AcrossReport)の帳票データを、自動で印字・PDF/PNG 保存するアプリケーションです。

## 特長

- システムトレイに常駐(画面を開かずに動作)
- **監視フォルダ**:データファイルを置くだけで、自動で描画・印字・保存
- **HTTP API**:他のシステムから帳票データを送って印字・保存
- 出力:PDF / PNG / 両方、自動印字
- 設定はブラウザの設定画面で変更(日本語 / English / Français)
- 受信データ・出力ファイルの保持日数を指定して自動削除

## 動作環境

| OS | 状況 |
|---|---|
| Windows x64 | 対応 |
| Linux x64 | 対応 |
| macOS(Apple Silicon) | 対応予定 |
| macOS(Intel) | 対応予定 |

### Windows

- 対応する Windows のバージョン:Windows 11 以上
- 自動印字には、PDF を開けるアプリ(PDF の既定アプリ)が必要です。印刷先は OS の既定プリンタです

### Linux

- 対応:Ubuntu 24.04 以上(x64)
- 次のライブラリが必要です。入っていない場合は、次のコマンドでインストールしてください

```
sudo apt install libgtk-3-0 libxdo3 libayatana-appindicator3-1
```

- 印字は OS の印刷の仕組み(CUPS の `lp` コマンド)で行います。OS に既定のプリンタが設定されていない場合は、設定画面の「印刷先プリンタ名」にプリンタ名を入力してください。登録されているプリンタ名は `lpstat -p` で確認できます
- トレイのアイコンは、画面上部のバーに表示されます。Ubuntu 以外の GNOME では、AppIndicator の拡張機能が必要です。アイコンが表示されない場合も ACR Viewer は動いていて、ブラウザで `http://localhost:8765/` を開けば設定画面を使えます

## ダウンロード

[Releases](https://github.com/acrossreport/acr-viewer/releases) から、お使いの OS 用のファイルをダウンロードしてください。

- Windows x64:`acr_viewer-v0.1.0-win-x64.zip`
- Linux x64:`acr_viewer-v0.1.0-linux-x64.zip`

## インストールと起動

### Windows

1. ダウンロードした zip を任意のフォルダに展開します
2. `acr_viewer.exe` を実行します。システムトレイにアイコンが表示されます
3. トレイアイコンのメニューから「Web設定画面を開く」を選ぶと、ブラウザで設定画面が開きます(`http://localhost:8765/`)

### Linux

1. ダウンロードした zip を任意のフォルダに展開します
2. 端末でそのフォルダに移動し、`./acr_viewer` を実行します。画面上部のバーにアイコンが表示されます
3. アイコンをクリックし、メニューから「Web設定画面を開く」を選ぶと、ブラウザで設定画面が開きます(`http://localhost:8765/`)

### フォルダ

初回起動時に、次のフォルダが「ドキュメント」(Linux は `~/Documents`)の下に自動で作られます。設定画面で変更できます。

| フォルダ | 役割 |
|---|---|
| `AcrViewer/Watch` | 監視フォルダ(ここにファイルを置く) |
| `AcrViewer/Templates` | デザイン定義フォルダ |
| `AcrViewer/Output` | PDF / PNG の保存先 |
| `AcrViewer/Processed` | 処理済みファイルの移動先 |
| `AcrViewer/Error` | 処理できなかったデータの移動先 |

## トレイメニュー

| メニュー | 説明 |
|---|---|
| 自動印字 | 自動印字の有効・無効を切り替える |
| PDF保存 | PDF保存の有効・無効を切り替える |
| Web設定画面を開く（フォルダ保存先・プリンタ・ポート） | ブラウザで設定画面を開く |
| 終了 | AcrViewer を終了する |

表示言語は設定画面の言語（日本語 / English / Français）に合わせます。変更は AcrViewer の再起動後に反映されます。

## 使い方

### 方法1:データファイルで定義を指定する(おすすめ)

1. デザイン定義ファイル(例:`invoice.json`)を **デザイン定義フォルダ** に置いておきます
2. データファイルのヘッダ(`Parameters`)に、定義ファイル名を書きます

```json
{
  "Parameters": {
    "TemplateFile": "invoice.json",
    "...": "...",
    "Data": [ ... ]
  }
}
```

3. このデータを `〇〇.data.json` という名前で **監視フォルダ** に置くと、自動で描画・印字・保存されます
4. 成功したデータは処理済みフォルダへ、定義が見つからない・描画に失敗したデータはエラーフォルダへ移動します

`TemplateFile` は帳票の項目名には使わないでください(予約名です)。

### 方法2:定義とデータを組にして置く

同じ名前の組で監視フォルダに置きます。組がそろった時点で処理されます。

```
〇〇.template.json   … デザイン定義
〇〇.data.json       … データ
```

同じ名前の `.template.json` が監視フォルダにある場合は、方法2が優先されます。

### 出力ファイル名

```
yyyymmddhhmmss_定義名.pdf   … PDF
yyyymmddhhmmss_定義名.zip   … PNG(全ページを1つの ZIP にまとめたもの)
```

PNG は、ACR CLI と同じ ACR-PNG-PACKAGE 形式の ZIP ファイル1つで保存します。ZIP の中には `manifest.json` と、`pages/001.png`・`pages/002.png`… が入っています。

### HTTP API

- `POST /api/print`:`{ "template": {...}, "data": {...}, "design_name": "..." }` を送ると、描画・保存・印字します
  - `template` を省略した場合は、`data` の `Parameters.TemplateFile` でデザイン定義フォルダから定義を読みます

HTTP サーバは `0.0.0.0` で待ち受けます。社外のネットワークからアクセスできない環境でお使いください。

## トラブル調査(ログ)

ログは既定では出力しません。調査のためにログを出す場合は、ACR Viewer を終了し、次の設定ファイルの `"debug_log": false` を `"debug_log": true` に書き換えてから、もう一度起動してください。Linux では端末から起動すると、その端末にログが表示されます。調査が終わったら `false` に戻してください。

| OS | 設定ファイル |
|---|---|
| Windows | `%APPDATA%\AcrViewer\config.json` |
| Linux | `~/.config/AcrViewer/config.json` |

`"debug_log"` の項目がない場合は、追加してください。

## 出力について

ライセンス未登録の場合、出力(PDF・PNG・印字)にウォーターマークが入ります。設定画面の「ライセンス登録」で、メールアドレスとライセンスキーを登録すると、ウォーターマークなしで出力できます。詳しくは [公式サイト](https://acrossreport.com) をご覧ください。

## 関連リンク

- ACR Designer:https://github.com/acrossreport/acr-designer
- ACR Generator:https://github.com/acrossreport/acr-generator
- ACR 仕様(JSON テンプレート):https://github.com/acrossreport/acr-spec
- 公式サイト:https://acrossreport.com

## ライセンス

本ソフトウェアのソースコードは公開していません。利用条件は [LICENSE](LICENSE) をご確認ください。

## お問い合わせ

across.support@gmail.com

---

© Across Systems Corporation
ACR の中間描画命令アーキテクチャは特許出願中です。
