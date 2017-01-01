## マルチシグ {#multi-sig}

ビットコインに対して所有権を共有することができる。  
そのためには`ScriptPubKey` を表す**m-of-n multi sig**を作るのだが、これが意味するのは、ビットコインを使うためには与えられた**n**個の異なる公開鍵に対して、**m**個の秘密鍵で署名する必要があるということだ。

ビットコインを使うためにはボブ、アリスそしてサトシのうち2人が署名する必要があるマルチシグを作ってみよう。

```cs
Key bob = new Key();
Key alice = new Key();
Key satoshi = new Key();

var scriptPubKey = PayToMultiSigTemplate
    .Instance
    .GenerateScriptPubKey(2, new[] { bob.PubKey, alice.PubKey, satoshi.PubKey });

Console.WriteLine(scriptPubKey);
```

```
2 0282213c7172e9dff8a852b436a957c1f55aa1a947f2571585870bfb12c0c15d61 036e9f73ca6929dec6926d8e319506cc4370914cd13d300e83fd9c3dfca3970efb 0324b9185ec3db2f209b620657ce0e9a792472d89911e0ac3fc1e5b5fc2ca7683d 3 OP_CHECKMULTISIG
```

見てのとおり、`scriptPubkey`が次のような形式となっている：

`<sigsRequired> <pubkeys…> <pubKeysCount> OP_CHECKMULTISIG`

署名のプロセスは`Transaction.Sign`と呼ばれるものと比べて少し複雑になっていて、それではマルチシグでは署名できない。

そのテーマについてはあとでより詳しく話すとして、マルチシグのトランザクションへの署名には`TransactionBuilder`を使うことにしよう。

マルチシグの`scriptPubKey`が`received`と呼ばれているトランザクション中のビットコインを受け取ったところを想像してほしい。

```cs
var received = new Transaction();
received.Outputs.Add(new TxOut(Money.Coins(1.0m), scriptPubKey));
```

ボブとアリスはニコに彼のサービスへの対価として1.0BTCを支払うことに同意した。  
だから彼らはトランザクションから、すでに受け取った`Coin`を取得する。

```cs
Coin coin = received.Outputs.AsCoins().First();
```

![](../assets/coin.png)

それから`TransactionBuilder`を使って**まだ署名されていないトランザクション**を生成する。

```cs
BitcoinAddress nico = new Key().PubKey.GetAddress(Network.Main);
TransactionBuilder builder = new TransactionBuilder();
Transaction unsigned = 
    builder
      .AddCoins(coin)
      .Send(nico, Money.Coins(1.0m))
      .BuildTransaction(sign: false);
```

そのトランザクションはまだ署名されていない。ここでどのようにアリスが署名するかを示す。

```cs
Transaction aliceSigned =
    builder
        .AddCoins(coin)
        .AddKeys(alice)
        .SignTransaction(unsigned);
```

![](../assets/aliceSigned.png)

それからボブの署名は以下のとおり。

```cs
Transaction bobSigned =
    builder
        .AddCoins(coin)
        .AddKeys(bob)
        .SignTransaction(aliceSigned);
```

![](../assets/bobSigned.png)

そして、ボブとアリスは1つのトランザクションに彼らの署名を結合する。

```cs
Transaction fullySigned =
    builder
        .AddCoins(coin)
        .CombineSignatures(aliceSigned, bobSigned);
```

![](../assets/fullySigned.png)

```cs
Console.WriteLine(fullySigned);
```

```json
{
  ...
  "in": [
    {
      "prev_out": {
        "hash": "9df1e011984305b78210229a86b6ade9546dc69c4d25a6bee472ee7d62ea3c16",
        "n": 0
      },
      "scriptSig": "0 3045022100a14d47c762fe7c04b4382f736c5de0b038b8de92649987bc59bca83ea307b1a202203e38dcc9b0b7f0556a5138fd316cd28639243f05f5ca1afc254b883482ddb91f01 3044022044c9f6818078887587cac126c3c2047b6e5425758e67df64e8d682dfbe373a2902204ae7fda6ada9b7a11c4e362a0389b1bf90abc1f3488fe21041a4f7f14f1d856201"
    }
  ],
  "out": [
    {
      "value": "1.00000000",
      "scriptPubKey": "OP_DUP OP_HASH160 d4a0f6c5b4bcbf2f5830eabed3daa7304fb794d6 OP_EQUALVERIFY OP_CHECKSIG"
    }
  ]
}
```

こうしてトランザクションはネットワークに送信できる準備ができた。

ビットコインネットワークがここに説明したとおりマルチシグをサポートしているとしても、1つ質問するに値することがある。「`scriptPubKey`が、以前の章で見てきたようにビットコインアドレスを使うのに簡単な方法で表現することができないのに、ビットコインに対して知見を持ち合わせていない人に対して、どのようにサトシ、アリスまたはボブのマルチシグに対して支払うようお願いできるのだろうか？」

`scriptPubKey`をビットコインアドレスと同じくらい簡単に、そしてコンパクトに表現できるとしたら、素晴らしいことだと思わないだろうか？

そう。これは可能で、**Bitcoin Script Address**、またの名をPay to Script Hash\(P2SH\)と呼ばれている。

最近では、ここで見た**Pay To Multi Sigそのもの**や**P2PKそのもの**は、説明したとおりに直接使われることは決してなく、**Pay To Script Hash**による支払いにラップされている。

