# トランザクション {#transaction}

> \([Mastering Bitcoin](https://github.com/bitcoinbook/bitcoinbook/)\) トランザクションは、ビットコインシステムの中で最も重要な部分です。システムの他の要素はすべて、トランザクションが作成され、ビットコインネットワークを伝搬し、検証され、最後にグローバルなトランザクション元帳（ブロックチェーン）に追加されるという、一連の流れを支えるように作られています。トランザクションは、ビットコインシステムの参加者間の価値の移転をエンコードしたデータ構造です。個々のトランザクションは、複式簿記の元帳であるブロックチェーンに記された、誰でも見ることができる取引記録です。

トランザクションには受け取り手がいないかもしれないし、複数いることもあるかもしれない。**それは送り手にも同じことが言える！** ビットコインブロックチェーン上では、前章で示したとおり、送り手と受け取り手は、常にScriptPubKeyによって抽象化されている。

もしビットコインコアを使っていたら、Transactionsタブをクリックすると、このようにトランザクションを参照できる。

![](../assets/BitcoinCoreTransaction.png)

ここで**Transaction ID**に着目してみたい。上図のケースだと以下がTransaction IDである。

`f13dc48fb035bbf0a6e989a26b3ecb57b84f85e0836e777d6edf60d87a4a2d94`

> 注釈：Tranaction IDは「SHA256\(SHA256\(txbytes\)\)」と定義できる。
>
> 注釈：承認されていないトランザクションを扱うときにTransaction IDを使ってはいけない。Transaction IDは承認される前は改ざんできてしまうからだ。これを「トランザクション展性」という。

Blockchain.infoのようなブロックエクスプローラーでトランザクションを閲覧することができる。 [https://blockchain.info/tx/f13dc48fb035bbf0a6e989a26b3ecb57b84f85e0836e777d6edf60d87a4a2d94](https://blockchain.info/tx/f13dc48fb035bbf0a6e989a26b3ecb57b84f85e0836e777d6edf60d87a4a2d94)
しかし開発者としてはおそらく、より簡単にクエリを実行したりパースしたりすることができるサービスがほしいと思うだろう。
C\#の開発者、そしてNBitcoinのユーザーとしては、Nicolas Dorierの[QBit Ninja](http://docs.qbitninja.apiary.io/#)が最適な選択肢だと思う。ブロックチェーンに対してクエリを発行でき、またウォレットを追跡するためのオープンソースのWebサービスAPIである。
QBit NinjaはMicrosoft Azure Storageを基盤としている[NBitcoin.Indexer](https://github.com/MetacoSA/NBitcoin.Indexer)に依拠している。C\#開発者にはこのAPIのラッパーは開発せず、クライアントライブラリーである[NuGet client package](http://www.nuget.org/packages/QBitninja.Client)を使うことをお勧めする。

[http://api.qbit.ninja/transactions/f13dc48fb035bbf0a6e989a26b3ecb57b84f85e0836e777d6edf60d87a4a2d94](http://api.qbit.ninja/transactions/f13dc48fb035bbf0a6e989a26b3ecb57b84f85e0836e777d6edf60d87a4a2d94)にアクセスしてみると、トランザクションの内容を見ることができる。

![](../assets/RawTx.png)

次のコードを使って16進数表現のトランザクションをパースできる。

```cs
Transaction tx = Transaction.Parse("0100000...", Network.Main);
Console.Writeline(tx);
```

出力された情報の多さで怖くなる前に、出力タブを閉じてくれ。QBit NinjaクライアントがAPIに対して問い合わせをして情報をパースしてくれる。**QBitNinja.Client** の NuGet packageをインストールしてみよう。

![](../assets/QBitNuGet.png)

まずインポートする。

'''cs
using QBitNinja.Client;
using QBitNinja.Client.Models;
'''

Transaction IDを使ってトランザクションのクエリを発行してみよう。

```cs
// QBitNinjaクライアントオブジェクトを作成
QBitNinjaClient client = new QBitNinjaClient(Network.Main);
// 文字型のトランザクションIdをパースしてuint256型に変換。これで、クライアントオブジェクトに渡せる。
var transactionId = uint256.Parse("f13dc48fb035bbf0a6e989a26b3ecb57b84f85e0836e777d6edf60d87a4a2d94");
// トランザクションをQBitNinjaサーバー側に問い合わせる
GetTransactionResponse transactionResponse = client.GetTransaction(transactionId).Result;
```

変数 **transactionResponse** の型は**GetTransactionResponse**で、QBitNinja.Client.Modelsのネームスペースに定義されている。この変数から **NBitcoin.Transaction** 型のオブジェクトを取り出せる。

```cs
NBitcoin.Transaction transaction = transactionResponse.Transaction;
```

これら２つのクラスから Transaction IDを取り出す例を見てみよう。

```cs
Console.WriteLine(transactionResponse.TransactionId); // f13dc48fb035bbf0a6e989a26b3ecb57b84f85e0836e777d6edf60d87a4a2d94
Console.WriteLine(transaction.GetHash()); // f13dc48fb035bbf0a6e989a26b3ecb57b84f85e0836e777d6edf60d87a4a2d94
```

**GetTransactionResponse** は、トランザクションデータに含まれない追加的な情報として、トランザクションの中で消費されようとしているインプットの値や、インプットのScriptPubKeyを含んでいる。

ここで言及したい重要な部分は **インプット** と **アウトプット** である。1つのScriptPubKeyに13.19683492BTCが送られていることがわかるだろう。

```cs
List<ICoin> receivedCoins = transactionResponse.ReceivedCoins;
foreach (var coin in receivedCoins)
{
    Money amount = (Money) coin.Amount;

    Console.WriteLine(amount.ToDecimal(MoneyUnit.BTC));
    var paymentScript = coin.TxOut.ScriptPubKey;
    Console.WriteLine(paymentScript);  // これはScriptPubKey
    var address = paymentScript.GetDestinationAddress(Network.Main);
    Console.WriteLine(address); // 1HfbwN6Lvma9eDsv7mdwp529tgiyfNr7jc
    Console.WriteLine();
}
```

QBitNinjaのGetTransactionResponseクラスを使って、「受け取った」BTCの情報を表示した。
**Exercise :** QBitNinjaのGetTransactionResponseクラスを使って、「使われた」BTCの情報を表示してみよう！

さて今度は、NBitcoinのTransactionクラスを使って、QBitNinjaで表示した「受け取った」BTCの情報と同じものを、どのように表示するか見てみよう。

```cs
var outputs = transaction.Outputs;
foreach (TxOut output in outputs)
{
    Money amount = output.Value;

    Console.WriteLine(amount.ToDecimal(MoneyUnit.BTC));
    var paymentScript = output.ScriptPubKey;
    Console.WriteLine(paymentScript);  // これは ScriptPubKey
    var address = paymentScript.GetDestinationAddress(Network.Main);
    Console.WriteLine(address);
    Console.WriteLine();
}
```

では、インプットとなっているトランザクションを見てみよう。見てみると、１つ前のトランザクションのアウトプットが参照されているのに気づくだろう。各インプットトランザクションが、いま着目しているトランザクションにBTCを充てるためにどのアウトプットを使っているかを示している。

```cs
var inputs = transaction.Inputs;
foreach (TxIn input in inputs)
{
    OutPoint previousOutpoint = input.PrevOut;
    Console.WriteLine(previousOutpoint.Hash); // １つ前のトランザクションのハッシュ値＝トランザクションID
    Console.WriteLine(previousOutpoint.N); // １つ前のトランザクションのアウトプットのインデックス値。これは、今回のトランザクションで消費される。
    Console.WriteLine();
}
```

**TxOut**、**Output**と**out**は同義語である。
**OutPoint** と混同してはいけないが、この点については後ほど触れる。

要約すると、あるTxOutとはビットコインの額とその受け取り手の**ScriptPubKey**の組み合わせを示す。

![](../assets/TxOut.png)
上図のとおり、いま対象のトランザクションにある一番最初のScriptPubKeyから21BTCを支払うトランザクションアウトプットを作ってみよう。

```cs
Money twentyOneBtc = new Money(21, MoneyUnit.BTC);
var scriptPubKey = transaction.Outputs[0].ScriptPubKey;
TxOut txOut = transaction.Outputs.CreateNewTxOut(twentyOneBtc, scriptPubKey);
```

すべての**トランザクションアウトプット**は、それを包含するトランザクションのIDと、そしてトランザクションの中で何番目のアウトプットを用いるかを示すインデックスにより、ブロックチェーン全体の中で一意に識別される。その一意に識別できる情報（つまりトランザクションとトランザクションインデックス）のことを**OutPoint**と呼ぶ。

![](../assets/OutPoint.png)

例えば、いま対象にしているトランザクションの13.19683492 BTCのアウトプットのOutpointは以下である。\(f13dc48fb035bbf0a6e989a26b3ecb57b84f85e0836e777d6edf60d87a4a2d94, 0\).

```cs
OutPoint firstOutPoint = receivedCoins[0].Outpoint;
Console.WriteLine(firstOutPoint.Hash); // f13dc48fb035bbf0a6e989a26b3ecb57b84f85e0836e777d6edf60d87a4a2d94
Console.WriteLine(firstOutPoint.N); // 0
```

さて、インプット（すなわちトランザクションインプット）に目を向けてみよう。

![](../assets/TxIn.png)

**トランザクションインプット**は支払いに使おうとしているトランザクションアウトプットの**Outpoint**と**ScriptSig**（ScriptSigは所有権の証明とも言える）によって構成されている。今題材にしているトランザクションでは現に9つのインプットがある。

```cs
Console.WriteLine(transaction.Inputs.Count); // 9
```

過去のoutpointのトランザクションIDを使って、そのトランザクションに関連付けられた情報を参照できる。

```cs
OutPoint firstPreviousOutPoint = transaction.Inputs[0].PrevOut;
var firstPreviousTransaction = client.GetTransaction(firstPreviousOutPoint.Hash).Result.Transaction;
Console.WriteLine(firstPreviousTransaction.IsCoinBase); // コインベーストランザクションではない
```

> 日本語版注：firstPreviousOutPointは、NBitcoinで１つめのインプットの元となった過去のトランザクションのアウトポイントを参照している。2行目ではアウトポイントから得られたトランザクションハッシュ/トランザクションIDを、QBit Ninjaクライアントに渡してトランザクションの情報を取得している。

やろうと思えばこの方法を使って、**コインベーストランザクション**、つまりマイナーによって新しく発掘されたコインのトランザクション（すなわち、それ以前のトランザクションは存在しない）にたどり着くまで、トランザクションIDをさかのぼり続けることができる。
**Exercise**：題材にしているトランザクションの1番目のインプットを過去にさかのぼり、コインベーストランザクションを見つけよう！
ヒント：私は数分後、30~40トランザクションさかのぼったときに諦めた。

そう、君の推測は正しい。これを行うのは効率的な方法ではないが、良い練習になる。

今の題材の例では、過去のアウトプットの合計は13.19**70**3492 BTCであった。

```cs
Money spentAmount = Money.Zero;
var spentCoins = transactionResponse.SpentCoins;

foreach (var spentCoin in spentCoins)
{
    spentAmount = (Money)spentCoin.Amount.Add(spentAmount);
}
Console.WriteLine(spentAmount.ToDecimal(MoneyUnit.BTC)); // 13.19703492
```

このトランザクションでは、13.19**68**3492 BTCが受け取られている。

**Exercise：** 使用したコインを取得したのと同じように、受け取ったコインの総量を取得してみよう。

0.0002 BTC（言い換えれば13.19**70**3492 - 13.19**68**3492）が計上されていないということだ！インプットとアウトプットの差は**トランザクション手数料**あるいは**マイニング手数料**と言われている。これはマイナーに、与えられたトランザクションをブロックに含めてもらうために渡す手数料である。

```cs
var fee = transaction.GetFee(spentCoins.ToArray());
Console.WriteLine(fee);
```

注意してほしいのだが、**コインベーストランザクション** だけは、そのアウトプットの値がインプットの値より高い。これは事実上コインの創造に相当する。よって定義上、コインベーストランザクションに手数料は存在しない。コインベーストランザクションはすべてのブロックの最初のトランザクションとなっている。
コンセンサスルールによって、コインベーストランザクションの中のアウトプットの値の合計は、ブロック内のトランザクション手数料の合計とマイニング報酬の和を超えないようになっている。
