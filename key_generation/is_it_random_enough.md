## Is it random enough? {#is-it-random-enough}

**new Key()** でコンストラクターを呼び出すと、内部ではPRNG(疑似乱数生成器)を使って秘密鍵を生成している。WindowsOS上では、Windows Crypto APIの.Netラッパーである **RNGCryptoServiceProvider** を使用している。

アンドロイドでは、私は **SecureRandom** を使用する。実際は **RandomUtils.Random** を使って自分独自の実装を使うことができる。

iOS上では、私はまだ実装したことはないが、自分で **IRandom** の実装クラスを作る必要がある。

コンピューターにとってランダムにすることは難しい。しかし、1番大きな問題は、ある一連の数値が、本当にランダムかどうかを知ることが不可能だということである。

もし、マルウェアがあなたのPRNGを改ざんした場合（そうすると、あなたが生成する数値を予測できる）、手遅れになって初めて改ざんされたことに気づくことになる。

これは、クロスプラットフォームもしくはネイティブ実装のPRNG（例えばコンピュータのクロックとCPUのスピードの組み合わせを利用）は危険であることを意味する。しかし、手遅れになるまで、それを知る由はない。

パフォーマンス上の理由から ほとんどのPRNGは同じように機能する。**シード** とよばれるランダムな数値が1つ選ばれ、呼び出す度に、結果が予測可能な式によって次の値が生成される。

シードのランダムさの度合は、**エントロピー** と呼ばれる計測値で定義されるが、エントロピー度も、監視している人にも依存する。

例えば、あなたが自分のクロック時間をもとにシード値を生成したとしよう。
そして、クロックが1ミリ秒の精度を持ったとしよう。（現実には15ミリ秒以上。）

もしあなたが先週鍵を生成したと、攻撃者が知っているとすると、シード値は、
1000 \* 60 \* 60 \* 24 \* 7 = 604800000 通りある。

その攻撃者にとって、エントロピーは、log<sub>2</sub>(604800000) = 29.17 ビットである。

その程度の回数だと順番に処理するとして、私の自宅のコンピュータで処理させても2秒以下しかかからない。このように順番に全可能性を当たってみる処理のことを”総当たり式”と呼ぶ。

でも、例えば、シード値を生成するのに、クロック時間とプロセスIDを使ったとする。
そして、1024個の個別のプロセスIDが存在すると想像してみよう。

今度は、攻撃者は、604800000 \* 1024 回を順番に当たっていく必要があり、それには2000秒かかる。
さて、ここに、コンピューターを起動した日時の情報も足してみよう。攻撃者は私が今日起動したと知っているとしても、86400000 通りの可能性が追加できる。

これで、攻撃者は、604800000 \* 1024 \* 86400000 = 5,35088E+19通りの可能性に当たる必要がある。
しかし、覚えておいてほしいのは、もし攻撃者が私のコンピューターに侵入可能であれば、この追加の情報を取得できるので、可能性の数を減らし、エントロピーを下げることができる。

エントロピーは、log<sub>2</sub>(数値の取りえる可能性の数)で計算できるので、log<sub>2</sub>(5,35088E+19)= 65 ビットとなる。

これは十分だろうか。攻撃者が可能性を左右する情報をこれ以上持っていないと仮定するならば、多分そうだろう。

しかし、公開鍵のハッシュ値が20バイト = 160ビット ということは、すべてのアドレスの数よりまだ小さい。もう少し頑張れるかもしれない。

> **注意:** エントロピーを増やすことは線形的に難しいが、エントロピーを解読するのは、指数関数的に難しくなる。

エントロピーを生成する面白い方法は、人間を介在させるやり方だ。例えば、マウスを動かすような動作だ。

もし、あなたがプラットフォームのPRNGを完全に信頼できない（それはそれほど[おかしなことではない](http://android-developers.blogspot.fr/2013/08/some-securerandom-thoughts.html)）なら、NBitcoinが使用するPRNGからの出力に対して、エントロピーを足すことができる。

```cs
RandomUtils.AddEntropy("hello");
RandomUtils.AddEntropy(new byte[] { 1, 2, 3 });
var nsaProofKey = new Key();
```

あなたが **AddEntropy(data)** を呼び出したときにNBitcoinが処理するのは、
**additionalEntropy = SHA(SHA(data) ^ additionalEntropy)** だ。

そして、新しい数値を取得するときの結果は:
**result = SHA(PRNG() ^ additionalEntropy)** だ。

## 鍵導出関数 {#key-derivation-function}

しかし、最も重要なことは、可能性の数の多さではなく、攻撃者があなたの鍵を破ることに成功するのに、どれくらいの時間が必要かということである。そこでKDFの登場である。

KDF もしくは **鍵導出関数(Key Derivation Function)** とは、たとえエントロピーが低くても、より強い鍵をもつための方法である。

シード値を生成してみたいと想像してみよう。そして攻撃者が1千万個の可能性があることを知っていたとする。
そのようなシードであれば通常、クラックして破るのは非常に簡単だ。

しかし、もしあなたがその順番処理を遅くすることができたとしたらどうだろう？
KDFはハッシュ関数で意図的にコンピューターの演算リソースを無駄に使わせることができる。
これが例だ:

```cs
var derived = SCrypt.BitcoinComputeDerivedKey("hello", new byte[] { 1, 2, 3 });
RandomUtils.AddEntropy(derived);
```

もし仮に、攻撃者があなたのエントロピーの元が５つの文字だと知っていたとしても、Scryptを実行して合っているか可能性を確認しないといけない。私のコンピューターでは5秒かかる。

結局のところ、どういうことかというと、疑似乱数発生器に悪さをされることを極端に心配する必要はない。エントロピーの増加とKDFの使用、両方をすることで攻撃を弱めることができる。
覚えておいてほしいのは攻撃者は、あなたもしくはあなたのシステムの情報を集めることで、エントロピーを減らすことができるということだ。
もし、タイムスタンプを元にエントロピーを作るとして、あなたが先週キーを生成し、しかも午前９時から午後６時までしかコンピューターを使用しないということを攻撃者が知ってしまうと、エントロピーを減らすことになる。

前段で、特別なKDFである**Scrypt**について話した。そこで言及したとおり、KDFの目的は 総当たり攻撃をコストがかかるものにすることだ。

なので、KDFを使ってパスワードで、秘密鍵を暗号化する標準の方法がすでにあるとしても、別に驚くことではないだろう。それが、[BIP38](http://www.codeproject.com/Articles/775226/NBitcoin-Cryptography-Part)だ。

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

このような暗号化は、2つのケースで使用される。:

* 鍵の保存場所の提供者を信用していない。（ハッキングされるかもしれないから。）
* 他の人のために鍵を保存しようとしている。（あなたはその人の鍵を知りたくない。）

もし、自分のストレージを持っているなら、データベースが提供する暗号化で十分だろう。

もし、あなたのサーバーが復号化の機能を持つ場合、気を付けなければいけない。もしかすると、攻撃者が大量の鍵を復号化させるようなDDOS攻撃を仕掛けてくるかもしれないからだ。

可能であるならば、復号化は最終利用者に移譲すべきである。