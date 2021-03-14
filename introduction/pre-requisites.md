# 読み始める前に確認しておくこと {#pre-requisites}

## スキル {#skills}

* 関数型プログラミングとオブジェクト指向プログラミングに抵抗がない。
* C\#の基礎を理解していると良いが、C\#はJavaや他のC型言語習得者にも読みやすいと思われる。
* 数学の知識は必要ない。ただし、この本では、安全なサービスを作るために必要なレベル以上の暗号学の知識はカバーしない。
* ビットコインに対する深い知識は必要ない。

## ツール {#tools}

* 以下のいずれかの開発環境をインストールすること
  * [Visual Studio Community Edition](https://www.visualstudio.com/)（Windows）
  * [Xamarin Studio](https://store.xamarin.com) （Mac and Linux）
  * [Visual Studio Code](https://code.visualstudio.com/)
* [Bitcoin Core](https://bitcoin.org/en/bitcoin-core/) 持っていれば理想だが、持ってなくても読み進められる。

> **ヒント:** もしハードディスクの容量が足りなくてフルノードに躊躇してるのであれば、pruningモードでBitcoin Coreを動かすことを考えてほしい。古い履歴を廃棄すること以外は、フルノード（ビットコインコアをインストールしていて、すべての取引履歴＝ブロックを持っているノード）と同じ動きをする。
