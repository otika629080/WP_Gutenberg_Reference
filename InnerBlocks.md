# InnerBlocksコンポーネントを使用したGutebnergブロックのネスト

このカスタムブロックは、機能的には十分ですが、このままでは魅力的な外観とは言えません。訪問者の目を引く写真やグラフィックを追加することができます。

画像を追加すると、ブロックに複雑なレイヤーを重ねてしまう可能性がありますが、幸い一から作りなおす必要はありません。Gutenbergには、ネストしたブロック構造を作成する特別なコンポーネントがあります。

InnerBlocksコンポーネントは、以下のように定義されています。

InnerBlocksは、コンポーネントのペアをエクスポートし、ネストしたブロックコンテンツを実現するブロックを実装するもの。（英語原文の日本語訳）

まず、srcフォルダに.jsファイルを作成します。今回の例では、container.jsを作成します。

次に、新しいリソースをindex.jsファイルにインポートします。

```js
import './container';
```

container.jsに戻り、必要なコンポーネントをインポートします。

```jsx
import { registerBlockType } from "@wordpress/blocks";
import { __ } from "@wordpress/i18n";
import {
	useBlockProps, 
	InnerBlocks 
} from "@wordpress/block-editor";
```

次に、内部におけるブロックの配置を規定するテンプレートを定義します。この例では、2つのカラムからなるテンプレートを定義します。

それぞれのカラムに、コアとなる画像ブロックとカスタムブロックを配置します。

```jsx
const TEMPLATE = [ [ 'core/columns', { backgroundColor: 'yellow', verticalAlignment: 'center' }, [
	[ 'core/column', { templateLock: 'all' }, [
		[ 'core/image' ],
	] ],
	[ 'core/column', { templateLock: 'all' }, [
		[ 'ka-example-block/ka-example-block', { placeholder: 'Enter side content...' } ],
	] ],
] ] ];
```

テンプレートの構造は、blockTypes（ブロック名と任意の属性）の配列です。

上のコードでは、複数の属性を使用して、ColumnsブロックとColumnブロックを構成しています。

特に、templateLock: 'all'属性は、Columnブロックをロックし、不用意に既存のブロックを追加、並べ替え、削除できないようにしています。

templateLockは、以下のいずれかの値を取ります。

- all：InnerBlocksがロックされブロックの追加、並べ替え、削除ができなくなる
- insert：ブロックの並べ替えと削除のみ可能
- false：テンプレートがロックされない

テンプレートは、InnerBlocks要素に割り当てられます。

```jsx
<InnerBlocks
	template={ TEMPLATE }
	templateLock="all"
/>
```

互換性の問題の回避のため、InnerBlocksコンポーネントにtemplateLock属性も追加します（issue #17262とpull #26128参照）。

最終的なcontainer.jsファイルは、以下の通りです。

```jsx
registerBlockType('ka-example-block/ka-example-container-block', {
	title: __( 'KA Container block', 'ka-example-block' ),
	category: 'design',

	edit( { className } ) {
		
		return(
			<div className={ className }>
				<InnerBlocks
					template={ TEMPLATE }
					templateLock="all"
				/>
			</div>
		)
	},

	save() {
		const blockProps = useBlockProps.save();
		return(
			<div { ...blockProps }>
				<InnerBlocks.Content />
			</div>
		)
	},
});
```

## 細かな編集

これでブロックが機能するようになりましたが、さらにちょっとした編集を加えることで、ブロックを改良することもできます。

RichTextコンポーネントで生成される段落には、backgroundColor属性を割り当てましたが、コンテナのdivに背景色を割り当てた方が良いかもしれません。

その場合は、edit.jsファイルとsave.jsファイルのdivを以下のように変更します。

```jsx
<div 
	{ ...blockProps }
	style={ { backgroundColor: backgroundColor } }
>
...
</div>
```

これでブロック全体の背景を変更できます。

一方、関連性の高い変更が、useBlockPropsメソッドです。今回の例では、元のコードでは、定数blockPropsを以下のように定義しました。

```jsx
const blockProps = useBlockProps();
```

しかし、プロパティのセットを渡すことで、useBlockPropsを効果的に使用できます。例えば、classnamesモジュールからclassnamesをインポートして、それに応じたラッパークラス名を設定できます。

以下の例では、align属性の値に基づいてクラス名を割り当てています（edit.js）。

```jsx
import classnames from 'classnames';

...

export default function Edit( { attributes, setAttributes } ) {
	...
	
	const onChangeAlign = ( newAlign ) => {
		setAttributes( { 
			align: newAlign === undefined ? 'none' : newAlign, 
		} )
	}

	const blockProps = useBlockProps( {
		className: `has-text-align-${ align }`
	} );
	...
}
```

save.jsも同様に変更します。

```jsx
import classnames from 'classnames';

...

export default function save( { attributes } ) {
	...
	const { content, align, backgroundColor, textColor, kaLink, linkLabel, hasLinkNofollow } = attributes;

	const blockProps = useBlockProps.save( {
		className: `has-text-align-${ align }`
	} );
	...
}
```

これで終了です！本番用のビルドを実行しましょう。

```sh
npm run build
```

















































































































































































































































































































































