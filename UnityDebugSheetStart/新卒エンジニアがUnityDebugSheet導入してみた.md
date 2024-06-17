# 序論
どうも！ AM3・クライアントエンジニアの平塚です。

時に、

皆さんのプロジェクトでは **デバッグ機能** どうされてますでしょうか？<br>
配属2カ月ちょいの新卒エンジニアは他PJの開発事情など当然のように知らないのですが、<br>
AM3ではこれまで**SRDebugger**というアセットでデバッグ機能を管理してきました。

アイテム付与やストーリー解放といったデバッグコマンドへのアクセスを提供する他、コンソールやプロファイラを実機でも確認可能にするなど、とっても便利な代物です。

しかし...やがてSRDebuggerに対して、ある不満がチーム内で浮かび上がってきました。<br>
それは **「デバッグコマンドが増えすぎてすごく見づらい！」** というものです。

![SRDebugger](https://raw.githubusercontent.com/kamahir0/TechArticle/master/UnityDebugSheetStart/IMG_7799.PNG)
ﾐﾁｨ...

ここまで多すぎると目当てのデバッグコマンドを見つけるのも一苦労ですね。
そこで、今回導入することになったのが**UnityDebugSheet**です。

![UnityDebugSheet](https://raw.githubusercontent.com/kamahir0/TechArticle/master/UnityDebugSheetStart/IMG_7800.PNG)

UnityDebugSheetはデバッグコマンドへのアクセスを階層式に格納するシステムを提供します。目当てのコマンドが見つけやすくなるので開発効率アップが期待できます。

今回の記事は、そんなUnityDebugSheetを新卒エンジニアが導入してみた！というお話です。

# 2つの概念
UnityDebugSheetを使う上で知っておくべき「セル」「ページ」という2つの概念があります。簡単にご説明しましょう。

## セル
デバッグシート上に配置される**ボタン**、**スイッチ**、**スライダー**といった各要素です。<br>
ボタンを押すことで何らかの処理を呼び出したり、スイッチやスライダーならbool・floatなどの値をここから取得することも可能です。また、完全オリジナルのセルを作ることもできます。
![cell](https://raw.githubusercontent.com/kamahir0/TechArticle/master/UnityDebugSheetStart/IMG_3791.PNG)

## ページ
セルの集合体であり、デバッグシートの階層の単位であり、常に1つ表示されています。
上述の**セル**には「**ページリンクボタン**」というボタンが用意されており、これを押すことでページから別のページへと進むことができます。<br>
また、デバッグシートの最も根本の階層部にあたるページは特別に**ルートページ**と呼ばれます。
![page](https://raw.githubusercontent.com/kamahir0/TechArticle/master/UnityDebugSheetStart/IMG_3790.PNG)

# 良い点・悪い点
## 良い点

## 悪い点

# 導入にあたり行った工夫
