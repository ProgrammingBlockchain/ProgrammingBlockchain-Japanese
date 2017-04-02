## ビットコインを使ってみる {#spend-your-coin}

さあ今や **ビットコインアドレス**、**ScriptPubKey**、**秘密鍵**や**マイナー** とは何かがわかったので、自分の手で初めての**トランザクション**を作れるだろう。

このレッスンを進めて1行ごとにコードを追加していくと、最終的にTwitterライクなメッセージで、この本へのフィードバックを残すプログラムを作るようになっている。

では前のレッスンでやったように、消費したい **トランザクションアウトプット** を含む **トランザクション** を見てみよう。

新しい **コンソールプロジェクト**（.NET4.5以上）を作り、**QBitNinja.Client** のNuGetパッケージをインストールしてほしい。

すでに秘密鍵を作成して、メモを取っただろうか？そして、その鍵に対応するビットコインアドレスを取得して、ビットコインを送金しただろうか？もしまだやってなくても心配しないでいい。どうやってやるかここで簡単に繰り返そう。

```cs
var network = Network.Main;

var privateKey = new Key();
var bitcoinPrivateKey = privateKey.GetWif(network);
var address = bitcoinPrivateKey.GetAddress();

Console.WriteLine(bitcoinPrivateKey);
Console.WriteLine(address);
```

**bitcoinPrivateKey** と **address** の値をメモし、そこにいくらかのコインを送り、そのトランザクションIDをメモしてくれ。（トランザクションIDは、自分のウォレットアプリに（多分）表示される、それか、[blockchain.info](https://blockchain.info/)のようなブロックエクスプローラーで（アドレスから）見つけることができる。）

次に、秘密鍵をインポートする。

```cs
var bitcoinPrivateKey = new
BitcoinSecret("cSZjE4aJNPpBtU6xvJ6J4iBzDgTmzTjbq8w2kqnYvAprBCyTsG4x");
var network = bitcoinPrivateKey.Network;
var address = bitcoinPrivateKey.GetAddress();

Console.WriteLine(network); // TestNet
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

さあ、これでトランザクションを生成する情報はすべて揃った。で、必要な質問はこれだ。「**どこから、どこへ、いくら？**」

### どこから？

このケースでは、前のトランザクションの2番目のoutpointを使いたいのだが、どのようにそれを見つけたのかを示す。

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
    throw new Exception("エラー：どのトランザクションアウトプットも送ったコインのScriptPubKeyを持ってない！");
Console.WriteLine("{0}番目のアウトポイントを使いたい。", outPointToSpend.N + 1);
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
**トランザクションインプット** を組み立ててトランザクションに加えることが、「どこから？」という質問への答えだった。  
**トランザクションアウトプット** を組み立ててトランザクションに加えることは、残っている２つの質問への答えである。

この本への寄付アドレス：[1KF8kUVHK42XzgcmJF4Lxz4wcL5WDL97PB](https://blockchain.info/address/1KF8kUVHK42XzgcmJF4Lxz4wcL5WDL97PB)  
寄付されたお金は本の残りを書いている間、私が食事できて素直にいられるように使う「コーヒー＆寿司ウォレット」という名前の僕のウォレットに入る。  
もしこのチャレンジに成功すると、[http://n.bitcoin.ninja/](http://n.bitcoin.ninja/)の **成功者の殿堂リスト** の中に自分の寄付を見つけることができるだろう（気前の良さ順に表示される）。

```cs
var hallOfTheMakersAddress = new BitcoinPubKeyAddress("1KF8kUVHK42XzgcmJF4Lxz4wcL5WDL97PB");
```

もしTestNet上でコードを動かしているなら、どのTestNetのアドレスにでも良いのでコインを送ってみよう。

```cs
var hallOfTheMakersAddress = BitcoinAddress.Create("mzp4No5cmCXjZUpf112B1XWsvWBfws5bbB");
```

### いくら？

もし **1BTC** を伴う **トランザクションインプット** から **0.5BTC** を使いたいとしても、確実にすべてのコインを使い切らなければならない！  
図解が以下に示すとおり、**トランザクションアウトプット** が **0.5** BTCを成功者の殿堂に、そしてあなたに**0.4999**BTCを戻すように仕分けている。  
残りの **0.0001BTC** はどうなったのだろう？これはマイナーの手数料となっていて、彼らがこのトランザクションを次のブロックに含めるためのインセンティブとなっているのであった。

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

*訳注：おつり（changeBackTxOut）がゼロの場合は changeBackTxOut をトランザクションには追加する必要はない。数量がゼロのTxOutはトランザクション実行時にエラーとなる。*

ここでコードの微調整をしてみる。  
以下のリンクから、この章のサンプルコードで使っているビットコインアドレスをブロックエクスプローラーでチェックできる。（TestNetのアドレスを使っているが。）

[http://tbtc.blockr.io/address/info/mzK6Jy5mer3ABBxfHdcxXEChsn3mkv8qJv](http://tbtc.blockr.io/address/info/mzK6Jy5mer3ABBxfHdcxXEChsn3mkv8qJv)

```cs
// ビットコインをいくら送金先に送りたいか
var hallOfTheMakersAmount = new Money(0.5m, MoneyUnit.BTC);
/* これを書いている時点では、マイニング手数料は５セント
 * だが、ビットコインのマーケット価格と
 * その時の推奨手数料レートに合わせて
 * 手数料を上げるか、もしくは、下げるかを考慮する必要がある。
*/
var minerFee = new Money(0.0001m, MoneyUnit.BTC);
// UTXOからいくらトータルでビットコインを 使いたいか。
var txInAmount = receivedCoins[(int) outPointToSpend.N].TxOut.Value;
Money changeBackAmount = txInAmount - hallOfTheMakersAmount - minerFee;
```

計算した値をトランザクションアウトプットに追加する。

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

そしてそれらをトランザクションに追加する。

```cs
transaction.Outputs.Add(hallOfTheMakersTxOut);
transaction.Outputs.Add(changeBackTxOut);
```

### ブロックチェーン上でのメッセージ送信

さあ、フィードバックを追加しよう！フィードバックは40バイト以下でなければならない。そうでないとアプリケーションがクラッシュする。  
このフィードバックはトランザクションと一緒に、（トランザクションが承認された後に）[成功者の殿堂](http://n.bitcoin.ninja/)の中に現れる。

```cs
var message = "nopara73 loves NBitcoin!";
var bytes = Encoding.UTF8.GetBytes(message);
transaction.Outputs.Add(new TxOut()
{
    Value = Money.Zero,
    ScriptPubKey = TxNullDataTemplate.Instance.GenerateScriptPubKey(bytes)
});
```

署名する前に、ここまでをまとめるため、トランザクション全体を見てほしい。  
3つの**トランザクションアウトプット**があり、2つは**コイン付き**で、1つは**コインなし**（メッセージあり）となっている。そこで、「普通の」**トランザクションアウトプット** の **ScriptPubKey** とメッセージ付きの **トランザクションアウトプット** の **ScriptPubKey** の違いに気づくだろう。

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

**トランザクションインプット**により着目してほしい。そこには **prev\_out**と**scriptSig**がある。  
**Exercise**：読み進む前に、このコードの中で **scriptSig** に何を入れるのか、そして、どのようにそれをコーディングして取得できるかを考えてみよう！

ブロックエクスプローラーで、**prev\_out**の**ハッシュ**をチェックしてみよう。

[http://tbtc.blockr.io/tx/info/e44587cf08b4f03b0e8b4ae7562217796ec47b8c91666681d71329b764add2e3](http://tbtc.blockr.io/tx/info/e44587cf08b4f03b0e8b4ae7562217796ec47b8c91666681d71329b764add2e3)  
ここで**prev\_out**のnの値\(インデックス\)は1だ。インデックスは0から始まるから、トランザクションの2番目のアウトプットを使いたいということになる。  
ブロックエクスプローラーの中で該当するアドレスが`mzK6Jy5mer3ABBxfHdcxXEChsn3mkv8qJv`であるとわかるので、このようにアドレスからscriptSigが得られる。

```cs
var address = BitcoinAddress.Create("mzK6Jy5mer3ABBxfHdcxXEChsn3mkv8qJv");
transaction.Inputs[0].ScriptSig = address.ScriptPubKey;
```

### トランザクションに署名する

さあトランザクションを作成したので、署名しなければならない。言い換えれば、私たちはトランザクションインプットの中で参照した前トランザクションのアウトプットが自分のものであると証明しなければならないのだ。

署名は[複雑](https://en.bitcoin.it/w/images/en/7/70/Bitcoin_OpCheckSig_InDetail.png)だと思われるかもしれないが、我々がそれを簡単にしよう。

最初に**scriptSig**に**入れた値**から、どのようにそれを得たかをコードから振り返ってみよう。思い出してほしい、ブロックエクスプローラーから上記のアドレスをコピー＆ペーストしただろうが、今度はビットコインシークレットから取得してみよう。

```cs
transaction.Inputs[0].ScriptSig =  bitcoinPrivateKey.ScriptPubKey;
```

そして秘密鍵を署名のためにパラメーターとして与える必要がある。

```cs
transaction.Sign(bitcoinPrivateKey, false);
```

### トランザクションを伝搬させる

これで君の最初のトランザクションに署名したのだ。おめでとう！トランザクションはもうその役目を果たす準備ができている。残るはマイナーがそのトランザクションを把握できるようにネットワークに伝搬させるだけだ。

#### QBitNinjaを使って：

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

#### ローカルにインストールされているBitcoin Coreを使って：

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

ここでは_using\*_コードブロックで、ノードとのコネクションを閉じる処理をさせている。これがすべてだ！

ビットコインネットワークに直接接続することもできるが、自分の信用できるノードに接続することをおすすめする（そのほうがより早いし、より簡単だ）。

## より多くの演習が必要なら：

Youtube: [How to make your first transaction with NBitcoin](https://www.youtube.com/watch?v=X4ZwRWIF49w)  
CodeProject: [Create a Bitcoin transaction by hand.](http://www.codeproject.com/Articles/1151054/Create-a-Bitcoin-transaction-by-hand)  
CodeProject: [DotNetWallet - Build your own Bitcoin wallet in C\#](https://www.codeproject.com/script/Articles/ArticleVersion.aspx?waid=214550&aid=1115639)
