## P2W\* over P2SH {#p2w-over-p2sh}

ビットコインの要求をスクリプト化するための**witness scriptPubKey**を使うことが魅力的だと思われてきている一方で、実のところ最近のウォレットのほとんどが、P2PKHあるいはP2SHしかサポートしていない。

segwitの利点を利用するために、古いソフトウェアが共存している間は、P2SH上でP2Wを使うことができる。古いBitcoinCoreを使っているノードにとっては、それは普通のP2SHを利用した支払いに見える。

どんな**P2W\***でも**P2SH上のP2W\***に変換できる。以下の手順を踏む。

1. **ScriptPubKey**を、同じ内容を示すP2SHで置き換える
2. 変換前の**ScriptPubKey**はトランザクションインプットの**scriptSig**の中にただ1つのプッシュとして示される。
3. すべての他のデータはトランザクションインプットのwitnessに示される。

心配しないでほしいのだが、もしこれが複雑だと思われたとしても、TransactionBuilderによって効果的にトランザクションを生成できる。

P2SH上のP2WPKH、または略称の**P2SH\(P2WPKH\)**の例を見てみよう。

**ScriptPubKey**を表示する。

```cs
var key = new Key();
Console.WriteLine(key.PubKey.WitHash.ScriptPubKey.Hash.ScriptPubKey);
```

> 注意：これはとても畏怖の念を感じさせるコードだ。

そうするとよく親しみのあるP2SHの**scriptPubKey**が表示される。

```
OP_HASH160 b19da5ca6e7243d4ec8eab07b713ff8768a44145 OP_EQUAL
```

そしてこのアウトプットを使う、署名されたトランザクションは以下のようになる。

```json
"in": [
    {
      "prev_out": {
        "hash": "674ece694e5e28956138efacab96fc0bffd7c6cc1af7bb2729943fedf8f0b8b9",
        "n": 0
      },
      "scriptSig": "001404100ab485c95701bf0f4d73e3fe7d69ecc4f0ea",
      "witness": "3045022100f4c14cf383c0c97bbdaf520ea06f7db6c61e0effbc4bd3dfea036a90272f6cce022055b0fc058759a7961e718d48a3dc4dd5580fffc310557925a0865dbe467a835901 0205b956a5afe8f34a01337f0949f5733b5e376caaea57c9624e40e739a0b1d16c"
    }
  ],
```

**scriptSig**は先ほど出てきたScriptPubKey（言い換えると、**key.PubKey.WitHash.ScriptPubKey**）のP2SH化されたredeem scriptのプッシュでしかない。witnessは完全に通常の**P2WPKH**による支払いと同じになっている。

NBitcoinでは、**P2SH\(P2WPKH\)**に署名することは、ScriptCoinを用いた通常のP2SHとほぼ同じようなものだ。

同じ原則に則って、**P2SH\(P2WSH\)**がどのように見えるかを見てみよう。この場合、2つの異なるredeem scriptを扱わなければならない。トランザクションインプットの**scriptSig**に入れる必要のある**P2SHのredeem script**と、witnessに入れる必要のある**P2WSHのredeem script**だ。

最初のルールにもとづいて、**scriptPubKey**を表示してみよう。

1. **ScriptPubKey**をP2SHと同等の情報で置き換える。

   ```cs
   var key = new Key();
   Console.WriteLine(key.PubKey.ScriptPubKey.WitHash.ScriptPubKey.Hash.ScriptPubKey);
   ```

   ```
   OP_HASH160 d06c0058175952afecc56d26ed16558b1ed40e42 OP_EQUAL
   ```

   > **注意**：理解できるから、ここでキレて回線切断しないで！

2. 置き換え前の**ScriptPubKey**はトランザクションインプットの**scriptSig**にたった1つのプッシュとして記録される。

3. すべての他のデータはトランザクションインプットのwitnessにプッシュされる。

項番3の「**他のデータ**」というのは、P2WSHにおける支払いの文脈では、**P2WSHのredeem script**のプッシュに続く、**P2WSHのredeem script**のパラメータを意味する。

```json
"in": [
    {
      "prev_out": {
        "hash": "1d23fa744a26cf6433f0841e9de7e088cf95e6f953e584b98d0de6ef4216765f",
        "n": 0
      },
      "scriptSig": "0020c54eb79829b2e26b71d15fd3b490b6e95cbdab361a45eed2cdfe642497480a6c",
      "witness": "3045022100d7570c3bf87149a0be3ba2e8bfccbdd35c3da44f741695e9962014795fabc4fc02203183cfa55a85728520b0f1ac59ac3ffa1a8526634fe619f99fac0f76016f366e01 2103146e87d7fcc81f3e044f97c6b262c01826f40a9ab9acae0f689983a5890a1f4dac"
    }
  ],
```

要約すると、P2SHのRedeem Scriptは、通常のP2WSHによる支払いとしてP2WSHのscriptPubKeyを得るためにハッシュされる。そして通常のP2SHの支払いとして、P2WSHのscriptPubKeyはハッシュされて置き換わり、まさにP2SHを作るために使われる。

もし、P2SH/P2WSH/P2SH\(P2WSH\)/P2SH\(P2WPKH\)が複雑に思えていても、怖がることはない。  
NBitcoinでは、**すべての支払い形式において**、**ScriptCoin**を作ることだけしか求めない。それは**P2SH**の章で説明したとおりで、P2WSHかP2SHのRedeem ScriptとScriptPubKeyを与えてやれば作ることができる。

NBitcoinに関して言えば、使いたいと思っているトランザクションアウトプットを使い、正しいredeem scriptがあれば、前章の「**Multi Sig」**の章で説明されたように、そして次の「**Using the TransactionBuilder**」の章でも説明するが、**TransactionBuilder**が正しい署名の仕方を把握してくれる。

![](../assets/ScriptCoin.png)

**P2SH/P2WSH/P2SH\(P2WSH\)/P2SH\(P2WPKH\)それぞれに互換性があるのだ。**

P2WPKHまたはP2WSHの支払いの例をさらに見たい場合は、[http://n.bitcoin.ninja/checkscript](http://n.bitcoin.ninja/checkscript)で見られる。

