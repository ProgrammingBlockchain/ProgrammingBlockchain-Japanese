## Using the TransactionBuilder {#using-the-transactionbuilder}

初めての**P2SH**や**マルチシグ**のトランザクションに署名するときに**TranasctionBuilder**をどのように動かすか見てきた。

次はより複雑なトランザクションに対して署名するために、そのフルパワーをどのように活かすかを見てみよう。

**TranactionBuilder**を用いると以下ができるようになる。

* 以下の支払いができる
  * **P2PK**、**P2PKH**
  * **マルチシグ**
  * **P2WPK**、**P2WSH**
* 前章のようなredeem scriptにもとづいて**P2SH**を支払いに使える
* **ステルスコイン**を使える（ダークウォレット）
* **カラードコイン**を発行し、移動できる（次の章で説明するオープンアセットのこと） 
* **部分的に署名されたトランザクション**を結合できる 
* **署名されていないトランザクション**の最終的な**サイズ**とそれにかかる**手数料**を計算できる
* **トランザクション**が**十分に署名されたか**を確認できる 

**TransactionBuilder**のゴールはインプットとしての**コイン**と**鍵**を取得し、**十分に**または**部分的に署名されたトランザクション**として戻すことである。

![](../assets/SignedTransaction.png)

**TransactionBuilder**だけで、なんの**コイン**を使い、なにを署名すべきかを把握してくれる。

![](../assets/TransactionBuilder.png)

TransactionBuilderを使うためには4つのステップがある。

* 使う**コイン**を集める
* 持っている**鍵**を集める
* どの**scriptPubKey**に対していくら**コインを**支払いたいかを数える
* **トランザクション**を生成して署名する
* **オプション**：誰かに**トランザクション**を送り、署名してもらうか生成を続けてもらう

さあ**コイン**を集めてみよう。そのコインを充足させるために見せかけの**トランザクション**を作ろう。  
ではここでは、その**トランザクション**には**P2PKH**のコイン、**P2PK**のコインと、ボブとアリスの**マルチシグ**のコインがあることにしよう。

```cs
// Create a fake transaction
var bob = new Key();
var alice = new Key();

Script bobAlice = 
    PayToMultiSigTemplate.Instance.GenerateScriptPubKey(
        2, 
        bob.PubKey, alice.PubKey);

var init = new Transaction();
init.Outputs.Add(new TxOut(Money.Coins(1m), bob.PubKey)); // P2PK
init.Outputs.Add(new TxOut(Money.Coins(1m), alice.PubKey.Hash)); // P2PKH
init.Outputs.Add(new TxOut(Money.Coins(1m), bobAlice));
```

さらにここではサトシに支払うためにこのトランザクションの`coins`を使いたいということにしよう。

```cs
var satoshi = new Key();
```

まずは**コイン**を集めなければならない。

```cs
Coin[] coins = init.Outputs.AsCoins().ToArray();
Coin bobCoin = coins[0];
Coin aliceCoin = coins[1];
Coin bobAliceCoin = coins[2];
```

では`bob`は0.2BTC、`alice`は0.3BTCを送りたく、2人が0.5BTCを送るために`bobAlice`を使うことに合意したとしよう。

```cs
var builder = new TransactionBuilder();
Transaction tx = builder
        .AddCoins(bobCoin)
        .AddKeys(bob)
        .Send(satoshi, Money.Coins(0.2m))
        .SetChange(bob)
        .Then()
        .AddCoins(aliceCoin)
        .AddKeys(alice)
        .Send(satoshi, Money.Coins(0.3m))
        .SetChange(alice)
        .Then()
        .AddCoins(bobAliceCoin)
        .AddKeys(bob, alice)
        .Send(satoshi, Money.Coins(0.5m))
        .SetChange(bobAlice)
        .SendFees(Money.Coins(0.0001m))
        .BuildTransaction(sign: true);
```

Then you can verify it is fully signed and ready to send to the network.

```cs
Console.WriteLine(builder.Verify(tx)); // True
```

The nice thing about this model is that it works the same way for **P2SH, P2WSH, P2SH\(P2WSH\)**, and **P2SH\(P2PKH\)** except you need to create **ScriptCoin**.

![](../assets/ScriptCoinFromCoin.png)

```cs
init = new Transaction();
init.Outputs.Add(new TxOut(Money.Coins(1.0m), bobAlice.Hash));

coins = init.Outputs.AsCoins().ToArray();
ScriptCoin bobAliceScriptCoin = coins[0].ToScriptCoin(bobAlice);
```

Then the signature:

```cs
builder = new TransactionBuilder();
tx = builder
        .AddCoins(bobAliceScriptCoin)
        .AddKeys(bob, alice)
        .Send(satoshi, Money.Coins(0.9m))
        .SetChange(bobAlice.Hash)
        .SendFees(Money.Coins(0.0001m))
        .BuildTransaction(true);
Console.WriteLine(builder.Verify(tx)); // True
```

For **Stealth Coin**, this is basically the same thing. Except that, if you remember our introduction on Dark Wallet, I said that you need a **ScanKey** to see the **StealthCoin**.

![](../assets/StealthCoin.png)

Let’s create darkAliceBob stealth address as in previous chapter:

```cs
Key scanKey = new Key();
BitcoinStealthAddress darkAliceBob =
    new BitcoinStealthAddress
        (
            scanKey: scanKey.PubKey,
            pubKeys: new[] { alice.PubKey, bob.PubKey },
            signatureCount: 2,
            bitfield: null,
            network: Network.Main
        );
```

Let’s say someone sent this transaction:

```cs
//Someone sent to darkAliceBob
init = new Transaction();
darkAliceBob
    .SendTo(init, Money.Coins(1.0m));
```

The scanner will detect the StealthCoin:

```cs
//Get the stealth coin with the scanKey
StealthCoin stealthCoin
    = StealthCoin.Find(init, darkAliceBob, scanKey);
```

And forward it to bob and alice, who will sign:

```cs
//Spend it
tx = builder
        .AddCoins(stealthCoin)
        .AddKeys(bob, alice, scanKey)
        .Send(satoshi, Money.Coins(0.9m))
        .SetChange(bobAlice.Hash)
        .SendFees(Money.Coins(0.0001m))
        .BuildTransaction(true);
Console.WriteLine(builder.Verify(tx)); // True
```

> **Note:** You need the scanKey for spending a StealthCoin



