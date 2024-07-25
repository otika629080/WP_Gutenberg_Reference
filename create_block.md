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

```jsx
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

```jsx
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

前述したように、サーバーサイドのブロック登録はメインの.phpファイルで行われます。しかし、ここでは.phpファイル内で設定を定義しません。代わりに、block.jsonファイルを使用します。

もう一度block.jsonを開いて、デフォルトの設定を詳しく見てみましょう。

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

### スクリプトとスタイル
editorScript、editorStyle、styleプロパティには、フロントエンドとバックエンドのスクリプトとスタイルへの相対パスを指定します。

ここで定義したスクリプトとスタイルは、WordPressによって自動的に登録され、キューに入れられるため、手動で登録する必要はありません。

buildフォルダ内のindex.jsスクリプトは、正しくキューに入れられています。PHPコードを記述する必要はありません。

### UIラベル
titleとdescriptionプロパティには、エディター上でのブロックの識別に必要なラベルを指定します。

### キーワード
前述したように、プロパティと属性を使用してブロックを設定できます。例えば、1つ以上のkeywordsを追加して、ブロックを検索しやすくできます。

```json
"keywords": [ 
	"kinsta", 
	"academy", 
	"superhero"
      ],
```

クイックインサーターに「kinsta」「academy」または「superhero」を入力すると、エディターに「Kinsta Academy」ブロックが表示されます。

### ローカライズ

JSONファイル内の文字列をローカライズする方法については、こちらをご覧ください。

JavaScript内では、@wordpress/blocksパッケージのregisterBlockTypeFromMetadataメソッドと、block.jsonファイルから読み込んだメタデータを使用して、ブロックタイプを登録できるようになりました。すべてのローカライズされたプロパティは、PHPのregister_block_type_from_metadataと同様に、自動的に_x（@wordpress/i18nパッケージ）関数呼び出しにラップされます。唯一の要件として、block.jsonファイルにtextdomainプロパティを設定してください。

ここでは、registerBlockTypeFromMetadataの代わりにregisterBlockType関数を使用します。Gutenberg 10.7以降、registerBlockTypeFromMetadataが非推奨になったためですが、仕組みは同じです。

### 組み込みコンポーネントの使用─RichTextコンポーネント

Gutenbergブロックを構成する要素は、Reactコンポーネントであり、wpグローバル変数でアクセスできます。例えば、ブラウザのコンソールにwp.editorと入力すると、wp.editorモジュールに含まれるコンポーネントの一覧が表示されます。(外観>エディタで、サイドバーを表示している場合)

リストをスクロールして、コンポーネントの名前から中身を推測してみてください。

同様に、wp.componentsモジュールに含まれるコンポーネントの一覧を確認できます。

Info
モジュールプログラミングは、ソフトウェア設計技法の1つで、プログラムの機能を、独立した交換可能なモジュールに分離します。各モジュールには、機能の特定部分の実行に必要なコードのみが含まれます（出典：Wikipedia）。

edit.jsファイルに戻り、スクリプトを詳しく見てみましょう。

```js
import { __ } from '@wordpress/i18n';
import { useBlockProps } from '@wordpress/block-editor';
import './editor.scss';

export default function Edit() {
	return (
		<p { ...useBlockProps() }>
			{ __(
				'Kinsta Academy Block – hello from the editor!',
				'ka-example-block'
			) }
		</p>
	);
}
```

このコードは、シンプルで編集できないテキストを含む静的ブロックを生成しますが、簡単に変更できます。

テキストを編集可能にするには、現在の<p>タグを、入力内容を編集できるコンポーネントで置き換えます。Gutenbergには、この目的で使用できる組み込みのRichTextコンポーネントがあります。

組み込みコンポーネントは、5つのステップでブロックに追加できます。

1. WordPressパッケージから必要なコンポーネントをインポートする
2. JSXコードに対応する要素を追加する
3. block.jsonファイルに必要な属性を定義する
4. イベントハンドラを定義する
5. データを保存する

#### ステップ1. WordPressパッケージから必要なコンポーネントをインポートする

edit.jsファイルを開き、import文を変更します。

```js
import { useBlockProps } from '@wordpress/block-editor';
```

上のimport文を以下のように変更してください。

```js
import { useBlockProps, RichText } from '@wordpress/block-editor';
```

@wordpress/block-editorパッケージから、useBlockProps関数とRichTextコンポーネントをインポートしています。

#### useBlockProps
useBlockProps Reactフックは、ブロックのラッパー要素をマークします。

APIバージョン2を使用するには、ブロックのedit関数内で新しいuseBlockPropsフックを使用して、ブロックのラッパー要素をマークする必要があります。useBlockPropsフックは、ブロックの動作の有効化に必要な属性とイベントハンドラを挿入します。ブロック要素にはuseBlockPropsを介して属性を渡し、戻り値は要素に展開する必要があります。

つまり、useBlockPropsは、ラッパー要素（この例ではp要素）に自動的に属性とクラスを割り当てます。

ラッパー要素からuseBlockPropsを削除すると、単純なテキスト文字列になってしまい、ブロックの機能やスタイルにアクセスできません。

後で触れますが、useBlockPropsには、プロパティのオブジェクトを渡しても、出力をカスタマイズできます。

#### RichText

RichTextコンポーネントには編集可能な入力領域があり、ユーザーはコンテンツを編集し、書式を設定できます。

GitHubのRichTextコンポーネントのドキュメントをご覧ください。

### ステップ2. JSXコードに対応する要素を追加する

```jsx
const blockProps = useBlockProps();

return (
	<RichText 
		{ ...blockProps }
		tagName="p"
		onChange={ onChangeContent }
		allowedFormats={ [ 'core/bold', 'core/italic' ] }
		value={ attributes.content }
		placeholder={ __( 'Write your text...' ) }
	/>
);
```

各行は以下の通りです。

- tagName：編集可能なHTML要素のタグ名
- onChange：要素の内容が変更されたときに呼び出される関数
- allowedFormats：許可されるフォーマットの配列（デフォルトでは、すべてのフォーマットが許可されます）
- value：編集可能になるHTML文字列
- placeholder：要素が空のときに表示されるプレースホルダテキスト

### ステップ3. block.jsonファイルに必要な属性を定義する

属性は、リッチコンテンツ、背景色、URLなど、ブロックが保存するデータに関する情報を保持します。

attributesオブジェクト内には、キーと値のペアで任意の数の属性を設定できます。キーは属性名、値は属性定義です。

block.jsonファイルを開き、以下のattributesプロパティを追加します。

```json
"attributes": {
	"content": {
		"type": "string",
		"source": "html",
		"selector": "p"
	}
},
```

content属性は、編集可能フィールドに入力されたテキストを格納できます。

- typeは、属性によって保存されるデータの種類を示します。このプロパティは、enumプロパティを定義しない限り必須です。
- sourceは、投稿のコンテンツから、どのように属性値を抽出するかを定義します。この例では、HTMLコンテンツです。注意）このプロパティを指定しなければ、データはブロックデリミタに格納されます（詳細はこちら）。
- selectorは、HTMLタグ、またはクラス名やid属性などのその他のセレクタです。
- Edit関数には、プロパティのオブジェクトを渡します。edit.jsファイルに戻り、以下のように変更してください。

```jsx
export default function Edit( { attributes, setAttributes } ) { ... }
```

### ステップ4. イベントハンドラを定義する

RichText要素にはonChange属性があり、要素の内容が変更されたときに呼び出される関数を指定します。

以下は、この関数を定義したedit.jsスクリプトの全文です。

```jsx
import { __ } from '@wordpress/i18n';
import { useBlockProps, RichText } from '@wordpress/block-editor';
import './editor.scss';

export default function Edit( { attributes, setAttributes } ) {
	const blockProps = useBlockProps();

	const onChangeContent = ( newContent ) => {
		setAttributes( { content: newContent } )
	}

	return (
		<RichText 
			{ ...blockProps }
			tagName="p"
			onChange={ onChangeContent }
			allowedFormats={ [ 'core/bold', 'core/italic' ] }
			value={ attributes.content }
			placeholder={ __( 'Write your text...' ) }
		/>
	);
}
```

useBlockProps は WordPress のブロックエディタ（Gutenberg）で使用される React フックです。このフックは、ブロックに必要な属性やイベントハンドラを提供し、ブロックの編集インターフェースを適切に機能させるために重要な役割を果たします。
主な特徴と機能：

1. 標準属性の提供：
ブロックに必要な標準的な属性（クラス名、ID など）を自動的に追加します。
2. イベントハンドラの追加：
ブロックの選択、フォーカス、ドラッグアンドドロップなどの操作に必要なイベントハンドラを提供します。
3. スタイルの適用：
ブロックエディタ内でのブロックの表示に必要なスタイルを適用します。
4. アクセシビリティの向上：
ARIA 属性などのアクセシビリティ関連の属性を自動的に追加します。
5. ブロック型の識別：
ブロックの種類を識別するための属性を追加します。

ファイルを保存して、ターミナルウィンドウでnpm run startを実行します。WordPressの管理画面に戻り、新規投稿または固定ページを作成し、Kinsta Academyブロックを追加します。

テキストを入力して、コードビューに切り替えます。コードは以下のようになります。

```jsx
<!-- wp:ka-example-block/ka-example-block -->
<p class="wp-block-ka-example-block-ka-example-block">Kinsta Academy Block – hello from the saved content!</p>
<!-- /wp:ka-example-block/ka-example-block -->
```

ここでページを保存してフロントエンドの表示をチェックしても、残念なことに変更はサイトに反映されていません。save.jsファイルを修正して、投稿を保存する際、データベースに入力を保存する必要があります。

### ステップ5. データを保存する
save.jsファイルを開き、スクリプトを次のように変更します。

```jsx
import { __ } from '@wordpress/i18n';
import { useBlockProps, RichText } from '@wordpress/block-editor';

export default function save( { attributes } ) {
	const blockProps = useBlockProps.save();
	return (
		<RichText.Content 
			{ ...blockProps } 
			tagName="p" 
			value={ attributes.content } 
		/>
	);
}
```

ここでは以下を実行しています。

- block-editorパッケージからRichTextコンポーネントをインポートする
- save関数にオブジェクト引数を介して複数のプロパティを渡す（この例ではattributesプロパティのみを渡しています）
- RichTextコンポーネントのコンテンツを返す

Important

save関数を変更するたびに、エディターキャンバスのブロックインスタンスを削除し、再度インクルードする必要があります。詳細についてはこのページの「妥当性検証」セクションをご覧ください。

RichTextコンポーネントの詳細については、ブロックエディターハンドブック、プロパティの一覧については、Githubを参照してください。

次は、ブロックツールバーにコントロールを追加します。

## ブロックツールバーへのコントロールの追加

ブロックツールバーには、ブロックコンテンツの一部を操作できるコントロールが並びます。各ツールバーコントロールには、コンポーネントがあります。

例えば、ブロックにはテキスト配置コントロールを追加できます。必要な作業は、@wordpress/block-editorパッケージから2つのコンポーネントをインポートするだけです。

前の例と同じ手順で進めていきます。

- WordPressパッケージから必要なコンポーネントをインポートする
- JSXコードに対応する要素を追加する
- block.jsonファイルに必要な属性を定義する
- イベントハンドラを定義する
- データを保存する

### ステップ1. @wordpress/block-editorからBlockControlsとAlignmentControlコンポーネントをインポートする

ブロックツールバーに配置コントロールを追加するには、2つのコンポーネントが必要です。

- BlockControlsは、コントロールの動的ツールバーをレンダリングします（ドキュメントなし）。
- AlignmentControlは、選択されたブロックの整列方法を表示するドロップダウンメニューをレンダリングします（詳細についてはこちら）。

edit.jsを開き、以下のようにimport文を編集します。

```jsx
import { 
	useBlockProps, 
	RichText, 
	AlignmentControl, 
	BlockControls 
} from '@wordpress/block-editor';
```

### ステップ2. BlockControlsとAlignmentControl要素を追加する

Edit関数に移動し、<BlockControls />要素を<RichText />と同じレベルで挿入します。次に、<BlockControls />の中に<AlignmentControl />を追加します。

```jsx
export default function Edit( { attributes, setAttributes } ) {
	const blockProps = useBlockProps();
	return (
		<>
			<BlockControls>
				<AlignmentControl
					value={ attributes.align }
					onChange={ onChangeAlign }
				/>
			</BlockControls>
			<RichText 
				{ ...blockProps }
				tagName="p"
				onChange={ onChangeContent }
				allowedFormats={ [ 'core/bold', 'core/italic' ] }
				value={ attributes.content }
				placeholder={ __( 'Write your text...' ) }
				style={ { textAlign: attributes.align } }
			/>
		</>
	);
}
```

上のコードで<>と</>は、Reactで複数の要素を返す「フラグメント」を宣言する短い記法です。

この例では、AlignmentControlに2つの属性があります。

- valueは、この要素の現在の値を表します
- onChangeは、値が変更されたときに実行するイベントハンドラを指定します

また、RichText要素に追加の属性も定義しています（例付きの属性の一覧についてはこちら）。

### ステップ3. block.jsonにalgin属性を定義する

block.jsonファイルに移動して、align属性を追加します。

```json
"align": {
	"type": "string",
	"default": "none"
}
```

ブロックエディターに戻り、ページを更新してブロックを選択します。ブロックツールバーに、配置コントロールが表示されるはずです。

これは、まだイベントハンドラを定義していないのが原因です。

### ステップ4. イベントハンドラを定義する
次に、onChangeAlignを以下のように定義します。

```jsx
const onChangeAlign = ( newAlign ) => {
	setAttributes( { 
		align: newAlign === undefined ? 'none' : newAlign, 
	} )
}
```

newAlignがundefinedであれば、noneに設定し、そうでなければ、newAlignを使用します。

これで、edit.jsスクリプトは（一旦）完了です。

```jsx
export default function Edit( { attributes, setAttributes } ) {
	const blockProps = useBlockProps();
	const onChangeContent = ( newContent ) => {
		setAttributes( { content: newContent } )
	}
	const onChangeAlign = ( newAlign ) => {
		setAttributes( { 
			align: newAlign === undefined ? 'none' : newAlign, 
		} )
	}
	return (
		<>
			<BlockControls>
				<AlignmentControl
					value={ attributes.align }
					onChange={ onChangeAlign }
				/>
			</BlockControls>
			<RichText 
				{ ...blockProps }
				tagName="p"
				onChange={ onChangeContent }
				allowedFormats={ [ 'core/bold', 'core/italic' ] }
				value={ attributes.content }
				placeholder={ __( 'Write your text...' ) }
				style={ { textAlign: attributes.align } }
			/>
		</>
	);
}
```

これでエディターに戻って、ブロックのコンテンツを配置することができます。ブロックに配置ツールバーが表示されているはずです。

なお、投稿を保存すると、ブロックのコンテンツがブロックエディターで表示されるようにフロントエンドで配置されないことがあります。この場合、save関数を修正して、ブロックのコンテンツと属性をデータベースに保存する必要があります。

### ステップ5. データを保存する

save.jsを開き、save関数を以下のように変更してください。

```jsx
export default function save( { attributes } ) {
	const blockProps = useBlockProps.save();
	return (
		<RichText.Content 
			{ ...blockProps } 
			tagName="p" 
			value={ attributes.content } 
			style={ { textAlign: attributes.align } }
		/>
	);
}
```

最後に、分割代入構文を使用して、attributeオブジェクトから個々のプロパティを抽出し、コードを整理しましょう。

```jsx
export default function save( { attributes } ) {
	const blockProps = useBlockProps.save();
	const { content, align } = attributes;
	return (
		<RichText.Content 
			{ ...blockProps } 
			tagName="p" 
			value={ content } 
			style={ { textAlign: align } }
		/>
	);
}
```

edit.jsファイルでも同じことができます。

ファイルを保存して、コードエディターモードでエディターに戻ります。コードは以下のようになるはずです。

```jsx
<!-- wp:ka-example-block/ka-example-block {"align":"right"} -->
<p class="wp-block-ka-example-block-ka-example-block" style="text-align:right">This is my first editable <strong>Gutenberg</strong> <em>block</em> 😎</p>
<!-- /wp:ka-example-block/ka-example-block -->
```

これで完了です。ブロックツールバーに配置コントロールを追加できました🤓

ブロックツールバーコントロールの詳細については、ブロックエディターハンドブックの「ブロックツールバーと設定サイドバー」を参照してください。

## ブロック設定サイドバーの編集

コントロールは、ブロック設定サイドバーにも追加できます（アプリケーション用に新規サイドバーを作成することも）。

APIとしては、InspectorControlsコンポーネントがあります。

ブロックエディターハンドブックで、設定サイドバーの使い方を以下のように説明しています。

設定サイドバーは、あまり使わない設定や、大きな画面スペースが必要な設定で使用されます。設定サイドバーはブロックレベル設定でのみ使用してください。

ブロック内の選択したコンテンツでのみ有効な設定は（例：段落内の選択したテキストに対する「太字」設定）、設定サイドバーの中に置かないでください。設定サイドバーはHTMLモードでブロックを編集するときにも表示されるため、ブロックレベルの設定のみを配置する必要があります。

再び、同じ手順で進めます。

- WordPressパッケージから必要なコンポーネントをインポートする
- JSXコードに対応する要素を追加する
- block.jsonファイルに必要な属性を定義する
- イベントハンドラを定義する
- データを保存する

### ステップ1. @wordpress/block-editorからInspectorControlsとPanelColorSettingsコンポーネントをインポートする

ブロックの特定部分をカスタマイズできるようにするには、必要なコントロールを追加します。例えば、カラーコントロールパネルを実装するには、block-editorモジュールからInspectorControlsとPanelColorSettingsコンポーネントをインポートします。

```jsx
import { 
	useBlockProps, 
	RichText, 
	AlignmentControl, 
	BlockControls,
	InspectorControls,
	PanelColorSettings
} from '@wordpress/block-editor';
```

### ステップ2. JSXコードに対応する要素を追加する

次に、Edit関数が返すJSXに、対応する要素を追加します。

```jsx
export default function Edit( { attributes, setAttributes } ) {
	const blockProps = useBlockProps();

	const { content, align, backgroundColor, textColor } = attributes;

	const onChangeContent = ( newContent ) => {
		setAttributes( { content: newContent } )
	}
	const onChangeAlign = ( newAlign ) => {
		setAttributes( { 
			align: newAlign === undefined ? 'none' : newAlign, 
		} )
	}
	return (
		<>
			<InspectorControls>
				<PanelColorSettings 
					title={ __( 'Color settings', 'ka-example-block' ) }
					initialOpen={ false }
					colorSettings={ [
						{
						  value: textColor,
						  onChange: onChangeTextColor,
						  label: __( 'Text color', 'ka-example-block' )
						},
						{
						  value: backgroundColor,
						  onChange: onChangeBackgroundColor,
						  label: __( 'Background color', 'ka-example-block' )
						}
					] }
				/>
			</InspectorControls>
			<BlockControls>
				<AlignmentControl
					value={ align }
					onChange={ onChangeAlign }
				/>
			</BlockControls>
			<RichText 
				{ ...blockProps }
				tagName="p"
				onChange={ onChangeContent }
				allowedFormats={ [ 'core/bold', 'core/italic' ] }
				value={ content }
				placeholder={ __( 'Write your text...' ) }
				style={ { textAlign: align, backgroundColor: backgroundColor, color: textColor } }
			/>
		</>
	);
}
```

なお、RichText要素のstyle属性も更新します。

```jsx
<RichText 
	 { ...blockProps }
	 tagName="p"
	 onChange={ onChangeContent }
	 allowedFormats={ [ 'core/bold', 'core/italic' ] }
	 value={ content }
	 placeholder={ __( 'Write your text...' ) }
	 style={ { textAlign: align, backgroundColor: backgroundColor, color: textColor } }
/>
```

ステップ3. block.jsonに必要な属性を定義する
ここで、block.jsonファイルにbackgroundColorとtextColor属性を定義します。

```jsx
"attributes": {
	"content": {
		"type": "string",
		"source": "html",
		"selector": "p"
	},
	"align": {
		"type": "string",
		"default": "none"
	},
	"backgroundColor": {
		"type": "string"
	},	 
	"textColor": {
		"type": "string"
	}
},
```

### ステップ4. イベントハンドラを定義する

次に、入力に応じてbackgroundColorとtextColorを更新する2つの関数を定義します。

```jsx
const onChangeBackgroundColor = ( newBackgroundColor ) => {
	setAttributes( { backgroundColor: newBackgroundColor } )
}

const onChangeTextColor = ( newTextColor ) => {
	setAttributes( { textColor: newTextColor } )
}
```

### ステップ5. データを保存する

最後にsave.jsファイルを開き、スクリプトを以下のように変更します。

```jsx
export default function save( { attributes } ) {
	const blockProps = useBlockProps.save();
	const { content, align, backgroundColor, textColor } = attributes;
	return (
		<RichText.Content 
			{ ...blockProps } 
			tagName="p" 
			value={ content } 
			style={ { textAlign: align, backgroundColor: backgroundColor, color: textColor } }
		/>
	);
}
```

ファイルを保存して、エディターでブロックを確認します。ブロックに予期せぬ、または無効なコンテンツが含まれていることを知らせるエラーメッセージが表示される場合があります。

このエラーは、save.jsファイルが変更され、データベースに保存されたコードがエディターで使用されているコードと一致しない時に発生します。

これを修正するには、ページを更新し、ブロックのインスタンスを一度削除して、再度投稿に追加してください。

変更を加え、投稿を保存して、フロントエンドで表示します。ブロックエディターで行った変更が、サイトにも反映されているはずです。

### 外部リンクの追加と編集
次に、ブロックタイプに新しいコンポーネントを追加します。

- カスタムブロックにカスタマイズ可能なリンクを追加できるようにするExternalLinkコンポーネント
- リンクの設定をカスタマイズできる複数のサイドバーコントロール

#### ステップ1. @wordpress/componentsからコンポーネントをインストールする

@wordpress/componentsから複数のコンポーネントをインポートする必要があります。edit.jsを開き、以下のimport文を追加してください。

```jsx
import {
	TextControl,
	PanelBody,
	PanelRow,
	ToggleControl,
	ExternalLink
} from '@wordpress/components';
```

- PanelBody：設定サイドバーに折りたたみ可能なコンテナを追加
- PaneRow：サイドバーコントロール用の汎用コンテナを生成
- TextControl：テキスト入力用のコントロール
- ToggleControl：特定のオプションの有効・無効を切り替えられるトグル用コントロール
- ExternalLink：外部リンクを追加するシンプルなコンポーネント

#### ステップ2. JSXコードに対応する要素を追加する

まず、divコンテナ内に、RichTextと同じレベルでExternalLink要素を追加します。

```jsx
<div { ...blockProps }>
	<RichText 
		...
	/>
	<ExternalLink 
		href={ kaLink }
		className="ka-button"
		rel={ hasLinkNofollow ? "nofollow" : "" }
	>
			{ linkLabel }
	</ExternalLink>

</div>
```

なお、ExternalLinkコンポーネントはドキュメント化されていないため、コンポーネントのソースコードを参照して利用可能な属性の一覧を取得しました。ここでは、href、className、rel属性を使用しています。

デフォルトでは、rel属性の値はnoopener noreferrerに設定されます。このコードでは、トグルコントロールがオンのとき、結果のaタグのrel属性にnofollowキーワードが追加されます。

ブロックサイドバーにリンク設定を追加します。まず、InspectorControls内に、PanelColorSettingsと同じレベルでPanelBody要素を追加します。

```jsx
<InspectorControls>
	<PanelColorSettings 
	...
	/>
	<PanelBody 
		title={ __( 'Link Settings' )}
		initialOpen={true}
	>
	...
	</PanelBody>
</InspectorControls>
```

今回は、以下を実行しています。

- title属性でパネルのタイトルを指定
- initialOpenで初期状態でパネルが開いているかどうかを設定
  
次に、PanelBodyの中に2つのPanelRow要素を追加し、それぞれのPanelRowの中にTextControl要素を追加します。

```jsx
<PanelBody 
	title={ __( 'Link Settings', 'ka-example-block' )}
	initialOpen={true}
>
	<PanelRow>
		<fieldset>
			<TextControl
				label={__( 'KA link', 'ka-example-block' )}
				value={ kaLink }
				onChange={ onChangeKaLink }
				help={ __( 'Add your Academy link', 'ka-example-block' )}
			/>
		</fieldset>
	</PanelRow>
	<PanelRow>
		<fieldset>
			<TextControl
				label={__( 'Link label', 'ka-example-block' )}
				value={ linkLabel }
				onChange={ onChangeLinkLabel }
				help={ __( 'Add link label', 'ka-example-block' )}
			/>
		</fieldset>
	</PanelRow>
</PanelBody>
```

上のコードは、もうかなり簡単に思えるはず。それから、2つのテキストコントロールにリンクラベルとURLを設定します。

さらに、ToggleControlを含むPanelRowを追加して、属性を含めるかどうかなど、特定のオプションの有効、無効を切り替えられるようにします。

```jsx
<PanelRow>
	<fieldset>
		<ToggleControl
			label="Add rel = nofollow"
			help={
				hasLinkNofollow
					? 'Has rel nofollow.'
					: 'No rel nofollow.'
			}
			checked={ hasLinkNofollow }
			onChange={ toggleNofollow }
		/>
	</fieldset>
</PanelRow>
```

#### ステップ3. block.jsonに必要な属性を定義する

block.jsonファイルにkaLink、linkLabel、hasLinkNofollow属性を定義します。

```jsx
"kaLink": {
	"type": "string",
	"default": ""
},
"linkLabel": {
	"type": "string",
	"default": "Check it out!"
},
"hasLinkNofollow": {
	"type": "boolean",
	"default": false
}
```

これ以上、何かを追加する必要はありません。イベントハンドラ関数の定義に移りましょう。

#### ステップ4. イベントハンドラを定義する

```jsx
const { content, align, backgroundColor, textColor, kaLink, linkLabel, hasLinkNofollow } = attributes;

const onChangeKaLink = ( newKaLink ) => {
	setAttributes( { kaLink: newKaLink === undefined ? '' : newKaLink } )
}

const onChangeLinkLabel = ( newLinkLabel ) => {
	setAttributes( { linkLabel: newLinkLabel === undefined ? '' : newLinkLabel } )
}

const toggleNofollow = () => {
	setAttributes( { hasLinkNofollow: ! hasLinkNofollow } )
}
```

この関数は、入力に対応する属性値を更新します。

#### ステップ5. データを保存する

最後に、save.jsのsave関数を更新します。

```jsx
export default function save( { attributes } ) {
	
	const { content, align, backgroundColor, textColor, kaLink, linkLabel, hasLinkNofollow } = attributes;

	const blockProps = useBlockProps.save( {
		className: `has-text-align-${ align }`
	} );
	
	return (
		<div 
			{ ...blockProps }
			style={ { backgroundColor: backgroundColor } }
		>
			<RichText.Content 
				tagName="p" 
				value={ content } 
				style={ { color: textColor } }
			/>
			<p>
				<a 
					href={ kaLink }
					className="ka-button"
					rel={ hasLinkNofollow ? "nofollow" : "noopener noreferrer" }
				>
					{ linkLabel }
				</a>
			</p>
		</div>
	);
}
```

なお、今回の例では、ExternalLinkの代わりに、通常のa要素を使用しています。

結果は以下のようになります。

### 複数のブロックスタイルの追加

前のセクションでは、テキストの配置を設定できる、ブロックツールバーコントロールの追加方法について学びました。同じ手順でブロックツールバーにさらにスタイルコントロールを追加できますが、ワンクリックで選択できる、定義済みのブロックスタイルを追加する方法もあります。

これにはブロックAPIの便利な機能ブロックスタイルを使用します。

block.jsonのstylesプロパティを定義して、対応するスタイルをスタイルシートで宣言します。

例えば、次のようなスタイルの配列を追加できます。

```json
"styles": [
	{
		"name": "default",
		"label": "Default",
		"isDefault": true
	},
	{
		"name": "border",
		"label": "Border"
	}
],
```

これで、デフォルトのスタイルと、border（枠）と呼ばれるスタイルが追加されました。ブロックエディターに戻ります。

スタイルは、ブロックスイッチャーをクリックし、ブロック設定サイドバーの「スタイル」パネルから利用できます。

スタイルを選択し、p要素に適用されるクラスを確認します。ブロック上で右クリックして、「検証」を選択してください。次の形式の名前の新しいクラスが追加されています。

```html
is-style-{style-name}
```

「Border」スタイルを選択すると、p要素にis-style-borderクラスが追加されます。「Default」スタイルを選択すると、代わりにis-style-defaultクラスが追加されます。

あとは、CSSのプロパティを宣言するだけです。editor.scssを開き、現在のスタイルを以下のように置き換えます。

```css
.wp-block-ka-example-block-ka-example-block {
    padding: 4px;
}
```

次に、style.scssにも同様に追加します。前述したように、style.scssで定義されたスタイルは、フロントエンドとエディターの両方に適用されます。

```css
.wp-block-ka-example-block-ka-example-block {
	&.is-style-default{
		border: 0;
        background-color: #FFE2C7;
	}
	&.is-style-border{
		border: 2px solid #000;
        border-radius: 16px;
        background-color: #F6F6F6;
	}
}
```

これで完了です。ページを更新して、新たなブロックスタイルをお楽しみください。






