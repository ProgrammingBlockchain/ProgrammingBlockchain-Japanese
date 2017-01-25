## ビットコインアドレス {#bitcoin-address}

**ビットコインアドレス**が、支払いを受けるために公開するものであることは知ってのとおりだ。  

![](../assets/BitcoinAddress.png)  

このアドレス宛てにあなたがもらったビットコインを使うために、ウォレットが**秘密鍵**を使うということも、たぶん知っているだろう。  
![](../assets/PrivateKey.png)

秘密鍵はネットワークには保管されず、インターネットにアクセスすることなく生成することができる。

秘密鍵を生成してみよう。

```cs
Key privateKey = new Key(); // ランダムな秘密鍵を生成
```

秘密鍵から**公開鍵**を作成する。これは一方向のみの生成となっていて、逆に公開鍵から秘密鍵は作れない。

![](../assets/PrivKeyPubKey.png)

```cs
PubKey publicKey = privateKey.PubKey;
Console.WriteLine(publicKey); // 0251036303164f6c458e9f7abecb4e55e5ce9ec2b2f1d06d633c9653a07976560c
```

ビットコインには2つの**ネットワーク**がある。

* **TestNet** は開発を目的としたビットコインのネットワーク。このネットワーク上のビットコインに価値はない。
* **MainNet** は普通のユーザーが使うネットワークである。

> 注釈：TestNetのコインは**faucets（蛇口）**サイトを使うことですぐに手に入る。「get testnet bitcoins」で検索してほしい。

公開鍵とアドレスを使用するネットワークの種類から、ビットコインアドレスを簡単に作ることができる。

![](../assets/PubKeyToAddr.png)

```cs
Console.WriteLine(publicKey.GetAddress(Network.Main)); // 1PUYsjwfNmX64wS368ZR5FMouTtUmvtmTY
Console.WriteLine(publicKey.GetAddress(Network.TestNet)); // n3zWAo2eBnxLr3ueohXnuAa8mTVBhxmPhq
```

**正確に言うと、ビットコインアドレスはバージョンを示す先頭1バイト（TestNetかMainNetかで異なる）と公開鍵ハッシュで構成されており、それらが結合された後、Base58Checkにエンコードされる。**

![](../assets/PubKeyHashToBitcoinAddress.png)

```cs
var publicKeyHash = publicKey.Hash;
Console.WriteLine(publicKeyHash); // f6889b21b5540353a29ed18c45ea0031280c42cf
var mainNetAddress = publicKeyHash.GetAddress(Network.Main);
var testNetAddress = publicKeyHash.GetAddress(Network.TestNet);
```

> 注釈：公開鍵ハッシュは、まず公開鍵をSHA256でハッシュ化し、その結果に対してRIPEMD160でさらにハッシュ化してビッグエンディアンで並べている。プログラミング関数なら、「RIPEMD160\(SHA256\(pubkey\)\)」のように表せるだろう。

Base58Checkエンコーディングには誤字を防止するためのチェックサムがあったり、「0（ゼロ）」や「O（オー）」のようにどちらとも見える文字を使わないようにするという気の利いた機能がある。  
Base58Checkエンコードされたビットコインアドレスは、ウォレットを使っているユーザーが、間違えて意図していないネットワークにビットコインを確実に送らないようにする。

```cs
Console.WriteLine(mainNetAddress); // 1PUYsjwfNmX64wS368ZR5FMouTtUmvtmTY
Console.WriteLine(testNetAddress); // n3zWAo2eBnxLr3ueohXnuAa8mTVBhxmPhq
```

> 参考：MainNetでビットコイン プログラミングを練習すれば、起こしたミスは、忘れられないものになるだろう。
