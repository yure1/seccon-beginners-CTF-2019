# [Crypto] Party
暗号方式をコードから読み取り、復号する問題です。  

問題URL：https://score.beginners.seccon.jp/files/f107d24b40270e3873d5e0dcfd3c93c5_party.tar.gz  
*技術用語：Python, sympy*

## 解答例
1. 問題URLよりダウンロードしたファイルを、`tar`コマンドで解凍する。
1. `encrypt.py`というPython3のコードを読み、暗号化方式を読み取る。
1. 復号するコードを作成し、`encrypted`というファイルに適用し、flagを取得する。

## 詳細
### 0. この問題について
連立方程式を解かなければならない箇所などがあり、アルゴリズム力やライブラリの知識が必要とされます。  
ここでは、sympyの勉強も兼ねて方程式を解いていきます。  
（なお、筆者、本番では連立方程式を解くアルゴリズムを必死こいて考えて解いた模様…。）

### 1～2. 準備
まず、問題URLからダウンロードしてきたファイルを`tar`コマンドで解凍します。
~~~
$ tar -xvf ファイル名
~~~
解凍したところ、encryptedとencrypt.pyという二つのファイルが得られます。  

encryptedには、よくわからない数字が書かれたリストが書いてあるので、encrypt.pyを読み解いていきます。  
~~~Python3
from flag import FLAG
from Crypto.Util.number import bytes_to_long, getRandomInteger, getPrime


def f(x, coeff):
    y = 0
    for i in range(len(coeff)):
        y += coeff[i] * pow(x, i)
    return y


N = 512
M = 3
secret = bytes_to_long(FLAG)
assert(secret < 2**N)

coeff = [secret] + [getRandomInteger(N) for i in range(M-1)]
party = [getRandomInteger(N) for i in range(M)]

val = map(lambda x: f(x, coeff), party)
output = list(zip(party, val))
print(output)
~~~
流れとしては、以下のような処理をしていると考えられます。  
1. FLAGのバイト文字列をlong型の整数とし、secretに格納する。
1. secretと、512bitのランダムな整数2つから構成されるリストcoeffを作成する。
1. 512bitのランダムな整数3つから構成されるリストpartyを作成する。
1. 以下の3つの式を計算し、valというリストを作成する。
    1. val[0] = coeff[0] * pow(party[0], 0) + coeff[1] * pow(party[0], 1) + coeff[2] * pow(party[0], 2)
    1. val[1] = coeff[0] * pow(party[1], 0) + coeff[1] * pow(party[1], 1) + coeff[2] * pow(party[1], 2)
    1. val[2] = coeff[0] * pow(party[2], 0) + coeff[1] * pow(party[2], 1) + coeff[2] * pow(party[2], 2)
1. partyとval、二つのリストの要素をまとめて出力する。  

ここまで、分かれば後は逆算していくだけです。

### 3. 暗号化された文を復号していく。
暗号化方式を逆算してflag取得を目指します。  
sympyを用いて、連立方程式を解いています。

~~~Python3
import sympy
import Crypto.Util.number

def find_coeff(party, val):
    x = sympy.Symbol('x') # coeff[0]
    y = sympy.Symbol('y') # coeff[1]
    z = sympy.Symbol('z') # coeff[2]

    expr1 = x * pow(party[0], 0) + y * pow(party[0], 1) + z * pow(party[0], 2) - val[0]
    expr2 = x * pow(party[1], 0) + y * pow(party[1], 1) + z * pow(party[1], 2) - val[1]
    expr3 = x * pow(party[2], 0) + y * pow(party[2], 1) + z * pow(party[2], 2) - val[2]

    solve_expr = sympy.solve([expr1, expr2, expr3])

    return solve_expr[x]

def main():
    party = []
    val = []

    with open("encrypted", "r") as fin:
        text = fin.read()
        text = text[1:-2]
        
        # partyとvalの取得
        text_list = text.split(", ")
        for i in range(0, 6, 2):
            party.append(int(text_list[i][1:]))
            val.append(int(text_list[i+1][:-1]))

        secret = find_coeff(party, val)
        print(Crypto.Util.number.long_to_bytes(secret))

if __name__ == "__main__":
    main()
~~~
上記のコードを実行して、flagを取得することができます。
