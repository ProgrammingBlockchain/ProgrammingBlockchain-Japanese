## HDウォレット (BIP 32) {#hd-wallet-bip-32}

それでは、我々が解決したい問題を再度心にとめておこう:

* バックアップが古くなってしまうのを防ぎたい
* 信頼してない相手に鍵やアドレスの生成を委譲したい

決定性ウォレットは、バックアップの問題を解決するだろう。そのようなウォレットでは、保管しなければならないのはシード値だけだ。そしてこのシードから、一連の秘密鍵を何度も生成することができる。

これが、"決定性"の意味するところである。
以下のように、マスターキーから新しい鍵をどんどん生成できる。

```cs
ExtKey masterKey = new ExtKey();
Console.WriteLine("Master key : " + masterKey.ToString(Network.Main));
for (int i = 0; i < 5; i++)
{
    ExtKey key = masterKey.Derive((uint)i);
    Console.WriteLine("Key " + i + " : " + key.ToString(Network.Main));
}
```

```
Master key : xprv9s21ZrQH143K3JneCAiVkz46BsJ4jUdH8C16DccAgMVfy2yY5L8A4XqTvZqCiKXhNWFZXdLH6VbsCsqBFsSXahfnLajiB6ir46RxgdkNsFk
Key 0 : xprv9tvBA4Kt8UTuEW9Fiuy1PXPWWGch1cyzd1HSAz6oQ1gcirnBrDxLt8qsis6vpNwmSVtLZXWgHbqff9rVeAErb2swwzky82462r6bWZAW6Ty
Key 1 : xprv9tvBA4Kt8UTuHyzrhkRWh9xTavFtYoWhZTopNHGJSe3KomssRrQ9MTAhVWKFp4d7D8CgmT7TRzauoAZXp3xwHQfxr7FpXfJKpPDUtiLdmcF
Key 2 : xprv9tvBA4Kt8UTuLoEZPpW9fBEzC3gfTdj6QzMp8DzMbAeXgDHhSMmdnxSFHCQXycFu8FcqTJRm2kamjeE8CCKzbiXyoKWZ9ihiF7J5JicgaLU
Key 3 : xprv9tvBA4Kt8UTuPwJQyxuZoFj9hcEMCoz7DAWLkz9tRMwnBDiZghWePdD7etfi9RpWEWQjKCM8wHvKQwQ4uiGk8XhdKybzB8n2RVuruQ97Vna
Key 4 : xprv9tvBA4Kt8UTuQoh1dQeJTXsmmTFwCqi4RXWdjBp114rJjNtPBHjxAckQp3yeEFw7Gf4gpnbwQTgDpGtQgcN59E71D2V97RRDtxeJ4rVkw4E
Key 5 : xprv9tvBA4Kt8UTuTdiEhN8iVDr5rfAPSVsCKpDia4GtEsb87eHr8yRVveRhkeLEMvo3XWL3GjzZvncfWVKnKLWUMNqSgdxoNm7zDzzD63dxGsm
```

**マスターキー** だけを保存する必要がある。なぜなら、同じ一連の秘密鍵を何度も生成できるからである。

見ればわかるように、これらの鍵は **ExtKey** であり、使い慣れた **Key** ではない。しかし、内部に本物の秘密鍵を内包しているので、使うのを躊躇する必要はない。

![](../assets/ExtKey.png)

**Key** と **ChainCode** を **ExtKey** のコンストラクターに渡すことで **Key** から **ExtKey** に戻すことも可能である。以下のようにする:

```cs
ExtKey extKey = new ExtKey();
byte[] chainCode = extKey.ChainCode;
Key key = extKey.PrivateKey;

ExtKey newExtKey = new ExtKey(key, chainCode);
```

**base58** 型の **ExtKey** は、 **BitcoinExtKey** という。

しかし、どうやって2つ目の問題を解決するのだろう: 潜在的にハッキングされる可能性のある相手（支払いサーバー）にアドレスの作成を委譲するには？

トリックは、マスターキーの無性化ができるということである。そして、秘密鍵を持たない公開可能なバージョンのマスターキーを持つことができる。無性化されたマスターキーから、第三者機関は、秘密鍵なしに公開鍵を作成できるのだ。

```cs
ExtPubKey masterPubKey = masterKey.Neuter();
for (int i = 0 ; i < 5 ; i++)
{
    ExtPubKey pubkey = masterPubKey.Derive((uint)i);
    Console.WriteLine("PubKey " + i + " : " + pubkey.ToString(Network.Main));
}
```

```
PubKey 0 : xpub67uQd5a6WCY6A7NZfi7yGoGLwXCTX5R7QQfMag8z1RMGoX1skbXAeB9JtkaTiDoeZPprGH1drvgYcviXKppXtEGSVwmmx4pAdisKv2CqoWS
PubKey 1 : xpub67uQd5a6WCY6CUeDMBvPX6QhGMoMMNKhEzt66hrH6sv7rxujt7igGf9AavEdLB73ZL6ZRJTRnhyc4BTiWeXQZFu7kyjwtDg9tjRcTZunfeR
PubKey 2 : xpub67uQd5a6WCY6Dxbqk9Jo9iopKZUqg8pU1bWXbnesppsR3Nem8y4CVFjKnzBUkSVLGK4defHzKZ3jjAqSzGAKoV2YH4agCAEzzqKzeUaWJMW
PubKey 3 : xpub67uQd5a6WCY6HQKya2Mwwb7bpSNB5XhWCR76kRaPxchE3Y1Y2MAiSjhRGftmeWyX8cJ3kL7LisJ3s4hHDWvhw3DWpEtkihPpofP3dAngh5M
PubKey 4 : xpub67uQd5a6WCY6JddPfiPKdrR49KYEuXUwwJJsL5rWGDDQkpPctdkrwMhXgQ2zWopsSV7buz61e5mGSYgDisqA3D5vyvMtKYP8S3EiBn5c1u4
```

では、支払いサーバーが公開鍵1を生成したと想定してみよう。すると対応した秘密鍵を、秘密のマスターキーを使って得ることができる。

```cs
masterKey = new ExtKey();
masterPubKey = masterKey.Neuter();

//The payment server generate pubkey1
ExtPubKey pubkey1 = masterPubKey.Derive(1);

//You get the private key of pubkey1
ExtKey key1 = masterKey.Derive(1);

//Check it is legit
Console.WriteLine("Generated address : " + pubkey1.PubKey.GetAddress(ScriptPubKeyType.Legacy, Network.Main));
Console.WriteLine("Expected address : " + key1.PrivateKey.PubKey.GetAddress(ScriptPubKeyType.Legacy, Network.Main));
```

```
Generated address : 1Jy8nALZNqpf4rFN9TWG2qXapZUBvquFfX
Expected address : 1Jy8nALZNqpf4rFN9TWG2qXapZUBvquFfX
```

**ExtPubKey** は **ExtKey** に似ている。が、両者の違いは、 **PubKey** （公開鍵）を保持しているが **Key** （秘密鍵）は保持していない。

![](../assets/ExtPubKey.png)

ここまで、決定性鍵がどうやって問題を解決してくれるのかを見てきた。では、次は、 **階層的** が何を意味するかについて議論してみよう。

前段の演習では、マスターキーとインデックス番号の組み合わせにより 鍵を生成するのを見てきた。このプロセスを **派生** と呼び、マスターキーは **親の鍵** で、生成される鍵は、**子供の鍵** である。

しかし、子供の鍵から派生した、さらに子供の鍵を複数作ることができる。これが **階層的** の意味である。

なので、概念的かつ一般的にこのように言うことができる: 親の鍵 ＋ 鍵の階層パス → 子供の鍵

![](../assets/Derive1.png)

![](../assets/Derive2.png)

この図に書かれているように、親から2つのやり方で子供(1,1)を派生させることができる。

```cs
ExtKey parent = new ExtKey();
ExtKey child11 = parent.Derive(1).Derive(1);
```

もしくは、

```cs
ExtKey parent = new ExtKey();
ExtKey child11 = parent.Derive(new KeyPath("1/1"));
```

なので、まとめると:

![](../assets/DeriveKeyPath.png)

**ExtPubKey** も同じように取り扱うことができる。

なぜ 階層的な鍵が必要になるのだろう？なぜなら、自分の複数の口座のための鍵を目的に別に分類するのは、良い方法だと思われるからだ。詳しくは、[BIP44](https://github.com/bitcoin/bips/blob/master/bip-0044.mediawiki)。

そして、 組織の内部において口座の権限を分けて管理することができるようにもなる。

自分をある会社のCEOだとしてみよう。すると、会社のウォレットをすべて管理したいだろう。しかし、経理部に、マーケティング部のお金を使ってほしくない。

すると最初に思いつくアイディアは、1つの部署に1つの階層を生成することだろう。

![](../assets/CeoMarketingAccounting.png)

しかし、この場合、**経理部** と **マーケティング部**  はCEOの秘密鍵を逆生成できるかもしれない。

そのような子供鍵は **非強化** であると言われる。

![](../assets/NonHardened.png)

```cs
ExtKey ceoKey = new ExtKey();
Console.WriteLine("CEO: " + ceoKey.ToString(Network.Main));
ExtKey accountingKey = ceoKey.Derive(0, hardened: false);

ExtPubKey ceoPubkey = ceoKey.Neuter();

//Recover ceo key with accounting private key and ceo public key
ExtKey ceoKeyRecovered = accountingKey.GetParentExtKey(ceoPubkey);
Console.WriteLine("CEO recovered: " + ceoKeyRecovered.ToString(Network.Main));
```

```
CEO: xprv9s21ZrQH143K2XcJU89thgkBehaMqvcj4A6JFxwPs6ZzGYHYT8dTchd87TC4NHSwvDuexuFVFpYaAt3gztYtZyXmy2hCVyVyxumdxfDBpoC
CEO recovered: xprv9s21ZrQH143K2XcJU89thgkBehaMqvcj4A6JFxwPs6ZzGYHYT8dTchd87TC4NHSwvDuexuFVFpYaAt3gztYtZyXmy2hCVyVyxumdxfDBpoC
```

別の言い方でいうと、 **非強化鍵** は階層を"登る"ことができる。**非強化鍵** は単一管理されるタイプの口座にだけ使用されるべきでる。

なので、我々のケースでは、CEOは **強化鍵** を作成しなければならない。そうすれば、経理部は、階層を登って秘密鍵を見つけることはできない。

```cs
ExtKey ceoKey = new ExtKey();
Console.WriteLine("CEO: " + ceoKey.ToString(Network.Main));
ExtKey accountingKey = ceoKey.Derive(0, hardened: true);

ExtPubKey ceoPubkey = ceoKey.Neuter();

ExtKey ceoKeyRecovered = accountingKey.GetParentExtKey(ceoPubkey); //Crash
```

**ExtKey.Derivate**(**鍵階層パス)** を使うときに、子のインデックス値の後ろにアポストロフィを付けることでも、強化鍵を作成できる。

```cs
var nonHardened = new KeyPath("1/2/3");
var hardened = new KeyPath("1/2/3'");
```

では経理部は、顧客ごとに1つの親鍵を生成したと想定してみよう。そして、顧客からの支払いごとにその子供鍵を使う。

CEOであるあなたは、そのうちの1つのアドレスから支払いたい場合、こうやって行う。

```cs
ceoKey = new ExtKey();
string accounting = "1'";
int customerId = 5;
int paymentId = 50;
KeyPath path = new KeyPath(accounting + "/" + customerId + "/" + paymentId);
//Path : "1'/5/50"
ExtKey paymentKey = ceoKey.Derive(path);
```
