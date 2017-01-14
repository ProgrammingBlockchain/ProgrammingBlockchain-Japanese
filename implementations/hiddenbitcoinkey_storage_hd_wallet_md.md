# HiddenBitcoin: Managing keys \(HD wallet\) {#hiddenbitcoin-keystorage}

\([nopara73](https://github.com/nopara73)\)私は[HiddenWallet](https://github.com/nopara73/HiddenWallet)というプライバシー重視のビットコインウォレットを開発している。[HiddenBitcoin](https://github.com/nopara73/HiddenWallet)のライブラリーはNBitcoinとユーザーインターフェースの間の抽象化レイヤーの導入となっている。

ビットコインウォレットには3つの重要な機能があり、このケーススタディーはその3つで構成するつもりだ。

1. 安全に鍵を保管し、鍵へのアクセスを管理する
2. ビットコインブロックチェーン上のこれらの鍵や他の鍵を監視する
3. トランザクションを生成し、それらをブロードキャストする

このレッスンでは鍵のストレージ機能に取り組むつもりだ。  
もしより広くコードを確かめてみたければ、[GitHub](https://github.com/nopara73/HiddenBitcoin)にソリューションがあるので見てほしい。  
もしただ早くウォレットをセットアップして使う方法を知りたければ、[CodeProject](https://www.codeproject.com/articles/1096320/hiddenbitcoin-high-level-csharp-bitcoin-wallet-lib)によりハイレベルなチュートリアルがある。

**どのくらいハイレベルか？**私見では、GUIディベロッパーやデザイナーでもそんなに多くのミスは起こさないはずだ。彼らはインプット、アウトプットやscriptPubKeyについて知らないかもしれない。ただ彼らはビットコインアドレス、秘密鍵とウォレットのレベルはしっかり理解しているはずだ。NBitcoinも十分に抽象化されるようにしている。

## 鍵を保管するデザインの決定

秘密鍵を保管するときになにを決めなければならないかとそれらを作るときになにを気に留めるべきかについての、テンプレートを提示するのにとても良いタイミングだ。

### 1つの鍵しか使わないケース

これが手っ取り早い。僕がこの道を行くのが正しいと思うシチュエーションはそんなに多くはない。しかし、あなたのニーズに最も適していることもありえなくはないだろう。  
悪い例としてここに私が作った、1つしか鍵を使わないビットコインウォレットのイラストレーションを示そう。結論を考えるために残しておこう。

![](../assets/TransparentWallet2.png)

### JBOKウォレット

JBOKウォレットはひと束の鍵（**J**ust a **B**unch **O**f **K**eys）をまとめるものだ。リファレンスを書いている時点で、鍵を保管するためにクライアントでこのメソッドを使っている。  
これの問題はユーザーが定期的にバックアップをとらなければならないということだ。しかしもし鍵をインポートしたり、鍵を紙にメモしたり、パスワードを変えたりすることをできるようにしたいなら、この機能を使うか、この機能と決定性ウォレットをあわせて使うようなことをする必要がある。私はこれを使わないと決めた。というのは私のHiddenWalletをプライバシー向上のために改良しようとしていたということと、これではなくてより良いウォレット構造を持ち得るからだ。

### BIP38 \(Part 2\) - 信頼できないサードパーティーの鍵生成

何度も言うことになるが、これは鍵の生成機構のためにパスフレーズを生成するという思想だ。パスフレーズを使えば、パスワードや知らなかったりまったく秘密鍵を知らなかったりしても、暗号化された鍵を生成することができる。  
HiddenWalletはデスクトップウォレットだ（そしておそらくしばらく変えるつもりはない）。ということで鍵の生成や鍵を保管する目的で信頼できないサードパーティーを使う必要がない。だからまだこの機能は実装しないことに決めた。

### SHDウォレット

これは私が組み込んだウォレット構造だ。そう、降参だ。この言葉は私がたった今思いついた言葉だ。この言葉は標準的な言葉としては存在していないし誰も使っていない。では私の考えの中でこれは何を表しているかというと、不可視な階層的決定性ウォレットだ（Stealth and Hierarchical Deterministic wallet）。これが私が作成したものを表現するのに最適な言葉だ。  
コードに入っていく前に不可視にする機能だけ実装したことを注釈しておきたい。なぜならばそれが低いところに成っているフルーツだからだ。ステルスアドレスがビットコイン業界において将来的にどんな使用法にしても使われるかどうかは私は確信を持てていない。

ステルスアドレスとはこのようなものだ。 `waPXAvDCDGv8sXYRY6XDEymDGscYeepXBV5tgSDF1JHn61rzNk4EXTuBfx22J2W9rPAszXFmPXwD2m52psYhXQe5Yu1cG26A7hkPxs`

## ブラックボックス

**Safe**というクラスを実装した。ブラックボックスとしてこのクラスは直感的に使える。

```cs
var network = Network.MainNet;
```

この**Network**は**NBitcoin.Network**ではない。というのはGUIディベロッパーはNBitcoinを使っているとは知らないはずだからだ。また、NBitcoinではより多くのネットワークの選択肢があるが、HiddenBitcoinではそれらを扱うことはできない。現時点でHiddenBitcoinは`MainNet`と`TestNet`をサポートしている。  
ネットワークはenumとなっていて、**HiddenBitcoin.DataClasses**のネームスペースで見つけることができる。

```cs
string mnemonic;
Safe safe = Safe.Create(out mnemonic, "password", walletFilePath: @"Wallets\hiddenWallet.hid", network);
Console.WriteLine(mnemonic);
```

また、safeクラスをロードしたり復元したりすることができる。

```cs
Safe loadedSafe = Safe.Load("password", walletFilePath: @"Wallets\hiddenWallet.hid");
if (network != loadedSafe.Network)
    throw new Exception("WrongNetwork");

Safe recoveredSafe = Safe.Recover(mnemonic, "password", walletFilePath: @"Wallets\sameHiddenWallet.hid", network);
```

safeクラスから文字列で鍵を取得することもできる。

```cs
Console.WriteLine("Seed private key: " + safe.Seed);
Console.WriteLine("Seed public key: " + safe.SeedPublicKey);
Console.WriteLine("Third child address: " + safe.GetAddress(2));
Console.WriteLine("First child private key: " + safe.GetPrivateKey(0));
Console.WriteLine("Second child private key and the corresponding address: ");
Console.WriteLine(safe.GetPrivateKeyAddressPair(1).PrivateKey);
Console.WriteLine(safe.GetPrivateKeyAddressPair(1).Address);
Console.WriteLine("The stealth address: " + safe.StealthAddress);
Console.WriteLine("Scan and spend private keys for stealth payments:");
Console.WriteLine(loadedSafe.ScanPrivateKey);
Console.WriteLine(loadedSafe.SpendPrivateKey);
```

```
Seed private key: xprv9s21ZrQH143K4RBm26TMm3qwTtR3Eyh22xDEN3TBebgfAvHPPSjxxFnFGDtnNHvqZ7pihGmAc8o9y1UvfEzcxSzyXAnmvTBowCNi69nXsqJ
Seed public key: xpub661MyMwAqRbcGuGE87zN8Bng1vFXeSQsQB8qARroCwDe3icXvz4DW46j7U6fX8NsKhqcxR7K1mDX4gTbtvCGdeJz5M7py3yEqMsjUH2DYhb
Third child address: 17pGpPX1A2sCdqJXsC5BiwdFphFVgJR9nk
First child private key: xprv9ubnoo3dgCYfrWbYBEM71WoBvzwTtQemEdjW836CeWJYunYBskQhq3nrJMvNBCCFpnU5GbgbL1b2QbPHA4rRPESEhqfKzae5oWe7SAMuxAV
Second child private key and the corresponding address:
xprv9ubnoo3dgCYfuE1hVB3F3Sh5YFJUNUjyZ68PDzPNhpmtqWDtD45zucZYMUAjY22HNxaY6tsvGAdJdcyALCMm2mTAvA4pEp1m7y3BSccKY4r
19FHdsj2YT79TuxbWcDMz9opTU28L1memr
The stealth address: vJmuFuLggpgzivm3UUjQguLhMA6C1SnYFJu5N6QkmXYRCU3nG1Ww36VcXy6zXpJvGeVTidxcsu7U19sfB1rxHhzvSNV5eGGLk6G1Cb
Scan and spend private keys for stealth payments:
L5CTS4U27umRfSBu2ztxsyUeMEYzJJk3HvCp3deSQBJWmRSUqCLg
KyXveppF4Xm3KwJgG7EBSi6SxfMTkaDXYYmv7c7xWRcF7yUNpswp
```

**注釈**：理想的にはシードは決して使わないこと。safeクラスの鍵を生成するメソッド（後述）を使って鍵生成を繰り返すことを始めるほうがよりよいプラクティスだ。

## ホワイトボックス

### Safe.Create

```cs
// Creates a mnemonic, a seed, encrypts it and stores in the specified path.
public static Safe Create(out string mnemonic, string password, string walletFilePath, Network network)
{
    var safe = new Safe(password, walletFilePath, network);
    mnemonic = safe.SetSeed(password).ToString();
    safe.Save(password, walletFilePath, network);

    return safe;
}
```

`safe.SetSeed` creates a mnemonic and set the `_seedPrivateKey`. Finally it returns the mnemonic, so we can give it back to the user of the class.

![](../assets/RootKey.png)

```cs
private ExtKey _seedPrivateKey;
private Mnemonic SetSeed(string password)
{
    var mnemonic = new Mnemonic(Wordlist.English, WordCount.Twelve);

    _seedPrivateKey = mnemonic.DeriveExtKey(password);

    return mnemonic;
}
```

### safe.Save

ウォレットのファイルを保存する。問題はその中に何を保存するかだ。

```json
{
  "EncryptedSeed":"6PYXR8U5Nu9UoGZcU95DWWKCXppKnYBUKyJgze6DX6bQDNwFzNdJApUzXT",
  "ChainCode":"C+2MiZU7R/33bkvgdDqdQp7xx3nXHSIzS6bUgRsnaus=",
  "Network":"MainNet"
}
```

ウォレットファイルはJSONフォーマットだ。  
拡張鍵からチェーンコードと秘密鍵を取得することができるし、逆もできる。

```cs
Key privateKey = _seedPrivateKey.PrivateKey;
byte[] chainCode = _seedPrivateKey.ChainCode;
```

最後に秘密鍵を暗号化する。

![](../assets/EncryptedKey.png)

```cs
string encryptedBitcoinPrivateKeyString = privateKey.GetEncryptedBitcoinSecret(password, _network).ToWif();
string chainCodeString = Convert.ToBase64String(chainCode);
string networkString = network.ToString();
```

### Safe.Load

保存するプロセスを逆方向にたどってみよう。

```cs
public static Safe Load(string password, string walletFilePath)
{
    if (!File.Exists(walletFilePath))
        throw new Exception("WalletFileDoesNotExists");

    var walletFileRawContent = WalletFileSerializer.Deserialize(walletFilePath);

    var encryptedBitcoinPrivateKeyString = walletFileRawContent.EncryptedSeed;
    var chainCodeString = walletFileRawContent.ChainCode;

    var chainCode = Convert.FromBase64String(chainCodeString);

    Network network;
    var networkString = walletFileRawContent.Network;
    if (networkString == Network.MainNet.ToString())
        network = Network.MainNet;
    else if (networkString == Network.TestNet.ToString())
        network = Network.TestNet;
    else throw new Exception("NotRecognizedNetworkInWalletFile");

    var safe = new Safe(password, walletFilePath, network);

    var privateKey = Key.Parse(encryptedBitcoinPrivateKeyString, password, safe._network);
    var seedExtKey = new ExtKey(privateKey, chainCode);
    safe._seedPrivateKey = seedExtKey;

    return safe;
}
```

ここにSafeクラスのコンストラクタの中で何をしているかを示す。

```cs
private Safe(string password, string walletFilePath, Network network)
{
    SetNetwork(network);

    SetSeed(password, mnemonicString);

    WalletFilePath = walletFilePath;
}
```

### SetNetwork

このクラスの中では`NBitcoin.Network`を使いたい。だからそれをprivateのメンバーとしてセットしよう。

```cs
private NBitcoin.Network _network;
private void SetNetwork(Network network)
{
    if (network == Network.MainNet)
        _network = NBitcoin.Network.Main;
    else if (network == Network.TestNet)
        _network = NBitcoin.Network.TestNet;
    else throw new Exception("WrongNetwork");
}
```

### Safe.Recover

```cs
public static Safe Recover(string mnemonic, string password, string walletFilePath, Network network)
{
    var safe = new Safe(password, walletFilePath, network, mnemonic);
    safe.Save(password, walletFilePath, network);
    return safe;
}
```

これを動かすために、コンストラクタを拡張しなければならない。

```cs
private Safe(string password, string walletFilePath, Network network, string mnemonicString = null)
{
    SetNetwork(network);

    if (mnemonicString != null)
    {
        var mnemonic = new Mnemonic(mnemonicString);
        _seedPrivateKey = mnemonic.DeriveExtKey(password);
    }

    WalletFilePath = walletFilePath;
}
```

### Getters

ここにはどのようにして鍵を引き出すかを示してある。私の目的としては複雑なキーパスを使うことはあまり意味をなさない。

```cs
public PrivateKeyAddressPair GetPrivateKeyAddressPair(int index)
{
    var foo = _seedPrivateKey.Derive(index, true).GetWif(_network);
    return new PrivateKeyAddressPair
    {
        PrivateKey = foo.ToWif(),
        Address = foo.ScriptPubKey.GetDestinationAddress(_network).ToWif()
    };
}
```

### Stealth

```cs
private Key _spendPrivateKey => _seedPrivateKey.PrivateKey;
public string SpendPrivateKey => _spendPrivateKey.GetWif(_network).ToWif();
private Key _scanPrivateKey => _seedPrivateKey.Derive(0, hardened: true).PrivateKey;
public string ScanPrivateKey => _scanPrivateKey.GetWif(_network).ToWif();

public string StealthAddress => new BitcoinStealthAddress
    (_scanPrivateKey.PubKey, new[] {_spendPrivateKey.PubKey}, 1, null, _network
    ).ToWif();
```



