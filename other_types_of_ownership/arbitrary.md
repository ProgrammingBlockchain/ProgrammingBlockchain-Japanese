## Arbitrary {#arbitrary}

ビットコインコアのバージョン0.10から、**RedeemScript**は任意とすることができるようになった。それはビットコインのスクリプト言語を用いて、「所有権」が何を意味するかに関する自分の定義を作ることができるようになったということだ。

たとえば、僕が、UTF8にバイト化された僕の誕生日を知っている人、もしくは秘密鍵：**1KF8kUVHK42XzgcmJF4Lxz4wcL5WDL97PB**を知っている人になら誰でもビットコインをあげられるのだ。

スクリプト言語の詳細についてはスコープ外だが、いろいろなウェブサイト上で簡単にドキュメントを見つけられるし、スタックベースの言語なのでアセンブラを触っていた人ならだれでも読むことができるはずだ。

> **注釈**：\([nopara73](https://github.com/nopara73)\)僕は [Davide De Rosa's tutorial](http://davidederosa.com/basic-blockchain-programming/bitcoin-script-language-part-one/) が1番楽しいチュートリアルだと思う。

さあ最初は**RedeemScript**を作ってみよう。

> **注釈**：このコードが正しく動くようにするために、**プロジェクト** -&gt; **参照の編集** -&gt; **Sysmtem.Numerics** の順番にクリックして参照を設定しよう。

```cs
BitcoinAddress address = BitcoinAddress.Create("1KF8kUVHK42XzgcmJF4Lxz4wcL5WDL97PB");
var birth = Encoding.UTF8.GetBytes("18/07/1988");
var birthHash = Hashes.Hash256(birth);
Script redeemScript = new Script(
    "OP_IF "
        + "OP_HASH256 " + Op.GetPushOp(birthHash.ToBytes()) + " OP_EQUAL " +
    "OP_ELSE "
        + address.ScriptPubKey + " " +
    "OP_ENDIF");
```

This **RedeemScript** means that there is 2 way of spending such **ScriptCoin**: either you know the data that give **birthHash** \(my birthdate\), either you own the bitcoin address.

Let’s say I sent money to such **redeemScript**:

```cs
var tx = new Transaction();
tx.Outputs.Add(new TxOut(Money.Parse("0.0001"), redeemScript.Hash));
ScriptCoin scriptCoin = tx.Outputs.AsCoins().First().ToScriptCoin(redeemScript);
```

So let’s create a transaction that want to spend such output:

```cs
//Create spending transaction
Transaction spending = new Transaction();
spending.AddInput(new TxIn(new OutPoint(tx, 0)));
```

The first option is to know my birth date and to prove it in the **scriptSig**:

```cs
////Option 1 : Spender knows my birthdate
Op pushBirthdate = Op.GetPushOp(birth);
Op selectIf = OpcodeType.OP_1; //go to if
Op redeemBytes = Op.GetPushOp(redeemScript.ToBytes());
Script scriptSig = new Script(pushBirthdate, selectIf, redeemBytes);
spending.Inputs[0].ScriptSig = scriptSig;
```

You can see that in the **scriptSig** I push **OP\_1** so I enter in the **OP\_IF** of my **RedeemScript**.  
Since there is no backed-in template, for creating such **scriptSig**, you can see how to build a P2SH **scriptSig** by hand.

Then you can check that the **scriptSig** prove the ownership of the **scriptPubKey**:

```cs
//Verify the script pass
var result = spending
                .Inputs
                .AsIndexedInputs()
                .First()
                .VerifyScript(tx.Outputs[0].ScriptPubKey);
Console.WriteLine(result); // True
```

The second way of spending the coin is by proving ownership of **1KF8kUVHK42XzgcmJF4Lxz4wcL5WDL97PB**.

```
////Option 2 : Spender knows my private key
BitcoinSecret secret = new BitcoinSecret("...");
var sig = spending.SignInput(secret, scriptCoin);
var p2pkhProof = PayToPubkeyHashTemplate
    .Instance
    .GenerateScriptSig(sig, secret.PrivateKey.PubKey);
selectIf = OpcodeType.OP_0; //go to else
scriptSig = p2pkhProof + selectIf + redeemBytes;
spending.Inputs[0].ScriptSig = scriptSig;
```

And ownership is also proven:

```cs
//Verify the script pass
result = spending
                .Inputs
                .AsIndexedInputs()
                .First()
                .VerifyScript(tx.Outputs[0].ScriptPubKey);
Console.WriteLine(result); // True
///////////
```



