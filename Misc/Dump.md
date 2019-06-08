# [Misc] Dump
pcapファイルから、大量の8進数文字列を抽出する問題です。  
問題URL：https://score.beginners.seccon.jp/files/fc23f13bcf6562e540ed81d1f47710af_dump  
6月前半まで。  
*技術用語：WireShark、バイナリと文字列の変換、TCP通信の基本*

## 解答例
1. `file`コマンドにより、ダウンロードしてきたファイルの種類を判別する。  
1. WireSharkでダウンロードファイルを開く。  
1. WebShellを用いていることを把握し、実行されたコマンドを読み取る。
1. `hexdump`された数値の文字列を抽出する。
1. 数値の文字列をバイナリとして保存し、作成されたファイルを`file`コマンドで確認する。
1. `tar`コマンドで解凍し出てきたファイルを閲覧すると、flagゲット。

## 詳細
### 0. この問題について
pcapファイルから、大量の8進数の文字列を抽出する問題です。  
一番下のほうにすべての値がまとまった箇所があるのですが、とても開くのに時間がかかります。

### 1～2. 準備
まず、問題URLからファイルをダウンロードしてきます。  
どんなファイルか判別できないため、`file`コマンドで判別します。
~~~
$ file ファイル名
ファイル名：pcap capture file, microsecond ts (little-endian) - version 2.4 (Ethernet, capture length 262144)
~~~
すると、pcapファイル、つまりWiresharkなどを用いて、パケットをキャプチャしたファイルであるということが分かります。  
そこで、WireSharkにて、該当ファイルを開きます。

### 3. WebShellを用いていることを把握し、実行されたコマンドを読み取る。
pcapファイルを読んでいきます。  
まず、IPアドレス`192.168.75.230`に対して`192.168.75.1`より`ping`コマンドを実行したパケットが、8番目まで続きます。  

その後、TCPによる通信が始まります。  
読み進めていくと、12番のところで、WebShell.phpというところに対し、GETメソッドで以下のクエリ文字列を送っています。  
~~~
?cmd=ls%20%2Dl%20%2Fhome%2Fctf4b%2Fflag
    ⇒ ls -l  /home/ctf4b/flag
~~~
URLエンコーディングされているので、読み替えたものが下です。  
`ls -l`コマンドで、ファイル`flag`の詳細な情報を取得しようとしているようです。  

14番でコマンドの実行結果が返ってきます。以下に、抜粋を示します。  
~~~html
<html>\n
<head>\n
<title>Web Shell</title>\n
</head>\n
<pre>\n
  -rw-r--r-- 1 ctf4b ctf4b 767400 Apr  7 19:46 /home/ctf4b/flag\n
</pre>\n
</html>\n
~~~
どうやらファイルは存在するようです。  

次に、22番で再びGetメソッドにて`hexdump`コマンドを実行しています。  
~~~
cmd=hexdump%20%2De%20%2716%2F1%20%22%2502%2E3o%20%22%20%22%5Cn%22%27%20%2Fhome%2Fctf4b%2Fflag
    ⇒ hexdump -e '16/1 "%02.3o " "\n"' /home/ctf4b/flag
~~~
ここではflagファイルを、16回出力ごとに1度改行しながら、8進数でダンプしているようです。  
この後のパケットは`hexdump`コマンドの出力が続きます。

### 4～5. 8進数文字列を抽出し、バイナリファイルとして保存し、flag取得を目指す。
さて、hexdumpの出力は続きますが、さすがに数千パケットを結合していくということはつらいです。  
pcapファイルの下のほうを見ていきますと、3193番目にHTTPレスポンスがあることが分かります。  
少し重いですがそちらを見てみますと、どうやらこれまでの出力がまとまっているようです。  
3193番目のパケットの8進数の部分をコピーなどで抽出し、8進数の数値を変換し、バイナリとして保存します。 
ここでは、hexdump.txtに保存しました。  

次に、python3を用いてバイナリへと変換します。  
~~~Python3
def main():
    with open("hexdump.txt", "r") as fin:
        text = fin.read()
        text_list = text.replace("\n", " ").split(" ")
        ans = b"";

        for oct_text in text_list:
            ans += int(oct_text, 8).to_bytes(1, "little")

        with open("output", "wb") as fout:
            fout.write(ans)

if __name__ == "__main__":
    main()
~~~
できたファイルoutputを`file`コマンドでファイルの種別を判別します。  
すると、`tar`ファイルのようですので、`tar`コマンドで解凍します。  
出てきたpngファイルを開くと、flagゲットです。
