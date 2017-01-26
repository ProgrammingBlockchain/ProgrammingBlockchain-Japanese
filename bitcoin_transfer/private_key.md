## 秘密鍵 {#private-key}

秘密鍵はよく、ビットコインアドレスと同じBase58Check形式の、**ビットコインシークレット**で表現される。これはウォレットにインポートするフォーマット（**Wallet Import Format, WIF**）としても知られている。

![](../assets/BitcoinSecret.png)

```cs
Key privateKey = new Key(); // ランダムな秘密鍵の生成
BitcoinSecret mainNetPrivateKey = privateKey.GetBitcoinSecret(Network.Main);  // メインネット用のビットコインシークレットを取得
BitcoinSecret testNetPrivateKey = privateKey.GetBitcoinSecret(Network.TestNet);  // テストネット用のビットコインシークレットを取得
Console.WriteLine(mainNetPrivateKey); // L5B67zvrndS5c71EjkrTJZ99UaoVbMUAK58GKdQUfYCpAa6jypvn
Console.WriteLine(testNetPrivateKey); // cVY5auviDh8LmYUW8AfafseD6p6uFoZrP7GjS3rzAerpRKE9Wmuz

bool WifIsBitcoinSecret = mainNetPrivateKey == privateKey.GetWif(Network.Main);
Console.WriteLine(WifIsBitcoinSecret); // 正
```

注目してほしいのだが、**ビットコインシークレット** から **秘密鍵** を導出するのは簡単だ。一方で、ビットコインアドレスから公開鍵を導くことはできない。なぜならば、ビットコインアドレスは公開鍵のハッシュを含んでいるのであって、公開鍵自体を含んではないからだ。  
次の2つのコードブロックの比較することで、上記の違いを理解してみよう。

```cs
Key privateKey = new Key(); // ランダムな秘密鍵の生成
BitcoinSecret bitcoinSecret = privateKey.GetWif(Network.Main); // L5B67zvrndS5c71EjkrTJZ99UaoVbMUAK58GKdQUfYCpAa6jypvn
Key samePrivateKey = bitcoinSecret.PrivateKey;
Console.WriteLine(samePrivateKey == privateKey); // 正
```

```cs
PubKey publicKey = privateKey.PubKey;
BitcoinPubKeyAddress bitcoinPubicKey = publicKey.GetAddress(Network.Main); // 1PUYsjwfNmX64wS368ZR5FMouTtUmvtmTY
//PubKey samePublicKey = bitcoinPubicKey.ItIsNotPossible;// 戻すのは不可能
```

### 練習 :

1. MainNet向けの秘密鍵を生成し、それをメモする。
2. 秘密鍵からビットコインアドレスを生成する。
3. そのビットコインアドレスにビットコインを送る。無くすには惜しいくらいの額を送金すれば、次のレッスンでそれらを取り戻すために集中し、モチベーションを持続できるだろう。
