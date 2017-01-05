## Project Setup {#project-setup}

説明を始める前に、どのようにVisual StudioまたはXamarin等でプロジェクトをセットアップするかを示す。

1. プロジェクトを作成する（.NET 4.5以上）
2. ソリューションエクスプローラーの「パッケージ」で右クリックし、「パッケージの追加」をクリック
3. Windowsでは「**NBitcoin**」、MacやLinuxでは「**NBitcoin.Mono**」を追加する。
   ![](../assets/nuget.png)  

> 参考 **:** MacやLinuxで「NBitcoin」をインストールしてしまうと、必要なクラスが揃わないので注意すること。

NBitcoinは、この本のメイン筆者であるNicolas Dorierがメンテナンスしている、.NETで作られたオープンソースライブラリである。C\#でビットコインのソフトウェアを開発するのであれば、このライブラリを使うべきだろう。NBitcoinはクロスプラットフォームアプリケーションをサポートする。

### NBitcoinのソースコードをデバッグする方法（オプション）

NBitcoinはより理解が進むようにそのコードをデバッグすることができる。この特徴を使えるようにするためには、設定で「source server support」をチェックする必要がある。  
![](../assets/visualstudio_enablesourceserversupport.png)

そうすると、もしNBitcoinのコードにステップインをすると、ソースコードがGitHubから自動的に取得され、visual studioのデバッガーに表示される。

