# [Crypto] So Tired
ぼくのかんがえたさいきょうのあんごうです。

問題URL：https://score.beginners.seccon.jp/files/7e9eb0636de2cad98b1eee3b667aea6c_so_tired.tar.gz  
6月前半まで。  
*技術用語：base64、zlib*

## 解答例
1. 問題URLのファイルを`tar`コマンドで解凍する。
1. 解凍して取得したテキストencrypted.txtを見て、base64であると検討をつけ、デコードする。
1. 得られたファイルoutputの種類を`file`コマンドを用いて、zlibで圧縮されたファイルであると判別する。
1. zlibファイルを解凍する。
1. 再びbase64のテキストが得られる。
1. base64デコード ⇒ zlibの解凍をflag取得まで繰り返す。

## 詳細
### 0. この問題について
僕の考えた最強の暗号の危うさを伝えている気がする問題です。  
計算がPCにとって簡単な問題は、方式が分かると一瞬で解かれてしまいます。

### 1～5. 準備
問題より取得されたファイルは、tar.gzで圧縮されているので`tar`コマンドで解凍します。
~~~
$ tar -xvf ファイル名
~~~
すると、encrypted.txtというテキストファイルが得られます。  
一見規則性のなさそうな文字列ですが、ファイルの末尾のほうを見てみると、`==`で終わっています。  
また、文字種は`a-z、A-Z、0-9、/、+、=`であるようです。  
encrypted.txtに書かれた文字列はbase64でエンコードされた文字列の特徴を持っています。  

`base64`コマンドでデコードしてみます。
~~~
$ base64 -d encrypted.txt > output
~~~
今回は、outputというファイルに保存しました。  

さて、outputの中身を見てみると、まったくよくわからない文字列です。  
どんなファイルか気になったので、`file`コマンドを使用してファイルの種類を判別します。
~~~
$ file output
output: zlib compressed data
~~~ 
どうやら、zlibで圧縮されたデータのようです。  
ここでは、python3を用いて解凍します。
~~~Python3
import zlib

with open("output", "rb") as fin:
  bin_text = fin.read()
  bin_text = zlib.decompress(bin_text)
  print(bin_text)
~~~
また、base64の文字列が出てきました。  
 
以上から、base64エンコードとzlibによる圧縮を繰り返した暗号であることが分かります。
 
### 6. base64デコードとzlibの解凍を繰り返す。
何度繰り返せばよいのか分からないので、zlib解凍後の文字列の長さが50より小さくなるまで、繰り返すことにします。  
以下、python3を用いたbase64デコードとzlibの解凍を繰り返すプログラムです。
~~~Python3
import base64
import zlib

def main():

    with open("encrypted.txt", "rb") as fin:
        bin_text = fin.read()

        while True:
            base64_decode_bin_text = base64.b64decode(bin_text)
            bin_text = zlib.decompress(base64_decode_bin_text)
            print(bin_text)

            if len(bin_text) < 50:
                break

if __name__ == "__main__":
    main()
~~~
実行したら、flag取得です。
