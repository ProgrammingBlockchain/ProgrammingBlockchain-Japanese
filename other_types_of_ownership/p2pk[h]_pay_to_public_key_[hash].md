## P2PK\[H\] \(Pay to Public Key \[Hash\]\) {#p2pk-h-pay-to-public-key-hash}

### P2PKH - 要点の振り返り

**ビットコインアドレス**は**公開鍵のハッシュ**であることを学んだ。

```cs
var publicKeyHash = new Key().PubKey.Hash;
var bitcoinAddress = publicKeyHash.GetAddress(Network.Main);
Console.WriteLine(publicKeyHash); // 41e0d7ab8af1ba5452b824116a31357dc931cf28
Console.WriteLine(bitcoinAddress); // 171LGoEKyVzgQstGwnTHVh3TFTgo5PsqiY
```

また、ビットコインブロックチェーンに関する限り、**ビットコインアドレス**というようなものはないということも学んだ。ビットコインブロックチェーンは**ScriptPubKey**を用いて受け取り手を認識し、その**ScriptPubKey**はビットコインアドレスから生成できる。

```cs
var scriptPubKey = bitcoinAddress.ScriptPubKey;
Console.WriteLine(scriptPubKey); // OP_DUP OP_HASH160 41e0d7ab8af1ba5452b824116a31357dc931cf28 OP_EQUALVERIFY OP_CHECKSIG
```

逆に言えば以下のとおり。

```cs
var sameBitcoinAddress = scriptPubKey.GetDestinationAddress(Network.Main);
```

### P2PK\(Pay to Public Key\)

しかし、すべての**ScriptPubKey**がビットコインアドレスを表現しているわけではない。たとえばジェネシスと呼ばれているビットコインブロックチェーンの最初のトランザクションがそうだ。

```cs
Block genesisBlock = Network.Main.GetGenesis();
Transaction firstTransactionEver = genesisBlock.Transactions.First();
var firstOutputEver = firstTransactionEver.Outputs.First();
var firstScriptPubKeyEver = firstOutputEver.ScriptPubKey;
var firstBitcoinAddressEver = firstScriptPubKeyEver.GetDestinationAddress(Network.Main);
Console.WriteLine(firstBitcoinAddressEver == null); // True
```

```cs
Console.WriteLine(firstTransactionEver);
```

```json
{
…
  "out": [
    {
      "value": "50.00000000",
      "scriptPubKey": "04678afdb0fe5548271967f1a67130b7105cd6a828e03909a67962e0ea1f61deb649f6bc3f4cef38c4f35504e51ec112de5c384df7ba0b8d578a4c702b6bf11d5f OP_CHECKSIG"
    }
  ]
}
```

**scriptPubKey**の形式がちがうことがわかる。

```cs
Console.WriteLine(firstScriptPubKeyEver); // 04678afdb0fe5548271967f1a67130b7105cd6a828e03909a67962e0ea1f61deb649f6bc3f4cef38c4f35504e51ec112de5c384df7ba0b8d578a4c702b6bf11d5f OP_CHECKSIG
```

ビットコインアドレスは次のように表す：**OP\_DUP OP\_HASH160 &lt;hash&gt; OP\_EQUALVERIFY OP\_CHECKSIG**

しかし今表示しているものはこうだ：**&lt;pubkey&gt; OP\_CHECKSIG**

事実として最初は、**公開鍵**が直接**scriptPubKey**に使われていた。

```cs
var firstPubKeyEver = firstScriptPubKeyEver.GetDestinationPublicKeys().First();
Console.WriteLine(firstPubKeyEver); // 04678afdb0fe5548271967f1a67130b7105cd6a828e03909a67962e0ea1f61deb649f6bc3f4cef38c4f35504e51ec112de5c384df7ba0b8d578a4c702b6bf11d5f
```

今は主に公開鍵のハッシュを使っている。

![](../assets/PPKH.png)

```cs
key = new Key();
Console.WriteLine("Pay to public key : " + key.PubKey.ScriptPubKey);
Console.WriteLine();
Console.WriteLine("Pay to public key hash : " + key.PubKey.Hash.ScriptPubKey);
```

```
Pay to public key : 02fb8021bc7dedcc2f89a67e75cee81fedb8e41d6bfa6769362132544dfdf072d4 OP_CHECKSIG
Pay to public key hash : OP_DUP OP_HASH160 0ae54d4cec828b722d8727cb70f4a6b0a88207b2 OP_EQUALVERIFY OP_CHECKSIG
```

これら2つの支払い方法は**P2PK** \(pay to public key\)とか**P2PKH** \(pay to public key hash\)とかとして言われている。

サトシは後に、2つの理由でP2PKではなくP2PKHを使うことを決めた。

* 楕円曲線暗号（**公開鍵**や**秘密鍵**に使われれている暗号）が、楕円曲線上の離散対数問題を解くための改良されたショアのアルゴリズムによって解かれてしまうから。簡単に言い換えるとそれが意味するのは、理論上、量子コンピューターがそう遠くない未来に**公開鍵から秘密鍵を導出してしまう**ということだ。ビットコインを使うときだけ公開鍵を算出することによって、そういった攻撃を無力化することができる（一度使われたビットコインアドレスが二度と使われない前提で）。
* ハッシュがより小さくなるので（20バイトになる）、印刷するにも小さくできるしQRコードのような小さい記録媒体に埋め込むことが簡単になる。

最近ではP2PKを直接使う理由がないが、後に述べるP2SHと組み合わせてまだ使われている。

> （[議論](https://www.reddit.com/r/Bitcoin/comments/4isxjr/petition_to_protect_satoshis_coins/d30we6f/)）もしP2PKの使用についてより早く取り組まなければ、ビットコインの価値に深刻な影響をおよぼすだろう。

### Exercise

\([nopara73](https://github.com/nopara73)\) この章を読んでいる間、略語（P2PK、P2PKH、P2Wなど）がとてもややこしいことに気づいた。  
僕はそこで、レッスンを進める中でその略語に遭遇する都度、十分にその言葉を発音することを自分に強制した。そうすると突然すべてがわかるようになった。あなたにも同じことをするのをおすすめする。

