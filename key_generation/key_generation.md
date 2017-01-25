<!---
# Key generation and encryption {#key-generation-encryption}
--->
# 鍵生成と暗号化 {#key-generation-encryption}  
<!---
## Is it random enough? {#is-it-random-enough}
--->
## それは本当にランダムだろうか？ {#is-it-random-enough}
<!---
When you call **new Key()**, under the hood, you are using a PRNG (Pseudo-Random-Number-Generator) to generate your private key. On windows, it uses the **RNGCryptoServiceProvider**, a .NET wrapper around the Windows Crypto API.
--->

**new Key()** でコンストラクターを呼び出すと、内部ではPRNG(疑似乱数生成器)を使って秘密鍵を生成している。WindowsOS上では、Windows Crypto APIの.Netラッパーである **RNGCryptoServiceProvider** を使用している。  

<!---
On Android, I use the **SecureRandom**, and in fact, you can use your own implementation with **RandomUtils.Random**.
--->
アンドロイドでは、私は **SecureRandom** を使用する。実際は **RandomUtils.Random** を使って自分独自の実装を使うことができる。  

<!---
On IOS, I have not implemented it and you need to create your **IRandom** implementation.
--->
iOS上では、私はまだ実装したことはないが、自分で **IRandom** の実装クラスを作る必要がある。  

<!---
For a computer, being random is hard. But the biggest issue is that it is impossible to know if a series of number is really random.
--->
コンピューターにとってランダムにすることは難しい。しかし、1番大きな問題は、ある一連の数値が本当にランダムかどうかを知ることが不可能だということである。  

<!---
If malware modifies your PRNG (and so, can predict the numbers you will generate), you won’t see it until it is too late.
--->
もし、マルウェアがあなたのPRNGを改ざんした場合（なので、あなたが生成する数値を予測できる）、手遅れになって初めて改ざんされたことに気づくことになる。

<!---
It means that a cross platform and naïve implementation of PRNG (like using the computer’s clock combined with CPU speed) is dangerous. But you won’t see it until it is too late.
--->
これは、クロスプラットフォーム、もしくは、ネイティブ実装のPRNG（コンピュータのクロックとCPUのスピードの組み合わせを利用）は危険であることを意味する。しかし、手遅れになるまで、それを知る由はない。  

<!---
For performance reasons, most PRNG works the same way: a random number, called **Seed**, is chosen, then a predictable formula generates the next numbers each time you ask for it.
--->
パフォーマンス上の理由から ほとんどのPRNGは同じように機能する。**シード** とよばれるランダムな数値が1つ選ばれ、呼び出す度に、結果が予測可能な式によって次の値が生成される。  

<!---
The amount of randomness of the seed is defined by a measure we call **Entropy**, but the amount of **Entropy** also depends on the observer.
--->
シードのランダムさの度合は、**エントロピー** と呼ばれる計測値で定義されるが、エントロピー度は、観測者にも依存する。  

<!---
Let’s say you generate a seed from your clock time.  
And let’s imagine that your clock has 1ms of resolution. (Reality is more ~15ms.)
--->
例えば、あなたが自分のクロック時間をもとにシード値を生成したとしよう。  
そして、クロックが１ミリ秒の精度を持ったとしよう。（現実には１５ミリ秒以上。）  

<!---
If your attacker knows that you generated the key last week, then your seed has  
1000 \* 60 \* 60 \* 24 \* 7 = 604800000 possibilities.
--->
もしあなたが先週鍵を生成したと、攻撃者が知っているとすると、シード値は、  
1000 \* 60 \* 60 \* 24 \* 7 = 604800000 通りある。  

<!---
For such attacker, the entropy is LOG(604800000;2) = 29.17 bits.
--->
そのような攻撃者にとって、エントロピーは、log<sub>2</sub>(604800000) = 29.17 ビットである。  

<!---
And enumerating such number on my home computer took less than 2 seconds. We call such enumeration “brute forcing”.
--->
その程度の回数だと順番に処理するとして、私の自宅のコンピュータで処理させても２秒以下しかかからない。このように順番に全可能性を当たってみる処理のことを”総当たり式”と呼ぶ。  

<!---
However let’s say, you use the clock time + the process id for generating the seed.  
Let’s imagine that there are 1024 different process ids.
--->
でも、例えば、シード値を生成するのに、クロック時間とプロセスIDを使ったとする。  
そして、1024個の個別のプロセスIDが存在すると想像してみよう。  

<!---
So now, the attacker needs to enumerate 604800000 \* 1024 possibilities, which take around 2000 seconds.  
Now, let’s add the time when I turned on my computer, assuming the attacker knows I turned it on today, it adds 86400000 possibilities.  
--->
今度は、攻撃者は、604800000 \* 1024 回を順番に当たっていく必要があり、それには2000秒かかる。  
さて、ここに、コンピューターを起動した日時の情報も足してみよう。攻撃者は私が今日起動したと知っているとしても、86400000 通りの可能性が追加できる。  

<!---
Now the attacker needs to enumerate 604800000 \* 1024 \* 86400000 = 5,35088E+19 possibilities.  
However, keep in mind that if the attacker infiltrate my computer, he can get this last piece of info, and bring down the number of possibilities, reducing entropy.
--->
これで、攻撃者は、604800000 \* 1024 \* 86400000 = 5,35088E+19　通りの可能性に当たる必要がある。  
しかし、覚えておいてほしいのは、もし攻撃者が私のコンピューターに侵入可能であれば、この追加の情報を取得できるので、可能性の数を減らし、エントロピーを下げることができる。  

<!---
Entropy is measured by **LOG(possibilities;2)** and so LOG(5,35088E+19; 2) = 65 bits.
--->
エントロピーは、log<sub>2</sub>(数値の取りえる可能性の数)で計算できるので、log<sub>2</sub>(5,35088E+19)= 65 ビットとなる。  

<!---
Is it enough? Probably. Assuming your attacker does not know more information about the realm of possibilities.
--->
これは十分だろうか。攻撃者が可能性を左右する情報をこれ以上もってないと仮定するならば、多分そうだろう。  

<!---
But since the hash of a public key is 20 bytes = 160 bits, it is smaller than the total universe of the addresses. You might do better.
--->
しかし、公開鍵のハッシュ値は、20バイト = 160ビット ということは、すべてのアドレスの数よりまだ小さい。もう少し頑張れるかもしれない。  

<!---
> **Note:** Adding entropy is linearly harder, cracking entropy is exponentially harder
--->
> **注意:** エントロピーを増やすことは線形的に難しいが、エントロピーを解読するのは、指数関数的に難しくなる。  

<!---
An interesting way of generating entropy quickly is by asking human intervention. (Moving the mouse.)
--->
エントロピーを生成する面白い方法は、人間を介在させるやり方だ。(マウスを動かす。)  

<!---
If you don’t completely trust the platform PRNG (which is [not so paranoic](http://android-developers.blogspot.fr/2013/08/some-securerandom-thoughts.html)), you can add entropy to the PRNG output that NBitcoin is using.  
--->
もし、あなたがプラットフォームのPRNGを完全に信頼できない（それはそれほど[おかしな事ではない](http://android-developers.blogspot.fr/2013/08/some-securerandom-thoughts.html)）なら、NBitcoinが使用する、PRNGからの出力にたいしてエントロピーを足すことができる。

```cs
RandomUtils.AddEntropy("hello");
RandomUtils.AddEntropy(new byte[] { 1, 2, 3 });
var nsaProofKey = new Key();
```  
<!---
What NBitcoin does when you call **AddEntropy(data)** is:
--->
あなたが **AddEntropy(data)** を呼び出すときNBitcoinが処理するのは、  
**additionalEntropy = SHA(SHA(data) ^ additionalEntropy)** だ。  

<!---
Then when you generate a new number:  
--->
そして、新しい数値を取得するときの結果は:  
**result = SHA(PRNG() ^ additionalEntropy)** だ。

<!---
## Key Derivation Function {#key-derivation-function}
--->
## 鍵導出関数 {#key-derivation-function}
<!---
However, what is most important is not the number of possibilities. It is the time that an attacker would need to successfully break your key. That’s where KDF enters the game.
--->
しかし、もっとも重要なことは、可能性の数の多さではなく、攻撃者が、あなたの鍵を破ることに成功するのに、どれくらいの時間が必要かということである。そこで、KDFの登場である。

<!---
KDF, or **Key Derivation Function** is a way to have a stronger key, even if your entropy is low.
--->
KDF もしくは **鍵導出関数(Key Derivation Function)** が、たとえエントロピーが低くても、より強い鍵をもつための方法である。  

<!---
Imagine that you want to generate a seed, and the attacker knows that there are 10.000.000 possibilities.  
Such a seed would be normally cracked pretty easily.
--->
シード値を生成してみたいと想像してみよう、そして攻撃者が１千万個の可能性があることをしっていたとする。  
そのようなシードは通常、クラックして破るのは非常に簡単だ。  
<!---
But what if you could make the enumeration slower?  
A KDF is a hash function that waste computing resources on purpose.  
Here is an example:
--->

しかし、もしあなたがその順番処理を遅くすることができたとしたらどうだろう？  
KDFはハッシュ関数で意図的にコンピューターの演算リソースを無駄に使わせることができる。  
これが例だ:  

```cs
var derived = SCrypt.BitcoinComputeDerivedKey("hello", new byte[] { 1, 2, 3 });
RandomUtils.AddEntropy(derived);
```  
<!---
Even if your attacker knows that your source of entropy is 5 letters, he will need to run Scrypt to check a possibility, which take 5 seconds on my computer.
--->
もし仮に、攻撃者があなたのエントロピーの元が５つの文字だと知っていたとしても、Scryptを実行して合っているか可能性を確認しないといけない。私のコンピューターでは５秒かかる。  

<!--
The bottom line is: There is nothing paranoid into distrusting a PRNG, and you can mitigate an attack by both adding entropy and also using a KDF.  
-->
結局のところ、どういうことかというと、疑似乱数発生器に悪さをされる事を極端に心配する必要はない。エントロピーの増加とKDFの使用、両方をすることで攻撃を弱めることができる。  
<!--
Keep in mind that an attacker can decrease entropy by gathering information about you or your system.  
If you use the timestamp as entropy source, then he can decrease the entropy by knowing you generated the key last week, and that you only use your computer between 9am and 6pm.
-->
覚えておいてほしいのは、攻撃者は、あなた、もしくは、あなたのシステムの情報を集めることでエントロピーを減らすことができるということだ。  
もし、タイムスタンプを元にエントロピーを作るとして、あなたが先週キーを生成し、しかも午前９時から午後６時までしかコンピューターを使用しないということを攻撃者が知ってしまうとエントロピーを減らすことになる。  

<!--
In the previous part I talked quickly about a special KDF called **Scrypt.** As I said, the goal of a KDF is to make brute force costly.  
-->
前に、特別なKDFである**Scrypt**について話した。そこで言った通り、KDFの目的は 総当たり攻撃をコストがかかるものにすることだ。  

<!--
So it should be no surprise for you that a standard already exists for encrypting your private key with a password using a KDF. This is
-->
なので、秘密鍵をKDFを使ってパスワードで暗号化する標準の方法がすでにあるとしても、別に驚くことではないだろう。それが、[BIP38](http://www.codeproject.com/Articles/775226/NBitcoin-Cryptography-Part)だ。  

![](../assets/EncryptedKey.png)  

```cs
var privateKey = new Key();
var bitcoinPrivateKey = privateKey.GetWif(Network.Main);
Console.WriteLine(bitcoinPrivateKey); // L1tZPQt7HHj5V49YtYAMSbAmwN9zRjajgXQt9gGtXhNZbcwbZk2r
BitcoinEncryptedSecret encryptedBitcoinPrivateKey = bitcoinPrivateKey.Encrypt("password");
Console.WriteLine(encryptedBitcoinPrivateKey); // 6PYKYQQgx947Be41aHGypBhK6TA5Xhi9TdPBkatV3fHbbKrdDoBoXFCyLK
var decryptedBitcoinPrivateKey = encryptedBitcoinPrivateKey.GetSecret("password");
Console.WriteLine(decryptedBitcoinPrivateKey); // L1tZPQt7HHj5V49YtYAMSbAmwN9zRjajgXQt9gGtXhNZbcwbZk2r

Console.ReadLine();
```  
<!--
Such encryption is used in two different cases:  
-->

このような暗号化は、２つのケースで使用される。:  

<!--
*   You don not trust your storage provider (they can get hacked)  
*   You are storing the key on the behalf of somebody else (and you do not want to know his key)  
-->
*   鍵の保存場所の提供者を信用していない。（ハッキングされるかもしれないから。）
*   他の人のために鍵を保存しようとしている。（あなたはその人の鍵を知りたくない。）  

<!--
If you own your storage, then encrypting at the database level might be enough.  
-->
もし、自分の保存先を持っているならデータベースが提供する暗号化で十分だろう。  

<!--
Be careful if your server takes care of decrypting the key, an attacker might attempt to DDOS your server by forcing it to decrypt lots of keys.
-->
もし、あなたのサーバーが復号化の機能を持つ場合、気を付けなければいけない。もしかすると、攻撃者が大量の鍵を復号化させるようなDDOS攻撃を仕掛けてくるかもしれないからだ。  

<!--攻撃者が
Delegate decryption to the ultimate user when you can.  
-->
可能であるならば、復号化は最終ユーザーに移譲すべきである。  

<!--
## Like the good ol’ days {#like-the-good-ol-days}
-->
## 古き良き時代のように {#like-the-good-ol-days}

<!--
First, why generating several keys?  
The main reason is privacy. Since you can see the balance of all addresses, it is better to use a new address for each transaction.  
-->
まず最初に、なぜ複数の鍵を生成しなければならないのだろうか。  
一番の理由はプライバシーである。すべてのアドレスの残高は、だれでも見ることができるのだから、取引ごとに新しいアドレスを使うほうが良い。  

<!--
However, in practice, you can also generate keys for each contact which makes this a simple way to identify your payer without leaking too much privacy.  
-->
しかし、実際のところ、相手ごとにも鍵を生成することもできる。それにより、支払元を簡単に特定でき、プライバシーの露出も防げる。  

<!--
You can generate key, like you did from the beginning:
-->
これまでのコードでもやってきたように、以下のように鍵を生成できる：
```cs
var privateKey = new Key()
```  
<!--
However, you have two problems with that:  
-->
しかし、これには、２つの問題がある:

<!--
*   All backups of your wallet that you have will become outdated when you generate a new key.  
*   You cannot delegate the address creation process to an untrusted peer.  
-->
*   新しい鍵を生成することで、自分の持つウォレットのバックアップが古くて使えなってしまう。
*   アドレスの作成処理を信用できない相手に委譲することができなくなる。

<!--
If you are developing a web wallet and generate key on behalf of your users, and one user get hacked, she will immediately start suspecting you.  
-->
もし、ウェブウォレットを開発していて、ユーザーのために、キーを生成している場合、あるユーザーの鍵が破られたとすると、即座にその人は、あなたを疑い始めるだろう。

<!--
## BIP38 (Part 2) {#bip38-part-2}
-->
## BIP38 (パート２) {#bip38-part-2}
<!--
We already saw BIP38 for encrypting a key, however this BIP is in reality two ideas in one document.  
-->
我々はBIP38をすでに見ているが、実は、このBIPは、２つのアイデアを１つにしたものだ。  

<!--
The second part of the BIP, shows how you can delegate Key and Address creation to an untrusted peer. It will fix one of our concerns.  
-->
２つめについての部分では、どうやって鍵とアドレスの生成を信頼できない相手に委譲することができるかを書いている。ので、２つの懸念点のうちの１つを解決する。  

<!--
**The idea is to generate a PassphraseCode to the key generator. With this PassphraseCode, he will be able to generate encrypted keys on your behalf, without knowing your password, nor any private key. **
-->
**そのアイデアとは、鍵生成をする人へ渡すパスフレーズコードを作るというものだ。このパスフレーズコードで、その相手は、あなたのために、暗号化された鍵を作ることができ、しかも、あなたのパスワードも秘密鍵も知ることはない。**

この **パスフレーズコード** は WIFフォーマットにして鍵生成する人に渡すことができる。

<!--
> **Tip**: In NBitcoin, all types prefixed by “Bitcoin” are Base58 (WIF) data.  
-->
> **ティップス**: NBitcoinでは、Bitcoinが頭についている型は全て Base58 （WIF）フォーマットのデータである。

<!--
So, as a user that wants to delegate key creation, first you will create the **PassphraseCode**.
-->
なので、鍵の生成を委譲したいユーザーとして、最初に、 **パスフレーズコード** を作ることになる。  

![](../assets/PassphraseCode.png)  

```cs
var passphraseCode = new BitcoinPassphraseCode("my secret", Network.Main, null);
```
<!--
**You then give this passphraseCode to a third party key generator.**
-->
**そしてこのパスフレーズコードを外部の鍵生成してくれる人に渡す。**

<!--
The third party will then generate new encrypted keys for you.
-->
その鍵生成者は、暗号化された鍵をあなたに作ってくれる。  

![](../assets/PassphraseCodeToEncryptedKeys.png)  

```cs
EncryptedKeyResult encryptedKeyResult = passphraseCode.GenerateEncryptedSecret();
```  
<!--
This **EncryptedKeyResult** has lots of information:  
-->
この **暗号化された鍵**　は、たくさんの情報を持っている。  

![](../assets/EncryptedKeyResult.png)  

<!--
First: the **generated bitcoin address**,  
-->
まずは、生成されたビットコインアドレス  
```cs
var generatedAddress = encryptedKeyResult.GeneratedAddress; // 14KZsAVLwafhttaykXxCZt95HqadPXuz73
```  
<!--
then the **EncryptedKey** itself, (as we have seen in the previous, **Key Encryption** lesson),  
-->
それから、暗号化された鍵 自身、（前にやった **鍵の暗号化** のレッスンでみたように）
```cs
var encryptedKey = encryptedKeyResult.EncryptedKey; // 6PnWtBokjVKMjuSQit1h1Ph6rLMSFz2n4u3bjPJH1JMcp1WHqVSfr5ebNS
```  
<!--
and last but not the least, the **ConfirmationCode**, so that the third party can prove that the generated key and address correspond  to your password.
-->
そして、最後だけれども、重要である **確認コード**。これにより、その第三者は、生成された鍵とアドレスが あなたのパスワードに対応していることを証明することができる。
```cs
var confirmationCode = encryptedKeyResult.ConfirmationCode; // cfrm38VUcrdt2zf1dCgf4e8gPNJJxnhJSdxYg6STRAEs7QuAuLJmT5W7uNqj88hzh9bBnU9GFkN
```  
<!--
As the owner, once you receive this information, you need to check that the key generator did not cheat by using **ConfirmationCode.Check**, then get your private key with your password:
-->
あなたは所有者として、この情報を手に入れたらすぐ、**ConfirmationCode.Check** を使って、鍵作成者が、インチキをしていないか確認する必要がある。そして、パスワードをつかって秘密鍵を取得する。

```cs
Console.WriteLine(confirmationCode.Check("my secret", generatedAddress)); // True
var bitcoinPrivateKey = encryptedKey.GetSecret("my secret");
Console.WriteLine(bitcoinPrivateKey.GetAddress() == generatedAddress); // True
Console.WriteLine(bitcoinPrivateKey); // KzzHhrkr39a7upeqHzYNNeJuaf1SVDBpxdFDuMvFKbFhcBytDF1R
```  
<!--
So, we have just seen how the third party can generate encrypted key on your behalf, without knowing your password and private key.
-->
我々は、第三者がどうやってあなたのパスワードと秘密鍵を知ることなしに暗号化された鍵を作成することができるかを見てきた。  

![](../assets/ThirdPartyKeyGeneration.png)  
<!--
However, one problem remains:
-->
しかし、まだ１つ問題が残っている:  

<!--
*   All backups of your wallet that you have will become outdated when you generate a new key.
-->
*   新しい鍵を生成することで、自分の持つウォレットのバックアップがすべて古いものになってしまう。

<!--
BIP 32, or Hierarchical Deterministic Wallets (HD wallets) proposes another solution, and is more widely supported.
-->
BIP 32、もしくは 階層的決定性ウォレット(HDウォレット)は、別の解決方法を提案している。そして、それは広くサポートされた方法だ。

<!--
## HD Wallet (BIP 32) {#hd-wallet-bip-32}
-->
## HDウォレット (BIP 32) {#hd-wallet-bip-32}
<!--
Let’s keep in mind the problems that we want to resolve:
-->
それでは、我々が解決したい問題を心にとめておこう:

<!--
*   Prevent outdated backups
*   Delegating key / address generation to an untrusted peer
-->
*   バックアップが古くなってしまうのを防ぎたい
*   信頼してない相手に鍵やアドレスの生成を委譲したい

<!--
A “Deterministic” wallet would fix our backup problem. With such wallet, you would have to save only the seed. From this seed, you can generate the same series of private keys over and over.  
-->
決定性ウォレットは、バックアップの問題を解決するだろう。そのようなウォレットでは、シード値だけは保存しなければならない。そして、このシードから、一連の秘密鍵を何度も生成することができる。  

<!--
This is what the “Deterministic” stands for.  
As you can see, from the master key, I can generate new keys:  
-->
これが、"決定性"の意味するところである。  
以下のように、マスターキーから、新しい鍵をどんどん生成できる。

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
<!--
You only need to save the **masterKey**, since you can generate the same suite of private keys over and over.  
-->
**マスターキー** だけを保存する必要がある。なぜなら、同じ一連の秘密鍵を何度も生成できるからである。  

<!--
As you can see, these keys are **ExtKey** and not **Key** as you are used to. However, this should not stop you since you have the real private key inside:  
-->
見ればわかるように、これらの鍵は、 **ExtKey** であり、使い慣れた **Key** ではない。しかし、内部に本物の秘密鍵をもっているので、使うのを躊躇する必要はない。  

![](../assets/ExtKey.png)  
<!--
You can go back from a **Key** to an **ExtKey** by supplying the **Key** and the **ChainCode** to the **ExtKey** constructor. This works as follows:  
-->

**Key** と **ChainCode** を **ExtKey** のコンストラクターに渡すことで **Key** から **ExtKey** に戻すことも可能である。以下のようにする:  

```cs
ExtKey extKey = new ExtKey();
byte[] chainCode = extKey.ChainCode;
Key key = extKey.PrivateKey;

ExtKey newExtKey = new ExtKey(key, chainCode);
```  

<!--
The **base58** type equivalent of **ExtKey** is called **BitcoinExtKey**.
-->
**base58** 型の **ExtKey** は、 **BitcoinExtKey** という。  

<!--
But how can we solve our second problem: delegating address creation to a peer that can potentially be hacked (like a payment server)?
-->
しかし、どうやって２つ目の問題を解決するのだろう: 潜在的にハッキングされる可能性のある相手（支払サーバー）にアドレスの作成を委譲するには？

<!--
The trick is that you can “neuter” your master key, then you have a public (without private key) version of the master key. From this neutered version, a third party can generate public keys without knowing the private key.
-->
トリックは、マスターキーの無性化ができるということである。そして、秘密鍵をもたない公開可能なバージョンのマスターキーを持つことができる。無性化されたマスターキーから、第三者機関は、秘密鍵なしに公開鍵を作成できるのだ。  

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
<!--
So imagine that your payment server generates pubkey1, you can get the corresponding private key with your private master key.
-->
では、支払サーバーが公開鍵1を生成したと想定してみよう。すると対応した秘密鍵を秘密のマスターキーを使って得ることができる。  
```cs
masterKey = new ExtKey();
masterPubKey = masterKey.Neuter();

//The payment server generate pubkey1
ExtPubKey pubkey1 = masterPubKey.Derive((uint)1);

//You get the private key of pubkey1
ExtKey key1 = masterKey.Derive((uint)1);

//Check it is legit
Console.WriteLine("Generated address : " + pubkey1.PubKey.GetAddress(Network.Main));
Console.WriteLine("Expected address : " + key1.PrivateKey.PubKey.GetAddress(Network.Main));
```  

```
Generated address : 1Jy8nALZNqpf4rFN9TWG2qXapZUBvquFfX
Expected address : 1Jy8nALZNqpf4rFN9TWG2qXapZUBvquFfX
```  
<!--
**ExtPubKey** is similar to **ExtKey** except that it holds a **PubKey** and not a **Key**.
-->
**ExtPubKey** は **ExtKey** に似ている。が、違いは、 **PubKey** （公開鍵）を保持しいるが **Key** （秘密鍵）は保持していない。  



![](../assets/ExtPubKey.png)  

<!--
Now we have seen how Deterministic keys solve our problems, let’s speak about what the “hierarchical” is for.
-->
ここまで、決定性鍵が、どうやって問題を解決してくれるのかを見てきた。では、次は、 **階層的** が何を意味するかについて議論してみよう。  

<!--
In the previous exercise, we have seen that by combining master key + index we could generate another key. We call this process **Derivation**, master key is the **parent key**, and the generated key is called **child key**.
-->
前の演習では、マスターキーとインデックス番号の組み合わせにより 鍵を生成するのを見てきた。このプロセスを **派生** と呼び、マスターキーは **親の鍵** で、生成される鍵は、**子供の鍵** である。  

<!--
However, you can also derivate children from the child key. This is what the “hierarchical” stands for.
-->
しかし、子供の鍵から派生した さらに子供の鍵を複数作ることができる。これが **階層的** の意味である。  

<!--
This is why conceptually more generally you can say: Parent Key + KeyPath => Child Key  
-->
なので、概念的かつ一般的に、 こうやって言うことができる: 親の鍵 ＋ 鍵の階層パス → 子供の鍵  

![](../assets/Derive1.png)  

![](../assets/Derive2.png)  

<!--
In this diagram, you can derivate Child(1,1) from parent in two different way:  
-->
この図に書かれているように、親から ２つのやり方で子供(1,1)を派生させることができる。
```cs
ExtKey parent = new ExtKey();
ExtKey child11 = parent.Derive(1).Derive(1);
```  

もしくは、    

```cs
ExtKey parent = new ExtKey();
ExtKey child11 = parent.Derive(new KeyPath("1/1"));
```  
<!--
So in summary:  
-->
なので、まとめると:

![](../assets/DeriveKeyPath.png)  

<!--
It works the same for **ExtPubKey**.  
-->
**ExtPubKey** も同じように取り扱うことができる。  

<!--
Why do you need hierarchical keys? Because it might be a nice way to classify the type of your keys for multiple accounts. More on [BIP44](https://github.com/bitcoin/bips/blob/master/bip-0044.mediawiki).
-->
なぜ 階層的な鍵が必要になるのだろう？ なぜなら、自分の複数の口座のための鍵を目的に別に分類するのは、良い方法だと思われるからだ。  詳しくは、[BIP44](https://github.com/bitcoin/bips/blob/master/bip-0044.mediawiki)。

<!--
It also permits segmenting account rights across an organization.
-->
そして、 組織の内部において口座の権限を分けて管理することができるようにもなる。  

<!--
Imagine you are CEO of a company. You want control over all wallets, but you don’t want the Accounting department to spend the money from the Marketing department.
-->
自分をある会社のCEOだとしてみよう。すると、会社のウォレットをすべて管理したいだろう。しかし、経理部に、マーケティング部のお金を使ってほしくない。  

<!--
So your first idea would be to generate one hierarchy for each department.  
-->
すると最初に思いつくアイデアは、１つの部署に１つの階層を生成することだろう。  

![](../assets/CeoMarketingAccounting.png)   

<!--
However, in such case, **Accounting** and **Marketing** would be able to recover the CEO’s private key.
-->
しかし、そのような場合、**経理部** と **マーケティング部**  はCEOの秘密鍵を 逆生成できるかもしれない。  

<!--
We define such child keys as **non-hardened**.  
-->
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
<!--
In other words, a **non-hardened key** can “climb” the hierarchy.**Non-hardened keys** should only be used for categorizing accounts that belongs to a **single control**.
-->
別の言い方でいうと、 **非強化鍵** は　階層を "登る" ことができる。**非強化鍵** は単一管理されるタイプの口座にだけ使用されるべきでる。  

<!--
So in our case, the CEO should create a **hardened key**, so the accounting department will not be able to climb.
-->
なので、我々のケースでは、CEOは 強化鍵を作成しなければならない。そうすれば、経理部は、階層を登って秘密鍵を見つけることはできない。  

```cs
ExtKey ceoKey = new ExtKey();
Console.WriteLine("CEO: " + ceoKey.ToString(Network.Main));
ExtKey accountingKey = ceoKey.Derive(0, hardened: true);

ExtPubKey ceoPubkey = ceoKey.Neuter();

ExtKey ceoKeyRecovered = accountingKey.GetParentExtKey(ceoPubkey); //Crash
```  

<!--
You can also create hardened keys by via the **ExtKey.Derivate**(**KeyPath)**, by using an apostrophe after a child’s index:
-->
**ExtKey.Derivate**(**鍵階層パス)** を使うときに、子のインデックス値の後ろにアポストロフィを付けることでも、強化鍵を作成できる。  

```cs
var nonHardened = new KeyPath("1/2/3");
var hardened = new KeyPath("1/2/3'");
```  
<!--
So let’s imagine that the Accounting Department generates 1 parent key for each customer, and a child for each of the customer’s payments.
-->
では、経理部は、顧客ごとに１つの親鍵を生成したと想定してみよう。そして、顧客からの支払いごとにその子供鍵を使う。  

<!--
As the CEO, you want to spend the money on one of these addresses. Here is how you would proceed.  
-->
CEOであるあなたは、そのうちの１つのアドレスから支払をしたいとした場合、こうやって行う。  

```cs
ceoKey = new ExtKey();
string accounting = "1'";
int customerId = 5;
int paymentId = 50;
KeyPath path = new KeyPath(accounting + "/" + customerId + "/" + paymentId);
//Path : "1'/5/50"
ExtKey paymentKey = ceoKey.Derive(path);
```  

<!--
## Mnemonic Code for HD Keys (BIP39) {#mnemonic-code-for-hd-keys-bip39}
-->
## HD鍵のためのネモニック(記憶しやすい)コード(BIP39){#mnemonic-code-for-hd-keys-bip39}
<!--
As you have seen, generating HD keys is easy. However, what if we want an easy way to transmit such key by telephone or hand writing?
-->
これまで見たように、HD鍵を作成することは簡単である。しかし、もし、そんな鍵を電話や手書きで人に伝える簡単な方法があればどうだろうか？  

<!--
Cold wallets like Trezor, generate the HD Keys from a sentence that can easily be written down. They call such sentence “the seed” or “mnemonic”. And it can eventually be protected by a password or a PIN.  
-->
Tresorのようなコールドウォレットは、簡単に書き留められるようなある文章をもとに、HD鍵を生成する。そういった文章は **ザ・シード** と言われたり **ネモニック(記憶文)** と言われる。そして、それはパスワードやPIN番号でさらに暗号化される。

![](../assets/Trezor.png)  

<!--
The language that you use to generate your easy to write sentence is called a **Wordlist**  
-->
書き留めやすい文章を生成するために使用される言葉は、**単語リスト** と呼ばれる。

![](../assets/RootKey.png)  
```cs
Mnemonic mnemo = new Mnemonic(Wordlist.English, WordCount.Twelve);
ExtKey hdRoot = mnemo.DeriveExtKey("my password");
Console.WriteLine(mnemo);
```  

```minute put grant neglect anxiety case globe win famous correct turn link```  

<!--
Now, if you have the mnemonic and the password, you can recover the **hdRoot** key.  
-->

ここで、もし、ネモニックとパスワードを持っていたら、 **階層ルート** の鍵を見つけることができる。  

```cs
mnemo = new Mnemonic("minute put grant neglect anxiety case globe win famous correct turn link",
                Wordlist.English);
hdRoot = mnemo.DeriveExtKey("my password");
```  
<!--
Currently supported **wordlist** are, English, Japanese, Spanish, Chinese (simplified and traditional).  
-->
現時点でサポートされている **単語リスト** の言語は、英語、日本語、スペイン語、中国語（簡体字と繁体字）である。  

<!--
## Dark Wallet {#dark-wallet}
-->
## ダークウォレット {#dark-wallet}

<!--
This name is unfortunate since there is nothing dark about it, and it attracts unwanted attention and concerns. Dark Wallet is a practical solution that fix our two initial problems:
-->
この名前がついたのは、不運である。なぜなら、まったくもってこのウォレットは、ダークではないが、不要な注目や懸念を生んでしまっている。ダークウォレットは、我々の最初の２つの問題に対する実践的な解決策である。  

<!--
*   Prevent outdated backups
*   Delegating key / address generation to an untrusted peer
-->
*   古くなって使えなくなるウォレットのバックアップを防ぐ
*   鍵とアドレスの生成を信用していない相手に移譲すること

<!--
But it has a bonus killer feature.
-->
しかし、このウオレットは、ボーナス的なすごい機能をもつのだ。  

<!--
You have to share only one address with the world (called **StealthAddress**), without leaking any privacy.
-->
あなたは、唯一（ **ステルスアドレス** と呼ばれる）アドレスだけを、ほかの人たちにシェアするだけでよいのだ。よって、プライバシーを失うことがない。  

<!--
Let’s remind us that if you share one **BitcoinAddress** with everybody, then all can see your balance by consulting the blockchain… That’s not the case with a **StealthAddress**.
-->
ここで思い出してほしいのは、１つの **ビットコインアドレス** をみんなと共有すると、すべての人たちは、ブロックチェーンをみれば、あなたの持っているビットコインの残高を知ることができる。しかし、**ステルスアドレス** を使えば、それには当たらない。  

<!--
This is a real shame it was labeled as **dark** since it solves partially the important problem of privacy leaking caused by the pseudo-anonymity of Bitcoin. A better name would have been: **One Address**.
-->
このウォレットがダークと命名されたのは、本多王に残念なことである。なぜかというと、それは 部分的にでもビットコインの疑似匿名性が原因として起きるプライバシー問題を解決するからである。よりよい名前は、**ワン・アドレス** だったかもしれない。  

<!--
In Dark Wallet terminology, here are the different actors:
-->
ダークウォレットの用語を使うと、以下のように 参加者を表現する:

<!--
*   The **Payer** knows the **StealthAddress** of the **Receiver**
*   The **Receiver** knows the **Spend Key**, a secret that will allow him to spend the coins he receives from one of such transaction.
*   **Scanner** knows the **Scan Key**, a secret that allows him to detect the transactions those belong to the **Receiver**.
-->
*   **支払者** は、 **受領者** の **ステルスアドレス** を知っている。
*   **受領者** は、 **スペンド鍵** を知っていて、その秘密のコードで、それにより彼が受け取るコインを消費することができる。
*   **スキャナー** は、**スキャン鍵** を知っていて、 その秘密のコードで、 その **受領者** に属する取引を特定することができる。

<!--
The rest is operational details.Underneath, this **StealthAddress** is composed of one or several **Spend PubKey** (for multi sig), and one **Scan PubKey**.  
-->
以降は、操作の詳細になる。 内部的には、このステルスアドレスは、１つもしくは複数(複数署名の場合)の **スペンド公開鍵** と１つの **スキャン公開鍵** で構成されている。

![](../assets/StealthAddress.png)  

```cs
var scanKey = new Key();
var spendKey = new Key();
BitcoinStealthAddress stealthAddress
    = new BitcoinStealthAddress
        (
        scanKey: scanKey.PubKey,
        pubKeys: new[] { spendKey.PubKey },
        signatureCount: 1,
        bitfield: null,
        network: Network.Main);
```  
<!--
The **payer**, will take your **StealthAddress**, generate a temporary key called **Ephem Key** and will generate a **Stealth Pub Key**, from which the Bitcoin address to which the payment will be done is generated.  
-->
**支払者** は、あなたの **ステルスアドレス** を使って、一時的に使用する **エフェム鍵** を作成し、そこから **ステルス公開鍵** を生成する。それから、支払が行われるビットコインアドレスが作成される。  

![](../assets/EphemKey.png)

<!--
Then, he will package the **Ephem PubKey** in a **Stealth Metadata** object embedded that in the OP_RETURN of the transaction (as we have done for the first challenge)
-->
そして、（最初のチャレンジでやったように）トランザクションのOP_RETURNに埋め込まれた **ステルスメタデータ** の中に **エフェム公開鍵** をパッケージする。  

<!--
He will also add the output to the generated bitcoin address. (the address of the **Stealth pub key**)
-->
さらに、トランザクション・アウトプットを生成されたビットコインアドレス（ **ステルス公開鍵** のアドレス）へ追加する。

![](../assets/StealthMetadata.png)  

```cs
var ephemKey = new Key();
Transaction transaction = new Transaction();
stealthAddress.SendTo(transaction, Money.Coins(1.0m), ephemKey);
Console.WriteLine(transaction);
```  
<!--
The creation of the **EphemKey** being an implementation detail, you can omit it, NBitcoin will generate one automatically:  
-->
**エフェム鍵** の生成の実装については、詳細すぎるので、無視してよいだろう。NBitcoinが自動的に生成してくれる:  

```cs
Transaction transaction = new Transaction();
stealthAddress.SendTo(transaction, Money.Coins(1.0m));
Console.WriteLine(transaction);
```  

```json
{
  "hash": "7772b0ad19acd1bd2b0330238a898fe021486315bd1e15f4154cd3931a4940f9",
  "ver": 1,
  "vin_sz": 0,
  "vout_sz": 2,
  "lock_time": 0,
  "size": 93,
  "in": [],
  "out": [
    {
      "value": "0.00000000",
      "scriptPubKey": "OP_RETURN 060000000002b9266f15e8c6598e7f25d3262969a774df32b9b0b50fea44fc8d914c68176f3e"
    },
    {
      "value": "1.00000000",
      "scriptPubKey": "OP_DUP OP_HASH16051f68af989f5bf24259c519829f46c7f2935b756 OP_EQUALVERIFY OP_CHECKSIG"
    }
  ]
}
```  
<!--
Then the payer add and signs the inputs, then sends the transaction on the network.
-->
そして、支払者は、トランザクション・インプットをトランザクションに追加し署名をする。それからビットコインネットワークにトランザクションを送信する。  

<!--
The **Scanner** knowing the **StealthAddress** and the **Scan Key** can recover the **Stealth PubKey** and so expected **BitcoinAddress** payment.  
-->
**ステルスアドレス** と **スキャン鍵** を知る **スキャナー** は、**ステルス公開鍵** を取得できるので、予定された **ビットコインアドレス** への支払いも確認できる。   
![](../assets/ScannerRecover.png)  

<!--
Then the scanner checks if one of the output of the transaction correspond to such address. If it is, then **Scanner** notifies the **Receiver** about the transaction.
-->
それから **スキャナー** は、トランザクションのアウトプットの１つがそのアドレスに対するものかをチェックし、もし、そうであるなら **スキャナー** は、 **受領者** にたいしてトランザクションについて通知する。  

<!--
The **Receiver** can then get the private key of the address with his **Spend Key**.  
-->
**受領者** は、**スペンド鍵** を使うことでそのアドレスの秘密鍵を取得できる。  

![](../assets/ReceiverStealth.png)  

<!--
The code explaining how, as a Scanner, to scan a transaction and how, as a Receiver, to uncover the private key, will be explained later in the **TransactionBuilder** (Other types of ownership) part.
-->
スキャナーとして、どうやってトランザクションをスキャンするか。レシーバーとして、どうやって秘密鍵を見つけるか、についてのコードは、あとの **トランザクションビルダー** の説明する。  

<!--
It should be noted that a **StealthAddress** can have multiple **spend pubkeys**, in which case, the address represent a multi sig.
-->
ここで知ってほしいのは、**ステルスアドレス** は、複数の **スペンド公開鍵** を持つことができることだ。それを使うばあい、そのアドレスは、マルチシグ用（複数人による署名）のアドレスということになる。  

<!--
One limit of Dark Wallet is the use of **OP_RETURN**, so we can’t easily embed arbitrary data in the transaction as we have done for in Bitcoin Transfer. (Current bitcoin rules allows only one OP_RETURN of 40 bytes, soon 80, per transaction)  
-->
ダークウォレットの唯一の制限事項は、**OP_RETURN** を使用するということである。すなわち、トランザクションに任意のデータを埋め込むことが簡単にできない。(現在のビットコインのルールでは。OP_RETURNには４０バイトしか許されていない、もうすぐ１トランザクションに対して８０バイトになる予定だ。)

<!--
> ([Stackoverflow](http://bitcoin.stackexchange.com/a/29648/26859)) As I understand it, the "stealth address" is intended to address a very specific problem. If you wish to solicit payments from the public, say by posting a donation address on your website, then everyone can see on the block chain that all those payments went to you, and perhaps try to track how you spend them.  
>
With a stealth address, you ask payers to generate a unique address in such a way that you (using some additional data which is attached to the transaction) can deduce the corresponding private key. So although you publish a single "stealth address" on your website, the block chain sees all your incoming payments as going to separate addresses and has no way to correlate them. (Of course, any individual payer knows their payment went to you, and can trace how you spend it, but they don't learn anything about other people's payments to you.)  
>
But you can get the same effect another way: just give each payer a unique address. Rather than posting a single public donation address on your website, have a button that generates a new unique address and saves the private key, or selects the next address from a long list of pre-generated addresses (whose private keys you hold somewhere safe). Just as before, the payments all go to separate addresses and there is no way to correlate them, nor for one payer to see that other payments went to you.
>
So the only difference with stealth addresses is essentially to move the chore of producing a unique address from the server to the client. Indeed, in some ways stealth addresses may be worse, since very few people use them, and if you are known to be one of them, it will be easier to connect stealth transactions with you.  
>
It doesn't provide "100% anonymity". The fundamental anonymity weakness of Bitcoin remains - that everyone can follow the chain of payments, and if you know something about one transaction or the parties to it, you can deduce something about where those coins came from or where they went.
-->
> ([Stackoverflow](http://bitcoin.stackexchange.com/a/29648/26859)) 私が理解するに、"ステルスアドレス"とは、ある特定の問題に対応するために作られた。一般の人々からお金を集めたいと思ったら、例えば寄付のためのアドレスを自分のウエブサイトに載せるとか、そうすると、ブロックチェーン上ですべての支払いの情報が見られてしまう。もしかしたら、あなたがそのアドレスから何に支払ったかを追跡されるだろう。
>
ステルスアドレスを使うケースでは、支払者にユニークなアドレスを生成するように依頼する。そうすることで、そのトランザクションにくっついた追加情報を使うだけで、対応した秘密鍵を推測することができる。そして、あなたは、１つのステルスアドレスをウェブサイトに公開したにもかかわらず、ブロックチェイン上では、すべて違ったアドレスに支払が行われることになり、それぞれアドレスの関連性を見つけることは不可能だ。（もちろん支払者は、支払先アドレスをしっているので。そこからどこにお金を送ったかは見ることができるが、ほかの支払者には、それはみることができない。）
>
しかし、ほかの方法でもどうような効果を得ることはできる:それぞれの支払者にたいして、固有の別々のアドレスを渡せばよい。１つの公開アドレスをウェブサイト載せる代わりに、例えば、押されるごとの新しいアドレスを生成し、秘密鍵を保存するようなボタンを置いておくとか、事前に用意しておいた公開アドレスのリストから次々とアドレスを選んで使うようなボタンを配置する（もちろん秘密鍵は、安全な場所に保存しておく）。ステルスアドレスを使用するときと同様に、 支払はすべて個別のアドレスへ行われ、アドレス間の相関は全くないし、１りの支払者が、別の支払情報を見ることもできない。
>
なので、ステルスアドレスを使用することとの唯一の違いは、サーバーが面倒なアドレス作成の処理をしなくてよいということである。実は、ある意味でステルスアドレスを使用するのは悪いことかもしれない。なぜなら、ステルスアドレスを使用する人が少ないため、もしあなたがステルスアドレスを使用してると知られてしまうと、ステルスアドレスのトランザクションは、あなたのものであると分かってしまうからだ。
>
この方法は"１００％の匿名性"を提供しない。ビットコインの基本的な匿名性に対する弱さは、残ってしまう。すなわち、だれでも、支払のいチェーン、つながりを見ることができるということだ。なので、もし、あなたが、あるトランザクションそのもの、もしくは、そのトランザクションの関係者だと知るならば、そのビットコインがどこからきて、どこへいったのを見ることができるのだ。
