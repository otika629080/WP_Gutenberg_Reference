#

## スターターブロックのひな形の探索
create-blockとcreate-guten-blockのどちらの開発ツールを選択しても、ブロックプラグイン構築の出発点として利用可能なブロックのひな形を取得できます。

ブロックのひな形とは何でしょうか。

ブロックのひな形とは、WordPressからブロックとして認識されるために必要なディレクトリ構造を表す言葉です。通常、このディレクトリにはindex.php、index.js、style.cssなどのファイルがあり、ファイルの内部にはregister_block_typeなどの呼び出しが含まれます。

ブロックエディターハンドブックでも使用されている公式のCreate Block開発ツールを使用します。ただし、create-guten-blockのようなサードパーティツールでも大きな違いはないはずです。

それでは、create-blockツールを見ていきましょう。

## 開発ツール「Create Block」とは

前述のように、Create Blockは、Gutenbergブロックを作成する公式のコマンドラインツールです。ターミナルで@wordpress/create-blockを実行すると、新しいブロックタイプの登録に必要な、PHP、JS、SCSSファイルとコードが生成されます。

```sh
npx @wordpress/create-block [options] [slug]
```

[slug]（任意）：ブロックのスラッグの割り当てとプラグインのインストールに使用される
[options]（任意）：利用可能なオプション
デフォルトでは、ESNextテンプレートが割り当てられます。すなわち、JSX構文が追加された次バージョンのJavaScriptが作成されます。

ブロック名を省略すると、コマンドは対話モードで実行され、ファイル生成前に複数の設定を編集することができます。

```sh
npx @wordpress/create-block
```

ブロックプラグインの主なファイルとフォルダを確認します。

## プラグインファイル

メインのプラグインファイルでは、サーバーにブロックを登録します。

```php
<?php
/**
 * Plugin Name:       Kinsta Academy Block
 * Plugin URI:        https://kinsta.com/
 * Description:       An example block for Kinsta Academy students
 * Requires at least: 5.9
 * Requires PHP:      7.0
 * Version:           0.1.0
 * Author:            Kinsta Students
 * License:           GPL-2.0-or-later
 * License URI:       https://www.gnu.org/licenses/gpl-2.0.html
 * Text Domain:       ka-example-block
 *
 * @package           ka-example-block
 */

/**
 * Registers the block using the metadata loaded from the `block.json` file.
 * Behind the scenes, it registers also all assets so they can be enqueued
 * through the block editor in the corresponding context.
 *
 * @see https://developer.wordpress.org/reference/functions/register_block_type/
 */
function ka_example_block_ka_example_block_block_init() {
	register_block_type( __DIR__ . '/build' );
}
add_action( 'init', 'ka_example_block_ka_example_block_block_init' );
```

register_block_type関数は、block.jsonファイルに格納されたメタデータを使用して、サーバーにブロックタイプを登録します。

この関数は2つのパラメータを取ります。

- 名前空間を含むブロックタイプ名、またはjsonファイルがあるフォルダのパス、または完全なWP_Block_Typeオブジェクト
- ブロックタイプの引数の配列

上のコードでは、ブロックタイプの引数に、マジック定数「__DIR__」が指定されています。これは、プラグインのファイルと同じフォルダにblock.jsonファイルが存在することを意味します。

## package.jsonファイル

package.jsonファイルは、プロジェクトのJavaScriptプロパティとスクリプトを定義します。ここには、プロジェクトの依存関係もインストールできます。

コードエディターでファイルを開き、内容を確認してください。

```json
{
	"name": "ka-example-block",
	"version": "0.1.0",
	"description": "An example block for Kinsta Academy students",
	"author": "Kinsta Students",
	"license": "GPL-2.0-or-later",
	"homepage": "https://kinsta.com/",
	"main": "build/index.js",
	"scripts": {
		"build": "wp-scripts build",
		"format": "wp-scripts format",
		"lint:css": "wp-scripts lint-style",
		"lint:js": "wp-scripts lint-js",
		"packages-update": "wp-scripts packages-update",
		"plugin-zip": "wp-scripts plugin-zip",
		"start": "wp-scripts start"
	},
	"devDependencies": {
		"@wordpress/scripts": "^24.1.0"
	},
	"dependencies": {
		"classnames": "^2.3.2"
	}
}
```

scriptsプロパティは、コマンドを含む連想配列です。コマンドはnpm run [cmd]を使用して、パッケージのライフサイクルの様々なタイミングで実行されます。

この記事では、以下のコマンドを使用します。

- npm run build：（圧縮された）本番用ビルドを作成
- npm run start：（圧縮されていない）開発用ビルドを作成

dependenciesとdevDependenciesは、パッケージ名とバージョンを対応付ける2つのオブジェクトです。dependenciesは本番用ビルドで、devDependencesはローカル開発でのみ必要です（詳細はこちら）。

デフォルトのdevDependenciesは、@wordpress/scriptsパッケージのみで、これは「WordPress開発専用の、再利用可能なスクリプト集」として定義されています。

## block.jsonファイル

WordPress 5.8以降、正式なブロックタイプの登録には、block.jsonメタデータファイルを使用します。

block.jsonファイルには、パフォーマンスの向上や、WordPressプラグインディレクトリでの視認性の向上など、様々な利点があります。

パフォーマンスの観点から、テーマがアセットの遅延ロードをサポートするとき、block.jsonで登録されたブロックは、デフォルトでアセットのキュー処理が最適化されます。styleまたはscriptプロパティにリストされたフロントエンドCSSやJavaScriptアセットは、ブロックがページ上に存在するときのみキューに入れられ、結果、ページサイズが縮小します。

@wordpress/create-blockコマンドを実行すると、以下のblock.jsonファイルが生成されます。

```json
{
	"$schema": "https://schemas.wp.org/trunk/block.json",
	"apiVersion": 2,
	"name": "ka-example-block/ka-example-block",
	"version": "0.1.0",
	"title": "Kinsta Academy Block",
	"category": "widgets",
	"icon": "superhero-alt",
	"description": "An example block for Kinsta Academy students",
	"supports": {
		"html": false
	},
	"textdomain": "ka-example-block",
	"editorScript": "file:./index.js",
	"editorStyle": "file:./index.css",
	"style": "file:./style-index.css"
}
```

以下は、デフォルトのプロパティの一覧です。

- piVersion：ブロックが利用するAPIのバージョン（現在のバージョンは2）
- name：名前空間を含むブロックの一意な識別子
- version：ブロックの現在のバージョン
- title：ブロックの表示タイトル
- category：ブロックのカテゴリ
- icon：Dashiconのスラッグまたは自作SVGアイコン
- description：ブロックインスペクタに表示される短い説明
- supports：エディターで使用される機能を制御するオプション群
- textdomain：プラグインのテキストドメイン
- editorScript：エディタースクリプト定義
- editorStyle：エディタースタイル定義
- style：ブロックの代替スタイルを指定
