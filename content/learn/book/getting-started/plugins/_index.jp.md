+++
title = "プラグイン"
weight = 4
sort_by = "weight"
template = "book-section.html"
page_template = "book-section.html"
+++

Bevyの基本原則の一つはモジュール化です。Bevyのエンジン機能はすべてプラグインとして実装されています。これにはレンダラーなどの内部機能も含まれますが、ゲーム自体もプラグインとして実装されています。これにより、開発者は必要な機能を選択することができます。UIはいらない？{{rust_type(type="struct" crate="bevy_ui", name="UiPlugin")}}を登録する必要はありません。ヘッドレスサーバを構築したいですか？{{rust_type(type="struct" crate="bevy_render" name="RenderPlugin")}}を登録しないでください。

これはまた、気に入らないコンポーネントを自由に交換できることを意味します。必要性を感じたら、独自の{{rust_type(type="struct" crate="bevy_ui" name="UiPlugin")}}を構築することは歓迎されますが、便利だと思ったら [Bevyに貢献する](/learn/book/contributing)ことを検討してください!

しかし、ほとんどの開発者はカスタムエクスペリエンスを必要とせず、手間をかけずに「フルエンジン」のエクスペリエンスを得たいと考えています。そのために、Bevyは「デフォルトプラグイン」のセットを提供しています。 

## Bevyのデフォルト・プラグイン

Bevy の "デフォルト・プラグイン" を追加して、アプリをより面白くしてみましょう。
`add_plugins(DefaultPlugins)` は、2D/3Dレンダラー、アセットローディング、UIシステム、ウィンドウ、入力など、多くの人がエンジンに期待する機能を追加します。

```rs
fn main() {
    App::build()
        .add_plugins(DefaultPlugins)
        .add_startup_system(add_people.system())
        .add_system(hello_world.system())
        .add_system(greet_people.system())
        .run();
}
```

もう一度、`cargo run`を実行してみましょう。

うまくいけば、2つのことに気づくはずです。
* **ウィンドウがポップアップします**。これは、ウィンドウインターフェイスを定義する{{rust_type(type="struct" crate="bevy_window" name="WindowPlugin")}}と、[winit library](https://github.com/rust-windowing/winit)を使用して、OSのネイティブウィンドウAPIを使用してウィンドウを作成する{{rust_type(type="struct" crate="bevy_winit" name="WinitPlugin")}}があるからです。
* **あなたのコンソールは今、"hello"メッセージでいっぱいです**。これは、{{rust_type(type="struct" crate="bevy" name="DefaultPlugins")}}がアプリケーションに「イベントループ」を追加しているためです。アプリケーションのECSスケジュールは、「フレーム」ごとに1回、ループで実行されるようになりました。コンソールのスパムをすぐに解決します。

なお、`add_plugins(DefaultPlugins)`は、以下のものと同じです。
```rs
fn main() {
    App::build()
        .add_plugin(CorePlugin::default())
        .add_plugin(InputPlugin::default())
        .add_plugin(WindowPlugin::default())
        /* その他のプラグインは簡略化のため省略されています */
        .run();
}
```

どのようなアプローチをとるかは、ご自由にどうぞ

## 最初のプラグインの作成

整理しやすいように、"hello"のロジックをすべてプラグインに移してみましょう。プラグインを作成するには、 {{rust_type(type="trait" name="Plugin" crate="bevy_app" no_mod=true)}} インターフェースを実装するだけです。以下のコードを `main.rs` ファイルに追加します。

```rs
pub struct HelloPlugin;

impl Plugin for HelloPlugin {
    fn build(&self, app: &mut AppBuilder) {.
        // ここにアプリに必要なものを追加する
    }
}
```

そして、以下のようにAppにプラグインを登録します。
```rs
fn main() {
    App::build()
        .add_plugins(DefaultPlugins)
        .add_plugin(HelloPlugin)
        .add_startup_system(add_people.system())
        .add_system(hello_world.system())
        .add_system(greet_people.system())
        .run();
}
```

あとは、システムを `HelloPlugin` に移動させるだけです。プラグインの`build()`関数の`app`変数は、 `main()`関数で使っているのと同じビルダータイプです。

```rs
impl Plugin for HelloPlugin {
    fn build(&self, app: &mut AppBuilder) { 。
        app.add_startup_system(add_people.system())
            .add_system(hello_world.system())
            .add_system(greet_people.system());
    }
}

fn main() {
    App::build()
        .add_plugins(DefaultPlugins)
        .add_plugin(HelloPlugin)
        .run();
}
```

アプリをもう一度実行してみてください。以前と同じように動作するはずです。次のセクションでは、Resourcesを使って "hello" スパムを修正します。
