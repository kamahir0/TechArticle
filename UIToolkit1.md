# UI Toolkitにちょっと入門してみた【エディタ拡張】

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
`OnEnable`関数というのは、ウィンドウが開かれたときに呼び出される関数です（もちろんUnityエディタを起動したときにも）。<br>

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
public void OnEnable()
{
    VisualElement rootFromUXML = windowLayout.Instantiate();
    rootVisualElement.Add(rootFromUXML);
    
    BindAllies(); // <--追加
}

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
`SetValueWithoutNotify`メソッドは、UI要素に対して変数の値をセットします。<br>
`RegisterValueChangedCallback`メソッドは、UI側で入力内容が変化したときのコールバックを登録します。今回は見ての通り、newValueを対応する変数に代入しています。<br>
`SetValueWithoutNotify`の方はコレなんの意味があんの？？と思われるかもしれません。こちらは、後述のセーブ機能と併せて効力を発揮します。<br>
このバインディング処理よりも早いタイミングで、復元した値を変数に入れておくというワケですね。で、その値をセットすると。<br>

# 入力内容の保存・復元
最後に、ウィンドウの入力内容をセーブする仕組みを作ります。ウィンドウを開く度 / Unityを起動する度に、毎回ステータスを入力するのはダルすぎますからね。<br>
例によってまずはコードです。
```
public void OnEnable()
{
    VisualElement rootFromUXML = windowLayout.Instantiate();
    rootVisualElement.Add(rootFromUXML);
    
    RestoreState(); // <--追加
    BindAllies();
}

public void OnDisable()
{
    SaveState();
}

private const string AlliesSettingsKey = "DebugBattleSetting_AllySettings";

/// <summary>
/// ウィンドウ状態の保存（閉じるとき呼ばれる）
/// </summary>
private void SaveState()
{
    // AllySettingsの保存
    var allySettingsData = new List<AllySetting>();
    allySettingsData.AddRange(new[] { Ally0, Ally1, Ally2, Ally3 });
    var alliesSettingData = new AlliesSettingData(){ Settings = allySettingsData };

    var allySettingsJson = JsonUtility.ToJson(alliesSettingData);
    EditorPrefs.SetString(AlliesSettingsKey, allySettingsJson);
}

/// <summary>
/// ウィンドウ状態の復元（開くとき呼ばれる）
/// </summary>
private void RestoreState()
{
    // AllySettingsの復元
    if (EditorPrefs.HasKey(AlliesSettingsKey))
    {
        var allySettingsJson = EditorPrefs.GetString(AlliesSettingsKey);
        var alliesSettingData = JsonUtility.FromJson<AlliesSettingData>(allySettingsJson);

        Ally0 = alliesSettingData.Get(0);
        Ally1 = alliesSettingData.Get(1);
        Ally2 = alliesSettingData.Get(2);
        Ally3 = alliesSettingData.Get(3);
    }
}

/// <summary>
/// データクラスのラッパー
/// </summary>
[Serializable]
private class AlliesSettingData
{
    public List<AllySetting> Settings;

    public AllySetting Get(int index)
    {
        return new AllySetting()
        {
            Name = Settings[index].Name,
            Level = Settings[index].Level,
            Hp = Settings[index].Hp,
            MaxHp = Settings[index].MaxHp,
            Atk = Settings[index].Atk,
        };
    }
}
```

ウィンドウが開かれたときに呼ばれる`OnEnable`と対になる、ウィンドウが閉じられたときに呼ばれる`OnDisable`関数もあります。<br>
`OnDisable`にて内容を保存する`SaveState`を呼び、`OnEnable`にて内容を復元する`RestoreState`を呼んでいますね。<br>

エディタ限定ですが、Unityには`EditorPrefs`というありがた～～い機能が備わっています。string値をキーにして、Jsonを非常に簡単に読み書きできるのです。<br>
今回の実装ではこの`EditorPrefs`で、ウィンドウの入力内容を保存・復元しています。<br>
`AlliesSettingData`クラスをわざわざ用意しているのはJsonUtilityで変換するためです。あとはList<AllySetting>のようなコレクションだけでなく、パーティ共通で単一の非コレクションな値も追加したくなるかもしれませんし（所持金...とか。ペルソナなんかでは混乱すると所持金をバラまいたりとかありますね）。<br>
そういうのも`AlliesSettingData`に含めれば一緒に復元できる、という。

# 結言
そんな感じで、ウィンドウの内容を保存できるようになりました。<br>
「RPGのデバッグ用戦闘シーンを開始するエディタウィンドウ」ということですから、<br>
後はこのデータを用いてバトルのModel的なのを生成し、そのModelでバトルを初期化して開始する...みたいな処理を作って、ボタンのコールバックに登録すればよい、と。<br>

以上で、基本的なウィンドウの作り方にちょっと入門してみた話でした。<br>
次回は超絶曲者のListViewについてです > < たいへん<br>

ご精読ありがとうございました。
