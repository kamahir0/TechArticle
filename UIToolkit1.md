UI Toolkitにやや入門してみた【エディタ拡張】

最近UI Toolkitをいじっていまして、エディタウィンドウを作れるようになったのでまとめてみます。<br>
今回やることの範囲としては以下のような感じです。<br>
 * UIBuilderでウィンドウをつくる
 * 入力した値をバインディングする
 * ウィンドウの入力内容を保存・復元する

本当に最低限のエディタウィンドウを作れるようになる程度の知識ですね。<br>

なおランタイム用のUIの作り方は取り扱いません（自分自身よくわかってない）。<br>
またListViewは結構ややこしく、これ単品で記事になりそうなのでそうしたいと思います。<br>

# ウィンドウを作る
作例として「RPGのデバッグ用戦闘シーンを開始するエディタウィンドウ」的なものを作ってみたいと思います。<br>
UI ToolkitはUI Builderのおかげでかなり楽にレイアウトを作ることができます。`Create / UI Toolkit / UI Document` からuxmlを作成して、ダブルクリックすればUI Builderで編集できます（お使いのUnityが対応してれば）。<br>

UIBuilderの説明はいったん省きます。いい感じにレイアウトを作れたら、次はC#を書きます。<br>
とりあえず↓のようにしてみました。<br>
```
using UnityEditor;
using UnityEngine;
using UnityEngine.UIElements;

public class DebugBattleSetting : EditorWindow
{
    [SerializeField] private VisualTreeAsset windowLayout = default;

    [MenuItem("Example/DebugBattleSetting")]
    public static void ShowExample()
    {
        DebugBattleSetting wnd = GetWindow<DebugBattleSetting>();
        wnd.titleContent = new GUIContent("DebugBattleSetting");
    }

    public void OnEnable()
    {
        VisualElement rootFromUXML = windowLayout.Instantiate();
        rootVisualElement.Add(rootFromUXML);
    }
}
```

`DebugBattleSetting`クラスは`EditorWindow`を継承しています。<br>
`OnEnable`関数というのは、ウィンドウが開かれたときに呼び出される関数です。<br>

Unityエディタに戻り、この.csファイルを選択して、インスペクターから先ほどのuxmlをアタッチします。<br>
今回のコードであれば、ツールバーから`Example / DebugBattleSetting`と選ぶことでウィンドウを出せるようになっているはずです。<br>
とはいえ、まだ単なるハリボテですが。<br>

# 複数のuxmlを組み合わせる
バインディングの説明に移る前に、レイアウトを整えたいと思います。<br>
今回は「RPGのデバッグ用戦闘シーンを開始するエディタウィンドウ」という体ですので、戦闘開始時の味方のステータスを設定できるようにしてみたいところです。<br>
まず、↓のようなレイアウトを作りました。<br>

![スクリーンショット 2024-08-14 155359](https://github.com/user-attachments/assets/4c685b34-8cc9-43f4-819b-d4f6148424cf)

これはHTLMなど全くミリしらな私がUI Toolkitを触ったうえでの私なりの理解になりますが、VisualElementは空のGameObjectのようなものと思っています。<br>
VisualElementは子にUI要素を持つことができて、LayoutGroupコンポーネントのように子のUI要素を縦あるいは横に整列させます。<br>
今回はHPの用のNumericFieldsを「残量 / 最大値」のように横に並べたいと思ったので、こうしました。<br>

そして、もう一つuxmlを作成します。<br>
新たに作成したuxmlを編集。`Library / Projectタブ / Assets` と選択し、Assets配下から先ほどのレイアウトを探します。それをHierarchyにD&Dすると、この新たなレイアウト内に先ほどのレイアウトを組み合わせることができます。<br>
今回は4人パーティのRPGという体で、↓のようにしてみました。<br>

![スクリーンショット 2024-08-14 155701](https://github.com/user-attachments/assets/649b9970-c842-4d19-8099-630ab55926de)

「DebugBattleStart」ボタンは、後に特定のシーンを開いてゲームを再生開始するような処理が結び付けられるであろうと思います。そこの処理はUI Toolkit解説という趣旨からは逸れるので割愛。<br>
また下の方にListViewがネタバレしていますが、先に述べた通り次回に単品で取り扱います。<br>

# バインディング
まずは最初にコードを置いておきます。これを前項の`DebugBattleSetting`クラスに追加します。<br>
```
[Serializable]
public class AllySetting
{
    public string Name;
    public int Level;
    public int Hp;
    public int MaxHp;
    public int Atk;
}

// UIとバインディングするもの
public AllySetting Ally0 = new ();
public AllySetting Ally1 = new ();
public AllySetting Ally2 = new ();
public AllySetting Ally3 = new ();

/// <summary>
/// Allyについてバインドする（OnEnableから呼ぶ）
/// </summary>
private void BindAllies()
{
    var ally0Elm = rootVisualElement.Q("Ally0Setting");
    BindDataToAllySetting(ally0Elm, Ally0);

    var ally1Elm = rootVisualElement.Q("Ally1Setting");
    BindDataToAllySetting(ally1Elm, Ally1);

    var ally2Elm = rootVisualElement.Q("Ally2Setting");
    BindDataToAllySetting(ally2Elm, Ally2);

    var ally3Elm = rootVisualElement.Q("Ally3Setting");
    BindDataToAllySetting(ally3Elm, Ally3);
}

/// <summary>
/// 各パラメータのバインディングを1つのメソッドにまとめた
/// </summary>
private static void BindDataToAllySetting(VisualElement allySettingElement, AllySetting field)
{
    // カスタムテンプレートの要素を取得
    var nameElm = allySettingElement.Q<TextField>("Name");
    var levelElm = allySettingElement.Q<IntegerField>("Level");
    var hpElm = allySettingElement.Q("Hp");
    var currentHpElm = hpElm.Q<IntegerField>("Current");
    var maxHpElm = hpElm.Q<IntegerField>("Max");
    var atkElm = allySettingElement.Q<IntegerField>("Atk");

    // フィールドの現在の値を設定
    nameElm?.SetValueWithoutNotify(field.Name);
    levelElm?.SetValueWithoutNotify(field.Level);
    currentHpElm?.SetValueWithoutNotify(field.Hp);
    maxHpElm?.SetValueWithoutNotify(field.MaxHp);
    atkElm?.SetValueWithoutNotify(field.Atk);

    // フィールドで値が書き変えられたときのコールバック
    nameElm?.RegisterValueChangedCallback(evt => field.Name = evt.newValue);
    levelElm?.RegisterValueChangedCallback(evt => field.Level = evt.newValue);
    currentHpElm?.RegisterValueChangedCallback(evt => field.Hp = evt.newValue);
    maxHpElm?.RegisterValueChangedCallback(evt => field.MaxHp = evt.newValue);
    atkElm?.RegisterValueChangedCallback(evt => field.Atk = evt.newValue);
}
```

`VisualElement.Q`関数は、そのVisualElementの配下から名前指定でVisualElementを取得できます。<br>
`VisualElement.Q<T>`関数になると、単なるVisualElementではなく「TextField」「IntegerField」のように返す型を指定できます（もちろん指定した名前のUI要素と合致していれば、ですが）。<br>
”名前”というのは、<br>

![スクリーンショット 2024-08-14 164731](https://github.com/user-attachments/assets/bc04dea2-da76-461d-b79b-bdb8f0b5ddc9)

このUI Builderのインスペクターで言えば "Ally0Setting" の部分のことです。<br>
上のコードの`BindDataToAllySetting`関数をよく見ると、HP周りの処理は他のパラメータと比べてひと手間かかっていますね。<br>
これはHPだけNumericFieldsを「残量 / 最大値」のように横に並べたくて、"Hp" というVisualElementの子にしたためです。<br>
なので、一旦`var hpElm = allySettingElement.Q("Hp");`のようにして"Hp"というVisualElementを取得し、そこから<br>
`var currentHpElm = hpElm.Q<IntegerField>("Current");`のようにして、子のUI要素を取得しています。<br>

あとは見ての通りです。<br>
`SetValueWithoutNotify`メソッドは、UI要素に対して値をセットします。<br>
`RegisterValueChangedCallback`メソッドは、UI側で入力内容が変化したときのコールバックを登録します。今回は見ての通り、newValueを対応する変数に代入しています。<br>

これらの変数を用いてバトルのModel的なものを生成し、デバッグバトルボタンからバトルをModelで初期化するような感じになるかと思います。<br>

# 入力内容の保存・復元
最後に、ウィンドウの入力内容をセーブする仕組みを作ります。ウィンドウを開く度に・Unityを起動する度に、毎回ステータスを入力するのはダルすぎますからね。




次回はListView
