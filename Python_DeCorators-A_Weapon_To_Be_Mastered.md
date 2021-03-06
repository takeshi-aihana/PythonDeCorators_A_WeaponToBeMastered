# Python デコレータ - 「習得すべき武器」

* (原文: [Python Decorators- A Weapon To Be Mastered](https://medium.com/analytics-vidhya/python-decorators-a-weapon-to-be-mastered-be310b519ac5))

---

こんにちは Pythonist の皆さん！
この記事では Python のデコレータについていろいろ学ぶ予定です。
まずは皆さんには、いきなりデコレータの動きを説明するのではなく Python の標準的な関数が持つ「能力」と、素晴らしい機能の一つであるデコレータをどのようにして作るのかを学び、そして感じとって欲しいと思っています。

Python の関数には三つの能力があることから、「ファーストクラス」のように至れり尽くせりなオブジェクトであることが知られています。

では、これらの能力とは何でしょうか？
それを使って何ができるのでしょうか？

まずは、そこから見てみることにしましょう：

---

#### 1. 関数は変数に割り当てることができる

```Python
In [1]: def parent(): 
   ...:     print('Inside parent function') 
   ...:

In [2]: fun = parent

In [3]: fun()
Inside parent function
```

ここでは変数の ``fun`` が ``parent()`` 関数のリファレンスを保持しています。このリファレンスが紐付いている先の ``parent()`` 関数は未だ呼び出されていません。

そして、この ``fun`` を関数を呼び出すかのように使うことができます。
すると実際に ``parent()`` 関数が実行されます。


#### 2. 関数は変数として他の関数に渡すことができる

```Python
In [4]: def neighbor(): 
   ...:     print('Hey, I am neighbor') 
   ...:

In [5]: def parent(func): 
   ...:     print('Hi, there!') 
   ...:     func() 
   ...:

In [6]: fun = parent

In [7]: fun(neighbor)
Hi, there!
Hey, I am neighbor

```

ここでは ``neighbor()`` 関数が引数として ``parent()`` 関数に渡され、その中で呼び出されます。


#### 3. 関数は変数として他の関数から返すことができる

```Python
In [20]: def neighbor(): 
    ...:     print('Hey, I am neighbor, where is your son?') 
    ...:     return 1 
    ...:

In [21]: def parent(func): 
    ...:     print('Hi there!') 
    ...:     call = func() 
    ...: 
    ...:     # nested function 
    ...:     def son(): 
    ...:         print('Hi neighbor, I am his son') 
    ...: 
    ...:     # nested function 
    ...:     def daughter(): 
    ...:         print('Hi neighbor, I am his daughter') 
    ...: 
    ...:     if call == 1: 
    ...:         return son 
    ...:     else: 
    ...:         return daughter 
    ...:

In [22]: fun = parent

In [23]: child = fun(neighbor)
Hi there!
Hey, I am neighbor, where is your son?

```

ここでは ``parent()`` 関数の内側で ``son()`` と ``daughter()`` の二つの関数を定義しています。
この二つの関数は直接（関数の外側から）呼びされることはなく、また外部の関数が呼び出されても自動的に実行されることはありません。
しかし、これら二つの関数のリファレンスを ``parent()`` 関数の外側へ返し、そこでそれらの関数を呼び出すことは可能です。


*これら三つの能力は Python 関数において、もっとも便利で強力な「武器」となるデコレータの構成要素となります。*


> そうです。デコレータは Python 開発者にとって多くの問題を解決するための武器なのです!!!

これら三つの要素を念頭においておけば、これから解説するデコレータの理解が深まることでしょう。

---

## デコレータとは何か？

**[教科書の定義]**

デコレータは、呼び出すことが可能なもう一つ別のオブジェクト（例えば関数やメソッド、あるいはクラス）を受け取って、そのオブジェクトを明示的に変更しないで処理を拡張することが可能な関数です。

簡単に言うと 〜 デコレータは呼び出しが可能なオブジェクトに巻き付いて、その挙動を変えるメカニズムです。

どうです、面白そうでしょう？

デコレータの例として、ここに小さな "*Hello World*" 風のプログラムを用意しました：

```Python
In [24]: def decorator(func): 
    ...:     def wrapper(): 
    ...:         print('Before the function get called.') 
    ...:         func() 
    ...:         print('After the function is executed.') 
    ...:     return wrapper 
    ...:

In [25]: def wrap_me(): 
    ...:     print('Hello Decorators!') 
    ...:

In [26]: wrap_me = decorator(wrap_me)

In [27]: wrap_me()

Before the function get called.
Hello Decorators!
After the function is executed.
```

## どのようにデコレータは動いているのか？

上記の例で、まさに Python 関数が持つ三つの能力が動作しているところ見てきました。

``wrap_me()`` 関数を引数にして ``decorator()`` を呼び出します。
``decorator()`` が呼び出されると：
クロージャーでもある ``wrapper()`` 関数が ``wrap_me()`` のリファレンスを保持するように定義され、それを呼び出した時の動作を変更するため引数として受け取った関数（すなわち ``wrap_me()``）に巻き付きます。

> ところでクロージャーって何？
>
> 一部のデータ（この場合は ``wrap_me()`` 関数のリファレンス）が関数など別のコードに「紐付ける」手法を、Python では「[クロージャー](https://medium.com/@amitrandive442/python-closures-b71e8847286f)」と呼んでいます。
>
> このデータのスコープは、データがそのスコープの外に出た場合でも記憶されます。


クロージャである ``wrapper()`` 関数は、デコレータが受け取った関数にアクセスすることができ、さらにその関数を呼び出す前と後で追加のコードを自由に実行することができます。


そして ``decorator()`` 関数は ``wrapper()`` 関数のリファレンスを返し、変数である ``wrap_me`` に格納され、その経数を関数のように呼び出すと実際には ``wrapper()`` 関数が呼び出される仕組みです。

この時点で、元々の ``wrap_me()`` 関数が実行される前と後で別の命令がそれぞれ実行されます。

> ちょっと難しいそうだと思うかもしれませんが、一度その動きをしっかりと理解すれば、間違いなくデコレータの魅力に取り憑かれてしまうことでしょう。

---

## @（パイ）記号

``wrap_me()`` 関数でデコレータを呼び出して、それを ``wrap_me`` 変数に割り当てる代わりに Python の ``@`` (Pie / パイ) 記号が使えます。


```Python
In [1]: def decorator(func): 
   ...:     def wrapper(): 
   ...:         print('Before the function gets called.') 
   ...:         func() 
   ...:         print('After the function is executed.') 
   ...:     return wrapper 
   ...:

In [2]: @decorator 
   ...: def wrap_me(): 
   ...:     print('Hello Decorators!') 
   ...:

In [3]: wrap_me()

Before the function gets called.
Hello Decorators!
After the function is executed.

```

``@`` 記号を使うということは、デコレータを呼び出すために一般的に使用するコードを短く表現したものを得るために「シンタックス・シュガー（糖衣構文）」を追加すると言うことと等価です。

---

## 引数を受け取るデコレータ


ここで誰もがするお決まりの質問があります ー 「もともと引数を受け取る関数にデコレータを適用するにはどうすれば良いですか？」

Python ではそれができます。
そして、それはかなり簡単です。
``*args`` とか ``**kwargs`` など関数で可変引数を扱うものがあることをご存知ですか？

これだけです！
この二つのキーワードを用意するだけで大丈夫です。

それでは例として、関数の引数情報をログに記録するデコレータを書いてみることにしましょう：

```Python
In [4]: def dump_args(func): 
   ...:     def wrapper(*args, **kwargs): 
   ...:         print(f'{args}, {kwargs}') 
   ...:         func(*args, **kwargs) 
   ...:     return wrapper 
   ...:

In [5]: @dump_args 
   ...: def wrap_me(arg1, arg2): 
   ...:     print(f'Arguments dumped') 
   ...:

In [6]: wrap_me('arg_dump1', 'arg_dump2')

('arg_dump1', 'arg_dump2'), {}
Arguments dumped

```

この例では ``wrapper()`` 関数の定義で ``*`` と ``**`` と言う二つの演算子を使い、位置引数とキーワード・オンリー引数の全てを受け取り、それぞれ ``args`` と ``kwargs`` という変数に格納し、デコレータが受け取った関数（すなわち ``wrap_me()``）に転送しています。

---

デコレータは再利用が可能です。
つまり、同じデコレータを複数の関数に対して使用できます。
さらに、他のモジュールからデコレータをインポートすることも可能です。

---

## 複数のデコレータを適用する

単一の関数に複数のデコレータを適用することも可能です。

例えば、引数の情報だけではなく、関数を実行した時のタイムスタンプもログとして記録する必要がある場合を見てみましょう：

```Python
In [7]: import datetime

In [8]: def dump_args(func): 
   ...:     def wrapper(*args, **kwargs): 
   ...:         print(f'{func.__name__} has arguments - {args}, {kwargs}') 
   ...:         func(*args, **kwargs) 
   ...:     return wrapper 

In [9]: def cal_time(func): 
   ...:     def wrapper(*args, **kwargs): 
   ...:         now = datetime.datetime.now() 
   ...:         print('start of execution :', now.strftime('%Y/%m/%d %H:%M:%S')) 
   ...:         func(*args, **kwargs) 
   ...:         now = datetime.datetime.now() 
   ...:         print('end of execution :', now.strftime('%Y/%m/%d %H:%M:%S')) 
   ...:     return wrapper 
   ...:

In [10]: @cal_time 
    ...: @dump_args 
    ...: def wrap_me(arg1, arg2): 
    ...:     print('Arguments dumped') 
    ...:
    
In [11]: wrap_me('arg_dump1', 'arg_dump2')

```

ここでは ``wrap_me()`` 関数に二つのデコレータを適用しています。
このあとの動きがどうなるか想像してみて下さい。

こちらが正解です：

```Python

start of execution : 2020/07/01 18:50:42
wrap_me has arguments - ('arg_dump1', 'arg_dump2'), {}
Arguments dumped
end of execution : 2020/07/01 18:50:42
```

積み重なった二つのデコレータが「下から上へ」適用されています。
デコレータが受け取った関数は、まず ``@dump_args()`` に巻き付かれ、次に処理された結果の（デコレータが適用された）関数が ``@cal_time()`` によって再び巻き付かれます。

---

実際の関数を呼び出す前に実行する必要がある「一般的な処理」はデコレータとして記述し、それを特定の関数呼び出しに「貼り付ける」だけです。

「一般的な処理」とは例えば：

* ログを記録する

* 認証処理

* 関数のパフォーマンスを計測する

* キャッシュ処理

など。

前に述べたようにデコレータは再利用が可能な「建物のブロック」みたいなもので、さらに強力な機能としてサードパーティ製のライブラリと同様に標準ライブラリの中でも頻繁に使用されています。

Python でオブジェクト指向なコードを書いていると、``@staticmethod`` とか ``@abstractmethod`` とか ``@classmethod`` といったキーワードに出くわす機会が多くありませんでしたか？
今のあなたなら、それらの本当の意味がわかるはずです。
この次、それらを関数に適用する時は本当のところ何をするものなのかよく調べてみて下さい。

しかしながらデコレータは全ての問題の解決策ではないので使いすぎないように注意して下さい。
とはいえ、デコレータがいつどこで使用すべきものなのかを理解しているのであれば、それは非常に役立ちます。

---

ここまで読んで、この記事で説明したことをすべてを理解したあなた、おめでとうございます!!
これで、Python で最も難しい概念の一つをマスターしました。

Cheers!!
