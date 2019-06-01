# [Reversing] Seccompare
elfファイルを解析する問題です。

問題URL：https://score.beginners.seccon.jp/files/seccompare_44d43f6a4d247e65c712d7379157d6a9.tar.gz
6月前半まで。
*技術用語：アセンブリ言語、逆アセンブラ*

## 解答例
1. `tar`コマンドでファイルを解凍する。
~~~
tar -xvf ファイル名
~~~
2. `file`コマンドで解凍されたファイルの種類が、elfファイルであることを判別する。
~~~
file seccompare
~~~
3. 逆アセンブラをするソフト(IDA, Ghidra, etc.)やデバッガ(gdb etc.)などを用いて解析し、処理の内容を把握する。
4. strcmp関数に代入されている文字列を見つけ出して、flag取得。

## 詳細
### 0. この問題について
皆さん大好きアセンブリ言語をゴリゴリ読んでいきましょう。  
デバッガで少しずつ答えを出していくもよし、逆アセンブラしてガツンと答えの部分を見つけるもよしです。

### 1～2. 問題を解くための準備
問題URLよりダウンロードしてきたファイルは、tar.gzファイルですのでまずは解凍をします。
~~~
tar -xvf ファイル名
~~~
ファイルが解凍され出現しますが、どんなファイルかわからないので`file`コマンドを用いて、ファイルの種類を判別します。
~~~
$ file seccompare
seccompare: ELF 64-bit LSB executable, x86-64,～
~~~
elfファイルであることが分かりました。  
実行してみます。
~~~
$ ./seccompare
usage: ./seccompare flag
~~~
実行の際、flagの文字列を入力するようです。  
適当な文字列で実行してみることにします。
~~~
$ ./seccompare ctf4b{aowejfoaj}
wrong
~~~
当たり前ですが、間違っているようです。  
そこで、逆アセンブルをしてみることにします。  

### 3～4. flag取得を目指す。
今回は、逆アセンブルにIDAのフリーウェア版を用います。  
先ほど解凍した実行ファイルをIDAで開きます。  
main関数の最初の処理から追っていきます。  
初めのずらずら書いてあるところは、後で使用する文字を格納する場所を指し示すポインタです。  

次に以下の処理について見ていきます。  
<img width="392" alt="無題" src="https://user-images.githubusercontent.com/51044014/58749431-080af080-84c1-11e9-9098-db2c255e237e.png">  
詳細は分かりませんが、コマンドライン引数が1つ以下だと`usage: ./seccompare flag`を表示して終了します。  
1つより多い場合は、loc_400630に分岐します。  

loc_400630では、まず1文字ずつスタックに積んでいきます。  
<img width="167" alt="2" src="https://user-images.githubusercontent.com/51044014/58750096-ddbd3100-84c8-11e9-9e8c-9b43aed11549.png">  
その後、先ほどスタックに積んだ文字列s1と、コマンドライン引数s2を引数としてstrcmp関数を実行します。  
<img width="185" alt="3" src="https://user-images.githubusercontent.com/51044014/58750139-7b186500-84c9-11e9-8ae1-5aaba5aed7db.png">  
s1とs2が一致していれば`correct`、一致していなければ`wrong`が表示されます。  

以上から、loc_400630の最初に積まれた文字列s1を読み取ればflagが取得できそうです。
IDAは勝手にasciiコードを文字に変換してくれないので、ascii表とにらめっこ、`echo -e "\x63\x74・・・"`で出力などで変換することで、flagが取得できます。  
また、今回はstrcmpという共有ライブラリの関数を使用しているため、`ltrace`コマンドが使用できます。（とても楽…。）
~~~
$ ltrace ./seccompare ctf4{aoeijf}
~~~
### 5. サンプルコード
今回の処理をコードにすると、このような感じです。  
問題の処理を完全に再現しているわけではないので、注意して下さい。
~~~
#include<stdio.h>
#include<string.h>

int main(int argc, char *argv[]) {
  if (argc > 1) {
    char s1[] = "ctf4b{samp1e}";
    int strcmp_value = -1;

    strcmp_value = strcmp(s1, argv[1]);

    if (strcmp_value != 0) {
      puts("wrong");
    } else {
      puts("correct");
    }

  } else {
    puts("usage: ./seccompare flag");
  }

  return 0;
}
~~~
