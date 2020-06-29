# Python デコレータ - 「習得すべき武器」

* (原文: [Python Decorators- A Weapon To Be Mastered](https://medium.com/analytics-vidhya/python-decorators-a-weapon-to-be-mastered-be310b519ac5))

---

こんにちは Pythonist の皆さん！
この記事では Python のデコレータについていろいろ学ぶ予定です。
いきなりデコレータの動きを説明するのではなく、まずは皆さんには Python の標準的な関数が持つ「能力」と、素晴らしい機能の一つであるデコレータをどのようにして作るのかを学び、そして感じとって欲しいと思っています。

Python の関数には三つの能力があることから、「ファーストクラス」のように至れり尽くせりなオブジェクトであることが知られています。

では、これらの能力とは何でしょうか？
それを使って何ができるのでしょうか？

まずは、そこから見てみることにしましょう 〜

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

ここで、変数の ``fun`` は ``parent()`` 関数のリファレンスを保持しています。このリファレンスが紐付いている先の ``parent()`` 関数は未だ呼び出されていません。

そして、この ``fun`` を関数のように呼び出すことができます。
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

ここで、``neighbor()`` 関数は引数として ``parent()`` 関数に渡され、その中で呼び出されます。


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

ここで、``parent()`` 関数の内側ではネストした二つの関数（ ``son()`` と ``daughter()``）を定義しています。
この二つの関数は直接（関数の外側から）呼びされることはなく、また外部の関数が呼び出されても自動的に実行されることはありません。
しかし、これらネストした二つの関数のリファレンスを関数の外側へ返し、そこでそれらの関数を呼び出すことは可能です。


*これら三つの能力は、Python 関数において、もっとも便利で強力な「武器」となるデコレータの構成要素となります。*


> そうです。デコレータは Python 開発者にとって多くの問題を解決するための武器なのです!!!

これら三つの要素を念頭においておけば、これから解説するデコレータの理解が深まることでしょう。

---

## デコレータとは何か？

