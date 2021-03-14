# ScriptPubKey {#payment-script}

ひょっとすると知らないかもしれないが、実は、ビットコインのブロックチェーンの内部には、ビットコインアドレスなどというものはない。内部的に、ビットコインのプロトコルでは、ビットコインを受け取る相手を**ScriptPubKey**で認識することになっている。

![](../assets/ScriptPubKey.png)
**ScriptPubKey**は、例えば、このように表現される。
`OP_DUP OP_HASH160 14836dbe7f38c5ac3d49e8d790af808a4ee9edcf OP_EQUALVERIFY OP_CHECKSIG`

この短いスクリプトこそ、ビットコインの所有権を主張するために、どんな条件が揃わなければならないかを説明するものだ。この本のレッスンを進めていく中で、**ScriptPubKey**の中にある命令をいくつか見ることになる。

ScriptPubKeyはビットコインアドレスから生成することができる。これはすべてのビットコインのクライアントが、「人間の目に易しい」ビットコインアドレスから、ブロックチェーンが理解できるアドレスへ変換するために行っているステップの1つである。

![](../assets/BitcoinAddressToScriptPubKey.png)

```cs
var publicKeyHash = new KeyId("14836dbe7f38c5ac3d49e8d790af808a4ee9edcf");

var testNetAddress = publicKeyHash.GetAddress(Network.TestNet);
var mainNetAddress = publicKeyHash.GetAddress(Network.Main);

Console.WriteLine(mainNetAddress.ScriptPubKey); // OP_DUP OP_HASH160 14836dbe7f38c5ac3d49e8d790af808a4ee9edcf OP_EQUALVERIFY OP_CHECKSIG
Console.WriteLine(testNetAddress.ScriptPubKey); // OP_DUP OP_HASH160 14836dbe7f38c5ac3d49e8d790af808a4ee9edcf OP_EQUALVERIFY OP_CHECKSIG
```

TestNetの**ScriptPubKey**とMainNetの**ScriptPubKey**が同じになっているのに気づいただろうか？
そして、**ScriptPubKey**には公開鍵ハッシュが含まれているのに気づいただろうか？
この時点ではまだ詳細には触れないでおく。しかし、ScriptPubKeyはビットコインアドレスと全く関係がないように見えるが、公開鍵ハッシュを持っているのだ。

ビットコインアドレスはそのアドレスが使われるネットワークを指し示す1バイトのバージョンバイトと公開鍵のハッシュ値で構成されている。だから**ScriptPubKey**の情報とネットワークを指し示す情報からビットコインアドレスに戻すことができる。

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

> 注釈：ScriptPubKeyには必ずしもビットコインを使う事ができる公開鍵ハッシュを含む必要があるというわけではなく、別の種類のスクリプトを入れてもよい。

さてここまでで、秘密鍵、公開鍵、公開鍵ハッシュ、ビットコインアドレスとScriptPubKeyの関係性を理解したと思う。

次の章からは基本的にScriptPubKeyだけを使う。ビットコインアドレスはユーザーインターフェースの発想から使われるものだからだ。
