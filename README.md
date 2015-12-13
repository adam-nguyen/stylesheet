# stylesheet

FLOCSSをベースにしたオブジェクト指向のスタイルシートです。以下のような特徴があります。

* [FLOCSS](https://github.com/hiloki/flocss)をベースにしたディレクトリ構成。
* OOCSSをベースにしたマルチクラス設計。
* [MindBEMding](http://csswizardry.com/2013/01/mindbemding-getting-your-head-round-bem-syntax/)をベースにした命名規則。
* レスポンシブ対応のグリッドシステム。

## ディレクトリ構成
FLOCSSの構成にいくつかのレイヤーを足した以下のような構成になっています。

1. Foundation
    1. Function
    2. Variable
    3. Mixin
    4. Vendor
    5. Vendor-extension
    6. Base
1. Layout
2. Object
    1. Component
    2. Project
    3. Utility

後ろのレイヤーになるほど具体的で詳細度が高く、カスケーディングにおいて強くする必要があります。そのため、レイヤーの順番を変更してはいけません。

### Foundation
FunctionとVariable、MixinにはSassの機能である関数と変数を定義しています。多くの関数や変数を定義する場合は機能ごとにファイルを分割します。特定のmixinやレイヤーでのみ使用する変数や関数は、その特定のファイル内で定義します。

```scss
$my-colors: (
    text: (
        base: #333,
        light: #666,
        dark: #000,
    ),
    link: (
        base: #2b70ba,
        visited: null,
        hover: lighten(#2b70ba, 15%),
    ),
    ui: (
        base: #aaa,
        light: lighten(#aaa, 15%),
        dark: darken(#aaa, 15%),
    ),
    bg: (
        base: #fff,
        light: lighten(#fff, 5%),
        dark: darken(#fff, 10%),
    ),
) !default;
```

```scss
@function my-colors($key, $tone: base) {
    @return map-get( map-get($my-colors, $key), $tone);
}
```

Vendorはnormalize.cssやBootstrapのような外部のライブラリやフレームワークです。スタイルの上書きはVendor-extensionレイヤーでそれぞれのファイルを作成しておこないます。これは[Sass Guidelines](http://sass-guidelin.es/#the-7-1-pattern)のVENDORS FOLDERのアイデアを取り入れています。

Baseにはプロジェクトにおける、基本的なベーススタイルを定義します。要素セレクタや属性セレクタなど、詳細度はできるかぎり低く保っておきます。

```scss
html {
    box-sizing: border-box;
}

*,
*:before,
*:after {
    box-sizing: inherit;
}

h1,
h2,
h3,
h4,
h5,
h6 {
    color: inherit;
    font-family: inherit;
    font-weight: 600;
    line-height: 1.2;
}
```

### Layout
ページを構成するヘッダーやフッター、メインコンテンツ部分などのコンテナブロックのスタイルを定義します。目安としてはワイヤーフレームに書かれるような大きな単位のブロックです。

ページ内で唯一の要素となるのでIDセレクタを使用することができます。IDセレクタはこのレイヤーでのみ指定することができます。

```scss
#page-header {}
#page-content {}
```

指定するスタイルは最小限度に止め、IDセレクタを親にしたセレクタは指定を禁止します。

クラスで指定するようなグリッドシステムはobject/componentレイヤーで定義します。

### Object
プロジェクトにおけるビジュアルパターンをObjectと定義します。抽象度や詳細度、拡張性などによって3つのレイヤーに分けます。

#### Component
多くのプロジェクトで横断的に再利用のできるような、小さな単位のモジュールを定義します。

buttonオブジェクトのベーススタイルや横幅を制限するwrapperオブジェクト、mediaやgridなどのレイアウトパターンがこれに該当します。**OOCSSで言うところの構造の機能を担う**ため、固有の幅や色やボーダーなどはできるだけ含めないようにします。**見た目の部分はProjectレイヤーで指定します**。

モディファイアを定義して、（主に`padding`による）サイズ違いや、配置（`float`や`vertical-align`など）を変更できるようにしておきます。これにはスタイルの重複や肥大化、似たような見た目のデザインをなるべく防ぐという目的もあります。

```html
<div class="c-media">
  <figure class="c-media__image c-media__image--rev">
    <img>
  </figure>
  <div class="c-media__body">
    <p></p>
  </div>
</div>
```

```scss
.c-media__image {
    float: left;
    margin-right: $media-gutter;
    > img {
        display: block;
        max-width: none;
    }
}

.c-media__image--rev {
    float: right; // スタイルの上書き
    margin-right: 0; // スタイルのリセット
    margin-left: $media-gutter; // スタイルの追加
}
```

#### Project
プロジェクト固有のパターンで、例えば記事一覧やモーダルといったユーザーインターフェイス（言葉、見た目、動きによってユーザーと対話をおこなうために使用するもの）が該当します。Componentとしてはスタイル（色やボーダーなど）を多く持っている見た目を定義したオブジェクトと考えます。

Componentのモディファイアで定義するのが適切でない場合はProjectレイヤーでスタイルを追加・上書きすることを許容します。ただし、その場合は詳細度を不必要に強くさせないために、Componentのオブジェクトを親セレクタとせず、詳細度ではなくカスケーディング（同じ詳細なら後に記述した方が優先される）によってスタイルを適応させます。

```html
<ol class="c-rank p-pagination">
  <li class="c-rank__item p-pagination__item">
    <a href="#" class="c-rank__link p-pagination__link p-pagination__link--prev">Prev</a>
  </li>
  <li class="c-rank__item p-pagination__item">
    <a href="#" class="c-rank__link p-pagination__link">1</a>
  </li>
  <li class="c-rank__item p-pagination__item">
    <a href="#" class="c-rank__link p-pagination__link">2</a>
  </li>
  <li class="c-rank__item p-pagination__item">
    <a href="#" class="c-rank__link p-pagination__link">3</a>
  </li>
  <li class="c-rank__item p-pagination__item">
    <a href="#" class="c-rank__link p-pagination__link p-pagination__link--next">Next</a>
  </li>
</ol>
```

```scss
// Componentレイヤー
.c-rank {
    @include my-clearfix;
    margin: 0;
    padding: 0;
    list-style-type: none;
}

.c-rank__item {
    float: left;
    padding: $my-rank-space-y $my-rank-space-x;
}

.c-rank__link {
    display: inline-block;
    margin: (-$my-rank-space-y) (-$my-rank-space-x);
    padding: $my-rank-space-y $my-rank-space-x;
}

// Projectレイヤー
.p-pagination {
    line-height: 1;
    text-align: center;
}

.p-pagination__link {
    display: inline-block;
    padding: $pagination-padding-y $pagination-padding-x;
}

.p-pagination__link--prev:before {
    content: "\003C" "\00A0";
}

.p-pagination__link--next:after {
    content: "\00A0" "\003E";
}
```

基本的にはマルチクラスを想定していますが、@extendを使用したシングルクラスの設計にすることもできます。

```scss
.p-pagination {
    @extend .c-stack;
    line-height: 1;
    text-align: center;
}

.p-pagination__link {
    @extend .c-stack__link;
    display: inline-block;
    padding: $pagination-padding-y $pagination-padding-x;
}
```

コンパイルするとこのようになります。

```css
.c-stack,
.p-pagination {
    margin: 0;
    padding: 0;
    list-style-type: none;
}

.c-stack__link,
.p-pagination__link {
    display: block;
    margin: -1em 0;
    padding: 1em 0;
}
```

@extendを使用する注意点として、ComponentレイヤーからProjectレイヤーへの継承、同一ファイル内での継承のみが許容されます。また、シングルクラスとして設計する場合は混乱を避けるため、マルチクラスと併用しないようにします。

#### Utility
ComponentのモディファイアやProjectのオブジェクトで定義することが適切でない場合はUtilityレイヤーのオブジェクトを使用できます。`width`や`padding`といった単一のスタイルやClearfixのような任意の目的を持つヘルパークラスが定義されています。

例えばgridオブジェクトを使用したグリッドシステムなどに使用します。`u-col-8of12-md`がUtilityレイヤーのオブジェクトです。

```scss
<div class="c-wrapper">
  <div class="c-grid c-grid--medium">
    <div class="c-grid__item u-col-8of12-md"></div>
    <div class="c-grid__item u-col-4of12-md"></div>
  </div>
</div>
```

```scss
@media screen and (min-width: 768px) {
    .u-col-4of12-md {
        width: 33.33333% !important;
    }
    .u-col-5of12-md {
        width: 41.66667% !important;
    }
    .u-col-6of12-md {
        width: 50% !important;
    }
```

## 命名規則
### Modifier
Modifierで使用する単語は以下のような名前にしています。

* `__item`は`li`要素のように複数の要素が同じ役割を持っている場合に使用します。
* `__link`は`a`要素（アンカータグ）で使用します。
* `__unit`は `li`要素をさらにラップする必要がある場合に使用します。
* `__title`は見出し要素である場合に使用します。
* `__head`はBlock内の最初に配置する場合に使用します。
* `__body`はBlock内の真ん中に配置する場合に使用します。
* `__foot`はBlock内の最後に配置する場合に使用します。
* `__list`はBlock内の`ul`や`ol`などに使用します。

## グリッドシステム
レスポンシブ対応のグリッドシステムが定義されています。`inline-block`を使用しているので`vertical-align`で縦位置を変更することもできます。

ベースになるスタイルとマークアップ例です。

```scss
.c-grid {
    display: block;
    margin: 0;
    padding: 0;
    /* カラム間の余白を除去する */
    font-size: 0;
    list-style-type: none;
}

.c-grid__item {
    display: inline-block;
    width: 100%;
    /* フォントサイズを戻す */
    font-size: 1rem;
    vertical-align: top;
}
```

```html
<div class="c-grid c-grid--medium">
  <div class="c-grid__item u-col-8of12-md"></div>
  <div class="c-grid__item u-col-4of12-md"></div>
</div>
```

デフォルトではガター（要素間の余白）は設定されていませんので、`c-grid`にmodifierで指定します。

```scss
.c-grid--small {
    margin-left: - ($my-grid-gutter / 2);
    > .c-grid__item {
        padding-left: ($my-grid-gutter / 2);
    }
}

.c-grid--medium {
    margin-left: - $my-grid-gutter;
    > .c-grid__item {
        padding-left: $my-grid-gutter;
    }
}

.c-grid--large {
    margin-left: - ($my-grid-gutter * 2);
    > .c-grid__item {
        padding-left: ($my-grid-gutter * 2);
    }
}
```

縦位置揃えのmodifierです。

```scss
.c-grid--middle {
    > .c-grid__item {
        vertical-align: middle;
    }
}

.c-grid--bottom {
    > .c-grid__item {
        vertical-align: bottom;
    }
}
```

1カラムレイアウトでセンタリングしたいときには`.c-grid--center`を、右寄せにしたいときには`.c-grid--right`を指定します。

```scss
.c-grid--center {
    text-align: center;
    > .c-grid__item {
        text-align: left;
    }
}

.c-grid--right {
    text-align: right;
    > .c-grid__item {
        text-align: left;
    }
}
```

2カラムレイアウトで左右のカラムを反転したいときは`c-grid--rev`を指定します。

```scss
.c-grid--rev {
    text-align: left;
    /* 文字表記を右から左に変更します。 */
    direction: rtl;
    > .c-grid__item {
        /* IE以外ではテキストが右寄せのままになってしまう */
        text-align: left;
        /* derectionプロパティを元に戻します。*/
        direction: ltr;
    }
}
```

`width`プロパティの指定は`object/utility/_col.scss`で@mixinによって定義されています。生成されるCSSはこのようになります。

```css
.u-col-11of12 { width: 91.66667% !important; }
.u-col-12of12 { width: 100% !important; }
@media screen and (min-width: 400px) {
    .u-col-1of12-sm { width: 8.33333% !important; }
    .u-col-2of12-sm { width: 16.66667% !important; }
```

`c-grid__item`はデフォルトで`width:100%;`が指定されているので、以下のようにマークアップをすると、ブレイクポイントに応じて1カラムから3カラムまで指定することができます。

```html
/* 1カラム → 2カラム → 3カラム */
<div class="c-grid__item u-col-6of12-md u-col-4of12-lg"></div>
```