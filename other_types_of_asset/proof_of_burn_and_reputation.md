## Proof of Burn と評判 {#proof-of-burn-and-reputation}

ここでの問題は単純だ。P2Pのマーケットでは法的な執行はあまりにもコストが高くつきすぎるが、どのようにして参加者が詐欺にあう可能性を最小化できるだろうか。

OpenBaazarは決定力のある評価としてproof of burnを使おうとしている[先駆者](https://gist.github.com/dionyziz/e3b296861175e0ebea4b)だ。

詐欺対策はいくつかの方法（エスクロー、公証人/仲裁人）があるが、ここで探求するのはProof Of Burnと呼ばれるものだ。

自分が、中世で何人かの地元の商人と一緒に小さい村に住んでいることを想像してほしい。  
ある日、旅人の商人があなたの村に来て、地元の価格と比べると信じられないくらい安い価格でいくつか商品を売ろうとしている。

しかしその旅人の商品は、悪質な商品で人を騙す手口に精通していた。なぜならば土着の商人と比べて、評価を失うことは旅人の商品にとっては取るに足らない代償だからだ。  
土着の商人は素晴らしい店、広告とその評価に投資していた。不幸な顧客は簡単にそれを壊すことができる。しかし旅人の商人は土着の店を持っていないし、一時的な評価なんぞ、人を騙さないようにするきっかけにはまったくならない。

インターネットでは、アイデンティティを作ることはとてもたやすく、すべての商人は中世以降の旅人の商人になりえる。  
マーケットの提供者の解決策は、マーケットにおけるすべての参加者の実際のアイデンティティを集め、そして法的執行を可能とすることだ。

もしAmazonやEbayで詐欺にあったら、銀行が結構な確率であなたにお金を戻してくれることだろう。なぜならばAmazonやEbayに連絡することで泥棒を見つける方法があるからだ。

単純なP2Pのマーケットでビットコインを使うとき、泥棒を見つけることはできない。もし詐欺にあうと、お金を失ってしまう。  
ということで、どのようにして買い手は旅人の商人を信じることができるだろうか？  
その答えは、その人が評価に対してどの程度投資をしているかをチェックすることだ。

善良な売り手なら、その自信を顧客に示したいと思うだろう。そのためには富を削るだろうし、すべての顧客はそれを見るだろう。これが「評価に投資する」ことの定義だ。

あなたが評価のために50BTCを使ったとしよう。そして顧客はあなたから2BTCの商品を買いたいとする。顧客はあなたが自分を騙さないと信じる良い理由がある。なぜならばあなたが、詐欺を働くことによって彼からだまし取ることができるもの以上に、評価に対して投資しているからだ。  
あなたが顧客を騙すことは経済的に有益ではないということになる。

技術的な詳細はきっと時間がたつにつれてバリエーションが増えたり変わったりするだろうが、ここにはProof of Burnの例を示す。

```cs
var alice = new Key();

//Giving some money to alice
var init = new Transaction()
{
    Outputs = 
    {
        new TxOut(Money.Coins(1.0m), alice),
    }
};

var coin = init.Outputs.AsCoins().First();

//Burning the coin
var burn = new Transaction();
burn.Inputs.Add(new TxIn(coin.Outpoint)
{
    ScriptSig = coin.ScriptPubKey
}); //Spend the previous coin

var message = "Burnt for \"Alice Bakery\"";
var opReturn = TxNullDataTemplate
                .Instance
                .GenerateScriptPubKey(Encoding.UTF8.GetBytes(message));
burn.Outputs.Add(new TxOut(Money.Coins(1.0m), opReturn));
burn.Sign(alice, false);

Console.WriteLine(burn);
```

```json
{
  ….
  "in": [
    {
      "prev_out": {
        "hash": "0767b76406dbaa95cc12d8196196a9e476c81dd328a07b30954d8de256aa1e9f",
        "n": 0
      },
      "scriptSig": "304402202c6897714c69b3f794e730e94dd0110c4b15461e221324b5a78316f97c4dffab0220742c811d62e853dea433e97a4c0ca44e96a0358c9ef950387354fbc24b8964fb01 03fedc2f6458fef30c56cafd71c72a73a9ebfb2125299d8dc6447fdd12ee55a52c"
    }
  ],
  "out": [
    {
      "value": "1.00000000",
      "scriptPubKey": "OP_RETURN 4275726e7420666f722022416c6963652042616b65727922"
    }
  ]
}
```

ビットコインブロックチェーンで一度、このトランザクションが作成されるだけで、アリスが自分のベーカリーにお金を投資したという否定できない証明となる。  
`ScriptPubKey OP_RETURN 4275726e7420666f722022416c6963652042616b65727922`を伴うコインはいかなる方法でも使えないようになっており、これらのコインは永遠に失われる。

