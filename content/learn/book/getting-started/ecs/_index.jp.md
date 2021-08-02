+++
title = "ECS"
weight = 3
sort_by = "weight"
template = "book-section.html"
page_template = "book-section.html"
+++

Bevyのすべてのアプリ・ロジックはEntity Component Systemのパラダイムを使用しており、これはしばしばECSと短縮されます。ECSはソフトウェアのパターンで、プログラムを **Entity**、**Components**、**Systems** に分割します。**Entity** は、ユニークな`モノ` であり、そこにコンポーネントのグループが割り当てられ、システムで処理されます。

例えば、あるエンティティは `Position` と `Velocity` というコンポーネントを持ち、別のエンティティは `Position` と `UI` というコンポーネントを持っているかもしれません。システムは、特定のコンポーネントタイプのセットで動作するロジックです。例えば、 `Position` と `Velocity` のコンポーネントを持つすべてのエンティティ上で動作する `movement` システムがあるかもしれません。

ECSパターンは、アプリのデータやロジックをコアコンポーネントに分割することで、クリーンでデカップリングされたデザインを促進します。また、メモリアクセスのパターンを最適化し、並列処理を容易にすることで、コードを高速化することができます。

# # Bevy ECS

Bevy ECSはBevyのECSパターンの実装です。複雑なライフタイム、trait、ビルダーパターン、マクロを必要とすることが多い他のRust ECSの実装とは異なり、Bevy ECSはこれらの概念のすべてに通常のRustデータ型を使用します。
* **Components**: 通常のRust構造体
    ```rs
    struct Position { x: f32, y: f32 }
    ```
* **Systems**: 通常のRust関数
    ```rs
    fn print_position_system(query: Query<&Transform>) {
        for transform in query.iter() {
            println!("position: {:?}", transform.translation);
        }
    }
    ```
* **Entities**: 一意の整数を含むシンプルな型
    ```rs
    struct Entity(u64);
    ```

では、実際にどのように動作するか見てみましょう。

## あなたの最初のシステム

以下の関数を `main.rs` ファイルに貼り付けます。

```rs
fn hello_world() {
    println!("hello world!");
}
```

これが私たちの最初のシステムになります。あとは、アプリに追加するだけです。

```rs
fn main() {
    App::build()
        .add_system(hello_world.system())
        .run();
}
```

`hello_world.system()` の関数呼び出しに注目してください。これは、`hello_world` 関数を {{rust_type(type="trait" crate="bevy_ecs" mod="system" no_mod=true name="System")}} 型に変換する `trait拡張メソッド` です。

また、 {{rust_type(type="struct" crate="bevy_app", name="AppBuilder" method="add_system" no_struct=true)}}関数は、アプリの{{rust_type(type="struct", crate="bevy_ecs", mod="schedule" no_mod=true name="Schedule")}} タイプにシステムを追加しますが、これについては後ほど詳しく説明します。

それでは、`cargo run` を使ってアプリを再度実行してみましょう。すると、ターミナルに `hello world!` と表示されるはずです。

## 最初のコンポーネント

世界中に挨拶するのは素晴らしいことですが、特定の人に挨拶したい場合はどうすればよいでしょうか？ECSでは、一般的に、人を定義するコンポーネントのセットを持つエンティティとしてモデル化します。まずはシンプルに、`Person` コンポーネントから始めましょう。

以下の構造体を `main.rs` に追加します。

```rs
struct Person;
```

しかし、人に名前を持たせたい場合はどうすればよいでしょうか。より伝統的な設計では、`Person` に `name: String` フィールドを追加するだけかもしれません。しかし、他のエンティティにも名前があるかもしれません。例えば、犬にも名前があるはずです。コードの再利用を促進するために、データタイプを小さく分割することはしばしば意味があります。そこで、 `Name` を独立したコンポーネントにしてみましょう。

```rs
struct Name(String);
```

次に、 `スタートアップシステム` を使って、{{rust_type(type="struct" crate="bevy_ecs" mod="world" no_mod=true name="World")}}に `People` を追加します。スタートアップシステムは通常のシステムと同じですが、アプリの起動時に、他のシステムよりも先に一度だけ実行されます。それでは、{{rust_type(type="struct" crate="bevy_ecs" mod="system" no_mod=true name="Commands")}}を使って、{{rust_type(type="struct" crate="bevy_ecs" mod="world" no_mod=true name="World")}}の中にいくつかのエンティティを生成してみましょう。

```rs
fn add_people(mut commands: Commands) {
    commands.spawn().insert(Person).insert(Name("Elaina Proctor".to_string()));
    commands.spawn().insert(Person).insert(Name("Renzo Hume".to_string()));
    commands.spawn().insert(Person).insert(Name("Zayna Nieves".to_string()));
}
```

ここで、起動システムを次のように登録します。

```rs
fn main() {
    App::build()
        .add_startup_system(add_people.system())
        .add_system(hello_world.system())
        .run();
}
```

今、このアプリを実行すると、 `add_people` システムが最初に実行され、次に `hello_world` が実行されます。しかし、新しい人々はまだ何もすることがありません! {{rust_type(type="struct" crate="bevy_ecs" mod="world" no_mod=true name="World")}}の新しい市民をきちんと迎えるシステムを作ってみましょう。

```rs
fn greet_people(query: Query<&Name, With<Person>>) {
    for name in query.iter() {
        println!("hello {}!", name.0);
    }
}
```

"システム関数" に渡すパラメータは、システムが実行するデータを定義します。今回の例では、`greet_people` は `Person` と `Name` コンポーネントを持つすべてのエンティティに対して実行されます。

上記のクエリを次のように解釈することができます。"Personコンポーネントを持つエンティティのすべてのNameコンポーネントを反復する"

あとはこのシステムをアプリに登録するだけです。

```rs
fn main() {
    App::build()
        .add_startup_system(add_people.system())
        .add_system(hello_world.system())
        .add_system(greet_people.system())
        .run();
}
```

このアプリを実行すると、次のような出力が得られます。

```
hello world!
hello Elaina Proctor!
hello Renzo Hume!
hello Zayna Nieves!
```

素晴らしい!

**クイックノート**: "hello world!" の表示順序が上記と異なる場合があります。これは、システムが可能な限りデフォルトで並行して動作するためです。
