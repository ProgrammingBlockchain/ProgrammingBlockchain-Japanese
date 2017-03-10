## 資産の発行 {#issuing-an-asset}

### 目的 {#objective}

このエクササイズのために、**BlockchainProgrammingコイン** を発行しよう。

僕に **0.004ビットコイン** を送るごとに **BlockchainProgrammingコイン** を1つ手に入れられる。  
もしなにか言葉を添えてもらえると **さらにもう1つ** 手に入れられる。  
さらにこれは[成功者の殿堂](http://n.bitcoin.ninja/)に名を連ねる絶好の機会だ。

どのようにこの特徴をコーディングするか見てみよう。

### コインの発行 {#issuance-coin}

オープンアセットでは、アセットIDが発行者の **scriptPubKey** から引き出される。  
もしカラードコインを発行したければ、その **scriptPubKey** の所有権を証明する必要がある。そしてビットコインブロックチェーンでそれをする唯一の方法はその **scriptPubKey** に帰属しているコインを支払いに使うことだ。

カラードコインを発行するために使うことを選んだコインは **NBitcoin** では「**Issuance Coin**」と呼ぶ。  
この本のビットコインアドレス：[1KF8kUVHK42XzgcmJF4Lxz4wcL5WDL97PB](https://www.smartbit.com.au/address/1KF8kUVHK42XzgcmJF4Lxz4wcL5WDL97PB)からアセットを発行したいと思う。

僕の残高を見てほしい。アセットを発行するために次のコインを使うことに決めたとしよう。

```json
{
          "transactionId": "eb49a599c749c82d824caf9dd69c4e359261d49bbb0b9d6dc18c59bc9214e43b",
          "index": 0,
          "value": 2000000,
          "scriptPubKey": "76a914c81e8e7b7ffca043b088a992795b15887c96159288ac",
          "redeemScript": null
}
```

こうして僕のissuance coinを作る。

```cs
var coin = new Coin(
    fromTxHash: new uint256("eb49a599c749c82d824caf9dd69c4e359261d49bbb0b9d6dc18c59bc9214e43b"),
    fromOutputIndex: 0,
    amount: Money.Satoshis(2000000),
    scriptPubKey: new Script(Encoders.Hex.DecodeData("76a914c81e8e7b7ffca043b088a992795b15887c96159288ac")));

var issuance = new IssuanceCoin(coin);
```

ここで **TransactionBuilder** の助けを借りて、トランザクションを生成して署名する必要がある。

```cs
var nico = BitcoinAddress.Create("15sYbVpRh6dyWycZMwPdxJWD4xbfxReeHe");
var bookKey = new BitcoinSecret("???????");
TransactionBuilder builder = new TransactionBuilder();

var tx = builder
    .AddKeys(bookKey)
    .AddCoins(issuance)
    .IssueAsset(nico, new AssetMoney(issuance.AssetId, quantity: 10))
    .SendFees(Money.Coins(0.0001m))
    .SetChange(bookKey.GetAddress())
    .BuildTransaction(true);

Console.WriteLine(tx);
```

```json
{
  …
  "out": [
    {
      "value": "0.00000600",
      "scriptPubKey": "OP_DUP OP_HASH160 356facdac5f5bcae995d13e667bb5864fd1e7d59 OP_EQUALVERIFY OP_CHECKSIG"
    },
    {
      "value": "0.01989400",
      "scriptPubKey": "OP_DUP OP_HASH160 c81e8e7b7ffca043b088a992795b15887c961592 OP_EQUALVERIFY OP_CHECKSIG"
    },
    {
      "value": "0.00000000",
      "scriptPubKey": "OP_RETURN 4f410100010a00"
    }
  ]
}
```

トランザクションアウトプットにOP\_RETURNアウトプットを含んでいることがわかるだろう。事実、これがカラードコインについての情報が詰め込まれる場所なのだ。

ここにOP\_RETURNの中のデータフォーマットを示す。

![](../assets/ColorMaker.png)

今のケースではカラードコインの数は10だけで、それが`nico`に対して僕が発行したアセットの数だ。Metadataは任意のデータだ。あとの説明で「Asset Definition」を示すURLを示すところだとわかる。  
**Asset Definition** はそのアセットが何なのかを表現する文書だ。定義するかどうかは任意なのでここでは使っていない（Ricardian Contractの章でこれに触れる）。

さらに詳しい説明は [Open Asset Specification](https://github.com/OpenAssets/open-assets-protocol/blob/master/specification.mediawiki)をチェックしてほしい。

トランザクションの正当性を確認したら、ネットワークに送る準備ができている。

```cs
Console.WriteLine(builder.Verify(tx));
```

### QBitNinjaを使ってブロードキャスト

```cs
var client = new QBitNinjaClient(Network.Main);
BroadcastResponse broadcastResponse = client.Broadcast(tx).Result;

if (!broadcastResponse.Success)
{
    Console.WriteLine("ErrorCode: " + broadcastResponse.Error.ErrorCode);
    Console.WriteLine("Error message: " + broadcastResponse.Error.Reason);
}
else
{
    Console.WriteLine("Success!");
}
```

### ローカルのビットコインコアを使ってブロードキャスト

```cs
using (var node = Node.ConnectToLocal(Network.Main)) //Connect to the node
{
    node.VersionHandshake(); //Say hello
    //Advertize your transaction (send just the hash)
    node.SendMessage(new InvPayload(InventoryType.MSG_TX, tx.GetHash()));
    //Send it
    node.SendMessage(new TxPayload(tx));
    Thread.Sleep(500); //Wait a bit
}
```

僕のビットコインウォレットは、本のアドレスとNicoのアドレスと両方持っている。

![](../assets/NicoWallet.png)

見てのとおり、ビットコインコアは僕が払った0.0001BTCの手数料しか表示してくれず、600satoshiのコインは無視している。これはスパムを抑止する機能があるからだ。

この古典的なビットコインウォレットではカラードコインをまったく認識しない。  
さらに悪いことに、古典的なビットコインウォレットでカラードコインを使うと、そこに関連づいているアセットを破壊し、**トランザクションアウトプット** にあるビットコインの価値、つまり600satoshiしか移動しない。

カラードコインをサポートしていないウォレットにユーザーが送ってしまうことを防止するために、オープンアセットでは独自のアドレスフォーマットがあって、それはカラードコインウォレットしか認識しないようになっている。

```cs
nico = BitcoinAddress.Create("15sYbVpRh6dyWycZMwPdxJWD4xbfxReeHe");
Console.WriteLine(nico.ToColoredAddress());
```

```
akFqRqfdmAaXfPDmvQZVpcAQnQZmqrx4gcZ
```

いま、Coinprismのようなオープンアセットと互換性のあるウォレットで見てみると、僕のアセットが正しく表示される。

![](../assets/Coinprism.png)

言ったとおり、アセットIDが発行者の **ScriptPubKey** から引き出されている。ここにコードでアセットIDを得る方法を示す。

```cs
var book = BitcoinAddress.Create("1KF8kUVHK42XzgcmJF4Lxz4wcL5WDL97PB");
var assetId = new AssetId(book).GetWif(Network.Main);
Console.WriteLine(assetId); // AVAVfLSb1KZf9tJzrUVpktjxKUXGxUTD4e
```
