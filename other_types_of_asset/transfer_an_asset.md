## 資産の移動 {#transfer-an-asset}

さあ今、僕が**BlockchainProgrammingコイン**を送ったと想定してみよう。  
どのようにして僕にそのコインを送り返せるだろうか。  
そのためには**カラードコイン**をビルドする必要がある。

Issuing an Assetで見たとおり、「nico」のアドレスで受け取った10アセットを使いたいということにしよう。  
これが使いたいコインである。

```json
{
  "transactionId": "fa6db7a2e478f3a8a0d1a77456ca5c9fa593e49fd0cf65c7e349e5a4cbe58842",
  "index": 0,
  "value": 600,
  "scriptPubKey": "76a914356facdac5f5bcae995d13e667bb5864fd1e7d5988ac",
  "redeemScript": null,
  "assetId": "AVAVfLSb1KZf9tJzrUVpktjxKUXGxUTD4e",
  "quantity": 10
}
```

そしてこれが、コードで上記のようなカラードコインを初期化する方法である。

```cs
var coin = new Coin(
    fromTxHash: new uint256("fa6db7a2e478f3a8a0d1a77456ca5c9fa593e49fd0cf65c7e349e5a4cbe58842"),
    fromOutputIndex: 0,
    amount: Money.Satoshis(2000000),
    scriptPubKey: new Script(Encoders.Hex.DecodeData("76a914356facdac5f5bcae995d13e667bb5864fd1e7d5988ac")));
BitcoinAssetId assetId = new BitcoinAssetId("AVAVfLSb1KZf9tJzrUVpktjxKUXGxUTD4e");
ColoredCoin colored = coin.ToColoredCoin(assetId, 10);
```

より簡単にカラードコインを得るために、ウェブサービスやカスタマイズされたコードをどのように使えるかをあとで説明するつもりだ。  
また、手数料を支払うためにさらにコイン（forFees）を必要とする。  
アセットの移動は**TransactionBuilder**を使えば本当に簡単である。

```cs
var book = BitcoinAddress.Create("1KF8kUVHK42XzgcmJF4Lxz4wcL5WDL97PB");
var nicoSecret = new BitcoinSecret("??????????");
var nico = nicoSecret.GetAddress(); //15sYbVpRh6dyWycZMwPdxJWD4xbfxReeHe

var forFees = new Coin(
    fromTxHash: new uint256("7f296e96ec3525511b836ace0377a9fbb723a47bdfb07c6bc3a6f2a0c23eba26"),
    fromOutputIndex: 0,
    amount: Money.Satoshis(4425000),
    scriptPubKey: new Script(Encoders.Hex.DecodeData("76a914356facdac5f5bcae995d13e667bb5864fd1e7d5988ac")));

TransactionBuilder builder = new TransactionBuilder();
var tx = builder
    .AddKeys(nicoSecret)
    .AddCoins(colored, forFees)
    .SendAsset(book, new AssetMoney(assetId, 10))
    .SetChange(nico)
    .SendFees(Money.Coins(0.0001m))
    .BuildTransaction(true);
Console.WriteLine(tx);
```

```json
{
  ….
  "out": [
    {
      "value": "0.00000000",
      "scriptPubKey": "OP_RETURN 4f410100010a00"
    },
    {
      "value": "0.00000600",
      "scriptPubKey": "OP_DUP OP_HASH160 c81e8e7b7ffca043b088a992795b15887c961592 OP_EQUALVERIFY OP_CHECKSIG"
    },
    {
      "value": "0.04415000",
      "scriptPubKey": "OP_DUP OP_HASH160 356facdac5f5bcae995d13e667bb5864fd1e7d59 OP_EQUALVERIFY OP_CHECKSIG"
    }
  ]
}
```

Confirmedされていないとはいえ、基本的にはアセットの移動はこれで成功だ。

![](../assets/Coinprism2.png)
