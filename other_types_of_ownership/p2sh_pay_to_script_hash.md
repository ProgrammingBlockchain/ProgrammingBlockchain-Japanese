## P2SH \(Pay To Script Hash\) {#p2sh-pay-to-script-hash}

前の章で見たとおり、マルチシグはコードとしても簡単に動かせるが、しかしP2SHを使わなければ、`Bitcoin Address`を顧客に渡すのと同じくらい簡単に、顧客にマルチシグの`scriptPubKey`に支払ってもらうようにお願いする方法はなかった。

**P2SH**または**Pay To Script Hash**は`scriptPubKey`を、それがどれだけ複雑だとしても、簡単な`BitcoinScriptAddress`に表現する方法である。

前の章ではこのマルチシグをこのように作っていた。

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

複雑だと思わないだろうか？

その代わりにその`scriptPubKey`が**P2SH**への支払いだとどのように見えるかを見てみよう。

```cs
Key bob = new Key();
Key alice = new Key();
Key satoshi = new Key();

var paymentScript = PayToMultiSigTemplate
    .Instance
    .GenerateScriptPubKey(2, new[] { bob.PubKey, alice.PubKey, satoshi.PubKey }).PaymentScript;

Console.WriteLine(paymentScript);
```

```
OP_HASH160 57b4162e00341af0ffc5d5fab468d738b3234190 OP_EQUAL
```

ちがいがわかるだろうか？このP2SHの`scriptPubKey`はマルチシグのスクリプトのハッシュを表している。これは`redeemScript.Hash.ScriptPubKey`という。

ハッシュなので、簡単にbase58エンコードの`BitcoinScriptAddress`に変換できる。

```cs
Key bob = new Key();
Key alice = new Key();
Key satoshi = new Key();

Script redeemScript =
    PayToMultiSigTemplate
    .Instance
    .GenerateScriptPubKey(2, new[] { bob.PubKey, alice.PubKey, satoshi.PubKey });
//Console.WriteLine(redeemScript.Hash.ScriptPubKey);
Console.WriteLine(redeemScript.Hash.GetAddress(Network.Main)); // 3E6RvwLNfkH6PyX3bqoVGKzrx2AqSJFhjo
```

このようなアドレスはどのクライアントのウォレットも認識する。そのウォレットが「マルチシグ」とはなにかを理解していないとしてもだ。

P2SHを使う支払いでは、ハッシュを得た`scriptPubKey`のことを**Redeem Script**と呼んでいる。

![](../assets/RedeemScript.png)

支払う人は**Redeem Scriptのハッシュ**についてだけ知っていれば良いから、**Redeem Script**を知ることはない。だから今のケースでは支払い者はボブ、サトシまたはアリスのマルチシグに対してビットコインを送金したことさえ知らないのだ。

このようなトランザクションに署名することは前の章（Bitcoin transferの章）で実践したものと似ている。1つだけちがいがあるのは、トランザクションを**TransactionBuilder**で生成するときは**Redeem Script**を生成しなければならないということだ。

マルチシグのP2SHが、`received`と名付けられたトランザクションの中にあるコインを受け取ったシーンをイメージしてみよう。

```cs
Script redeemScript =
    PayToMultiSigTemplate
    .Instance
    .GenerateScriptPubKey(2, new[] { bob.PubKey, alice.PubKey, satoshi.PubKey });
////Console.WriteLine(redeemScript.Hash.ScriptPubKey);
//Console.WriteLine(redeemScript.Hash.GetAddress(Network.Main));

Transaction received = new Transaction();
//Pay to the script hash
received.Outputs.Add(new TxOut(Money.Coins(1.0m), redeemScript.Hash));
```

> 注意：その支払いは`redeemScript`ではなく`redeemScript.Hash`に送っているから注意！

そして一度アリス、ボブまたはサトシが受け取ったビットコインを使いたいときは、`Coin`クラスのインスタンスを生成するのではなく、`ScriptCoin`クラスのインスタンスを生成する。

```cs
//Give the redeemScript to the coin for Transaction construction
//and signing
ScriptCoin coin = received.Outputs.AsCoins().First()
                                    .ToScriptCoin(redeemScript);
```

![](../assets/ScriptCoin.png)

トランザクション生成と署名に関する残りのコードは、マルチシグそのものを使ったトランザクションを説明した前章のコードとまったく同じである。

