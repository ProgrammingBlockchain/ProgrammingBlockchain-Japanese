## トランザクション {#transaction}

> \([Mastering Bitcoin](https://github.com/bitcoinbook/bitcoinbook/)\) トランザクションはビットコインシステムの中で最も重要な部分だ。ビットコインの中のあらゆるものが、トランザクションを作り、ネットワーク上を伝播し、確認し、最後にトランザクションのグローバルな台帳、つまりビットコインブロックチェーンに加わることを確認するように設計されている。トランザクションはビットコインシステムへの参加者たちの間の価値の移動を暗号に書き直すデータ構成となっている。1つ1つのトランザクションはビットコインブロックチェーン、つまりグローバルな複式記帳型の台帳における、公開された記帳となっている。

1つのトランザクションは受け取り手がいないかもしれないし、複数いることもあるかもしれない。そして送り手にも同じことが言えるのだ！ブロックチェーン上では、前章で示したとおり、送り手と受け取り手がいつもScriptPubKeyによって抽象化されている。

もしビットコインコアを使っていたら、Transactionsタブをクリックすると、このようにトランザクションを参照できる。

![](../assets/BitcoinCoreTransaction.png)

ここで**Transaction ID**に着目してみたい。上図のケースだと以下がTransaction IDである。

`f13dc48fb035bbf0a6e989a26b3ecb57b84f85e0836e777d6edf60d87a4a2d94`

> 注釈：Tranaction IDは「SHA256\(SHA256\(txbytes\)\)」と定義できる。
>
> 注釈：承認されていないトランザクションを扱うためにTransaction IDを使ってはいけない。Transaction IDは承認される前は複製できてしまうからだ。これを「トランザクション展性」という。

Blockchain.infoのようなブロックエクスプローラーでトランザクションを閲覧することができる。 [https://blockchain.info/tx/f13dc48fb035bbf0a6e989a26b3ecb57b84f85e0836e777d6edf60d87a4a2d94](https://blockchain.info/tx/f13dc48fb035bbf0a6e989a26b3ecb57b84f85e0836e777d6edf60d87a4a2d94)  
しかし開発者としては、クエリを実行したりパースしたりすることがより簡単なサービスがほしいとおそらく思うだろう。

C\#の開発者、そしてNBitcoinのユーザーとしては、Nicolas Dorierの[QBit Ninja](http://docs.qbitninja.apiary.io/#)が最適な選択肢と思われる。ビットコインブロックチェーンに対してクエリを発行でき、またウォレットを追跡するためのオープンソースのWebAPIである。  
QBit NinjaはMicrosoft Azure Storageを基盤としている[NBitcoin.Indexer](https://github.com/MetacoSA/NBitcoin.Indexer)に依拠している。C\#開発者にはこのAPIのラッパーを開発するのではなく、[NuGet client package](http://www.nuget.org/packages/QBitninja.Client)を使うことが期待されている。

[http://api.qbit.ninja/transactions/f13dc48fb035bbf0a6e989a26b3ecb57b84f85e0836e777d6edf60d87a4a2d94](http://api.qbit.ninja/transactions/f13dc48fb035bbf0a6e989a26b3ecb57b84f85e0836e777d6edf60d87a4a2d94)にアクセスしてみると、トランザクションの内容を見ることができる。

![](../assets/RawTx.png)

次のコードを使って16進数からトランザクションをパースできる。

```cs
Transaction tx = new Transaction("0100000...");
Console.Writeline(tx);
```

タブの情報の多さに怖がる前にさっとタブを閉じてほしい。QBit NinjaがAPIに対してクエリを発行して情報をパースしてくれるから、**QBitNinja.Client** NuGet packageをインストールしてみよう。

![](../assets/QBitNuGet.png)

Transaction IDを使ってトランザクションのクエリを発行してみよう。

```cs
// Create a client
QBitNinjaClient client = new QBitNinjaClient(Network.Main);
// Parse transaction id to NBitcoin.uint256 so the client can eat it
var transactionId = uint256.Parse("f13dc48fb035bbf0a6e989a26b3ecb57b84f85e0836e777d6edf60d87a4a2d94");
// Query the transaction
GetTransactionResponse transactionResponse = client.GetTransaction(transactionId).Result;
```

> 日本語版注：ファイルの先頭にQBitNinja.ClientとQBitNinja.Client.Modelsを使う宣言する必要がある。
>
> ```
> using QBitNinja.Client;
> using QBitNinja.Client.Models;
> ```

**transactionResponse**の型は、**GetTransactionResponse**である。QBitNinja.Client.Modelsのネームスペースに定義されている。同じ結果を**NBitcoin.Transaction**型でも取り出せる。

```cs
NBitcoin.Transaction transaction = transactionResponse.Transaction;
```

両方のクラスを使ってTransaction IDを取り出す例を見てみよう。

```cs
Console.WriteLine(transactionResponse.TransactionId); // f13dc48fb035bbf0a6e989a26b3ecb57b84f85e0836e777d6edf60d87a4a2d94
Console.WriteLine(transaction.GetHash()); // f13dc48fb035bbf0a6e989a26b3ecb57b84f85e0836e777d6edf60d87a4a2d94
```

**GetTransactionResponse**はトランザクションの中で使われるインプットの値やScriptPubKeyのようなトランザクションについての追加的な情報を含んでいる。

追加的な情報の中で関連性があるのはインプットとアウトプットである。1つのScriptPubKeyに13.19683492BTCが送られていることがわかるだろう。

```cs
List<ICoin> receivedCoins = transactionResponse.ReceivedCoins;
foreach (var coin in receivedCoins)
{
    Money amount = (Money) coin.Amount;

    Console.WriteLine(amount.ToDecimal(MoneyUnit.BTC));
    var paymentScript = coin.TxOut.ScriptPubKey;
    Console.WriteLine(paymentScript);  // It's the ScriptPubKey
    var address = paymentScript.GetDestinationAddress(Network.Main);
    Console.WriteLine(address);
    Console.WriteLine();
}
```

> 日本語版注：ファイルの先頭にSystem.Collections.Genericを使う宣言をする必要がある。
>
> ```
> using System.Collections.Generic;
> ```

QBitNinjaのGetTransactionResponseクラスを使って、受け取ったBTCの情報を表示した。  
**Exercise : **QBitNinjaのGetTransactionResponseクラスを使って、支払ったBTCの情報を表示してみよう！

NBitcoinのTransactionクラスを使って受け取ったBTCの情報の情報をどのように表示するか見てみよう。

```cs
var outputs = transaction.Outputs;
foreach (TxOut output in outputs)
{
    Money amount = output.Value;

    Console.WriteLine(amount.ToDecimal(MoneyUnit.BTC));
    var paymentScript = output.ScriptPubKey;
    Console.WriteLine(paymentScript);  // It's the ScriptPubKey
    var address = paymentScript.GetDestinationAddress(Network.Main);
    Console.WriteLine(address);
    Console.WriteLine();
}
```

インプットとなっているトランザクションを見てみよう。インプットとなっているトランザクションを見てみると、前段のアウトプットが参照されているのに気づくだろう。各インプットトランザクションが、いま着目しているトランザクションにBTCをあてるためにどのアウトプットを使っているかを示している。

```cs
var inputs = transaction.Inputs;
foreach (TxIn input in inputs)
{
    OutPoint previousOutpoint = input.PrevOut;
    Console.WriteLine(previousOutpoint.Hash); // hash of prev tx
    Console.WriteLine(previousOutpoint.N); // idx of out from prev tx, that has been spent in the current tx
    Console.WriteLine();
}
```

**TxOut**、**Output**と**out**は類義語である。  
**OutPoint**と混同してはいけないが、この点については後ほど触れる。

要約すると、TxOutはBTCの総計と受け取りての**ScriptPubKey**を示す。

![](../assets/TxOut.png)  
As illustration let's create a txout with 21 bitcoin from the first ScriptPubKey in our current transaction:

```cs
Money twentyOneBtc = new Money(21, MoneyUnit.BTC);
var scriptPubKey = transaction.Outputs.First().ScriptPubKey;
TxOut txOut = new TxOut(twentyOneBtc, scriptPubKey);
```

Every **TxOut** is uniquely addressed at the blockchain level by the ID of the transaction which include it and its index inside it. We call such reference an **Outpoint**.

![](../assets/OutPoint.png)

For example, the **Outpoint** of the **TxOut** with 13.19683492 BTC in our transaction is \(4788c5ef8ffd0463422bcafdfab240f5bf0be690482ceccde79c51cfce209edd, 0\).

```cs
OutPoint firstOutPoint = spentCoins.First().Outpoint;
Console.WriteLine(firstOutPoint.Hash); // 4788c5ef8ffd0463422bcafdfab240f5bf0be690482ceccde79c51cfce209edd
Console.WriteLine(firstOutPoint.N); // 0
```

Now let’s take a closer look at the inputs \(aka **TxIn**\) of the transaction:

![](../assets/TxIn.png)

The **TxIn** is composed of the **Outpoint** of the **TxOut** being spent and of the **ScriptSig** \( we can see the ScriptSig as the “Proof of Ownership”\) In our transaction there are actually 9 inputs.

```cs
Console.WriteLine(transaction.Inputs.Count); // 9
```

With the previous outpoint's transaction ID we can review the information associated with that transaction.

```cs
OutPoint firstPreviousOutPoint = transaction.Inputs.First().PrevOut;
var firstPreviousTransaction = client.GetTransaction(firstPreviousOutPoint.Hash).Result.Transaction;
Console.WriteLine(firstPreviousTransaction.IsCoinBase); // False
```

We could continue to trace the transaction IDs back in this manner until we reach a **coinbase transaction**, the transaction including the newly mined coin by a miner.  
**Exercise:** Follow the first input of this transaction and its ancestors until you find a coinbase transaction!  
Hint: After a few minutes and 30-40 transaction, I gave up tracing back.  
Yes, you've guessed right, it is not the most efficient way to do this, but a good exercise.

In our example, the outputs were for a total of 13.19**70**3492 BTC.

```cs
Money spentAmount = Money.Zero;
foreach (var spentCoin in spentCoins)
{
    spentAmount = (Money)spentCoin.Amount.Add(spentAmount);
}
Console.WriteLine(spentAmount.ToDecimal(MoneyUnit.BTC)); // 13.19703492
```

In this transaction 13.19**68**3492 BTC were received.

**Exercise:** Get the total received amount, as I have been done with the spent amount.

That means 0.0002 BTC \(or 13.19**70**3492 - 13.19**68**3492\) is not accounted for! The difference between the inputs and outputs are called **Transaction Fees** or **Miner’s Fees**. This is the money that the miner collects for including a given transaction in a block.

```cs
var fee = transaction.GetFee(spentCoins.ToArray());
Console.WriteLine(fee);
```

You should note that a **coinbase transaction** is the only transaction whose value of output are superior to the value of input. This effectively correspond to coin creation. So by definition there is no fee in a coinbase transaction. The coinbase transaction is the first transaction of every block.  
The consensus rule enforce that the sum of output's value in the coinbase transaction does not exceed the sum of transaction fees in the block plus the mining reward.

