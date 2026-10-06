# Shared Prime

- Platform: AlpacaHack
- Category: Crypto
- Difficulty: Medium
- Topic: RSA
- Released: 2026-10-03

## 問題概要

2つのRSA公開鍵 `n1`, `n2` と、同じ平文（フラグ）を暗号化した `c1`, `c2` が与えられる問題。
配布物は `chall.py` と `output.txt`。`output.txt` に `n1`, `n2`, `c1`, `c2` が書かれている。

## 方針

Crypto の基本は「暗号の理論ではなく、実装や使い方のミスを探す」こと。
まず `chall.py` を読んで、鍵がどう作られているかを確認する。

```bash
cat chall.py
```

## 解析

`chall.py` の鍵生成部分は次のようになっている。

```python
e = 65537

p = getPrime(1024)
q1 = getPrime(1024)
q2 = getPrime(1024)

n1 = p * q1
n2 = p * q2
```

- 素数 `p` が `n1` と `n2` で共有されている（問題名の Shared Prime）
- 公開指数は `e = 65537`
- 同じ `m` を `n1`, `n2` の両方で暗号化している

通常、巨大な `n` の素因数分解は現実的な時間では不可能。
しかし `n1 = p*q1`, `n2 = p*q2` なら、最大公約数を取るだけで `p` が出てくる。

```
gcd(n1, n2) = gcd(p*q1, p*q2) = p
```

gcd はユークリッドの互除法で一瞬で計算できる。
`p` が分かれば `q1 = n1 // p` も分かり、秘密鍵 `d` が計算できる。

### 復号の流れ

1. `p = gcd(n1, n2)`
2. `q1 = n1 // p`
3. `d = e^-1 mod (p-1)(q1-1)`
4. `m = c1^d mod n1`
5. `m` をバイト列に変換するとフラグ

`c2` は同じ平文なので、`c1` だけ復号すれば十分。

## 解法

`output.txt` と同じディレクトリに `solve.py` を作って実行する。

```bash
cat > solve.py << 'EOP'
from math import gcd
import re

d = dict(re.findall(r'(\w+) = (\d+)', open('output.txt').read()))
n1, n2, c1, c2 = [int(d[k]) for k in ('n1', 'n2', 'c1', 'c2')]
e = 65537

p = gcd(n1, n2)
q1 = n1 // p
dd = pow(e, -1, (p - 1) * (q1 - 1))
m = pow(c1, dd, n1)
print(m.to_bytes((m.bit_length() + 7) // 8, 'big'))
EOP
python3 solve.py
```

- `gcd(n1, n2)` : 手順 1（共通素数の取得）
- `pow(e, -1, phi)` : 手順 3（秘密鍵 d の計算、Python 3.8 以降）
- `pow(c1, dd, n1)` : 手順 4（RSA 復号）

出力された文字列がフラグ。

### chall.py を動かしたい場合

`chall.py` は `Crypto` モジュールが無いと動かない。ローカルで試したいときだけ入れる。

```bash
sudo apt install -y python3-pycryptodome
```

解くだけなら `chall.py` の実行は不要。

## 学んだこと

- 問題名が攻撃手法のヒントになっていることがある
- 素数を使い回した RSA 鍵は、gcd だけで素因数分解できてしまう
- Crypto は「アルゴリズム自体」より「鍵の作り方・使い方」のミスを突くことが多い
- 配布された `chall.py` を読むと、`e` の値や素数の作り方など攻撃の前提が確認できる
- `.tar.gz` は `unzip` ではなく `tar -xzvf` で展開する