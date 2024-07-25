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
- 
上のプロパティに加えて属性オブジェクトを定義でき、ブロックが保存するデータに関する情報を指定できます。block.jsonでは、キーと値のペアでいくつでも属性を設定できます。この時、キーは属性名、値は属性定義です。

以下は属性定義の例です。

```json
"attributes": {
	"content": {
		"type": "array",
		"source": "children",
		"selector": "p"
	},
	"align": {
		"type": "string",
		"default": "none"
	},
	"link": { 
		"type": "string", 
		"default": "https://kinsta.com" 
	}
},
```

block.jsonファイルについてはこの記事の後半で詳しくご説明しますが、ブロックエディターハンドブックでも、block.jsonファイルのメタデータや属性について詳しく解説されていますので、参照してみてください。

## srcフォルダ
- index.js
- edit.js
- save.js
- editor.scss
- style.scss

### index.js
index.jsファイルは出発点です。依存関係をインポートし、ブロックタイプをクライアントに登録します。

```json
import { registerBlockType } from '@wordpress/blocks';

import './style.scss';

import Edit from './edit';
import save from './save';
import metadata from './block.json';

registerBlockType( metadata.name, {
	/**
	 * @see ./edit.js
	 */
	edit: Edit,

	/**
	 * @see ./save.js
	 */
	save,
} );
```

最初の文では、@wordpress/blocksパッケージからregisterBlockType関数をインポートしています。続くimport文は、スタイルシート、Edit関数、save関数をインポートしています。

registerBlockType関数は、クライアントでコンポーネントを登録します。この関数は2つのパラメータを取ります。ブロック名の「名前空間/ブロック名」（サーバー上で登録されたものと同じ）とブロック構成オブジェクトです。

Edit関数はブロックエディターで描画されるブロックインターフェースを定義し、save関数はシリアライズされてデータベースに保存される構造を定義します（詳しくはこちら）。

### edit.js

edit.jsでは、ブロック管理インターフェースを構築します。

```js
import { __ } from '@wordpress/i18n';
import { useBlockProps } from '@wordpress/block-editor';
import './editor.scss';

export default function Edit() {
	return (
		<p {...useBlockProps()}>
			{__('My First Block – hello from the editor!', 'my-first-block')}
		</p>
	);
}
```

まず、@wordpress/i18nパッケージ（このパッケージには、JavaScript版の翻訳関数が含まれています）の__関数、useBlockProps Reactフック、editor.scssファイルをインポートしています。

続いて、Reactコンポーネントをエクスポートします（詳細はimport文とexport文をご覧ください）。

### save.js

save.jsファイルでは、データベースに保存するブロック構造を構築します。

```js
import { __ } from '@wordpress/i18n';
import { useBlockProps } from '@wordpress/block-editor';

export default function save() {
	return (
		<p {...useBlockProps.save()}>
			{__(
				'My First Block – hello from the saved content!',
				'my-first-block'
			)}
		</p>
	);
}
```

editor.scssとstyle.scss

スクリプトとは別に、2つのSASSファイルがsrcフォルダに存在します。editor.scssファイルにはエディターのコンテキストでブロックに適用されるスタイルが含まれ、style.scssファイルにはフロントエンドとエディターでの表示用のブロックのスタイルが含まれます。これらのファイルについては、この記事の後半で掘り下げていきます。

### node_modulesとbuildフォルダ
node_modulesフォルダには、nodeモジュールとその依存関係が格納されています。nodeパッケージは、この記事で扱う範囲を超えるため深入りしませんが、npmがパッケージをインストールする場所については、この記事をご覧ください。

buildフォルダには、ビルド処理で生成されたJSファイルとCSSファイルが格納されます。詳しいビルドプロセスについては、「ESNext構文」と「JavaScriptビルド環境のセットアップ」をご覧ください。

## 実践─Gutenbergブロックを作成しよう

それでは、実際にブロックを作っていきましょう。このセクションでは、CTA（コール・トゥ・アクション）ブロック、Kinsta Academyブロックを実装するプラグインの作成方法をご説明します。

ブロックは2つのカラムで構成され、左側に画像、右側にテキストの段落があります。テキストの下には、編集可能なリンクが付いたボタンが配置されます。

![custom-block-1](custom-block-1.jpg)

これは簡単な例ですが、Gutenbergブロック開発の基本を網羅することができます。一通り理解できたら、ブロックエディターハンドブックなどのリソースを参照して、より複雑なGutenbergブロックを作ってみてください。

今回は、ローカル開発環境で最新版のWordPressが稼働していることを前提とし、以下の順番でブロック開発をご紹介していきます。

- スターターブロックプラグインのセットアップ
- jsonの編集
- 組み込みコンポーネントの使用─RichTextコンポーネント
- ブロックツールバーへのコントロールの追加
- ブロック設定サイドバーの編集
- 外部リンクの追加と編集
- 複数のブロックスタイルの追加
- InnerBlocksコンポーネントを使用したブロックのネスト
- 細かな編集

では、始めましょう。

### スターターブロックプラグインのセットアップ方法

コマンドラインツールを起動して、/wp-content/pluginsフォルダに移動します。

以下のコマンドを実行します。

```sh
npx @wordpress/create-block
```

このコマンドは対話モードで、ブロック登録用のPHP、SCSS、JSファイルを生成します。対話モードでは、ブロックに必要なデータを簡単に設定できます。この記事では、以下の値を使用します。

- Template variant（テンプレート）：static（静的）
- Block slug（ブロックのスラッグ）：ka-example-block
  - WordPressのブロックエディタ（Gutenberg）で使用される、カスタムブロックの一意の識別子
- Internal namespace（内部名前空間）：ka-example-block
- Block display title（ブロック表示タイトル）：Kinsta Academy Block
- Short block description（ブロックの説明）：An example block for Kinsta Academy students（Kinstaアカデミー受講者向けブロックの例）
- Dashicon（アイコンフォント）：superhero-alt
- Category name（カテゴリ名）：widgets
- Do you want to customize the WordPress plugin?（WordPressプラグインを編集しますか？）：Yes
- The home page of the plugin（プラグインのトップページ）：https://kinsta.com/
- Current plugin version（現在のプラグインのバージョン）：1.0
- Plugin author（プラグイン開発者名） ：自分の名前
- License（ライセンス）：─
- Link to the license text（ライセンステキストへのリンク）：─
- Custom domain path for translation（翻訳用独自ドメインパス）：─

  プラグインとすべての依存関係をインストールするには、数分かかります。

  次に、/wp-content/pluginsフォルダから以下のコマンドを実行します。

  ```sh
  cd ka-example-block
  ```

  最後に、プラグインのフォルダ（この例ではka-example-block）から次のコマンドを入力して、開発を始めます。

  ```sh
  npm start
  ```

「プラグイン」画面を開き、「Kinsta Academy Block」プラグインを有効化してください。

新しい投稿を作成し、ブロックインサーターを開き、「デザイン」カテゴリまでスクロールします。「Kinsta Academy Block」をクリックして、投稿に追加してください。

### block.jsonの編集
