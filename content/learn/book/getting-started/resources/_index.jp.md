+++
title = "リソース"
weight = 5
sort_by = "weight"
template = "book-section.html"
page_template = "book-section.html"
+++

**Entity** と **Components** は、複雑で問い合わせ可能なデータ群を表現するのに適しています。しかし、ほとんどのアプリでは、ある種の「グローバルにユニークな」データを必要とします。Bevy ECSでは、**Resources**を使ってグローバルにユニークなデータを表現します。

以下は、**Resources**としてエンコードできるデータの例です。
* 経過時間
* アセットコレクション(サウンド、テクスチャ、メッシュ)
* レンダラー

## リソースで時間を追う

アプリの "hello spam" 問題を解決するために、2秒に1回だけ "hello" を表示することにしましょう。これには {{rust_type(type="struct" crate="bevy_core" name="Time")}} というリソースを使います。

簡単にするために、アプリから `hello_world` システムを削除します。そうすれば、`greet_people`システムを適応させるだけで済みます。

コンポーネントにアクセスするのと同じように、リソースにアクセスします。システム内の`Time`リソースには次のようにアクセスできます。

```rs
fn greet_people(time: Res<Time>, query: Query<&Name, With<Person>>) {
    for name in query.iter() {
        println!("hello {}!", name.0);
    }
}
```

`Res`と`ResMut`のポインターは、それぞれリソースへのリードとライトのアクセスを提供します。

`Time`の`delta_seconds`フィールドは、最後の更新からの経過時間を示しています。しかし、2秒に1回システムを実行するためには、一連の更新から経過した時間を追跡する必要があります。これを簡単にするために、Bevy は `Timer` 型を提供しています。それでは、`Timer`を使って経過時間を追跡するシステムのために、新しいResourceを作成してみましょう。

```rs
struct GreetTimer(Timer);

fn greet_people(
    time: Res<Time>, mut timer: ResMut<GreetTimer>, query: Query<&Name, With<Person>>) {
    // 前回の更新からの経過時間でタイマーを更新する
    // それによってタイマーが終了した場合、みんなに挨拶をする
    if timer.0.tick(time.delta()).just_finished() {...
        for name in query.iter() {。
            println!("hello {}!", name.0);
        }
    }
}
```

さて、あとは `HelloPlugin` に `GreetTimer` リソースを追加するだけです。
```rs
impl Plugin for HelloPlugin {
    fn build(&self, app: &mut AppBuilder) {
        // from_seconds を true フラグで呼ぶのは、タイマーを繰り返すためです。
        app.insert_resource(GreetTimer(Timer::from_seconds(2.0, true)))
            .add_startup_system(add_people.system())
            .add_system(greet_people.system());
    }
}
```

それでは、アプリを `cargo run` してみましょう。これで、それなりの割合で人を迎えることができるはずです。
