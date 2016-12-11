## ScriptPubKey {#payment-script}

ひょっとするとビットコインブロックチェーンに関する限りは、ビットコインアドレスなどというものはないとご存知かもしれない。内部的には、ビットコインのプロトコルではビットコインを受け取る人を**ScriptPubKey**で認識することになっている。

![](../assets/ScriptPubKey.png)  
**ScriptPubKey**とはこのように表現される。  
`OP_DUP OP_HASH160 14836dbe7f38c5ac3d49e8d790af808a4ee9edcf OP_EQUALVERIFY OP_CHECKSIG`

これはビットコインの所有権を主張するためにどんな条件が揃わなければならないかを説明する、短いスクリプトである。この本のレッスンを進めていく中で、スクリプトが示す命令をいくつか見ることになる。

ビットコインアドレスからScriptPubKeyを生成することができる。これはすべてのビットコインのクライアントが、「人間の目に易しい」ビットコインアドレスから、ビットコインブロックチェーンが読み込むことのできるアドレスへ変換するために行っているステップの1つである。

![](../assets/BitcoinAddressToScriptPubKey.png)

```cs
var publicKeyHash = new KeyId("14836dbe7f38c5ac3d49e8d790af808a4ee9edcf");

var testNetAddress = publicKeyHash.GetAddress(Network.TestNet);
var mainNetAddress = publicKeyHash.GetAddress(Network.Main);

Console.WriteLine(mainNetAddress.ScriptPubKey); // OP_DUP OP_HASH160 14836dbe7f38c5ac3d49e8d790af808a4ee9edcf OP_EQUALVERIFY OP_CHECKSIG
Console.WriteLine(testNetAddress.ScriptPubKey); // OP_DUP OP_HASH160 14836dbe7f38c5ac3d49e8d790af808a4ee9edcf OP_EQUALVERIFY OP_CHECKSIG
```

注目してほしいのだが、開発環境の**ScriptPubKey**と本番環境の**ScriptPubKey**が同じになっている。

もう1つ注目してほしいのだが、**ScriptPubKey**には公開鍵ハッシュが含まれている。

この時点では詳細には触れないでおくが、ScriptPubKeyはビットコインアドレスと全く関係がないように見えるが、公開鍵ハッシュを表しているのだ。

ビットコインアドレスはアドレスが使われる環境を指し示す1バイトと公開鍵で構成されているのであった。だから**ScriptPubKey**と環境を指し示す情報からビットコインアドレスに戻すことができる。

```cs
var paymentScript = publicKeyHash.ScriptPubKey;
var sameMainNetAddress = paymentScript.GetDestinationAddress(Network.Main);
Console.WriteLine(mainNetAddress == sameMainNetAddress); // True
```

または**ScriptPubKey**から公開鍵ハッシュに戻し、さらにそれからビットコインアドレスを生成することができる。

```cs
var samePublicKeyHash = (KeyId) paymentScript.GetDestination();
Console.WriteLine(publicKeyHash == samePublicKeyHash); // True
var sameMainNetAddress2 = new BitcoinPubKeyAddress(samePublicKeyHash, Network.Main);
Console.WriteLine(mainNetAddress == sameMainNetAddress2); // True
```

> 注釈：ScriptPubKeyは必ずしもビットコインを使うことが許されている公開鍵ハッシュを含まなくても良い。

さあ今、秘密鍵、公開鍵、公開鍵ハッシュ、ビットコインアドレスとScriptPubKeyの関係性を理解したと思う。

次の章からはScriptPubKeyだけを使う。ビットコインアドレスはユーザーインターフェースの発想から使われるものだからだ。

