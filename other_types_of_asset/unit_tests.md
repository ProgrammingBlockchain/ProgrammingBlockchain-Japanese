## 単体テスト {#unit-tests}

前の章では、**カラードコイン** 資産をコードするのに苦労していたように見えたかもしれない。  
なぜかというと **カラードコイン** から **トランザクション** を生成する方法を見せたかっただけだからだ。

現場では、カラードコインのトランザクションや資金を取得するためには、サードパーティーのAPIに頼ろうとするだろう。しかしそれは良いアイディアではひょっとするとないかもしれない。なぜならば、APIの提供者に対しての信頼に基づく、プログラムの依存関係ができてしまうからだ。

**NBitcoin** によって、ウェブサービスに頼っても良いし、カラードコインの **トランザクション** をプログラム自身に取得させても良くなる。これによって、コードの単体テストに柔軟性を持つことができるようになった。

2つの発行者を紹介しよう：シルバーとゴールドだ。そして3人の参加者を紹介しよう：ボブ、アリスとサトシだ。  
シルバー、ゴールドとサトシにビットコインを送る模擬のトランザクションを作ってみよう。

```cs
var gold = new Key();
var silver = new Key();
var goldId = gold.PubKey.ScriptPubKey.Hash.ToAssetId();
var silverId = silver.PubKey.ScriptPubKey.Hash.ToAssetId();

var bob = new Key();
var alice = new Key();
var satoshi = new Key();

var init = new Transaction()
{
    Outputs =
    {
        new TxOut("1.0", gold),
        new TxOut("1.0", silver),
        new TxOut("1.0", satoshi)
    }
};
```

**Init** はカラードコインの発行も移動も全く含んでいない。しかし、それを確信したいと思っている場面を想像してみよう。どのように進めればよいだろうか。

**NBitcoin** では、カラードコインの移動や発行の概要が **ColoredTransaction** と呼ばれるクラスに表現されている。

![](../assets/ColoredTransaction.png)

**ColoredTransaction** クラスによって以下がわかる。

* どの **トランザクションインプット** がどのアセットを使うのか
* どの **トランザクションアウトプット** がどのアセットを表すのか
* どの **トランザクションアウトプット** がどのアセットを移動するのか

しかし今私たちにとって興味をわかせるメソッドは **FetchColors** で、これによってトランザクションインプットに与えたトランザクションから、カラードコインの情報を抜き出すことができる。

FetchColorsが **IColoredTransactionRepository** に依存していることを見てみよう。

![](../assets/IColoredCoinTransactionRepository.png)

**IColoredTransactionRepository** はトランザクションIDから、**カラードコインのトランザクション** を示してくれるストアでしかない。しかしこれは **ITransactionRepository** に依存関係があることがわかるだろう。ITransactionRepositoryはトランザクションIDとそのトランザクションをマッピングしてくれるものだ。

**IColoredTransactionRepository** の実行は、カラードコインの操作を担う公開されたAPIである **CoinprismColoredTransactionRepository** の実行と同義だ。  
しかし、簡単に自分自身でそれらを行うことができる。それがここで示すように、**FetchColors** がどのように動くかでわかる。

最も単純なケースを見てみよう。**IColoredTransactionRepository** がカラードコインを知っているから、そのケースであれば **FetchColors** はその結果を取ってくるだけだ。

![](../assets/FetchColors.png)

2番目のケースでは、**IColoredTransactionRepository** がカラードコインのトランザクションについて何も知らない場合だ。  
そのときはオープンアセットの仕様に基づいて **FetchColors** を使って、カラードコインを算出する必要がある。

しかし、カラードコインを求めるためには、**FetchColors** はカラードコインの親トランザクションを必要とする。  
だから **ITransactionRepository** 上で各トランザクションを取得し、それらに対して **FetchColors** をコールするのだ。  
一度 **FetchColors** が再帰的に親トランザクションのカラードコインを解明すると、カラードコインのトランザクションを算出して、**IColoredTransactionRepository** にその結果を再度引き渡し、結果を得る。

![](../assets/FetchColors2.png)

これによって、将来的に発生する、トランザクションのカラードコインを取得するリクエストが素早く解決される。  
**IColoredTransactionRepository** は読み取り専用だ（**CoinprismColoredTransactionRepository** と同じでPutする操作が無視される）。

さあ、例に戻ろう。  
ユニットテストを書くときのトリックは、メモリ上の **IColoredTransactionRepository** を使うことだ。

```cs
var repo = new NoSqlColoredTransactionRepository();
```

さあ、**init** トランザクションを格納しよう。

```cs
repo.Transactions.Put(init);
```

注釈するが、Putは拡張メソッドなので、以下の参照を追加する必要がある。

```cs
using NBitcoin.OpenAsset;
```

ファイルの1番上に、参照を付け足して欲しい。

そしたら、カラードコインを抽出できる。

```cs
ColoredTransaction color = ColoredTransaction.FetchColors(init, repo);
Console.WriteLine(color);
```

```json
{
  "inputs": [],
  "issuances": [],
  "transfers": [],
  "destructions": []
}
```

想定どおり、**init** トランザクションはカラードコインのインプットも発行も移動も破壊も何も持っていない。

ということで、SilverとGoldに送られたコインを、発行用のコインとして使ってみよう。

```cs
var issuanceCoins =
    init
    .Outputs
    .AsCoins()
    .Take(2)
    .Select((c, i) => new IssuanceCoin(c))
    .OfType<ICoin>()
    .ToArray();
```

Goldが最初、次にSilverとなっている。

前の章でやったとおり、**TransactionBuilder** を使ってGoldをSatoshiに送り、リポジトリに結果として生成されるトランザクションを格納し、結果を表示できる。

```cs
{
  "inputs": [],
  "issuances": [
    {
      "index": 0,
      "asset": "ATEwaRSNeCgBjxjcur7JtfypFjqQgAtLJs",
      "quantity": 10
    }
  ],
  "transfers": [],
  "destructions": []
}
```

これは最初の **トランザクションアウトプット** には10goldあるということだ。

さて今、**サトシ** が **アリス** に4goldを送りたいということにしてみよう。  
最初にサトシはトランザクションから **カラードコイン** を取得する。

```cs
var goldCoin = ColoredCoin.Find(sendGoldToSatoshi, color).FirstOrDefault();
```

そしてこのようにトランザクションを生成する。

```cs
builder = new TransactionBuilder();
var sendToBobAndAlice =
        builder
        .AddKeys(satoshi)
        .AddCoins(goldCoin)
        .SendAsset(alice, new AssetMoney(goldId, 4))
        .SetChange(satoshi)
        .BuildTransaction(true);
```

**NotEnoughFundsException** が発生してしまうだろう。  
なぜかというと、このトランザクションはトランザクションインプット（**goldコイン**）に600satoshiあって、トランザクションアウトプットに1200satoshiある。（1つ目の **トランザクションアウトプット** はアリスにアセットを送るもので、もう1つはサトシにお釣りを送るものだ）

つまり600satoshi不足しているのだ。  
サトシに送られている **init** トランザクションにある最後の1BTCに相当する **コイン** をトランザクションに加えることで解決できる。

```cs
var satoshiBtc = init.Outputs.AsCoins().Last();
builder = new TransactionBuilder();
var sendToAlice =
        builder
        .AddKeys(satoshi)
        .AddCoins(goldCoin, satoshiBtc)
        .SendAsset(alice, new AssetMoney(goldId, 4))
        .SetChange(satoshi)
        .BuildTransaction(true);
repo.Transactions.Put(sendToAlice);
color = ColoredTransaction.FetchColors(sendToAlice, repo);
```

トランザクションとカラードコインの内容を見てみよう。

```cs
Console.WriteLine(sendToAlice);
Console.WriteLine(color);
```

```json
{
  ….
  "in": [
    {
      "prev_out": {
        "hash": "46117f3ef44f2dfd87e0bc3f461f48fe9e2a3a2281c9b3802e339c5895fc325e",
        "n": 0
      },
      "scriptSig": "304502210083424305549d4bb1632e2c67736383558f3e1d7fb30ce7b5a3d7b87a53cdb3940220687ea53db678b467b98a83679dec43d27e89234ce802daf14ed059e7a09557e801 03e232cda91e719075a95ede4c36ea1419efbc145afd8896f36310b76b8020d4b1"
    },
    {
      "prev_out": {
        "hash": "aefa62270999baa0d57ddc7d2e1524dd3828e81a679adda810657581d7d6d0f6",
        "n": 2
      },
      "scriptSig": "30440220364a30eb4c8a82cc2a79c54d0518b8ba0cf4e49c73a5bbd17fe1a5683a0dfa640220285e98f3d336f1fa26fb318be545162d6a36ce1103c8f6c547320037cb1fb8e901 03e232cda91e719075a95ede4c36ea1419efbc145afd8896f36310b76b8020d4b1"
    }
  ],
  "out": [
    {
      "value": "0.00000000",
      "scriptPubKey": "OP_RETURN 4f41010002060400"
    },
    {
      "value": "0.00000600",
      "scriptPubKey": "OP_DUP OP_HASH160 5bb41cd29f4e838b4b0fdcd0b95447dcf32c489d OP_EQUALVERIFY OP_CHECKSIG"
    },
    {
      "value": "0.00000600",
      "scriptPubKey": "OP_DUP OP_HASH160 469c5243cb08c82e78a8020360a07ddb193f2aa8 OP_EQUALVERIFY OP_CHECKSIG"
    },
    {
      "value": "0.99999400",
      "scriptPubKey": "OP_DUP OP_HASH160 5bb41cd29f4e838b4b0fdcd0b95447dcf32c489d OP_EQUALVERIFY OP_CHECKSIG"
    }
  ]
}
Colored :
{
  "inputs": [
    {
      "index": 0,
      "asset": " ATEwaRSNeCgBjxjcur7JtfypFjqQgAtLJs ",
      "quantity": 10
    }
  ],
  "issuances": [],
  "transfers": [
    {
      "index": 1,
      "asset": " ATEwaRSNeCgBjxjcur7JtfypFjqQgAtLJs ",
      "quantity": 6
    },
    {
      "index": 2,
      "asset": " ATEwaRSNeCgBjxjcur7JtfypFjqQgAtLJs ",
      "quantity": 4
    }
  ],
  "destructions": []
}
```

ついに外部的な依存なしで、アセットを発行し移動する単体テストを実行できたわけだ。

もしサードパーティーのサービスに依存したくなければ、**IColoredTransactionRepository** を自前で作ることもできる。

より詳しい内容は[NBitcoin tests](https://github.com/MetacoSA/NBitcoin/blob/master/NBitcoin.Tests/transaction_tests.cs)で見ることができるし、コードプロジェクトの中にある私の記事の1つの「[Build Them all](https://www.codeproject.com/articles/835098/nbitcoin-build-them-all)」でも見ることができる。（マルチシグでのカラードコインの発行や交換）
