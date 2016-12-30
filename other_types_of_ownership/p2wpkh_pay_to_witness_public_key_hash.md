## P2WPKH \(Pay to Witness Public Key Hash\) {#p2wpkh-pay-to-witness-public-key-hash}

2015年、Peter Wuilleが**Segregated Witness**、略して**segwit**と呼ばれる新しい特徴をビットコインに提案した。基本的には、Segregated Witnessは所有権の証明を、トランザクションの**scriptSig**から、トランザクションインプットの**witness**と呼ばれる新しい部分に移すものだ。

この新しいスキームを使うことにはいくつかの理由があるが、ここには要点だけ述べるので詳細はこのリンクを見てほしい：[https://bitcoincore.org/en/2016/01/26/segwit-benefits/](https://bitcoincore.org/en/2016/01/26/segwit-benefits/)

* **サードパーティーによるトランザクション展性に関する課題解決**：以前はサードパーティーが、トランザクションがブロックに取り込まれる前にそのトランザクションIDを変えてしまうことが可能だったが、segwitによってそれは不可能となった。
* **一次署名（Linear Sig）ハッシュのスケール**：トランザクションに対して署名することは、すべてのインプットに使われるトランザクション全体をハッシュすることが必要だった。これは大きいサイズのトランザクションを使ったDDos攻撃の可能性を示唆する。
* **トランザクションインプットの価値に署名する**：インプットに使われる総コイン数も署名される。それは署名者が一度確実に支払われた手数料をごまかせないことを意味する。
* **キャパシティの向上**：10分ごとに1MB以上、1.75MB程度までのトランザクションを今や発生させることができる。
* **詐欺の証明**：後々開発される予定だが、SPVウォレットが最も長いチェーンを追うことよりも信頼できるコンセンサスルールを有効にできる。

以前はトランザクションの署名がトランザクションIDの計算対象に入っていたが、もう入らない。

![](../assets/segwit.png)

署名はP2PKHを使用するのと同じ情報を含むが、scriptSigではなくwitnessに含まれる。`scriptPubKey` は

```
OP_DUP OP_HASH160 0067c8970e65107ffbb436a49edd8cb8eb6b567f OP_EQUALVERIFY OP_CHECKSIG
```

ではなく以下のように作られるようになる。

```
0 0067c8970e65107ffbb436a49edd8cb8eb6b567f
```

アップグレードしていないノードにとっては、これはスタックに対しての2つのプッシュとしてしか見えない。これはどういうことかというと、どんな`scriptSig` でもそれらのビットコインを使えてしまうということだ。だから署名がなくてさえも古いノードはどのトランザクションを有効とみなしてしまう。新しいノードは最初のプッシュを**witness version**と解釈し、2番目のプッシュを**witness program**とみなす。

しかし新しいノードはトランザクションを有効とするために署名を必要とする。

**NBitcoinでは、P2WPKHのアウトプットを使用することは普通のP2PKHを使用する方法と異なる。  
公開鍵を得るための**`ScriptPubKey`** を取得するために、**`PubKey.Hash`**ではなく**`PubKey.WitHash`**を使う。**

```cs
var key = new Key();
Console.WriteLine(key.PubKey.WitHash.ScriptPubKey);
```

そうするとこのような出力がされるだろう。

```
0 0067c8970e65107ffbb436a49edd8cb8eb6b567f
```

このようなビットコインを使用するための署名は「Using the `TransactionBuilder` part」で説明するが、P2PKHのアウトプットに署名するコードと何も変わらない。

`witness`はP2PKHの`scriptSig` に似ていて、`scriptSig`は空になる。

```json
"in": [
{
  "prev_out": 
    {
      "hash": "725497eaef527567a0a18b310bbdd8300abe86f82153a39d2f87fef713dc8177",
      "n": 0
    },
  "scriptSig": "",
  "witness": "3044022079d443be2bd39327f92adf47a34e4b6ad7c82af182c71fe76ccd39743ced58cf0220149de3e8f11e47a989483f371d3799a710a7e862dd33c9bd842c417002a1c32901 0363f24cd2cb27bb35eb2292789ce4244d55ce580218fd81688197d4ec3b005a67"
}
```

もう一度言うが、P2WPKHはP2PKHの文法と同じだ。ただ、署名がP2PKHの場合とは異なるところに記録される。

