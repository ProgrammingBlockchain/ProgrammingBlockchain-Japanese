## P2WPKH \(Pay to Witness Public Key Hash\) {#p2wpkh-pay-to-witness-public-key-hash}

2015年、Peter Wuilleが**Segregated Witness**、略して**segwit**と呼ばれる新しい機能をビットコインに提案した。基本的には、Segregated Witnessは所有権の証明を、トランザクションの**scriptSig**から、トランザクションインプットの**witness**と呼ばれる新しい部分に移すものだ。

この新しいスキームを使うことにはいくつかの理由があるが、ここには要点だけ述べるので詳細はこのリンクを見てほしい：[https://bitcoincore.org/en/2016/01/26/segwit-benefits/](https://bitcoincore.org/en/2016/01/26/segwit-benefits/)

* **第三者によるトランザクション展性に関する課題解決**：以前は第三者が、トランザクションが承認される前にそのトランザクションIDを変えてしまうことが可能だったが、それが不可能になる。
* **署名ハッシュの線形スケール**：トランザクションに署名とは、トランザクション全体を対象にしたハッシュ値計算を各インプットそれぞれに行うことである。これは大きいサイズのトランザクションを使うことで、DDOSベクター攻撃の危険性を持つが、それを防ぐことができる。
* **トランザクションインプットの金額に署名する**：インプットの消費金額も署名の対象になる。それは署名者が支払う手数料を、間違えないことを意味する。
* **キャパシティの向上**：10分ごとにトータル1MB以上（1.75MB程度まで）のトランザクション量に対応することができる。
* **詐欺への防御**：後々開発される予定だが、SPVウォレットでは、今の最も長いチェーンが正当とするルールだけでなくさらに、コンセンサスルールを追加することができる。

以前はトランザクションの署名がトランザクションIDの計算対象に入っていたが、segwitではもうそれは入らない。

![](../assets/segwit.png)

署名はP2PKHのと同じ情報をもつが、scriptSigにではなく、witnessエリアに配置される。しかし、`scriptPubKey` は

```
OP_DUP OP_HASH160 0067c8970e65107ffbb436a49edd8cb8eb6b567f OP_EQUALVERIFY OP_CHECKSIG
```

ではなく、以下のようになる。

```
0 0067c8970e65107ffbb436a49edd8cb8eb6b567f
```

アップグレードしていないノードにとっては、これはスタックに対しての2つのプッシュとしてしか見えない。これはどういうことかというと、どんな`scriptSig` でもそれらのビットコインを使えてしまうということだ。だから署名がなくても古いノードはどのトランザクションを有効とみなしてしまう。新しいノードは最初のプッシュを**witness バージョン**と解釈し、2番目のプッシュを**witness プログラム**とみなす。

しかし新しいノードはトランザクションを検証するために署名を必要とする。

**NBitcoinでは、P2WPKHのアウトプットを消費することは普通のP2PKHを使用する方法と同じである。  
公開鍵から P2WHPKHで使う **`ScriptPubKey`** を取得するために、**`PubKey.Hash`** でなく**`PubKey.WitHash`**を使う。**

```cs
var key = new Key();
Console.WriteLine(key.PubKey.WitHash.ScriptPubKey);
```

そうするとこのような出力がされるだろう。

```
0 0067c8970e65107ffbb436a49edd8cb8eb6b567f
```

このようなビットコインを使用するための署名は「Using the `TransactionBuilder` part」で説明するが、P2PKHのアウトプットに署名するコードと何も変わらない。

`witness`はP2PKHの`scriptSig`と同様で、`scriptSig`は空になる。

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

もう一度言うが、P2WPKHはP2PKHの文法と同じだ。ただ、署名がP2PKHの場合とは異なるところにセットされる。
