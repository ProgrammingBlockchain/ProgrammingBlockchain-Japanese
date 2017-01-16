## ビットコインアドレス {#bitcoin-address}

**ビットコインアドレス**が、支払いを受けるために公表するものであることは知ってのとおり。  
![](../assets/BitcoinAddress.png)  
このアドレス宛てにあなたがもらったビットコインを使うために、ウォレットが**秘密鍵**を使うということも、たぶん知っているだろう。  
![](../assets/PrivateKey.png)

秘密鍵はネットワークには保管されず、インターネットにアクセスすることなく生成される。

秘密鍵を生成してみよう。

```cs
Key privateKey = new Key(); // generate a random private key
```

秘密鍵から**公開鍵**を作成する。これは一方向のみの生成となっていて、逆に公開鍵から秘密鍵は作れない。

![](../assets/PrivKeyPubKey.png)

```cs
PubKey publicKey = privateKey.PubKey;
Console.WriteLine(publicKey); // 0251036303164f6c458e9f7abecb4e55e5ce9ec2b2f1d06d633c9653a07976560c
```

ビットコインには2つの**ネットワーク**がある。

* **TestNet** は開発環境を目的としたビットコインのネットワーク。このネットワーク上のビットコインに価値はない。
* **MainNet** はユーザー含めてあらゆる人が使うネットワークである。

> 注釈：TestNetのコインは**faucets**を使うことですぐに手に入る。「get testnet bitcoins」で検索してほしい。

公開鍵と生成されるアドレスが使われるネットワークから、ビットコインアドレスは簡単に作れる。

![](../assets/PubKeyToAddr.png)

```cs
Console.WriteLine(publicKey.GetAddress(Network.Main)); // 1PUYsjwfNmX64wS368ZR5FMouTtUmvtmTY
Console.WriteLine(publicKey.GetAddress(Network.TestNet)); // n3zWAo2eBnxLr3ueohXnuAa8mTVBhxmPhq
```

**正確に言うと、ビットコインアドレスは先頭1バイト（TestNetかMainNetかで異なる）と公開鍵ハッシュで構成されており、それらが結合され、Base58Checkにエンコードされる。**

![](../assets/PubKeyHashToBitcoinAddress.png)

```cs
var publicKeyHash = publicKey.Hash;
Console.WriteLine(publicKeyHash); // f6889b21b5540353a29ed18c45ea0031280c42cf
var mainNetAddress = publicKeyHash.GetAddress(Network.Main);
var testNetAddress = publicKeyHash.GetAddress(Network.TestNet);
```

> 注釈：公開鍵ハッシュは公開鍵をSHA256でハッシュ化し、その結果に対してRIPEMD160でハッシュ化してビッグエンディアンで並べている。「RIPEMD160\(SHA256\(pubkey\)\)」のように表せるだろう。

Base58Checkは誤字を防止するためのチェックサムがあったり、「0（ゼロ）」や「O（オー）」のようにどちらとも見える文字を使っていない点で均整が取れている。

Base58Checkエンコードされたビットコインアドレスは、ビットコインのウォレットを使っているユーザーが、意図していないネットワークにビットコインを送ってしまわないように確認できるようになっている。

```cs
Console.WriteLine(mainNetAddress); // 1PUYsjwfNmX64wS368ZR5FMouTtUmvtmTY
Console.WriteLine(testNetAddress); // n3zWAo2eBnxLr3ueohXnuAa8mTVBhxmPhq
```

> 参考：MainNetで実践的にビットコインプログラミングをすることで、ミスしたときに二度と同じミスをしないように気をつけられる。



