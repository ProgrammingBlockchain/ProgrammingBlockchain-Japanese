## ビットコインを支払ってみよう {#spend-your-coin}

さあ今や**ビットコインアドレス**、**ScriptPubKey**、**秘密鍵**や**マイナー**とはなにかがわかったのだから、自分の手で自分が作り出す初の**トランザクション**を作れるだろう。

As you proceed through this lesson you will add code line by line as it is presented to build a method that will leave feedback for the book in a Twitter style message.

前のレッスンでやったように、使いたい**トランザクションアウトプット**を含む**トランザクション**を見てみよう。

新しい**コンソールプロジェクト**（.NET4.5以上）を作り、**QBitNinja.Client**のNuGetパッケージをインストールしてほしい。

もう秘密鍵を作成し、表示させられるだろうか？もう対応するビットコインアドレスを取得し、そこにビットコインを送金できるだろうか？もしできなくても心配しないでほしい。どうやってそれができるかすぐに何度も繰り返そう。

```cs
var network = Network.Main;

var privateKey = new Key();
var bitcoinPrivateKey = privateKey.GetWif(network);
var address = bitcoinPrivateKey.GetAddress();

Console.WriteLine(bitcoinPrivateKey);
Console.WriteLine(address);
```

**bitcoinPrivateKey**と**address**の値をメモし、そこに複数コインを送ってそのトランザクションIDをメモしよう。（トランザクションIDは[blockchain.info](https://blockchain.info/)のようなウォレットやブロックエクスプローラーで見つけられる）

秘密鍵をインポートする。

```cs
var bitcoinPrivateKey = new 
BitcoinSecret("cSZjE4aJNPpBtU6xvJ6J4iBzDgTmzTjbq8w2kqnYvAprBCyTsG4x");
var network = bitcoinPrivateKey.Network;
var address = bitcoinPrivateKey.GetAddress();

Console.WriteLine(bitcoinPrivateKey); // cSZjE4aJNPpBtU6xvJ6J4iBzDgTmzTjbq8w2kqnYvAprBCyTsG4x
Console.WriteLine(address); // mzK6Jy5mer3ABBxfHdcxXEChsn3mkv8qJv
```

そして最後にトランザクションの情報を取得する。

```cs
var client = new QBitNinjaClient(network);
var transactionId = uint256.Parse("e44587cf08b4f03b0e8b4ae7562217796ec47b8c91666681d71329b764add2e3");
var transactionResponse = client.GetTransaction(transactionId).Result;

Console.WriteLine(transactionResponse.TransactionId); // e44587cf08b4f03b0e8b4ae7562217796ec47b8c91666681d71329b764add2e3
Console.WriteLine(transactionResponse.Block.Confirmations);
```

さあ、トランザクションを生成する情報はすべて揃った。必要な質問はこれだ。「**どこから、どこへ、いくら？**」

### どこから？

このケースでは、2番目のアウトポイントを使いたいが、どのようにそれを把握したかを示す。

```cs
var receivedCoins = transactionResponse.ReceivedCoins;
OutPoint outPointToSpend = null;
foreach (var coin in receivedCoins)
{
    if (coin.TxOut.ScriptPubKey == bitcoinPrivateKey.ScriptPubKey)
    {
        outPointToSpend = coin.Outpoint;
    }
}
if(outPointToSpend == null)
    throw new Exception("TxOut doesn't contain our ScriptPubKey");
Console.WriteLine("We want to spend {0}. outpoint:", outPointToSpend.N + 1);
```

支払いのためにはトランザクションの中でこのoutpointを参照する必要がある。以下のようにトランザクションを生成する。

```cs
var transaction = new Transaction();
transaction.Inputs.Add(new TxIn()
{
    PrevOut = outPointToSpend
});
```

### どこへ？

必要な質問は覚えているだろうか？「**どこから、どこへ、いくら？**」だ。  
**トランザクションインプット**を組み立ててトランザクションに加えることが、「どこから」という質問への答えだった。  
**トランザクションアウトプット**を組み立ててトランザクションに加えることが、残っている質問の1つへの答えである。

この本への寄付アドレス：[1KF8kUVHK42XzgcmJF4Lxz4wcL5WDL97PB](https://blockchain.info/address/1KF8kUVHK42XzgcmJF4Lxz4wcL5WDL97PB)  
寄付されたお金は本の残りを書いている間、僕が食事し、人の言いなりになるのに使う「Coffee and Sushi Wallet」という僕のウォレットに入る。  
もしこのチャレンジを完遂させることに成功すると、[http://n.bitcoin.ninja/](http://n.bitcoin.ninja/)の**Hall of the Makers**の中に自分の寄付を見つけることができるだろう（寛大さの順に表示される）。

```cs
var hallOfTheMakersAddress = new BitcoinPubKeyAddress("1KF8kUVHK42XzgcmJF4Lxz4wcL5WDL97PB");
```

もし開発環境上でコードを動かしているなら、どの開発環境のアドレスにでも良いのでコインを送ってみよう。

```cs
var hallOfTheMakersAddress = BitcoinAddress.Create("mzp4No5cmCXjZUpf112B1XWsvWBfws5bbB");
```

### いくら？

もし**1BTC**を伴う**トランザクションインプット**から**0.5BTC**を使いたいとしても、確実にすべてを使い切らないといけないのだ！  
図解が以下に示すとおり、**トランザクションアウトプット**が**0.5**BTCをHall of The Makersに、そしてあなたに**0.4999**BTCを戻すように仕分けている。  
残りの**0.0001BTC**はどうなったのだろう？これはマイナーの手数料となっていて、彼らがこのトランザクションを次のブロックに含めるためのインセンティブとなっているのであった。

![](../assets/SpendTx.png)

```cs
TxOut hallOfTheMakersTxOut = new TxOut()
{
    Value = new Money((decimal)0.5, MoneyUnit.BTC),
    ScriptPubKey = hallOfTheMakersAddress.ScriptPubKey
};

TxOut changeBackTxOut = new TxOut()
{
    Value = new Money((decimal)0.4999, MoneyUnit.BTC),
    ScriptPubKey = bitcoinPrivateKey.ScriptPubKey
};

transaction.Outputs.Add(hallOfTheMakersTxOut);
transaction.Outputs.Add(changeBackTxOut);
```

We can do some finetuning here.  
You can check the address on a blockexplorer I am working with on this whole chapter example \(I am working on the testnet\):   [http://tbtc.blockr.io/address/info/mzK6Jy5mer3ABBxfHdcxXEChsn3mkv8qJv](http://tbtc.blockr.io/address/info/mzK6Jy5mer3ABBxfHdcxXEChsn3mkv8qJv)

```cs
// How much you want to TO
var hallOfTheMakersAmount = new Money(0.5m, MoneyUnit.BTC);
/* At the time of writing the mining fee is 0.05usd
 * Depending on the market price and
 * On the currently advised mining fee,
 * You may consider to increase or decrease it
*/
var minerFee = new Money(0.0001m, MoneyUnit.BTC);
// How much you want to spend FROM
var txInAmount = receivedCoins[(int) outPointToSpend.N].TxOut.Amount;
Money changeBackAmount = txInAmount - hallOfTheMakersAmount - minerFee;
```

Let's add our calculated values to our TxOuts:

```cs
TxOut hallOfTheMakersTxOut = new TxOut()
{
    Value = hallOfTheMakersAmount,
    ScriptPubKey = hallOfTheMakersAddress.ScriptPubKey
};

TxOut changeBackTxOut = new TxOut()
{
    Value = changeBackAmount,
    ScriptPubKey = bitcoinPrivateKey.ScriptPubKey
};
```

And add them to our transaction:

```cs
transaction.Outputs.Add(hallOfTheMakersTxOut);
transaction.Outputs.Add(changeBackTxOut);
```

### Message on The Blockchain

Now add your feedback! This must be less than 40 bytes, or it will crash the application.  
This feedback, along with your transaction will appear \(after transaction is confirmed\) in the [Hall of The Makers](http://n.bitcoin.ninja/).

```cs
var message = "nopara73 loves NBitcoin!";
var bytes = Encoding.UTF8.GetBytes(message);
transaction.Outputs.Add(new TxOut()
{
    Value = Money.Zero,
    ScriptPubKey = TxNullDataTemplate.Instance.GenerateScriptPubKey(bytes)
});
```

To sum up take a look at my whole transaction before signing:  
I have 3 **TxOut**, 2 with **value**, 1 without **value** \(with the message\). You can notice the differences between the **scriptPubKey**s of the "normal" **TxOut**s and the **scriptPubKey** of the **TxOut** with the message:

```json
{
  "hash": "b7803df4b90fd615532bcbdb3b63eb1af5a2e4ae36f29a6fbf9f57d0a1842e0a",
  "ver": 1,
  "vin_sz": 1,
  "vout_sz": 3,
  "lock_time": 0,
  "size": 154,
  "in": [
    {
      "prev_out": {
        "hash": "e44587cf08b4f03b0e8b4ae7562217796ec47b8c91666681d71329b764add2e3",
        "n": 1
      },
      "scriptSig": ""
    }
  ],
  "out": [
    {
      "value": "0.50000000",
      "scriptPubKey": "OP_DUP OP_HASH160 d3a689bc36464b9d74e1721fd321d4686eae594e OP_EQUALVERIFYOP_CHECKSIG"
    },
    {
      "value": "0.62840112",
      "scriptPubKey": "OP_DUP OP_HASH160 ce2c16edb74aef1caa6db0078af9d3a5b8fd12d1 OP_EQUALVERIFYOP_CHECKSIG"
    },
    {
      "value": "0.00000000",
      "scriptPubKey": "OP_RETURN 6e6f706172613733206c6f766573204e426974636f696e21"
    }
  ]
}
```

Take a closer look at **TxIn**. We have **prev\_out** and **scriptSig** there.  
**Exercise:** try to figure out what will be and how to get the **scriptSig** in our code before you read further!

Let's check out the **hash** of **prev\_out** in a blockexplorer: [http://tbtc.blockr.io/tx/info/e44587cf08b4f03b0e8b4ae7562217796ec47b8c91666681d71329b764add2e3](http://tbtc.blockr.io/tx/info/e44587cf08b4f03b0e8b4ae7562217796ec47b8c91666681d71329b764add2e3)  
In **prev\_out** **n** is 1. Since we are indexing from 0, this means I want to spend the second output of the transaction.  
In the blockexplorer we can see the corresponding address is `mzK6Jy5mer3ABBxfHdcxXEChsn3mkv8qJv` and I can get the scriptSig from the address like this:

```cs
var address = BitcoinAddress.Create("mzK6Jy5mer3ABBxfHdcxXEChsn3mkv8qJv");
transaction.Inputs[0].ScriptSig = address.ScriptPubKey;
```

### Sign your transaction

Now that we have created the transaction, we must sign it. In other words, you will have to prove that you own the TxOut that you referenced in the input.

Signing can be [complicated](https://en.bitcoin.it/w/images/en/7/70/Bitcoin_OpCheckSig_InDetail.png), but we’ll make it simple.

First let's revisit the **scriptSig** of **in**, how we can get it from code. Remember, we copypasted the address above from a blockexplorer, now let's get it from our QBitNinja transactionResponse:

```cs
transaction.Inputs[0].ScriptSig =  bitcoinPrivateKey.ScriptPubKey;
```

Then you need to give your private key for signing:

```cs
transaction.Sign(bitcoinPrivateKey, false);
```

### Propagate your transactions

Congratulations, you have signed your first transaction! Your transaction is ready to roll! All that is left is to propagate it to the network so the miners can see it.

#### With QBitNinja:

```cs
BroadcastResponse broadcastResponse = client.Broadcast(transaction).Result;

if (!broadcastResponse.Success)
{
    Console.WriteLine("ErrorCode: " + broadcastResponse.Error.ErrorCode);
    Console.WriteLine("Error message: " + broadcastResponse.Error.Reason);
}
else
{
    Console.WriteLine("Success! You can check out the hash of the transaciton in any block explorer:");
    Console.WriteLine(transaction.GetHash());
}
```

#### With your own Bitcoin Core:

```cs
using (var node = Node.ConnectToLocal(network)) //Connect to the node
{
    node.VersionHandshake(); //Say hello
                             //Advertize your transaction (send just the hash)
    node.SendMessage(new InvPayload(InventoryType.MSG_TX, transaction.GetHash()));
    //Send it
    node.SendMessage(new TxPayload(transaction));
    Thread.Sleep(500); //Wait a bit
}
```

The **using** code block will take care of closing the connection to the node. That's it!

You can also connect directly to the Bitcoin network, however I advise you to connect to your own trusted node \(faster and easier\)

## Need more practice?

Youtube: [How to make your first transaction with NBitcoin](https://www.youtube.com/watch?v=X4ZwRWIF49w)  
CodeProject: [Create a Bitcoin transaction by hand.](http://www.codeproject.com/Articles/1151054/Create-a-Bitcoin-transaction-by-hand)  
CodeProject: [DotNetWallet - Build your own Bitcoin wallet in C\#](https://www.codeproject.com/script/Articles/ArticleVersion.aspx?waid=214550&aid=1115639)

