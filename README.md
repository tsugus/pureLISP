# pure LISP

## Abstruct

As the name suggests, this program is "pure LISP".

It supports super brackets '[', ']' to group and close brackets together.

## このプログラムについて。

純 LISP のインタプリタです。実際に純 LISP の腸循環評価器を動かせるものを目指しました（ただし、シミュレートされる LISP とする LISP とで細部の形式が微妙に異なりますが）。

## コンパイル方法

gcc もしくは clang で Makefile を実行してください。
それらのコンパイラのパスが通っている環境なら、

    % make

で OK。

余計なファイルが生成されてウザいなら、

    % make clean

でそれらを削除できます。

もちろん、このプログラムのご使用は（嫌いな言葉ですが）自己責任で。

## 使用方法

終了するには、quit 関数を実行してください。
すなわち、

    % (quit)

と入力してください（% はプロンプト）。

## スーパー括弧

「スーパー括弧」が使えます。これは、丸括弧の代わりに角括弧を使うと何重もの括弧のネストをまとめて閉じることができるものです。
左角括弧は右角括弧のこの機能をその深さで止めるために用いられます。
すなわち、((( ] や ([( ]) は ((( ))) と等しいです。

## イニシャル・ローディング機能

起動すると最初に init.txt を同じディレクトリ内から読み込みます。
ここに、あらかじめ読み込ませておきたい pure LISP プログラムを記述してください。
また、見当たらないときは空の init.txt を作っておいてください。

リポジトリの init.txt にはいろいろと基本的な関数や、いくつかのテストコードが書き込まれています。よって、起動したときに何か文字や数字の列がずらずらと表示されるでしょう。これらが邪魔ならば、すべて削除するか上書き保存したものを使ってください。

## 注意事項

シンボルを評価すると、その値に自動的に置換されます。
よって、普通の LISP とは多少挙動の違う部分が表面的にはあります。
新しく用意されたシンボルには自分自身が値として含まれています。
それゆえ、

    % a
    a

となって、違いは表面的には現れませんが（% はプロンプト）、

    % (setq a b)
    a

のようにして値 b をシンボル a に設定すると、

    % a
    b

となります。


### 注意事項その２

ダイナミック・スコープです。
けれども、昔の Common Lisp における「#'」が使えます。
これは、ラムダ項の前に付けて関数クロージャを生成することによって、一時的にレキシカル・スコープを導入します。

### 注意事項その３

ファイル名を指定しての読込・保存の機能は実装しておりません。

## 実装関数

LISP の関数には引数を評価するものとしないものとがあります。関数によっては評価される関数とそうでない関数が混在するものもあります。

そこで、以下の関数の構文表示においては、評価される引数に「'」を付けることとします。
実際、例えば リスト \(a b c\) の先頭を取り出したいときは *car* を用いますが、*car* は引数が評価される関数なので次のようにして用いることとなります。

    % (car '(a b c))
    a

したがって、評価される引数に「'」をつけて例示する表記法には正当性があります。
しかし、この「'」はその関数の構文上、必須というわけではないので、その点は注意してください。

### 基本７関数

- quote
- car
- cdr
- cons
- cond
- atom
- eq

LISP を齧ったことのある人なら、これらの意味はご存知でしょう。
もちろん、(quote ...) の略記法として '... が使えます。

### eval

    (eval '<form> '<env>)

の形で用います。

\<form\> は S-式、すなわちシンボルなどのアトムから構成されるリストです。
当処理系では数アトムがありませんので、シンボルを並べてリストにしたものを入れ子にしたツリー構造のリスト、もしくは 単独のシンボル、ということになります。

\<env\> は環境リストです。
x1, x2, ... をシンボル、A1, A2, ... を任意の S-式として、

    ((x1 . A1) (x2 . A2) ...)

のような構造を持っている特定のリストです。
すなわち、car 部がシンボルを指しているドット対の成すリストです。
これは、シンボル x1, x2, ... がそれぞれ式 A1, A2, ... に置き換えられることを意味しています。
同じシンボルが含まれているときは、先頭に近いものが優先されます。

*eval* は \<env\> の示す置き換え規則の下、式 \<form\> を評価します。

この eval 関数は、いわばユーティリティ版であり、当処理系でインタプリタを動かしている本物の eval 関数のラッパー関数です。
環境リストは、本来、インタプリタが行う関数呼び出しに応じて空リストの状態から自ずと生成されます。
したがって、ここでの環境リスト \<env\> は、インタプリタ内部で生成されている環境リストを置き換えるものではありません。

### apply

    (apply '<func> '<args>)

リスト \<args\> を引数の列とみなして関数 \<func\> を適用します。
例えば、

    (apply 'cons '(a (b c)))

は

    (cons 'a '(b c))

と同じ結果をもたらします。

### lambda

これも LISPer なら知っていて当然でしょう（これは説明を省きたいのだからこう言っているであって、実は私も初心者です。それなのに処理系を作るという……）。
当処理系において、これはリストが lambda-式であることを示す単なる印としてのシンボルです。

### de

    (de <func> (<x 1> ... <x n>) <form 1> ... <form n>)

の形で用い、シンボル \<func\> を関数として定義します。
\<x 1\>, ..., \<x n\> は仮引数としてのシンボルで、\<form 1\>, ..., \<form n\> は S-式です。
\<form 1\>, ..., \<form n\> は順次評価され、最後の結果が関数の値として返されます。
内部的には、lamda-式としてのリストへと変換して、そのリストへのポインタを値としてシンボル \<func\> 内に格納します。

de によって関数として定義されたシンボルは、oblist というリストに登録され、ガベージ・コレクションによって消去されなくなります。

ちなみに、当処理系においては、ユーザが定義した関数としてのシンボルと変数としてのシンボルとに実質的な違いはありません。
lambda-式を代入した変数が関数になります。
しかしながら、関数としてのシンボルをさらに他の関数の引数とするためには、後述する *funcall* を用いる必要があります。

## df

*df* は、引数を評価しない関数を定義すること以外、*de* と同じです。

### setq

    (setq <var 1> '<val 1> <var 2> '<val 2> ...)

の形で用います。
\<var 1\>, \<var 2\> ... は変数としてのシンボルです。
\<val 1\>, \<val 2\> ... は値としての任意の S-式です。
*setq* は、引数の並びにおいて直前の変数に直後の値を代入します。
値はそれぞれの代入の直前に評価されます。
したがって、代入の順序によっては結果に違いが生じます。

環境リストにそのシンボルがあるときは、それに対応する環境リスト内の値を書き換えます。

そうでないときは、シンボル内部に値を格納します。当処理系においては、このようなシンボルは oblist というリストに登録され、ガベージ・コレクションによって消去されなくなります。
（ただし、リストから削除する関数は作ってありません。
不便なのは何となくわかっているのですが、そうする必要に迫られるほど使っていないので。）

ちなみに当処理系においては、lambda 式をシンボルに代入すると（ただし、setq や psetq を使って代入するには式の前にクォート「'」を付ける必要があります）、そのシンボルが関数そのものになります。

### psetq

    (psetq <var 1> '<val 1> <var 2> '<val 2> ...)

の形で用います。
\<var 1\>, \<var 2\> ... は変数としてのシンボルです。
\<val 1\>, \<val 2\> ... は値としての任意の S-式です。
*psetq* は、引数の並びにおいて直前の変数に直後の値を代入します。
値はすべての代入に先立って一括して評価されます。
したがって、代入の順序による結果の違いは生じません。

あとの説明は setq と同じです。

### gc

    (gc)

ガベージ・コレクションを起動します。
ダミーとして *nil* を返します。

### while

    (while <condition> <form 1> ... <form n>)

の形で用います。
これは、前置判定ループに相当するものです。
\<condition\> は何らかの S-式です。
\<condition\> を評価した結果が空リストにならない間、*while* は式 \<form 1\>, \<form 2\>, ... \<form n\> を順次実行することを繰り返します。
最後の \<form n\> が実行された結果が、全体の返す結果となります。

### until

    (until <condition> <form 1> ... <form n>)

の形で用います。
これは、**否定型**の後置判定ループに相当するものです。
\<condition\> は何らかの S-式です。
\<condition\> を評価した結果が空リスト**以外になる**まで、*until* は式 \<form 1\>, \<form 2\>, ... \<form n\> を順次実行することを繰り返します。
最後の \<form n\> が実行された結果が、全体の返す結果となります。

この *until* は、私が勝手に導入したもので、昔の LISP 処理系には見られないものかもしれません。

### rplaca

    (rplaca '<list> '<form>)

の形で使います。
\<list\> はアトムであってはいけません。
\<form\> は任意の S-式です。
*rplacd* は、\<list\> の car 部に \<form\> を代入します。

    % (rplaca '(a b) '(c d))
    ((c d) b)

この rplacd は、rplacd を実装する必要性が出てきたので、ならば rplaca もないとおかしい、ということで実装したものです。
使い方によっては循環リストが作れてしまうので、注意してください。

### rplacd

    (rplacd '<list> '<form> )

の形で使います。
\<list\> はアトムであってはいけません。
\<form\> は任意の S-式です。
*rplacd* は、\<list\> の cdr 部に \<form\> を代入します。

    % (rplacd '(a b) '(c d))
    (b c d)

この *rplacd* は、ループ処理のできる趙循環評価器を作るときのために、いわば仕方なく実装したものです。
*while* のためには是非とも *setq* が必要で、*setq* のためには環境リストのドット対の cdr 部を書き換えると非常に簡単だったからです。
使い方によっては循環リストが作れてしまうので、注意してください。

### function

    (function <lambda-form>)

quote と同様に省略記法が使え、

    #'<lambda-form>

も同じ意味を持ちます。

\<lambda-form\> は lambda-項です。
lambda-項とは、lambda-式から一番外側の括弧を取り外し、さらに実引数を取り除いたものです。
*function* は、lambda-項を引数に取って funarg-式を生成します。

funarg-式とは、

    (funarg <lambda-term> <env>)

という形式であり、ここで \<lambda-term\> は lambda-項で、\<env\> は環境リストです。

当初理系においては、funarg-式はそのまま関数として使用できます。
実引数を環境 \<env\> の下で評価し、それらに \<lambda-term\> が適用されます。ただし、\<lambda-term\> が後述の *nlambda* を先頭に持つものであれば、実引数の評価は行われません。このように、funarg-式は関数クロージャとして機能します。

### funcall

    (funcall <func> '<arg1> '<arg2> ...)

<arg1> <arg2> ...　を引数としてそれらに関数 <func> を適用します。
引数はあらかじめ評価されます。

当初理系においては、lambda-式 を用いてもリストの先頭の変数に関数を代入することができないので、高階関数を定義する際には、*fucall* を用いる必要があります。
この辺りのところは昔の Common Lisp と共通です。


### dm

    (de <macro name> (<x 1> ... <x n>) <form 1> ... <form n>)

の形で、マクロを定義するために用います。
当処理系でのマクロは、*lambda* が *macro* に置き換わったようなものです。
実際、これら二つはシステムにあらかじめ用意された単なるシンボルであり、これらによってマーキングされた式はほぼ同じ処理を施されます。
唯一の違いは、マクロ式のほうは二度、評価されるということです。

*dm* は *de* が引数から lambda-式を作るように、macro-式を作って指定されたシンボル内部に格納します。
*dm* で定義したシンボルが評価されるときは、内部の macro-式がシンボルの代わりに評価されることになります。

### macro

これは関数ではなくシンボルです。関数に対する *lambda* のマクロ版です。

### backquote

*backquote* は関数です。

    (backquote <form>)

の形で用いられ、S-式 \<form\> は、以下に述べる一部の例外を除いて、評価されません。

    `<form>

と略記することができます。

### comma

*comma* は本質的には関数ではありませんが、backquote-式の外で使われたときにエラーにするために、関数の役割も持たせてあります。backquote-式の内部ではこれはシンボルとして扱われます。

すなわち、backquote-式の中の

    (comma <form>)

において、S-式 \<form\> は評価されます。

    ,<form>

と略記できます。

### atmark

    (atmark <form>)

あるいは

    ,@<form>

のように用います。
*atmark* は、次の一点を除いて *comma* と同じです。
式が展開、すなわち、S-式 \<form\> はただ評価されるだけではなくて、一番外側の括弧が取れた形になります。

また、*backquote* の直後の *atmark* はエラー扱いになります。

### quit

    (quit)

インタプリタを修了してコンソールに戻ります。

### num

    (num <number>)

の形で用います。
これには省略記法があって、

    #<number>

も同じ意味です。
「#」の後にはスペースを入れません。
\<number\> は小数点なしの数字の並びであり、*num* は与えられた数字をチャーチ数もどき（チャーチ数ではない）の集合論的な数に置き換えます。

すなわち、数アトムを持たない純 LISP において、自然数をリストの長さで代用する方法です。
当処理系では真を表すアトム「t」のみからなるリストをこの意味での「数」として生成します。

例えば

    % #6
    (t t t t t t)

となって、数字の 6 が「t」から成る長さ 6 のリストに変換されたことがわかります。

このようなタイプの「数」は実用的ではありませんが、純 LISP で自然数を原理的に実現する最も簡単な方法です。

num は C 言語の関数 atoi() を使っています。それゆえ、int の最大値を超える数を入力すると、途中までの数字しか認識されないはずです。また、「数」の構造上、マイナスを入力しても意味を持ちません。

### len

    (len '<list>)

の形で用います。\<list\> は任意のリストです。
len は、与えられたリストの長さを表す数字の並びを名前とするシンボルを生成し、それを返します。

この機能は、「チャーチ数もどき」の算用数字表示を得るために使えます。

    #'...

は

    (len ...)

として扱われます。

###　 cls

    (cls)

画面を消去します。

## 予約シンボル

- nil
  + 空リストを意味します。正しくはシンボルではありません。内部構造もシンボルとは違っていて、名前を格納するセルも、値へのボインタを格納するセルも持ちません。。
- t
    + 真偽値の真を意味します。
- lambda
    + 関数ではありません。先頭にあるとそのリスト全体が無名関数として扱われる、ただの印です。
- nlambda
    + これは、引数が前もって評価されない lambda 式を表す記号です。
- oblist
    + oblist を評価すると、すでに定義した関数や、値を設定したシンボルがリストになって表示されます。
- funarg
    + funarg 式を示す印です。
- macro
    + マクロであることを示す印です。
- comma
    + マクロの中で使うことのできるコンマ記法「,...」のためのものです。
- atmark
    + マクロの中で使うことのできる「,@...」という記法のためのものです。

## シンボルの内部構造

シンボル自体もセルから構成されています。すなわち、シンボルも当処理系においては一種のリストなのですが、ユーザからはリストとして扱えないようになっています。

> ((str_1 str_2 ... str_n) (*value* . *next*))

が、仮にリストとして表したときのシンボルです。

str_1, ..., str_n はそれぞれ4文字までの文字列で、連結されてシンボル名として表示されます。
ただし、文字列バッファとして配列を用いて処理しているので、文字数に限界があります（とりあえずは 100 文字。もちろん変更可能）。
str_n の使用されない部分にはヌル文字が詰め込まれます。

*value* は関数ポインタ（本物のポインタ）へのポインタ（セル番号）や、リストおよびアトムへのポインタ（セル番号）を格納するためのセルです。

*next* は、デフォルトでは 0 すなわち *nil* になっています。
けれども、インタプリタの内部処理においてポインタを連結させて扱う際に、次のシンボルへのポインタとして使われます。

## 付記

私の関心はこの処理系を簡素化した UrLISP にすでに移っています。

この pure LISP は喩えると、建て増しに建て増しを重ねたツリーハウスのようなちょっと奇妙なものです。
ツリーハウスは小さいからこそいいのであり、住む目的で建て増しを重ねるなら、最初から地面に立てられた家であるべきです。

その最も奇妙な点を具体的に挙げると、funcall を用いないと高階関数に関数を渡せないのに、名前空間は関数も変数も一緒だということです。
私はよく知りもしない古い LISP 処理系の仕様を表面的に取り入れたのです。
古い仕様のほうが原始的なはずだから、素人が自作するのに好都合だろうと考えたわけです。

しかし、どういう理由があってそういう仕様になっているかを理解せずに表面的に真似たため、上記のようにチグハグなことになった。
そして、今更、直そうにも、いろいろ仕組みが入り組んでしまって、直すのが困難です。

そういうわけで、この pure LISP を私はもうあんまり保守したくない。
でも、バグを見つけた人はよかったらお知らせください。
UrLISP と共通する部分に起因するものであれば、UrLISP に修正を反映させます。
