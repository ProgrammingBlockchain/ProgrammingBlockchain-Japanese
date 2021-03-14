## ビットコインを使ってみる {#spend-your-coin}

ここまでで **ビットコインアドレス**、**ScriptPubKey**、**秘密鍵**や**マイナー** とは何か理解いただけたと思うので、自分の手で初めての**トランザクション**を作ってみよう。

このレッスンを進めて1行ごとにコードを追加していくと、最終的にTwitterライクなメッセージで、この本へのフィードバックを残すプログラムを作るようになっている。

では前のレッスンでやったように、消費したい **トランザクションアウトプット** を含む **トランザクション** を見てみよう。

新しい **コンソールプロジェクト**（.NET4.5以上）を作り、**QBitNinja.Client** のNuGetパッケージをインストールしてほしい。

すでに秘密鍵を作成して、メモを取っただろうか？そして、その鍵に対応するビットコインアドレスを取得して、ビットコインを送金しただろうか？もしまだやってなくても心配しないでいい。どうやってやるかここで簡単に繰り返そう。

```cs
// もし本番のネットワークで以下を進めたい場合は、Network.Main としてください
var network = Network.TestNet;

var privateKey = new Key();
var bitcoinPrivateKey = privateKey.GetWif(network);
var address = bitcoinPrivateKey.GetAddress(ScriptPubKeyType.Legacy);

Console.WriteLine(bitcoinPrivateKey);
Console.WriteLine(address);
```

**bitcoinPrivateKey** と **address** の値をメモし、そこにいくらかのコインを送り、そのトランザクションIDをメモしよう。（トランザクションIDは、自分のウォレットアプリに（多分）表示される、それか、[blockchain.info](https://blockchain.info/)のようなブロックエクスプローラーで（アドレスから）見つけることができる。）

次に、秘密鍵をインポートする。

```cs
var bitcoinPrivateKey = new BitcoinSecret("cN5YQMWV8y19ntovbsZSaeBxXaVPaK4n7vapp4V56CKx5LhrK2RS");
var network = bitcoinPrivateKey.Network;
var address = bitcoinPrivateKey.GetAddress(ScriptPubKeyType.Legacy);

Console.WriteLine(bitcoinPrivateKey); // cN5YQMWV8y19ntovbsZSaeBxXaVPaK4n7vapp4V56CKx5LhrK2RS
Console.WriteLine(address); // mkZzCmjAarnB31n5Ke6EZPbH64Cxexp3Jp
```

そして最後にトランザクションの情報を取得する。

```cs
var client = new QBitNinjaClient(network);
var transactionId = uint256.Parse("0acb6e97b228b838049ffbd528571c5e3edd003f0ca8ef61940166dc3081b78a");
var transactionResponse = client.GetTransaction(transactionId).Result;

Console.WriteLine(transactionResponse.TransactionId); // 0acb6e97b228b838049ffbd528571c5e3edd003f0ca8ef61940166dc3081b78a
Console.WriteLine(transactionResponse.Block.Confirmations);
```

これでトランザクションを生成する情報はすべて揃った。トランザクションを生成するうえで、必要な質問はこれだ。「**どこから、どこへ、いくら？**」

### どこから？

このケースでは、先ほど取得したばばトランザクションの2番目のoutpointを使いたいのだが、どのようにそれを見つけたのかを示す。

```cs
var receivedCoins = transactionResponse.ReceivedCoins;
OutPoint outPointToSpend = null;
var myScriptPubKey = bitcoinPrivateKey.GetAddress(ScriptPubKeyType.Legacy).ScriptPubKey;
foreach (var coin in receivedCoins)
{
    if (coin.TxOut.ScriptPubKey == myScriptPubKey)
    {
        outPointToSpend = coin.Outpoint;
    }
}

if (outPointToSpend == null)
    throw new Exception("エラー：どのトランザクションアウトプットも自分のScriptPubKeyに送られていない");
else
    Console.WriteLine("{0}番目のアウトポイントを使いたい。", outPointToSpend.N);
```

支払いのためにはトランザクションの中でこのoutpointを参照する必要がある。以下のようにトランザクションを生成する。

```cs
var transaction = Transaction.Create(network);
transaction.Inputs.Add(new TxIn() {
    PrevOut = outPointToSpend
});
```

### どこへ？

必要な質問は覚えているだろうか？「**どこから、どこへ、いくら？**」だ。
**トランザクションインプット** を組み立ててトランザクションに加えることが、「どこから？」という質問への答えだった。
**トランザクションアウトプット** を組み立ててトランザクションに加えることは、残っている２つの質問への答えである。

> この本への寄付アドレス：[1KF8kUVHK42XzgcmJF4Lxz4wcL5WDL97PB](https://blockchain.info/address/1KF8kUVHK42XzgcmJF4Lxz4wcL5WDL97PB)
>
> 寄付されたお金は本の残りを書いている間、執筆者の Nicolas の「コーヒー＆寿司ウォレット」というウォレットに入る。Nicolasが食事できて素直に過ごせるようにするために使うものだ。
>
> もしこのチャレンジに成功すると、[http://n.bitcoin.ninja/](http://n.bitcoin.ninja/)の **成功者の殿堂リスト** の中に自分の寄付を見つけることができるだろう（気前の良さ順に表示される）。

```cs
var hallOfTheMakersAddress = new BitcoinPubKeyAddress("1KF8kUVHK42XzgcmJF4Lxz4wcL5WDL97PB", Network.Main);
```

もしTestNet上でコードを動かしているなら、以下のコードを使って、どのTestNetのアドレスにでも良いのでコインを送ってみよう。

```cs
var hallOfTheMakersAddress = BitcoinAddress.Create("mzp4No5cmCXjZUpf112B1XWsvWBfws5bbB", Network.TestNet);
```

### いくら？

Bitcoin には[いくつかの単位](https://en.bitcoin.it/wiki/Units)があるが、中でも三つは覚えておくべきだ。ビットコイン、ビットとサトシである。1ビットコインは 1,000,000 ビットであり、 100サトシが 1ビット。1サトシがビットコインネットワークの中では最小の単位である。

もし **0.0004 BTC** （数ドル）を **0.001 BTC** が入っている **まだ使っていないトランザクションアウトプット** から使いたいとする。実はここでは全てのコインを使い切らないといけないことに注意しよう！
図解が以下に示すとおりですが、**トランザクションアウトプット** が **0.0004 BTC** を成功者の殿堂に、そしてあなた自身に残りの **0.00053** BTCを戻すように仕分けている。
残りの **0.00007BTC** はどうなったのだろう？これが _マイナーへの手数料_ と言われるものだ。
このマイナーの手数料によって、あなたが作成するトランザクションをマイナーが取り込むインセンティブが生まれる。当然この手数料が高ければ高いほど、マイナーがあなたのトランザクションを次のブロックに含めるモチベーションが上がるので、それはつまりそのトランザクションが承認されるのがより早くなると言うことである。もしマイナーへの手数料をゼロにしたら、もしかするとあなたのトランザクションが承認されることはないかもしれない。

![](../assets/SpendTx.png)

```cs
// 成功者の殿堂に送る分
transaction.Outputs.Add(Money.Coins(0.0004m), hallOfTheMakersAddress.ScriptPubKey);

// 自分に戻す分 (おつり)
transaction.Outputs.Add(new Money(0.00053m, MoneyUnit.BTC), myScriptPubKey);
```

*訳注：自分に戻す分 (おつり)がゼロの場合は、その TxOut (上記では changeBackTxOut) をトランザクションに追加する必要はない。数量がゼロのTxOutはトランザクション実行時にエラーとなる。*

ここで、トランザクションアウトプットの追加に関するコードのリファクタリングをしてみよう。
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
transaction.Outputs.Add(hallOfTheMakersAmount, hallOfTheMakersAddress.ScriptPubKey);
// お釣り
transaction.Outputs.Add(changeAmount, myScriptPubKey);
```

### ブロックチェーン上でのメッセージ送信

さあ、フィードバックを追加しよう！フィードバックは80バイト以下でなければならない。そうでないとトランザクションは拒否される。
このフィードバックはトランザクションと一緒に、（トランザクションが承認された後に）[成功者の殿堂](http://n.bitcoin.ninja/)の中に表示される。

```cs
var message = "Long live NBitcoin and its makers!";
// Encoding を使うためには using System.Text を追加する必要がある
var bytes = Encoding.UTF8.GetBytes(message);
transaction.Outputs.Add(Money.Zero, TxNullDataTemplate.Instance.GenerateScriptPubKey(bytes));
```

### トランザクションの内容のまとめ

署名する前に、ここまでをまとめるため、トランザクション全体を見てほしい。
3つの**トランザクションアウトプット**があり、2つは**コイン付き**で、1つは**コインなし**（メッセージあり）となっている。そこで、「普通の」**トランザクションアウトプット** の **ScriptPubKey** とメッセージ付きの **トランザクションアウトプット** の **ScriptPubKey** の違いに気づくだろう。

```json
{
  "hash": "eeffd48b317e7afa626145dffc5a6e851f320aa8bb090b5cd78a9d2440245067",
  "ver": 1,
  "vin_sz": 1,
  "vout_sz": 3,
  "lock_time": 0,
  "size": 164,
  "in": [
    {
      "prev_out": {
        "hash": "0acb6e97b228b838049ffbd528571c5e3edd003f0ca8ef61940166dc3081b78a",
        "n": 0
      },
      "scriptSig": ""
    }
  ],
  "out": [
    {
      "value": "0.00040000",
      "scriptPubKey": "OP_DUP OP_HASH160 d3a689bc36464b9d74e1721fd321d4686eae594e OP_EQUALVERIFY OP_CHECKSIG"
    },
    {
      "value": "0.00053000",
      "scriptPubKey": "OP_DUP OP_HASH160 376b786582a3423bcdda4517ea87f0a7e862f27b OP_EQUALVERIFY OP_CHECKSIG"
    },
    {
      "value": "0.00000000",
      "scriptPubKey": "OP_RETURN 4c6f6e67206c697665204e426974636f696e20616e6420697473206d616b65727321"
    }
  ]
}
```

**トランザクションインプット**により着目してほしい。そこには **prev\_out**と**scriptSig**がある。
**Exercise**：読み進む前に、このコードの中で **scriptSig** に何を入れるのか、そして、どのようにそれをコーディングして取得できるかを考えてみよう！

ブロックエクスプローラーで、**prev\_out**の**ハッシュ**をチェックしてみよう。[アウトポイントの詳細](https://testnet.smartbit.com.au/tx/0acb6e97b228b838049ffbd528571c5e3edd003f0ca8ef61940166dc3081b78a)。0.001 BTCがアドレスに送信されるのが見えるだろう。

ここで **prev\_out** の **n** の値\(インデックス\)は 0 だ。インデックスは0から始まるから、トランザクションの最初のアウトプットを使いたいということになる。補足として二つ目のアウトプットは　1.0989548 BTC となっているが、これはお釣りである。

### トランザクションに署名する

ここまででトランザクションを作成したので、署名しなければならない。言い換えれば、私たちはトランザクションインプットの中で参照した前トランザクションのアウトプットが自分のものであると証明しなければならないのだ。

署名は[複雑](https://en.bitcoin.it/w/images/en/7/70/Bitcoin_OpCheckSig_InDetail.png)だと思われるかもしれないが、我々がそれを簡単にしよう。

最初に**scriptSig**に**入れた値**から、どのようにそれをコードから得られるのかを振り返ってみよう。ここで ScriptPubKey を使って ScriptSig を埋める方法は二つある。

```cs
// ビットコインアドレスから作成する方法
var address = BitcoinAddress.Create("mkZzCmjAarnB31n5Ke6EZPbH64Cxexp3Jp", Network.TestNet);
transaction.Inputs[0].ScriptSig = address.ScriptPubKey;

// もしくは秘密鍵から作成する方法
var bitcoinPrivateKey = new BitcoinSecret("cN5YQMWV8y19ntovbsZSaeBxXaVPaK4n7vapp4V56CKx5LhrK2RS", Network.TestNet);
transaction.Inputs[0].ScriptSig =  bitcoinPrivateKey.GetAddress(ScriptPubKeyType.Legacy).ScriptPubKey;
```

そして秘密鍵を署名のためにパラメーターとして与える必要がある。

```cs
transaction.Sign(bitcoinPrivateKey, receivedCoins.ToArray());
```

上記を実行すると、トランザクションインプットにある ScriptSig に署名が入っており、トランザクション全体が署名されたことがわかる。

[ここで](https://testnet.smartbit.com.au/tx/eeffd48b317e7afa626145dffc5a6e851f320aa8bb090b5cd78a9d2440245067)ブロックエクスプローラーを使って、テストネットのトランザクションを見ることができる。

### トランザクションを伝搬させる

これで君の最初のトランザクションに署名したのだ。おめでとう！トランザクションはもうその役目を果たす準備ができている。残るはマイナーがそのトランザクションを把握できるようにネットワークに伝搬させるだけだ。

#### QBitNinjaを使って

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

#### ローカルにインストールされているBitcoin Coreを使って

```cs
// using NBitcoin.Protocol を追加する必要があります
using (var node = Node.ConnectToLocal(network)) //Connect to the node
{
    node.VersionHandshake(); // Say hello
                             // Advertize your transaction (send just the hash)
    node.SendMessage(new InvPayload(InventoryType.MSG_TX, transaction.GetHash()));
    // Send it
    node.SendMessage(new TxPayload(transaction));
    // using System.Threading を追加する必要があります
    Thread.Sleep(500); // Wait a bit
}
```

ここでは_using\*_コードブロックで、ノードとのコネクションを閉じる処理をさせている。これがすべてだ！

ビットコインネットワークに直接接続することもできるが、自分の信用できるノードに接続することをおすすめする（そのほうがより早いし、より簡単だ）。

### より多くの演習が必要なら

Youtube: [How to make your first transaction with NBitcoin](https://www.youtube.com/watch?v=X4ZwRWIF49w)
CodeProject: [Create a Bitcoin transaction by hand.](http://www.codeproject.com/Articles/1151054/Create-a-Bitcoin-transaction-by-hand)
CodeProject: [DotNetWallet - Build your own Bitcoin wallet in C\#](https://www.codeproject.com/script/Articles/ArticleVersion.aspx?waid=214550&aid=1115639)
