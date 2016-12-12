## 秘密鍵 {#private-key}

秘密鍵はよく、ビットコインアドレスと同じBase58Check形式の、ビットコインシークレットで表現される。これはウォレットにインポートするフォーマット（**Wallet Import Format, WIF**）としても知られている。

![](../assets/BitcoinSecret.png)

```cs
Key privateKey = new Key(); // generate a random private key
BitcoinSecret mainNetPrivateKey = privateKey.GetBitcoinSecret(Network.Main);  // get our private key for the mainnet
BitcoinSecret testNetPrivateKey = privateKey.GetBitcoinSecret(Network.TestNet);  // get our private key for the testnet
Console.WriteLine(mainNetPrivateKey); // L5B67zvrndS5c71EjkrTJZ99UaoVbMUAK58GKdQUfYCpAa6jypvn
Console.WriteLine(testNetPrivateKey); // cVY5auviDh8LmYUW8AfafseD6p6uFoZrP7GjS3rzAerpRKE9Wmuz

bool WifIsBitcoinSecret = mainNetPrivateKey == privateKey.GetWif(Network.Main);
Console.WriteLine(WifIsBitcoinSecret); // True
```

注目してほしいのだが、秘密鍵からビットコインシークレットを導出するのは簡単だ。一方で、ビットコインアドレスから公開鍵を導くことはできない。なぜならば、ビットコインアドレスは公開鍵ハッシュを含んでいるのであって、公開鍵自体を含んではないからだ。

> 日本語版注：公開鍵ハッシュを含んでいて公開鍵に戻すことができないのはハッシュの特性である。

次の2つのコードブロックの類似性を見ることで、上記を体験してみよう。

```cs
Key privateKey = new Key(); // generate a random private key
BitcoinSecret bitcoinSecret = privateKey.GetWif(Network.Main); // L5B67zvrndS5c71EjkrTJZ99UaoVbMUAK58GKdQUfYCpAa6jypvn
Key samePrivateKey = bitcoinSecret.PrivateKey;
Console.WriteLine(samePrivateKey == privateKey); // True
```

```cs
PubKey publicKey = privateKey.PubKey;
BitcoinPubKeyAddress bitcoinPubicKey = publicKey.GetAddress(Network.Main); // 1PUYsjwfNmX64wS368ZR5FMouTtUmvtmTY
//PubKey samePublicKey = bitcoinPubicKey.ItIsNotPossible;
```

### Exercise : 

1. 本番環境向けの秘密鍵を生成し、それをメモする。
2. 秘密鍵からビットコインアドレスを生成する。
3. そのビットコインアドレスにビットコインを送る。失うことができないほどの量であれば、次のレッスンでそれらを取り戻すために集中できるだろう。 



