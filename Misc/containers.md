# [Misc] Containers
バイナリを読んで、ファイルの始端と終端を把握し切り出すという、ことをします。  
ほかの方のWriteUPは、より簡単に解かれているようなので、勉強していきたいところです。  
問題URL：https://score.beginners.seccon.jp/files/e35860e49ca3fa367e456207ebc9ff2f_containers  
6月前半まで。  
*技術用語：マジックナンバー*

## 解答例
1. ダウンロードしたファイルを、バイナリエディタで開く。
2. 格納されているpngファイルを、マジックナンバーとファイルの番号を基に切り出す。
3. 書かれている画像の文字を順番に読むと、flagが取得できる。

## 詳細
### 0. この問題について
containersファイルは、多くのファイルが一つのファイルとして保存されています。  
Dockerとかかなぁ、という無知を晒したのはいい思い出です。

### 1. ダウンロードしたファイルをバイナリエディタで開く
問題URLからダウンロードしたファイルは名前が長いので、準備としてリネームしました。  
~~~
$ mv ダウンロードファイル名 containers
~~~

さて、問題のファイルの種類を`file`コマンドで判別します。
~~~
$ file containers
containers: data
~~~
これだけでは、どんなファイルか検討もつきません。  
そこで、バイナリエディタでファイルを開いてみます。  
ここでは、Stirlingというソフトを用いて開きます。

### 2. バイナリを読み、pngファイルを切り出して、flag取得を目指す。
#### 2.1 マジックナンバー
マジックナンバーとは、ファイルの識別のためファイルの先頭や終端に書かれる文字列のことです。  
例えば、pngファイルは`89 50 4E 47(= 臼NG)`で始まり、`49 45 4E 44 (= IEND)`で終わります。  
このマジックナンバーを基に、pngファイルを切り出していきます。

#### 2.2 バイナリの観察
バイナリエディタで、バイナリを見ていきますと以下のような構造になっていることが分かります。
~~~
CONTAINER.FILE0
pngファイル
FILE1
pngファイル
～以下、これが続いていく～
~~~
このことから、pngファイルの始端と終端を切り出していけば全ファイルを抽出することができます。  
先ほどの、マジックナンバーを基にpngファイルの始端と終端を切り出す、ということをcontainersファイルの最後まで繰り返すことで、
すべての画像を抽出することができます。

### 3. 補足
上記の方法は非常にめんどくさいです。  
私は開催中思いつけなかったのですが、とても楽な方法に`foremost`コマンドを用いるというものがあります。  
この場合は、61の画像を以下のコマンド一つで抽出することができます。
~~~
foremost containers
~~~
とても楽ですね…。  
ただ、一枚は切り出してみると、ファイルの構造が分かってお勧めです（笑）