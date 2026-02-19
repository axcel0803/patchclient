# Patch Client — 使い方ガイド

Ragnarok Online プライベートサーバー向けの自動パッチクライアントです。

---

## フォルダ構成

```
パッチクライアント/
├── 蔵/                          ← プレイヤーに配布するファイル一式
│   ├── Patch Client.exe         ← 本体
│   ├── patch.conf               ← 設定ファイル（必須）
│   ├── jropatcher.exe           ← GRF更新ツール（任意）
│   └── patchclient/             ← 内部データ（触らなくてOK）
│       ├── assets/
│       │   └── default.htm      ← デフォルト表示HTML（info_url未設定時）
│       └── func/                ← DLL・Flutterランタイム
│
├── Webサーバー側/               ← サーバーにアップロードするファイル
│   ├── info.html                ← クライアント内WebViewに表示されるページ
│   ├── patch.txt                ← MD5ハッシュリスト（パッチ管理の要）
│   └── patch/                   ← パッチファイル（GRF等）置き場
│
└── md5ツール/
    └── getmd5.exe               ← patch.txt 作成用ツール
```

---

## セットアップ手順

### 1. patch.conf を編集する

`蔵/patch.conf` を開いて、最低限以下の項目を設定してください。

| キー | 説明 | 例 |
|------|------|----|
| `server` | サーバーのドメイン（http:// 不要） | `yourdomain.com` |
| `exename` | ゲーム開始ボタンで起動するEXE名 | `RagexeRE.exe` |
| `info_url` | WebViewに表示するHTMLのサーバー上パス | `/info.html` |
| `patch_txt` | patch.txt のサーバー上パス | `/patch.txt` |
| `patchdir` | パッチファイルが置いてあるサーバー上ディレクトリ | `/patch/` |

> **ポート指定が必要な場合:**  `server  yourdomain.com:8080`

---

### 2. Webサーバーにファイルをアップロードする

`Webサーバー側/` の中身をWebサーバーのルート（または任意のパス）にアップします。

```
サーバー上:
/
├── info.html       ← お知らせページ
├── patch.txt       ← MD5ハッシュリスト
└── patch/          ← パッチファイル置き場
```

---

### 3. patch.txt を作成する

`md5ツール/getmd5.exe` を使ってパッチファイルのMD5ハッシュを生成します。

**使い方:**
1. `getmd5.exe` を起動する
2. ハッシュ化したいファイル（GRF等）をウィンドウにドラッグ＆ドロップする
3. 出力されたテキストを `patch.txt` に貼り付ける

**patch.txt のフォーマット（1行1ファイル）:**
```
data/map001.grf  a1b2c3d4e5f6...
data/bgm.mp3     9f8e7d6c5b4a...
```

サブフォルダも対応しています。パスはサーバー上の `patchdir` からの相対パスで書きます。

---

### 4. プレイヤーに配布する

`蔵/` フォルダの中身をそのまま配布します。
プレイヤーは `Patch Client.exe` と同じ場所に `patch.conf` があれば起動できます。

> `patchclient/` フォルダも必ず一緒に配布してください。

---

## 任意設定

### GRF更新ボタンを表示する

`patch.conf` の `updater` キーを有効にすると、フッターに「GRF更新」ボタンが表示されます。

```ini
updater  jropatcher.exe
```

### パッチ必須モード

パッチが成功しないとゲーム起動できなくするには:

```ini
patch_required  1
```

### UIカラーのカスタマイズ

```ini
accent_color   FFD700   # ボタン・プログレスバーの色
bg_color       0D1117   # メイン背景色
panel_color    0F0F23   # フッターパネルの背景色
error_color    FF6B6B   # エラー時の色
```

### サーバー上の設定ファイルで上書き（リモート設定）

`server_ini` を使うと、配布後でも `exename` や `info_url` をサーバー側から変更できます。

```ini
server_ini  /patch_conf.txt
```

サーバー上の `/patch_conf.txt` に同じ書式で上書きしたいキーだけ書けばOKです。

### お知らせページ（info.html）をカスタマイズする

`Webサーバー側/info.html` を編集してサーバー名・レート・お知らせ内容を書き換えてください。

`info_url` を空にすると、`patchclient/assets/default.htm`（内蔵HTML）が代わりに表示されます。

---

## トラブルシューティング

| 症状 | 原因・対処 |
|------|-----------|
| 起動直後にクラッシュ | `patch.conf` が `Patch Client.exe` と同じフォルダにない |
| パッチが失敗する | `server` のドメインが間違っている、またはサーバーが応答していない |
| ゲーム開始ボタンが押せない | `patch_required 1` の状態でパッチが完了していない |
| WebViewが真っ白 | `info_url` のパスが間違っている（`/info.html` のようにスラッシュ始まりで書く） |
