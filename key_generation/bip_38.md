## BIP38 (パート2) {#bip38-part-2}

秘密鍵を暗号化するためにBIP38をすでに見てきたが、実はこのBIPは2つのアイディアを1つにしたものだ。

2つ目については、どうやって鍵とアドレスの生成を、信頼できない相手に委譲することができるかを書いている。なので、2つの懸念点のうちの1つを解決する。

**そのアイディアとは、鍵生成をする人へ渡すパスフレーズコードを作るというものだ。このパスフレーズコードで、その相手はあなたのために、暗号化された鍵を作ることができ、しかも、あなたのパスワードも秘密鍵も知ることはない。**

この **パスフレーズコード** は WIFフォーマットにして鍵生成する人に渡すことができる。

> **注釈**: NBitcoinでは、Bitcoinが頭についている型は全て Base58 （WIF）フォーマットのデータである。

なので、鍵の生成を委譲したいユーザーとして最初に、 **パスフレーズコード** を作ることになる。

![](../assets/PassphraseCode.png)

```cs
var passphraseCode = new BitcoinPassphraseCode("my secret", Network.Main, null);
```

**そしてこのパスフレーズコードを鍵を生成するサードパーティに渡す。**

そのサードパーティは、暗号化された鍵をあなたに作ってくれる。

![](../assets/PassphraseCodeToEncryptedKeys.png)

```cs
EncryptedKeyResult encryptedKeyResult = passphraseCode.GenerateEncryptedSecret();
```

この **暗号化された鍵** は、たくさんの情報を持っている。

![](../assets/EncryptedKeyResult.png)

まずは、生成されたビットコインアドレス。

```cs
var generatedAddress = encryptedKeyResult.GeneratedAddress; // 14KZsAVLwafhttaykXxCZt95HqadPXuz73
```

それから、暗号化された鍵自体（前段で述べた **鍵の暗号化** のレッスンでみたように）。

```cs
var encryptedKey = encryptedKeyResult.EncryptedKey; // 6PnWtBokjVKMjuSQit1h1Ph6rLMSFz2n4u3bjPJH1JMcp1WHqVSfr5ebNS
```

そして、最後だけれども重要である **確認コード**。これにより、その第三者は、生成された鍵とアドレスが あなたのパスワードに対応していることを証明することができる。

```cs
var confirmationCode = encryptedKeyResult.ConfirmationCode; // cfrm38VUcrdt2zf1dCgf4e8gPNJJxnhJSdxYg6STRAEs7QuAuLJmT5W7uNqj88hzh9bBnU9GFkN
```

あなたは所有者として、この情報を手に入れたらすぐ、**ConfirmationCode.Check** を使って、鍵作成者がインチキをしていないか確認する必要がある。そして、パスワードを使って秘密鍵を取得する。

```cs
Console.WriteLine(confirmationCode.Check("my secret", generatedAddress)); // True
var bitcoinPrivateKey = encryptedKey.GetSecret("my secret");
Console.WriteLine(bitcoinPrivateKey.GetAddress() == generatedAddress); // True
Console.WriteLine(bitcoinPrivateKey); // KzzHhrkr39a7upeqHzYNNeJuaf1SVDBpxdFDuMvFKbFhcBytDF1R
```

我々は、第三者がどうやってあなたのパスワードと秘密鍵を知ることなしに暗号化された鍵を作成することができるかを見てきた。

![](../assets/ThirdPartyKeyGeneration.png)

しかし、まだ1つ問題が残っている:

* 新しい鍵を生成することで、自分の持つウォレットのバックアップがすべて古いものになってしまう。

BIP 32、もしくは 階層的決定性ウォレット(HDウォレット)は、別の解決方法を提案しており、その方がより広くサポートされた方法だ。
