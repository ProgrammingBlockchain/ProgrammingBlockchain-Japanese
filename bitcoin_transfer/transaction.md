## Transaction {#transaction}

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

**transactionResponse**の型は**GetTransactionResponse**で、QBitNinja.Client.Modelsのネームスペースに定義されている。同じ結果を**NBitcoin.Transaction**型でも取り出せる。

```cs
NBitcoin.Transaction transaction = transactionResponse.Transaction;
```

両方のクラスを使ってTransaction IDを取り出す例を見てみよう。

```cs
Console.WriteLine(transactionResponse.TransactionId); // f13dc48fb035bbf0a6e989a26b3ecb57b84f85e0836e777d6edf60d87a4a2d94
Console.WriteLine(transaction.GetHash()); // f13dc48fb035bbf0a6e989a26b3ecb57b84f85e0836e777d6edf60d87a4a2d94
```

**GetTransactionResponse**はトランザクションの中で使われるインプットの値やScriptPubKeyのようなトランザクションについての追加的な情報を含んでいる。

本章で言及したいのはインプットとアウトプットである。1つのScriptPubKeyに13.19683492BTCが送られていることがわかるだろう。

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
**Exercise : **QBitNinjaのGetTransactionResponseクラスを使って、使われたBTCの情報を表示してみよう！

さて、NBitcoinのTransactionクラスを使って受け取ったBTCの情報の情報をどのように表示するか見てみよう。

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

次にインプットとなっているトランザクションを見てみよう。見てみると、前のトランザクションのアウトプットが参照されているのに気づくだろう。各インプットトランザクションが、いま着目しているトランザクションにBTCをあてるためにどのアウトプットを使っているかを示している。

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

**TxOut**、**Output**と**out**は同義語である。  
**OutPoint**と混同してはいけないが、この点については後ほど触れる。

要約すると、TxOutはBTCの総計と受け取り手の**ScriptPubKey**を示す。

![](../assets/TxOut.png)  
上図のとおり、いま題材としているトランザクションにある最初のScriptPubKeyから21BTCを支払うトランザクションアウトプットを作ってみよう。

```cs
Money twentyOneBtc = new Money(21, MoneyUnit.BTC);
var scriptPubKey = transaction.Outputs.First().ScriptPubKey;
TxOut txOut = new TxOut(twentyOneBtc, scriptPubKey);
```

> 日本語版注：ファイルの先頭にSystem.Linqを使う宣言をする必要がある。
>
> ```
> using System.Linq;
> ```

すべてのトランザクションアウトプットは、そのアウトプット自体とそのアウトプットがトランザクションの中で何番目かを示すインデックスを含む、トランザクションIDによってビットコインブロックチェーンの中で一意に識別されている。その一意に識別できる情報のことを**OutPoint**と呼ぶ。

![](../assets/OutPoint.png)

例えば、いま着目しているトランザクションのトランザクションアウトプットの中で、最初のOutpointは以下である。\(4788c5ef8ffd0463422bcafdfab240f5bf0be690482ceccde79c51cfce209edd, 0\).

```cs
OutPoint firstOutPoint = spentCoins.First().Outpoint;
Console.WriteLine(firstOutPoint.Hash); // 4788c5ef8ffd0463422bcafdfab240f5bf0be690482ceccde79c51cfce209edd
Console.WriteLine(firstOutPoint.N); // 0
```

> 日本語版注：QBitNinjaClientでOutPointを参照している。なお、spentCoinsはQBitNinjaのGetTransactionResponseクラスを使って、使われたBTCの情報を表示するときに設定する変数である。

さて、トランザクションのインプット（すなわち**TxIn**）に目を向けてみよう。

![](../assets/TxIn.png)

**TxIn**は支払いに使われた**TxOut**の**Outpoint**と**ScriptSig**（署名のことで所有権の証明とも言える）によって構成されている。今題材にしているトランザクションでは現に9つのインプットがある。

```cs
Console.WriteLine(transaction.Inputs.Count); // 9
```

前段のoutpointのトランザクションIDで、そのトランザクションに関連付けられた情報を参照できる。

```cs
OutPoint firstPreviousOutPoint = transaction.Inputs.First().PrevOut;
var firstPreviousTransaction = client.GetTransaction(firstPreviousOutPoint.Hash).Result.Transaction;
Console.WriteLine(firstPreviousTransaction.IsCoinBase); // False
```

> 日本語版注：firstPreviousOutPointは、NBitcoinでOutPointを参照している。2行目ではOutPointから得られたトランザクションハッシュから、QBitNinjaを使ってインプットとなったトランザクションを取得している。

やろうと思えばこの方法を使って、**コインベーストランザクション**、つまりマイナーによって新しく発掘されたコインを含むトランザクションにたどり着くまで、トランザクションIDをさかのぼり続けることができる。  
**Exercise**：題材にしているトランザクションの1番目のインプットをさかのぼり、コインベーストランザクションを見つけよう！  
ヒント：数分後または3、40分後に僕はさかのぼるのを諦めた。

そう、君の推測は正しい。これを行うのは最も効率的な方法ではないが、良い練習にはなる。

今の題材の例では、アウトプットの合計は13.19**70**3492 BTCであった。

```cs
Money spentAmount = Money.Zero;
foreach (var spentCoin in spentCoins)
{
    spentAmount = (Money)spentCoin.Amount.Add(spentAmount);
}
Console.WriteLine(spentAmount.ToDecimal(MoneyUnit.BTC)); // 13.19703492
```

このトランザクションでは、13.19**68**3492 BTCが受け取られている。

**Exercise:** Get the total received amount, as I have been done with the spent amount.

0.0002 BTC（言い換えれば13.19**70**3492 - 13.19**68**3492）が計上されていないということだ！インプットとアウトプットの差は**トランザクション手数料**あるいは**マイニング手数料**とか言われている。これはマイナーが、与えられたトランザクションをブロックに含めるためにかき集める手数料となっている。

```cs
var fee = transaction.GetFee(spentCoins.ToArray());
Console.WriteLine(fee);
```

注意してほしいのだが、**コインベーストランザクション**のみが、そのアウトプットの価値がインプットの価値より高い。これは事実上コインの創造に相当する。よって定義上、コインベーストランザクションに手数料は存在しない。コインベーストランザクションはすべてのブロックの最初のトランザクションとなっている。  
コンセンサスルールによって、コインベーストランザクションの中のアウトプットの価値の合計が、ブロック内のトランザクション手数料の合計とマイニング報酬の和を超えないこととなっている。

