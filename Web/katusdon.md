# [Web] katsudon
クーポンが何で暗号化されているか？、を調べる問題です。

問題URL1：https://katsudon.quals.beginners.seccon.jp/storelists  
問題URL2：https://katsudon.quals.beginners.seccon.jp/flag  
6月前半まで。  
*技術用語：base64、Rails*

## 解答例
1. クーポンが`--`で区切られていることに気づく。
1. 前半部分がBase64で暗号化されていると考えて復号し、flag取得。
~~~
echo "BAhJIiVjdGY0YntLMzNQX1kwVVJfNTNDUjM3X0szWV9CNDUzfQY6BkVU" | base64 -d
~~~

## 詳細
### 0. この問題について
「Rails5.2.1で作られたサイトです。」  
という怖い文言から始まる問題ですが、関係ありませんでした。  
katsudon okawariという問題が追加されたので、出題者の意図と違う問題だったのかもしれません。

### 1～2. flag取得を目指す
問題URL1に記載された暗号化されたクーポンを眺めてみます。  
そうすると、`--`の前後で使用されている文字種が、少し変化していることに気づきました。  
前半部分は、`a-z, A-Z, 0-9, =`、後半部分は、`a-z, 0-9`、のようです。  
このことから、前半部分と後半部分で異なる暗号化方式が利用されているかも？と考えました。  

さて、前半部分の文字種の一つである`=`は特徴的であると感じました。
これは、Base64で文字数合わせの際に用いられる記号です。  
つまり、このクーポン券の前半部分はBase64で暗号化されているのではないか？、と考えられます。  

そこで、以下のコマンドを実行し、問題URL2に記載されたクーポンの前半部分を復号してみます。
~~~
echo "BAhJIiVjdGY0YntLMzNQX1kwVVJfNTNDUjM3X0szWV9CNDUzfQY6BkVU" | base64 -d
~~~
そうすると、flagをすべて取得することができました。  

### 補足
この問題の問題文には、暗号化されたクーポンを復号するコードが記載されていました。(まだ、未実装だそうですが。）  
復号している箇所は主にここだと考えられます。
~~~
@coupon_id = Rails.application.message_verifier(:coupon).verify(serial_code)
~~~
Rails.application.message_verifierは、主にCookieの改ざんチェックに使用されているそうです。  
generateにより、暗号化することができ、
~~~
Base64でエンコードされた文字列--文字列のSHA1ハッシュ
~~~
という文字列が取得されるようです。  
verifyは、署名付きメッセージが改ざんされていないかどうか？、をチェックしているようです。  
`--`で区切られた前半部分と後半部分は同じものを示していると考えられます。  

参考にさせていただいたサイト：  
RailsのMessageVerifierの内部実装を追ってみた - Qiita  
https://qiita.com/johro/items/06b146d749946e13ae0d  
2019/6/1アクセス
