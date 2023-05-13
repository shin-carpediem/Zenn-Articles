---
title: "動的型付け言語こそ型を書こう"
emoji: "🍴"
type: "tech"
topics: ["動的型付け言語", "静的型付け言語"]
published: true
---

## なぜ型を書くべきか

予期しない値が混入して、アプリケーションがクラッシュしてしまうのを避けるためです。

動的型付け言語 Python で書かれた以下のコードを見てください。

```python
number_list = ["0", "1", "2", 3, "4"]

for number in number_list:
    int(number_list)
```

このコードは、number が 3 の時に、`int()`メソッドの引数に `str` 型ではないもの(`int`型)が渡されるので、クラッシュします。

静的型付け言語は、コンパイル時に変数の型が決まるので、アプリを実行する前に型の不一致による上述したエラーを検出できます。
一方動的型付け言語は、コンパイル時よりも後、プログラム実行時に変数の型が決まります。
なので、プログラムを実行するまで上述したエラーに、仕組みの側からは気づけません。

つまり、動的型付け言語の方が、アプリケーションがクラッシュしてしまうリスクが高いと言えます。

## コードは読み手に優しくあるべき

一般にプログラマは、コードを書く時間よりもコードを読む時間の方が長いとされます。
自分の書いたコードでも、半年も経てば、他者が書いたコードのように見えます。

自分がコードを書いている時は、自分が定義した変数に、何が代入されうるのかを想定できています。
しかし、他者が定義した変数にどんな値が入りうるかは、型が明記されていなければ、上流のコードを辿って行くしかありません。
それが何十行、何百行にもなると、
**最終的には途中でコードを読むのをやめて、型を予想し始めます。**
これがバグを産むきっかけです。

このリスクを下げるために、動的型付け言語でも、しっかりと型の定義をすべきなのです。

## 何から始めるべきか

2 つあります。

- 変数、関数の戻り値の型を極力明記する。
- コンパイル時 or それよりも前に、型の不一致を検出する拡張機能をインストールする。

### 変数、関数の戻り値の型を極力明記する

コードを読みやすくするため、できる限り全てに型を明記しましょう。

```python
number_list: list[str] = ["0", "1", "2", "3", "4"]
```

オプショナルかどうかもきちんと明記しましょう。

```python
number_list: list[str | None] = ["0", "1", "2", "3", None, "4"]
```

関数の場合も同様です。

```python
def appended_list(number: str) -> list[str | None]:
    return self.number_list.append("5")
```

関数の戻り値がない場合は、明示的に`None`としてあげると、書き忘れではないことを読み手に伝える事ができ、ベターです。

```python
def append_list(number: str) -> None:
    self.number_list.append("5")
```

型が複雑な場合は、タイプエイリアスを定義するのも手です。
いずれにしても、型は丁寧に明記していきましょう。

```python
number_type = str | int | float | None

number_list: list[snumber_type] = [0, "1", "2", "3", None, "4", 4.5]
```

### コンパイル時 or それよりも前に、型の不一致を検出する拡張機能をインストールする

静的型付け言語の一つ swift は、[Xcode](https://developer.apple.com/jp/xcode/) という専用の統合開発環境(IDE)があり、コンパイルよりもさらに前、コードを書いた瞬間に、型の不一致を検出し、エラーを検出します。

Python であれば、[mypy](https://github.com/python/mypy)という型捜査ライブラリがあり、おすすめです。

上述した 2 つを行えば、動的型付け言語であっても、プログラム実行時よりも前に、エラーに気づく事ができます。