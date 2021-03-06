## 所有権の証明による個人の認証 {#proof-of-ownership-as-an-authentication-method}

> \[[2016.05.02](https://www.youtube.com/watch?v=dZNtbAFnr-0)\] 私の名前はクレイグ・ライトだ。これからビットコインで最初に作成されたトランザクションに関連付けられた公開鍵を使ってメッセージに署名することをデモンストレーションしよう。

```cs
var bitcoinPrivateKey = new BitcoinSecret("XXXXXXXXXXXXXXXXXXXXXXXXXX");

var message = "I am Craig Wright";
string signature = bitcoinPrivateKey.PrivateKey.SignMessage(message);
Console.WriteLine(signature); // IN5v9+3HGW1q71OqQ1boSZTm0/DCiMpI8E4JB1nD67TCbIVMRk/e3KrTT9GvOuu3NGN0w8R2lWOV2cxnBp+Of8c=
```

これはそんなに難しいことだろうか？

クレイグ・ライトという人物を覚えているかもしれない。彼は自分がサトシ・ナカモトであると,是非とも信じてほしいと思っていた人物だ。  
彼はあるソーシャルエンジニアリングの方法を使って、一握りの影響力のあるビットコイン業界の人物やジャーナリストを納得させることに成功した。  
しかし、幸運なことにデジタル署名に関しては、そんなふうにうまくいかない。  
ジェネシスブロックにある、最初のビットコイントランザクションのアドレスを[インターネット](https://en.bitcoin.it/wiki/Genesis_block)でさっと見てみよう。[1A1zP1eP5QGefi2DMPTfTL5SLmv7DivfNa](https://blockchain.info/address/1A1zP1eP5QGefi2DMPTfTL5SLmv7DivfNa) とある。さあ彼の主張を確かめてみよう。

```cs
var message = "I am Craig Wright";
var signature = "IN5v9+3HGW1q71OqQ1boSZTm0/DCiMpI8E4JB1nD67TCbIVMRk/e3KrTT9GvOuu3NGN0w8R2lWOV2cxnBp+Of8c=";

var address = new BitcoinPubKeyAddress("1A1zP1eP5QGefi2DMPTfTL5SLmv7DivfNa");
bool isCraigWrightSatoshi = address.VerifyMessage(message, signature);

Console.WriteLine("Is Craig Wright Satoshi? " + isCraigWrightSatoshi);
```

ネタバレ注意！結果のブーリアンの値はfalseになるだろう。

ここにコインを移動させないで、どうやって特定のアドレスを自分のものだと証明できるかの方法を示そう。

**ビットコインアドレス：**  
[1KF8kUVHK42XzgcmJF4Lxz4wcL5WDL97PB](https://blockchain.info/address/1KF8kUVHK42XzgcmJF4Lxz4wcL5WDL97PB)  
**メッセージ：**  
Nicolas Dorier Book Funding Address  
**署名：**  
H1jiXPzun3rXi0N9v9R5fAWrfEae9WPmlL5DJBj1eTStSvpKdRR8Io6/uT9tGH/3OnzG6ym5yytuWoA9ahkC3dQ=

これらの情報でNicolas Dorierがこの本の秘密鍵を所有していることの証明ができる。  
**Exercise：**Nicolas先生がうそをついていないことを確かめてみよう！

### 追記

PGPがどのように動いているか知っているだろうか？ビットコインブロックチェーンととてもよく似ていないだろうか？  
たぶんビットコインブロックチェーンはよりユーザーフレンドリーな、PGPの代わりとしての基盤となり得るだろう。  
どうかNBitcoinでそれを作って欲しい。:-\)
