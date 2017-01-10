## Ricardian contracts {#ricardian-contracts}

本章は[Coinprismのブログ](http://blog.coinprism.com/2014/12/10/colored-coins-and-ricardian-contracts/)で書いた記事のコピーとなっている。書いた時点ではNBitcoinはRicardian Contractsに関連するコードはまったくなかった。

### Ricardian Contractとはなにか {#what-is-a-ricardian-contract}

一般的にアセットは特定の条件下で発行者に償還される権利を表現するものをいう。

* 会社の株式によって配当をもらう権利が与えられる
* 公債によって満期になれば元本を取り返す権利と毎期に利子を得られる権利を与えられる
* 投票権によって何らかの集合体を決定するために投票する権利を与えられる（会社や選挙）
* これらのミックスもありえる：株式によって会社の社長を選ぶ投票券を与えられもするだろう。

このような権利は典型的に契約の中に列挙され、発行者によって署名される。（必要であれば公証人のように信頼された第三者によっても署名される）

Ricardian Contractは暗号によって発行者から署名され、アセットと切り離すことができない契約のことをいう。

だからそういった契約は否定したり改ざんしたりすることはできない。明白に発行者によって署名されているからだ。  
このような契約は発行者と執行者との間で信頼された状態が保てるし、もしくは公表することもできる。

オープンアセットはすでにコアプロトコルを変えることなく、それらすべてをサポートしている。本章でどのように実装されているかを述べる。

### オープンアセット中のRicardian Contract {#ricardian-contract-inside-open-asset}

[このサイト](http://iang.org/papers/ricardian_contract.html)にRicardian Contractの公式の定義がある。

1. 発行者によって契約がアセットの所持者に示され、
2. アセットの所持者が有し、発行者によって執行される様々な権利に対応し、
3. 簡単に人が読めて（契約書のように）
4. プログラムによって処理でき（データベースのように解釈ができ）
5. デジタル署名されていて
6. 鍵とサーバーの情報を含み、
7. ユニークで安全な識別子と結び付けられている。

アセットIDは以下のような方法でオープンアセットによって特定される。

`AssetId = Hash160(ScriptPubKey)`

**ScriptPubKey**をP2SHで作ってみよう。

`ScriptPubKey = OP_HASH160 Hash(RedeemScript) OP_EQUAL`

RedeemScriptはこうだ。

`RedeemScript = HASH160(RicardianContract) OP_DROP IssuerScript`

**IssureScript**とは1人の発行者に対しては古典的なP2PKHになるが、発行に対して何人かの合意が必要ならばマルチシグ（たとえば発行者と公証人）となる。

Bitcoin 0.10から、IssureScriptは任意となり内容はなんでも良いということは注釈しておきたい。

**RicardianContract**は任意で、秘匿しておける。契約をもっている人なら誰でも、ScriptPubKeyのハッシュのおかげでその契約がこのアセットに適用されることを証明できる。

しかし、RicardianContractを可視化し、Asset Definition Protocolをもつウォレットクライアントで確認できるようにしてみよう。

A、BまたはCに対しての選挙権を発行していることにしてみよう。

オープンアセットのマーカーに、次のアセット定義のURL：`u=http://issuer.com/contract`を追加してみよう。

[http://issuer.com/contract](http://issuer.com/contract)で、次のような[アセットの定義ファイル](https://github.com/OpenAssets/open-assets-protocol/blob/8b945ba68a781358947325ac008cdd740c89adb3/asset-definition-protocol.mediawiki)を作ってみよう。

```json
{
    "IssuerScript" : IssuerScript,
    "name" : "MyAsset",
    "contract_url" : "http://issuer.com/readableContract",
    "contract_hash" : "DKDKocezifefiouOIUOIUOIufoiez980980",
    "Type" : "Vote",
    "Candidates" : ["A","B","C"],
    "Validity" : "10 jan 2015"
}
```

これでRicardianContractを定義できる。

`RicardianContract = AssetDefinitionFile`

This terminate our RicardianContract implemented in OA.

### チェックリスト {#check-list}

* **発行者によって契約がアセットの所持者に示され、**  
  契約が発行者によって主体的に執行され、変更されず、発行者が新しいアセットを発行する都度署名される。

* **アセットの所持者が有し、発行者によって執行される様々な権利に対応し、**  
  このサンプルの中での権利は2015年1月10日までに候補者A、BまたはCに投票する権利を与えられる。

* **簡単に人が読めて（契約書のように）**  
  The human readable contract is in the contract\_url, but the JSON might be enough.

* **プログラムによって処理でき（データベースのように解釈ができ）**  
  投票の詳細はJSONフォーマットの中の**AssetDefinitionFile**のうちにあり、契約の真正性は**IssureScript**と**ScriptPubKey**のハッシュを使ってソフトウェアによって確かめられる。

* **デジタル署名されていて**  
  **ScriptPubKey**は、発行者がアセット、つまり契約のハッシュまたはさらに言えば契約自体を発行するときに署名される。

* **鍵とサーバーの情報を含む。 informationIssureScript**は契約に含まれる。

* **ユニークで安全な識別子と結び付けられている。**  
  **アセットID**は、変更できず一意である**ScriptPubKeyのハッシュ**によって定義される。

### 何のためのものか？ {#what-is-it-for}

Ricardian Contractがないと、悪意のあるアセット発行者が、アセットの定義ファイルを変更したり否認したりすることが簡単になってしまう。

Ricardian Contractによって否認拒否を強制し、契約を変更できないようにして、アセットの償還を受ける者と発行者との間の裁定を容易にする。

また、アセットの定義ファイルは変更できないから、悪意のある発行者による、契約の破壊行為を防止しながら、アセットを償還者の手元で保管できるようにもなる。

