## 執筆途中．以下の内容には，不正確な情報や，謝った情報が含まれている可能性があります．

# Haskell を学ぶ意義

- ### 純粋な関数型言語の特徴を知ることができる
	- 副作用が(殆ど)ない
	- 何をするか(命令型)ではなく，何であるかを伝える
- ### (比較的)新しい言語を学ぶことで，技術的なトレンドを知ることができる
	- 型推論
	- 型クラス
	- 列挙表現(range)

- ### 学術(アカデミー)に近い言語なので，プログラミングの基礎を学べる
	- 型理論
	- ラムダ計算
	- プログラム意味論
	- 圏論
	- 形式手法(は，若干こじつけかも(^o^;) )

- ### その他
	- -(マイナス)の扱いめんどくさい．
	- 強い静的型付け言語の特徴を知ることができる
	- 遅延評価型の言語の特徴も知ることができる
	- 再帰的な考え方がみにつく

### 上記の特徴は，Haskell だけではなく，他の言語や，ソフトウェア科学(開発)一般において応用することが可能な，汎用的な知識と成る．

# 基本的な規則
- 型名：先頭大文字(例 Integer, String 等や，Eq，Ord　等の型クラスも)
- 変数名：上記規則があるため，先頭は小文字から．(あ，でも，型名が先頭大文字なのは，
下記の形変数が小文字(一文字)だからかもしれないので，変数名が小文字なことに関して因果関係はないかも)
- 型変数：先頭小文字(基本，「a」などの様に，小文字一文字)
- 関数名：関数名も先頭は小文字から
	-  アポストロフィ「’」を関数につけることで,少し変更したverの関数であることや，※正格評価版の関数をあらわす．
- 括弧のあり方について：基本的に，Haskell の関数適用には括弧を必要としないし，単純な適用ならば，括弧をつけないのが
作法だと思われる(そっちのが，λ計算の適用」(application)っぽいし．でも，やっぱり括弧があったほうがわかりやすくなったり
する場合もあるし，それ以外にも「$」を使った関数適用や「.」による関数合成などで分かり易い構文(表現)にすることができるので，
臨機応変にそれらを用いるべき．

- Haskell における変数

# 構文基本編 
- 結合規則
	- 関数適用(左結合)
		関数適用は，通常は左結合である．
	- 右結合の例
	- 演算の優先度
- 中置記法
	`hoge` とすることで，関数hogeを中置記法に適応させることができる
	例
```
Prelude> elem 2 [1,2,3]
True
Prelude> 2 `elem` [1,2,3]
True
``` 
- エスケープシーケンス
	基本的にC言語に近い．うまくいかなかったらggろう(適当)

## GHCi でのコマンド
- ### :t(:type) 型を調べる
	- :set t について
- ### :i(:info) 詳細を調べる
	:i と入力することで，関数や，演算子の基本的な情報を知ることができる．

	例1
	
	```
	Prelude> :i odd
	odd :: Integral a => a -> Bool 	-- Defined in ‘GHC.Real’
	```
	
	例2<br>
	※中置記法の演算子を調べるためには，括弧を補う必要がある...
	はずだが，つけなくても，自動的に補ってくれるっぽ．
	
	```
	Prelude> :info (+)
	class Num a where
	  (+) :: a -> a -> a
	  ...
	  	-- Defined in ‘GHC.Num’
	infixl 6 +
	```
	
	例2においては，演算子(中置換数)の型と定義場所だけではなく，
	「infixl 6 +」という，構文に関する情報も表示されている．
	このように，通常は左結合である関数適用についても，
	優先順位が設定されている場合があり，その場合は，結合則
	(括弧の対応)も変わってくる．また，infixlのlは左結合を表している．
	つまり，右結合の演算子もあり，例えば，
	
	例3
	```
	Prelude> :i ^
	(^) :: (Num a, Integral b) => a -> b -> a 	-- Defined in ‘GHC.Real’
	infixr 8 ^
	```
	上記の結合規則に関する問題を演習問題であげるので，挑戦してみて．
	
	若干 :i コマンドから話がそれてしまいましたが，:i コマンドには他にも，
	型クラスからインスタンスを調査するとか，その逆とかで非常に役に立ちます．
	型クラスのところでまた触れます．

- ### :it について
	it とは，ghciで用いられる特殊変数でなかなか便利です．itを用いると，
	直前で評価に「成功」した式の結果を参照できます．
	
	```
	Prelude> "My name is it."
	"My name is it."
	Prelude> it
	"My name is it."
	Prelude> it ++ "What's your name?"
	"My name is it.What's your name?"
	```
	さらに，評価に成功した式を参照するため，一種のセーフティーネットにもなる．
	
	```
	Prelude> it ++ My name is Shingo
	
	<interactive>:4:7: Not in scope: data constructor ‘My’
	
	<interactive>:4:10: Not in scope: ‘name’
	
	<interactive>:4:15:
	    Not in scope: ‘is’
	    Perhaps you meant one of these:
	      ‘id’ (imported from Prelude), ‘it’ (line 3)
	
	<interactive>:4:18: Not in scope: data constructor ‘Shingo’
	Prelude> it
	"My name is it.What is your name?"	 
	```
	このように，itは，直前の式で「かつ，評価に成功した」ものを保存している．

## List について
List は，型を引数として取る型(多相型)である．これは，タプルも同じ．
List は，コレクション(特定の要素のあつまり←かなり怪しい表現😥)であり，
その要素は全て「同じ型」である必要がある．

```
Prelude> [1,2,3]
[1,2,3]
Prelude> ['1','2','3']
"123"
👆 Charからなるリスト[Char］はString と同じなのである！！(Cっぽい？)
Prelude> ['1',2,'3']
<interactive>:4:6:
    No instance for (Num Char) arising from the literal ‘2’
    In the expression: 2
    In the expression: ['1', 2, '3']
    In an equation for ‘it’: it = ['1', 2, '3']
```
[Char] と String が同じ型であるこについて補足

```
Prelude> ['T','E','S','T'] == "TEST"
True
Prelude> ['T','E','S','T'] == "TeST"
False
```
- ### 無限リスト：
下記でも述べる，列挙表現を用いることで，簡単に「無限リスト」を作ることができる
```
Prelude> [1..]
結果は怖いから省略
```

- ### 内包表記：
こんな書き方もできるんやで
```
気持ち的には{ x | x　∈ N∪{0} , x < 8}
Prelude> [x | x <- [0..10], x < 8  ]
[0,1,2,3,4,5,6,7]
```
x<- の逆は怒られた
```
Prelude> [x |  [1..10] -> x, x < 8  ]
<interactive>:6:15: parse error on input ‘->’
```
述語(真偽を返す関数のようなもの)を用いるかのようにあつめることもできる
```
気持ち的には { x | 0≦x≦10 , 偶数(x)}
Prelude> [ x | x <- [0..10] , even x]
[0,2,4,6,8,10]
```
- ### 汎用的なList操作
List"操作"という表現は不適切だと思っている(個人的には．だって，操作って，オブジェクトを意識したような表現
に思えるんだもの…)．なので，一般的なListを引数にとる"関数"だと思って貰ったほうがいいかも
(つまり，[a] -> ... というようなものね)
```
Prelude> let list = [3,4,5,1,2]
Prelude> list
[3,4,5,1,2]
Prelude> head list
3
Prelude> tail list
[4,5,1,2]
Prelude> last list
2
Prelude> init list
[3,4,5,1]
Prelude> length list
5
Prelude> null list
False
Prelude> null []
True
Prelude> take 3 list
[3,4,5]
Prelude> take 7 [9..]
[9,10,11,12,13,14,15]
Prelude> take 100 [1,2]
[1,2]
```

- ### List の比較
List の要素が同じ型(👈厳しい表現😅)ならば，そのList同士は比較することができる．
辞書式順序で比較されるっぽい
```
LT:<, GT:<, EQ:=
Prelude> compare [1,2,3] [2,3,1]
LT
Prelude> [1,3,2] `compare` [1,2,3]
GT
👆 `hoge` は 関数 hoge の中置記法
Prelude> [1,3,2] `compare` [1,3,2]
EQ
Prelude> "abc" `compare` "abcd"
LT
👆文字列は，[Char]だった
```

- ### ファンキーなList
```
Prelude> :t [head,last]
[head,last] :: [[a] -> a]
Prelude> :t [(+),(-),(*)]
[(+),(-),(*)] :: Num a => [a -> a -> a]
```

## タプルにつてい
簡単に言えば，数学のn-組(タプル)と同じ．
集合は，{a,b} = {b,a} だけど
タプル(n-組) は，一般的に(a,b) ≠ (b,a) である．
つまり，要素の出現順(👈すごく怪しい表現)まで考慮されたものである．
例えば，Listにおいても，ListのListなるもの
(今回は，簡単のため [Int]からなるリストを考える)を作れるが
```
Prelude> [[1,2],[2],[1],[1,2,3]]
[[1,2],[2],[1],[1,2,3]]
```
このように，サイズが違っていても型的にはOKなので許されてしまう😔
そこで，タプルの出番
```
Prelude> [(1,2),(2,3,4)]
<interactive>:17:8:
    Couldn't match expected type ‘(t, t1)’
                with actual type ‘(Integer, Integer, Integer)’
    Relevant bindings include
      it :: [(t, t1)] (bound at <interactive>:17:1)
    In the expression: (2, 3, 4)
    In the expression: [(1, 2), (2, 3, 4)]
    In an equation for ‘it’: it = [(1, 2), (2, 3, 4)]
```
ここで，今回は，2-組(ペアともよぶ)の"List"だと
(乱暴に表現すれば[2-Tuple]？)と型が推論されたので，
2番めに3-組(トリプルともよぶ)の要素が来たのでErrorを吐かれた
```
Prelude> [(1,2),(2,3),(3,4)]
[(1,2),(2,3),(3,4)]
```
これなら安心😃
ただ，実際には，タプルであっても，そのまま使うのは，(型的には)そんなに
筋がいいわけではないかもしれない．実際に，型の安全性や柔軟性を
考慮して設計する場合には，タプルをオーバーラップするような型を
さらに定義するほうがいいのかも．つまり，生のままタプルはあまり
使うのは筋が良くない気がする(例えば，xy座標を表すにしても，
生ペアでは，抽象化が十分ではない)


# 基本的な表現
- ### - (マイナス)表現の注意<br>
負の数を表す意味で -3 なんかをghciに入力するときは注意
(-3) などのように，過去を補ってあげないと，Error はかれて悲しい．

- ### if else<br> 
Haskell において，if else 式は，命令形言語のif else と同じようで微妙に違う．
まず，(多くの)命令形言語においては，else を省略して，if (then) だけの式を
書いても問題ない．しかし，Haskell は else まで「必ず」セットで用いる必要がある．
命令形言語でif else は「文」であり，あくまで手続きを与えるためのものであるが，
Haskell においては「式」でなくてはならない．そのため．述語(分岐条件)が真でも
偽でも「同じ型」に評価されなくてはならない．

- ### 列挙表現(ragnge)<br>
こんな感じの
	```
	Prelude> [1..10]
	[1,2,3,4,5,6,7,8,9,10]
	Prelude> ['a'..'z']
	"abcdefghijklmnopqrstuvwxyz"
	```

	ナウめ？の言語では実装されているやつ．Listとかループ条件によくあるよね
	Haskell の range は，結構賢くていろいろしてくれるけど，落とし穴があったりする
	(特に，浮動小数点が絡んでくる場合)
	ただ，過信はできない．
	```
	Prelude> [10..1]
	[] 
	👆 [10,9,8,7,6,5,4,3,2,1] となってほしい
	Prelude> ['z'..'a']
	"" 
	👆 "zyxwvutsrqponmlkjihgfedcba" となってほしい 
	```
	そこで，下降方向には，減少値がわかるように教えてあげればいい
	```
	Prelude> [10,9..1]
	[10,9,8,7,6,5,4,3,2,1]
	Prelude> ['z','y'..'a']
	"zyxwvutsrqponmlkjihgfedcba"
	```

- ### 論理演算<br>
 論理演算は，基本的にはC言語の慣習を引き継いでいる．論理積は「&&」，論理和は「||」．真偽はそれぞれ，True，False である(先頭大文字)
等価は「==」であり，不等号は「>」「>=」「<」「<=」 などであるが，否定は「/=」であるので注意．(≠っぽいからみたい)

	```
	Prelude> True || False
	True
	Prelude> False || True
	True
	Prelude> True && True
	True
	Prelude> True && False
	False
	Prelude> 3 == 3
	True
	Prelude> 3 == 10
	False
	Prelude> 3 /= 3
	False
	Prelude> 3 /= 8
	True
	Prelude> 3 >= 2
	True
	Prelude> 2 >= 3
	False
	```
	また，型が異なるものの比較は，怒られちゃう

	```
	Prelude> 3 == True
	
	<interactive>:34:1:
	    No instance for (Num Bool) arising from the literal ‘3’
	    In the first argument of ‘(==)’, namely ‘3’
	    In the expression: 3 == True
	    In an equation for ‘it’: it = 3 == True
	Prelude> 3 /= True
	
	<interactive>:35:1:
	    No instance for (Num Bool) arising from the literal ‘3’
	    In the first argument of ‘(/=)’, namely ‘3’
	    In the expression: 3 /= True
	    In an equation for ‘it’: it = 3 /= True
	```

# 型初級編
## 型の話
Haskell は強い型システムをもつ．そして，強い型Systemが保証されるということはある種の
エラーがないということを実行前(コンパイル時)に検出してくれるということである．
また，型システムは，平坦なデータに意味を与えてくれる．このことによって，データ(構造)
を一つ抽象化することができる．抽象を導入することによって，低水準の詳細を意識
しなくてもよい(忘れても良い)といったメリットを享受できる．
たとえば，パラメタトリック多相によってもたらされる，汎用的なソーティングなどが
わかりやすい例かもしれない．

Haskell とは違い，動的型付けであるPythonは，型システムの作法が厳密でなく
簡単にかけるというメリットがある反面，プログラムの検証をするためには，
大量のtest code が必要になってくる(らしい)．
型errorの例

また，Haskellの最大の特徴は，モナド？などを用いることで，
型をさらに抽象的な型で包み込むことで，データのセキュア性を担保したり，
柔軟に扱えることかもしれないが，ここは闇なので言わなかったことにする．

- いろいろなものの型を調べてみよう
型は，:t コマンドで調べることができる
```
Prelude> :t 10
10 :: Num a => a
Prelude> :t 10.3
10.3 :: Fractional a => a
Prelude> :t 10.3333
10.3333 :: Fractional a => a
Prelude> :t "a"
"a" :: [Char]
Prelude> :t ['a']
['a'] :: [Char]
Prelude> :t (3)
(3) :: Num a => a
Prelude> :t (+)
(+) :: Num a => a -> a -> a
```

- List の型，タプルの型
[1,2,3]もListだし，['a','b','c'] も "eastern youth" も
[[1],[1,2],[]] だって [(+),(-)] だってListである．
それぞれ異なる型といえばそれはその通り．
でも，個別に実装があるというよりは，ある種List
を統一的に扱う手法だったり理論があったりする(はず)
タプルも同じで
()も(3,"a")も("a",3)も，((1,2),(+))だってタプルである．
これらもそれぞれ別の型であると考えられる一方で
やはり，統一的に扱う理論があるはずである．
その一つが，型変数だと思う.


- 型変数
Haskell には型を表す変数があり，基本的には
小文字のアルファベット1文字で表現される(が，先頭が小文字であるという以外の
制約はないかも)
例えば，先ほど述べたように，ListやTupleは，ある意味ではいろんな型の振る舞いを
取り込んでいる．例えば，[int] にしろ，Num a => [a -> a -> a]にしろ，
head や last といった関数を適応することが可能である．
これは，head や Last といった関数，あるいはList自身が型変数によって
多相的な振る舞いが規定されているからである
```
Prelude> :t head
head :: [a] -> a
```

- 型シグネチャ
あ，そういえば．:t したときの 名前::[Char] とか 名前::[a] -> a とかの::右側の型の
表示をシグネチャと呼ぶみたい．うんで，シグネチャに型変数がある場合，それは
多相的に(ある意味ではどんな型にでも)なれる．これはを「パラメトリック多相」と
よぶみたい．また，パラメトリック多相な関数を多相関数と呼ぶみたい．

	- ### 多相関数 
		- #### Javaとの比較
		Java での多相的なクイックソート(オリジナルなので，糞コードの可能性も否めないが)
		``` java
		import java.util.ArrayList;
		import java.util.Arrays;
		
		public class QuickSort {
		    static public <T extends Comparable<? super T>> ArrayList<T>
		            quicksort(ArrayList<T> list) {
		        if (list.isEmpty()) {
		            return list;
		        }
		        if (list.size() == 1) {
		            return list;
		        }
		        T pivot = list.remove(0);
		        ArrayList<T> smallerList = new ArrayList<>();
		        ArrayList<T> biggerList = new ArrayList<>();
		        for (T x : list) {
		            if (x.compareTo(pivot) < 0) {
		                smallerList.add(x);
		            } else {
		                biggerList.add(x);
		            }
		        }
		        ArrayList<T> ansList = quicksort(smallerList);
		        ansList.add(pivot);
		        ansList.addAll(quicksort(biggerList));
		        return ansList;
		    }
		```

		Haskell での多相的なクイックソート(コチラはすごいH本から引用．だから，多分良いコード)
		``` haskell
		quicksort :: (Ord a) => [a] -> [a]  
		quicksort [] = []  
		quicksort (x:xs) =   
		    let smallerSorted = quicksort [a | a <- xs, a <= x]  
		        biggerSorted = quicksort [a | a <- xs, a > x]  
		    in  smallerSorted ++ [x] ++ biggerSorted  
		```

		Java での多相クイックソートのコードが(私が書いたため)残念である可能性はあるにせよ，
		Haskell の方が楽に書けることが分かる．

- 型クラス
	型クラスのクラスは，JavaやC++などのclassとは別のもの(でも，Javaのインタフェースには近い気がするけど…)
	型クラスは，インスタンスとして型を持つ．型クラス自身は具体的な型ではないので注意．
	例 Enum型クラス(順序があり，要素を列挙できる型を規定する)
```
Prelude> :i Enum
class Enum a where
  succ :: a -> a
  pred :: a -> a
  toEnum :: Int -> a
  fromEnum :: a -> Int
  enumFrom :: a -> [a]
  enumFromThen :: a -> a -> [a]
  enumFromTo :: a -> a -> [a]
  enumFromThenTo :: a -> a -> a -> [a]
  	-- Defined in ‘GHC.Enum’
instance Enum Word -- Defined in ‘GHC.Enum’
instance Enum Ordering -- Defined in ‘GHC.Enum’
instance Enum Integer -- Defined in ‘GHC.Enum’
instance Enum Int -- Defined in ‘GHC.Enum’
instance Enum Char -- Defined in ‘GHC.Enum’
instance Enum Bool -- Defined in ‘GHC.Enum’
instance Enum () -- Defined in ‘GHC.Enum’
instance Enum Float -- Defined in ‘GHC.Float’
instance Enum Double -- Defined in ‘GHC.Float’
👆 お，bool関数のSuccsorがあるのか(面白い)
Prelude> succ False
True
Prelude> succ True
*** Exception: Prelude.Enum.Bool.succ: bad argument
👆（#^ω^）
```
1つの型は，いくつもの型クラスのインスタンスに成ることができるし，
1つの型クラスは，いくつもの型をインスタンスとしてもてる(は👆のとおり)
前者についてのエビデンス
```
Prelude> :i Eq
class Eq a where
  (==) :: a -> a -> Bool
  (/=) :: a -> a -> Bool
  	-- Defined in ‘GHC.Classes’
instance Eq a => Eq [a] -- Defined in ‘GHC.Classes’
instance Eq Word -- Defined in ‘GHC.Classes’
instance Eq Ordering -- Defined in ‘GHC.Classes’
instance Eq Int -- Defined in ‘GHC.Classes’
instance Eq Float -- Defined in ‘GHC.Classes’
instance Eq Double -- Defined in ‘GHC.Classes’
instance Eq Char -- Defined in ‘GHC.Classes’
instance Eq Bool -- Defined in ‘GHC.Classes’
(長いので中略)
```
# 関数の構文
Haskell で関数を定義したい．以下のように大きくわけて，「パターンマッチ」と「ガード」
による方法で定義をすることができる
ただ，実は，パターンマッチ使うよりも，foldr, flodl 使う辺りが常道だったする
問題も多いらしくて，見極める必要があるみたい・・・？

- ## パターンマッチ
主に構文的な方法で関数を定義する
そろそろ力が底をついてきたので，すごいH本のページで…
ポイント：マッチングにワイルドカードを使うべきところがある場合は，
かならず使うこと！！！！(なぜならば，不必要な値を変数に代入することは
バグの元だからです．とっとと捨てよう)

- ## ガード
主に，意味的(値)な方法で関数を定義する
もちろん，ガードの中でパターンマッチが出現することもある．
- ### Where

- ## let 

- ## case


# 再帰 
力つきた…

# 演習
- ## chapter1
	- ### πは定数piとして，prelude で定義されているが，ネイピア数 e を定義せよ．
	- ### 「:i +」「:i -」「:i \*」「:i ^」の結果を確認せよ．また，「 3 - 1 + 4 \* 2 \* 3 ^ 2 」の結果を考察し，上記結果となる理由を :i の結果から考察せよ
	- ### Prelude> ['L','i','s','t'] `compare` "Listlength" の結果を予測せよ 
- ## chapter2
	- ### list型，あるいは，タプル型に属する型が(理論上は)可算無限個存在する(つまり，自然数と一対一対応がとれる)ことを証明せよ．
	- ### 多相関数の例を挙げよ．また，それが多相的であることのメリットを述べよ
	- ### ソート関数を作るためには， Ord a => [a] -> [a] という型制約が必要である．何故か答えよ．

- ## chapter3
	- ### (多分難しい) [1,[2,3,4],[[2,[3]],4],[[[5]]]] みたいなList を [1,2,3,4,2,3,4,5] にする関数を作ろう．パターンマッチ(と再帰の力で)

- ## chapter4
	- ### ハノイの塔を解いてみて 
