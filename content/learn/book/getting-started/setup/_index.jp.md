+++
title = "セットアップ"
weight = 1
sort_by = "weight"
template = "book-section.html"
page_template = "book-section.html"
+++

ゲームを作りたくてたまらないのはわかりますが、まずは最低限のセットアップを行う必要があります。

## Rustのセットアップ

BevyのアプリとエンジンのコードはすべてRustで書かれています。そのため、始める前にRustの開発環境を整える必要があります。

### Rustのインストール

[Rust Getting Started Guide](https://www.rust-lang.org/learn/get-started)に従って、Rustをインストールします。

この作業が完了すると、パスに ```rustc``` コンパイラと ```cargo``` ビルドシステムがインストールされるはずです。

### OS の依存関係のインストール
* [Linux](https://github.com/bevyengine/bevy/blob/main/docs/linux_dependencies.md)
* Windows: [VS2019 ビルドツール](https://visualstudio.microsoft.com/thank-you-downloading-visual-studio/?sku=BuildTools&rel=16)を必ずインストールしてください。
* MacOS: 依存関係はありません。

### コードエディタ/IDE

コードエディタは何を使っても構いませんが、[Rust Analyzer](https://github.com/rust-analyzer/rust-analyzer)のプラグインがあるものを強くお勧めします。Rust Analyzer はまだ開発中ですが、すでに最高レベルのオートコンプリートとコードインテリジェンスを提供しています。[Visual Studio Code](https://code.visualstudio.com/)には、公式にサポートされた [Rust Analyzer Extension](https://marketplace.visualstudio.com/items?itemName=matklad.rust-analyzer)があります。

### Rust の学習リソース

本書は Bevy の学習を目的としているため、Rust の完全な教育にはなりません。Rust言語についてもっと知りたい方は、以下のリソースをチェックしてみてください。

* [**The Rust Book**](https://doc.rust-lang.org/book/): Rustをゼロから学ぶのに最適な場所です。
* [**Rust by Example**](https://doc.rust-lang.org/rust-by-example/): 実際のコーディング例を見ながらRustを学ぶことができます。


## 新しいBevyプロジェクトの作成

これで、Bevyプロジェクトを立ち上げる準備ができました! Bevy は通常の Rust の依存関係にあります。既存のRustプロジェクトに追加することも、新しいプロジェクトを作成することもできます。念のため、最初から始めると仮定して説明します。

### 新しいRust実行形式プロジェクトの作成

まず、新しいプロジェクトを作成したいフォルダに移動します。次に、以下のコマンドを実行して、Rust の実行可能なプロジェクトを含む新しいフォルダを作成します。

```sh
cargo new my_bevy_game
cd my_bevy_game
```

次に、``cargo run``を実行して、プロジェクトをビルドして実行します。すると、ターミナルに「Hello, world!」と表示されるはずです。お好みのコードエディタで ``my_bevy_game`` フォルダを開き、ファイルに目を通してみましょう。

``main.rs`` は、プログラムのエントリーポイントです。
```rs
fn main() {
    println!("Hello, world!");
}
```

``Cargo.toml`` は、あなたの「プロジェクトファイル」です。プロジェクトの名前、依存関係、ビルド構成など、プロジェクトに関するメタデータが含まれています。

```toml
[package]
name = "my_bevy_game"
version = "0.1.0"
edition = "2018"

[dependencies]
```

### プロジェクトのCargo.tomlにBevyを追加します。

Bevyは、Rustの公式パッケージリポジトリである[crates.ioでライブラリとして利用可能](https://crates.io/crates/bevy)です。最新のバージョン番号([![Crates.io](https://img.shields.io/crates/v/bevy.svg)](https://crates.io/crates/bevy))を探して、Cargo.tomlファイルに追加してください。

```toml
[package]
name = "my_bevy_game"
version = "0.1.0"
edition = "2018"

[dependencies]
bevy = "0.5" # make sure this is the latest version
```
### Fast Compiles を有効にする (オプション)

安定したRustでは、デフォルトの設定でもBevyは問題なくビルドできます。Bevyのダイナミックリンク機能を有効にしてください。

* **Bevyのダイナミックリンク機能を有効にする**: これは、最もインパクトのあるコンパイル時間の短縮方法です。もし`bevy`が依存関係にある場合、"dynamic "機能フラグ（ダイナミックリンクを有効にする）でバイナリをコンパイルすることができます。今のところ、これはWindowsでは動作しないことに注意してください。
    ``sh
    cargo run --features bevy/dynamic
    ```
    毎回の実行に `--features bevy/dynamic` を追加したくない場合は、このフラグを `Cargo.toml` で恒久的に設定することができます。
    ``toml
    [dependencies] (依存関係)
    bevy = { version = "0.5.0", features = ["dynamic"] }.
    ```
    注意: ゲームをリリースする前に、この設定を元に戻すことを忘れないでください。そうしないと、ゲームを走らせたいときに、`libbevy_dylib`を一緒にインクルードする必要があります。ダイナミック "機能を削除すると、ゲームの実行ファイルはスタンドアロンで動作します。

* **LLD リンカ**: Rust コンパイラは「リンク」のステップに多くの時間を費やしています。LLDはRustのデフォルトのリンカーよりも_はるかに高速にリンクを行います。LLDをインストールするには、以下の中からお使いのOSを選び、所定のコマンドを実行してください。
    * **Ubuntu**: Ubuntu**: `sudo apt-get install lld`.
    * **Arch**: sudo pacman -S lld`.
    * **Windows**: 最新の[cargo-binutils](https://github.com/rust-embedded/cargo-binutils)がインストールされていることを確認してください。
        ``sh
        cargo install -f cargo-binutils
        rustup component add llvm-tools-preview
        ```
    * **MacOS**: 現代のLLDはまだMacOSをサポートしていませんが、代わりにzldを使うことができます。`brew install michaeleisel/zld/zld`。
* **Nightly Rust Compiler**: 最新のパフォーマンス改善や「不安定な」最適化にアクセスできます。
    
    プロジェクトのルートに、``cargo.toml``の隣に``rust-toolchain``ファイルを作成します。
    ``toml
    [ツールチェイン]
    チャンネル = "nightly"
    ```
    詳細は[The rustup book: Overrides](https://rust-lang.github.io/rustup/overrides.html#the-toolchain-file)を参照してください。
* **一般的な共有**: クレートが単調なジェネリックコードを複製する代わりに共有できるようにします。場合によっては、ジェネリックコードを "プリコンパイル "して、反復コンパイルに影響を与えないようにすることができます。これは nightly Rust でのみ利用可能です。

高速なコンパイルを可能にするには、nightly rust コンパイラと LLD をインストールします。そして、[このファイル](https://github.com/bevyengine/bevy/blob/main/.cargo/config_fast_builds)を`YOUR_WORKSPACE/.cargo/config.toml`にコピーします。このガイドのプロジェクトでは、`my_bevy_game/.cargo/config.toml`となります。

何か問題が発生した場合は、[トラブルシューティングセクション](/learn/book/troubleshooting/) や [ask for help on our Discord](https://discord.com/invite/gMUk5Ph)をご覧ください。

### Bevy のビルド

再度、``cargo run``を実行します。Bevyの依存関係の構築が始まるはずです。基本的にエンジンを一から構築することになるので、多少時間がかかります。完全なリビルドが必要なのは一度だけです。この後のビルドはすべて高速になります!

これでBevyプロジェクトのセットアップが完了し、最初のBevyアプリを作り始める準備ができました。