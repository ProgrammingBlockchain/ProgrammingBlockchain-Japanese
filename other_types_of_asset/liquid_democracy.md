## 流動体民主主義 {#liquid-democracy}

### 概要 {#overview}

この章は単純にカラードコインの1つのアプリケーションの概念的なエクササイズとなっている。

投票の後、投資家によっていくつか決定がなされる会社を想像してほしい。

* 何人かの投資家はトピックを十分に知らないから、いくつかの議題を他の誰かに委任したいと思っている。
* かなりたくさんの投資家がいる可能性もある。
* CEOとして、会社に資金を提供するために投票権を売れるようにしたい
* CEOとして、決定するときに票を投じることができるようにしたい

どうやってカラードコインによって、このような投票を透明性を維持したままで行えるようにできるだろうか。

しかし始める前に、ブロックチェーン上で投票を行うときのいくつかの欠点に触れておきたい。

* だれも投票者が実際に誰なのかを知らない
* マイナーは投票の内容に気づけてしまうかもしれない（たとえそれが可能であったとして、興味はないと思われるが）
* たとえだれも投票者が実際に誰なのか知らないとしても、いくつかの票を通じた投票者のふるまいに対する分析によって、アイデンティティがひょっとすると明らかになってしまうかもしれない

これらのポイントが妥当かどうかは、その投票をとりまとめる人の決定次第である。

どのようにしてそれを実装するか、概要を見てみよう。

### 投票権を発行する {#issuing-voting-power}

すべては投資家に対して会社の「決定ができる権利」を売りたい会社の創設者（ボスと呼ぼう）によって始まる。決定権は、このエクササイズでは「Power Coin」と呼ぶことにするが、カラードコインを形作れる。

紫色で表現しよう。

![](../assets/PowerCoin.png)

たとえば3人が興味を持ったとして、その3人をサトシ、アリスとボブとしよう（そう、再登場だ）。  
そしてボスはPower Coinを0.1BTCで売ることに決めた。

`powerCoin`アドレス、`satoshi`、`alice`そして`bob`にビットコインを与えよう。

```cs
var powerCoin = new Key();
var alice = new Key();
var bob = new Key();
var satoshi = new Key();
var init = new Transaction()
{
    Outputs = 
    {
        new TxOut(Money.Coins(1.0m), powerCoin),
        new TxOut(Money.Coins(1.0m), alice),
        new TxOut(Money.Coins(1.0m), bob),
        new TxOut(Money.Coins(1.0m), satoshi),
    }
};

var repo = new NoSqlColoredTransactionRepository();
repo.Transactions.Put(init);
```

アリスが2つPower coinを買ったことにしよう。ここにそのトランザクションの作り方を示す。

![](../assets/Power2Alice.png)

```cs
var issuance = GetCoins(init,powerCoin)
                .Select(c=> new IssuanceCoin(c))
                .ToArray();
var builder = new TransactionBuilder();
var toAlice =
    builder
    .AddCoins(issuance)
    .AddKeys(powerCoin)
    .IssueAsset(alice, new AssetMoney(powerCoin, 2))
    .SetChange(powerCoin)
    .Then()
    .AddCoins(GetCoins(init, alice))
    .AddKeys(alice)
    .Send(alice, Money.Coins(0.2m))
    .SetChange(alice)
    .BuildTransaction(true);
repo.Transactions.Put(toAlice);
```

要約すると、powerCoinは2つのPower Coinをアリスに発行し、お釣りを自身に送っている。同様にアリスは0.2BTCをpowerCoinに送り、そのお釣りを自身に返す。

**GetCoins**はこのようになっている。

```cs
private IEnumerable<Coin> GetCoins(Transaction tx, Key owner)
{
    return tx.Outputs.AsCoins().Where(c => c.ScriptPubKey == owner.ScriptPubKey);
}
```

理由があってアリスはひょっとするとサトシに投票権のいくつかを売りたくなるかもしれない。

![](../assets/PowerCoin2.png)

**init**トランザクションからアリスのコインをダブルスペンドしていることがわかるだろう。  
_\*\*これはビットコインブロックチェーンでは受け入れられないだろう。しかしまだブロックチェーンから簡単に使われていないコインを引き出す方法を見ていないから、エクササイズではコインはダブルスペンドされていなかったと仮定しよう。_

今、アリスとサトシが投票権を持っている。どのようにしてボスが投票をさせるか見てみよう。

### 投票の実施 {#running-a-vote}

ブロックチェーンを調べれば、ボスはいつでもPower Coinを持っている**ScriptPubKey**を知ることができる。  
そしてボスはPower Coinを持っている人に、その投票権に比例したVoting Coinを送る。このケーススタディーではアリスに1voting coin、サトシに1voting coin送ることとしよう。

![](../assets/PowerCoin3.png)

最初に**votingCoin**に対してビットコインを付与する必要がある。

```cs
var votingCoin = new Key();
var init2 = new Transaction()
{
    Outputs = 
    {
        new TxOut(Money.Coins(1.0m), votingCoin),
    }
};
repo.Transactions.Put(init2);
```

そしてvoting coinを発行する。

```cs
issuance = GetCoins(init2, votingCoin).Select(c => new IssuanceCoin(c)).ToArray();
builder = new TransactionBuilder();
var toVoters =
    builder
    .AddCoins(issuance)
    .AddKeys(votingCoin)
    .IssueAsset(alice, new AssetMoney(votingCoin, 1))
    .IssueAsset(satoshi, new AssetMoney(votingCoin, 1))
    .SetChange(votingCoin)
    .BuildTransaction(true);
repo.Transactions.Put(toVoters);
```

### 投票の委任 {#vote-delegation}

問題は、投票はビジネスの財務的な側面にも議題が及んでおり、アリスは主にマーケティングの側面に関心があるということだ。

アリスは、財務的な問題に関して、自分よりも良い判断をすると信頼している誰かに投票権を譲渡することに決めた。彼女はボブを委任相手に選んだ。

![](../assets/PowerCoin4.png)

```cs
var aliceVotingCoin = ColoredCoin.Find(toVoters,repo)
                        .Where(c=>c.ScriptPubKey == alice.ScriptPubKey)
                        .ToArray();
builder = new TransactionBuilder();
var toBob =
    builder
    .AddCoins(aliceVotingCoin)
    .AddKeys(alice)
    .SendAsset(bob, new AssetMoney(votingCoin, 1))
    .BuildTransaction(true);
repo.Transactions.Put(toBob);
```

**SetChange**がないことがわかるだろう。それはインプットのカラードコインが完全に使われているからで、お釣りとして戻すものがないからである。

### 投票 {#voting}

サトシがとても忙しすぎて投票しないことに決めたとしよう。またボブは自分の意思決定を表さないといけない。  
その投票では会社が銀行に対して、新しい設備投資のために負債借り入れを申し入れるかどうかに関してだ。

ボスは会社のウェブサイトでこう周知した。

賛成なら 1HZwkjkeaoZfTSaJxDw6aKkxp45agDiEzN にコインを、反対なら 1F3sAm6ZtwLAUnj7d38pGFxtP3RVEvtsbV にコインを送ること。

ボブは会社は負債を借り入れるべきだと決めた。

![](../assets/PowerCoin5.png)

```cs
var bobVotingCoin = ColoredCoin.Find(toVoters, repo)
    .Where(c => c.ScriptPubKey == bob.ScriptPubKey)
    .ToArray();

builder = new TransactionBuilder();
var vote =
    builder
    .AddCoins(bobVotingCoin)
    .AddKeys(bob)
    .SendAsset(BitcoinAddress.Create("1HZwkjkeaoZfTSaJxDw6aKkxp45agDiEzN"),
                new AssetMoney(votingCoin, 1))
    .BuildTransaction(true);
```

さてボスは投票結果を自動計算できる。その結果、賛成が1、反対が0で賛成多数のため、負債を借り入れる。  
すべての参加者が自分たちでも結果を数えられる。

### 代案：リカーディアン・コントラクトを使う {#alternative-use-of-ricardian-contract}

これまでのエクササイズでは、ボスがブロックチェーンの外の世界、つまり会社のウェブサイトで投票の方法をアナウンスした。

これはうまく機能したが、ボブはそのウェブサイトが存在していることを知る必要があった。

もう1つの解決法はブロックチェーンで、つまり**Asset Definition File**の中で直接、投票の方法を記録してしまうことだ。そうするとソフトウェアが自動的にそれを把握し、ボブにそれを提示させることができる。

それによって変わるコードはほんの少しで、Voting Coinを投票する人に発行するところだ。

```cs
issuance = GetCoins(init2, votingCoin).Select(c => new IssuanceCoin(c)).ToArray();
issuance[0].DefinitionUrl = new Uri("http://boss.com/vote01.json");
builder = new TransactionBuilder();
var toVoters =
    builder
    .AddCoins(issuance)
    .AddKeys(votingCoin)
    .IssueAsset(alice, new AssetMoney(votingCoin, 1))
    .IssueAsset(satoshi, new AssetMoney(votingCoin, 1))
    .SetChange(votingCoin)
    .BuildTransaction(true);
repo.Transactions.Put(toVoters);
```

このケースでは、ボブがvoting coinを発行する中で**Asset Definition File**が記録されていたことを把握することができる。それはスキーマが部分的に[Open Assetに明示されている](https://github.com/OpenAssets/open-assets-protocol/blob/master/asset-definition-protocol.mediawiki)JSONの他の何物でもない。スキーマは以下のような情報を持つように拡張することができる。

* 投票の期限
* 各候補に対する投票先
* Asset Definition Fileに記録されている内容の、人間の目に優しい表現

しかしハッカーが投票権を盗みたいと思っているとしよう。彼はいつもJSONのドキュメントを操作することができる（もしくは中間者が攻撃したり、ボスのウェブサイトに物理的にアクセスしたりまたはボブのソフトウェアに物理的にアクセスしたり）ことで、ボブは騙されて異なる候補に投票してしまう。

署名することによって**Asset Definition File**を**リカーディアン・コントラクト**に置き換えることで、ボブのソフトウェアがあらゆる変更をすぐに検知できるようになるだろう（Asset Definition Protocolの[Proof Of Authenticity](https://github.com/OpenAssets/open-assets-protocol/blob/master/asset-definition-protocol.mediawiki)を見てほしい）。

