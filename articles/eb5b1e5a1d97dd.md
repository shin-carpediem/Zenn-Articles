---
title: "動的型付け言語で型を明記してみた"
emoji: "🍴"
type: "tech"
topics: ["動的型付け言語", "静的型付け言語"]
published: true
---

動的型付け言語 Python を書く機会がありました。
静的型付け言語に浸かっていた筆者が、慣れずに動的型付け言語を書き、バグを沢山産んでしまいました。

本記事では、動的型付け言語の初学者におすすめだと思う書き方を紹介します。

## 型を書くメリット

予期しない値が混入して、エラーとなるのを避けるためです。

動的型付け言語である Python で書かれた以下のコードを見てください。

```python
number_list = ["0", "1", "2", 3, "4"]

for number in number_list:
    int(number)
```

このコードは、`number` が 3 の時に、`str` 型を `int` 型に変換する`int()`関数の引数に str 型ではない型が渡されるので、エラーとなります。

Java, Swift, TypeScript のような静的型付け言語は、プログラムを実行する前に型の不一致による上述したエラーを検出できます。
一方 PHP, Python, JavaScript のような動的型付け言語は、プログラムを実行する時に変数の型が決まります。
なので、動的型付け言語はプログラムを実行する時まで、上述したエラーに言語機能からは気がつけません。

## 予期しない値が混入してしまう理由

すでに書かれた変数の型を読み間違って、コードが追加されてしまうからです。

一般にプログラマは、コードを書く時間よりもコードを読む時間の方が長いとされます。
**自分の書いたコードでも、半年も経てば、他の人が書いたコードのように見えます。**
(筆者も何度も痛い目を見ました 😅)

他の人が定義した変数に型が書かれていなければ、どんな値が入りうるかは、上流のコードをたどるしかありません。
変数の型を一つ間違えたら、間違った型を前提としたコードを追加してしまいます。

## 例

以下のコードを見てください。

```python
# DBに保存されているリスト(str型のみを保持)を取得する
def fetch_number_list_from_db():
    number_list = ...
    return number_list

# DBに保存されているリストに number を追加する
def append_number_to_list_in_db(number):
    ...

# DBに保存されているリストに number があるかどうか
def has_spesific_number_in_list(number):
    number_list = fetch_number_list_from_db()
    return (number in number_list)
```

ここで、関数`append_number_to_list_in_db`の引数`number`に`3`を渡したらどうなるでしょうか。
関数`has_spesific_number_in_list`は、
引数 number が`3`の時は True を返しますが、引数 number が`"3"`の時は、False を返してしまいます。

DB から取得した、`str`型の number を保持するリストに、予期しない`int`型の number が追加されてしまった事で、
リストは str 型と int 型が混在するようになってしまいました。

このように、あるコードで DB に予期しない値が混入し、別コードがそれを参照してしまうことは、よくある不具合例の 1 つです。

## 混入させないための方法 3 選

『テストを書く』は自明ですが、本記事では、テストを書きつつ + α で行うと良さげな 3 つの方法を説明します。
(もし他にも良い方法ありましたら、コメント欄でご指摘ください 🙏)

- 全ての変数、関数の引数・戻り値の型を書く
- 動的型付け言語にも型チェックを行わせる
- 関数を呼び出す際に引数名を省略しない

## 全ての変数、関数の引数・戻り値の型を書く

Python でコード例を示していきます。

### 変数

```python
number_list: list[str] = ["0", "1", "2", "3", "4"]
```

値に Null になり得る場合は、オプショナル型としてきちんと明記しましょう。

```python
number_list: list[str | None] = ["0", "1", "2", "3", None, "4"]
```

:::message
上記は Python 3.10 で可能になった書き方です。
Python 3.9 までは以下のように書く必要があります。

```python
from typing import Optional

number_list: list[Optional[str]] = ["0", "1", "2", "3", None, "4"]
```

:::

### タイプエイリアス

型が複雑な場合は、型の別名(`タイプエイリアス`)定義するのも手です。
ただし個人的には、`Null` とその他の型とは分けて定義した方が良いと考えています。
Null は 1 段階、型の抽象度が上がるからです。
いずれにしても、型は漏れなく明記していきましょう。

```python
NumberType = str | int | float

number_list: list[NumberType | None] = [0, "1", "2", "3", None, "4", 4.5]
```

:::message
Python 3.9 までの書き方

```python
from typing import Union

NumberType = Union[str, int, float]

number_list: list[Optional[NumberType]] = [0, "1", "2", "3", None, "4", 4.5]
```

:::

### 例外

以下のように右辺からコードから型が明らかな場合は、型の明記を省略しても構いません。

```python
hoge_class = HogeClass(param: str)
```

### 関数

関数の場合も、変数の場合と意識する事はほぼ同じです。引数、戻り値の型を漏れなく明記しましょう。

```python
def has_spesific_number_in_list(number: str) -> bool:
    number_list: list[str] = self.fetch_number_list_from_db()
    return (number in number_list)
```

関数の戻り値がない場合は、明示的に`None`[^1]としてあげると、書き忘れではないことを読み手に伝える事ができ、ベターです。
[^1]: ここでは`None`は Java や Swift の`Void`型と同じ扱いになります。

```python
def append_number_to_list_in_db(number: str) -> None:
    ...
```

## 動的型付け言語にも型チェックを行わせる

上述したように全ての変数、関数の引数・戻り値の型を明記すれば、コードの読み手が型の予想を強いられる頻度は、かなり減るはずです。

静的型付け言語の一つ Swift には、[Xcode](https://developer.apple.com/jp/xcode/) という専用の統合開発環境(IDE)があり、コンパイルよりもさらに前、コードを書いた瞬間に型走査が走り、型の不一致があればエラーを吐き出します。

動的型付け言語には型がないため、デフォルトでは型チェックをする機能はありません。
しかし、以下のような型チェックライブラリを用いれば、動的型付け言語であっても、プログラム実行時よりも前に、型の不一致に気付けます。

- Ruby: [Sorbet](https://sorbet.org/), [Steep](https://github.com/soutaro/steep)
- Python: [mypy](https://github.com/python/mypy)

## 関数を呼び出す際に引数名を省略しない

最後に、型を漏れなく明記しても、予期しない値が混入しやすくなる NG 例を取り上げます。
以下のような関数があったとします。

```python
def append_and_print(number: str, additional_message: str) -> None:
    self.number_list.append(number)
    print(f"One number has appneded. {additional_message}")
```

引数 `number` は list に追加され、引数 `additional_message` は、`print` 関数で出力される、という処理です。
この関数の引数の型は、どちらも `str` 型です。
以下のような引数の渡し方をしたらどうなるでしょうか。

```python
append_list_and_print("5 has appended.", "5")
```

渡すべき値の順番が逆なので、引数 number に`"5 has appended."`が渡され、引数 additional_message に`"5"`が渡されてしまいます。

このような値の渡しミスを防ぐ為にも、引数名は省略せず、明記すると良いです。

```python
append_and_print(number="5", additional_message="5 has appended.")
```

## まとめ

不安な時は型を書いて安心しよう！
