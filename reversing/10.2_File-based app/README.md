# C# File-based app

- Platform: AlpacaHack
- Category: Rev
- Difficulty: Easy
- Topic: .NET
- Released: 2026-10-02

## 問題概要

.NET 10 の File-based apps 機能により、`.cs` ファイルをそのまま実行できる問題。
配布物は `chal.cs` と `run-using-docker.sh`。入力したフラグが正しいかを判定するプログラムになっている。

## 方針

Rev の基本は「プログラムが正解をどう作っているかを読む」こと。
まず `chal.cs` を読んで、入力チェックの流れを把握する。

```bash
cat chal.cs
```

## 解析

`IsCorrect()` は次の順に処理している。

1. 埋め込まれた Base64 文字列をデコード（`Base64.DecodeFromUtf8`）
2. 結果を Brotli で展開（`BrotliDecoder.Decompress`）
3. 展開したバイト列と入力を比較（`SequenceEqual`）

プログラムは「正解の文字列」を自分で計算して入力と見比べているだけ。
つまり 1 と 2 を自分で再現すれば、正解（フラグ）が分かる。XOR などの追加変換はない。

### ヒントとの対応

| ヒント | 内容 |
|---|---|
| Implicit using directives | `using System;` などが省略されているだけ。コード読解では脳内補完すればよい |
| Brotli | 手順 2 の圧縮形式。展開方法を調べるヒント |

## 解法

埋め込まれた文字列に対して、プログラムと同じ処理をコマンドで行う。

```bash
sudo apt install -y brotli
echo '<埋め込まれたBase64>' | base64 -d | brotli -dc
```

- `base64 -d` : 手順 1（Base64 デコード）
- `brotli -dc` : 手順 2（Brotli 展開）

出力された文字列がフラグ。

### Python の場合

```bash
python3 -c "import base64,brotli; print(brotli.decompress(base64.b64decode('<埋め込まれたBase64>')).decode())"
```

## 学んだこと

- Rev は実行せずコードを読むだけで解けることが多い
- 「Base64 → 圧縮」は CTF の定番パターン
- 変換が複雑（XOR や独自アルゴリズム）な場合は、逆順にたどる必要がある
- `apt` が 404 で失敗したら `sudo apt update` してから再インストール