## P2WSH \(Pay to Witness Script Hash\) {#p2wsh-pay-to-witness-script-hash}

P2PKHとP2WPKHとの関係と同じで、P2SHとP2WSHとのちがいはP2SHにおける支払いで`scriptSig`に記録されていたものの場所と、生成される`scriptPubKey`だけだ。

`scriptPubKey`は以下のように変わる。まずP2SHの形式は以下のとおり。

`OP_HASH160 10f400e996c34410d02ae76639cbf64f4bdf2def OP_EQUAL`

それからP2WSHでは以下のように変わる。

`0 e4d3d21bab744d90cd857f56833252000ac0fade318136b713994b9319562467`

以下のコードでこのscriptPubKeyを導出できる。

```cs
var key = new Key();
Console.WriteLine(key.PubKey.ScriptPubKey.WitHash.ScriptPubKey);
```

そしてP2SHで`scriptSig`（署名とredeem Scriptの合成）にあったものは、`witness`に移動する。

```json
"in": [
    {
      "prev_out": {
        "hash": "ffa2826ba2c9a178f7ced0737b559410364a62a41b16440beb299754114888c4",
        "n": 0
      },
      "scriptSig": "",
      "witness": "304402203a4d9f42c190682826ead3f88d9d87e8c47db57f5c272637441bafe11d5ad8a302206ac21b2bfe831216059ac4c91ec3e4458c78190613802975f5da5d11b55a69c601 210243b3760ce117a85540d88fa9d3d605338d4689bed1217e1fa84c78c22999fe08ac"
    }
  ]
```

P2SHの支払いで説明したとおり、P2WSHでもまったく同様で署名するときは`ScriptCoin`クラスを使う。

