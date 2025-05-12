[toc]

## ice's polygon 笔记

### 简介

本笔记是 ice 在使用 [polygon](https://polygon.codeforces.com/) 进行出题时的笔记，旨在于帮助快速上手 polygon 并且能够出属于自己的题目。



### Statement



### Files



### Validator

Validator 用于检验你生成的数据，以及 cf 上的 hack 数据的输入是否合法。模板为

```cpp
#include <bits/stdc++.h>
#include "testlib.h"

using namespace std;

int main(int argc, char* argv[]){
	registerValidation(argc, argv);
	
}
```



在 testlib.h 里，可以通过 `inf.xxx()` 来从输入中执行某些函数，可以使用以下函数：

```cpp
inf.readInt(); //从输入中读入一个 int 类型整数
inf.readInt(min_value, max_value, variable_name); //从输入中读入一个 int 类型整数，且这个整数需要在 [min_value, max_value] 中，否则 Validator 会返回错误，即输入不合法。 variable_name 表示这个变量的名字。

inf.readLong(); //从输入中读如一个 ll 类型整数
inf.readLong(min_value, max_value, variable_name); //和 inf.readInt() 一样，只不过读入的是一个 ll 类型整数

inf.readChar(); //从输入中读入一个 char 类型字符
inf.readChar(c); //从输入中读入一个 char 类型字符，且这个字符一定要是 c

inf.readWord(); //从输入中读入一个 string 类字符串

inf.readSpace(); //从输入中读入一个空格

inf.readEoln(); //从输入中读入一个换行

inf.readEof(); //从输入中读入 EOF(文件结束标志)

```



**请注意，你的校验器 Validator 需要严格的读入空格和换行进行格式判断。**

假设输入样例为(包含 T 测)

```
2
4
1 3 5 9
4
2 2 2 2

```



那么 Validator 需要如下书写：

```cpp
#include <bits/stdc++.h>
#include "testlib.h"

using namespace std;

int main(int argc, char* argv[]){
	registerValidation(argc, argv);
	int T = inf.readInt();
  inf.readEoln(); //记住要读入换行
  while(T--){
    int n = inf.readInt();
    inf.readEoln(); //记住读入换行
    for(int i = 1; i <= n; i++){
      int x = inf.readInt();
      if(i != n) inf.readSpace(); //这里的输入样例没有行末空格，请注意判断！
    }
    inf.readEoln();
  }
  inf.readEof(); //请一定要记得读入 EOF
}
```



### Checker

Checker 用于检验参赛者的输出以及答案的输出是否一样，模板为

```cpp
#include <bits/stdc++.h>
#include "testlib.h"

using namespace std;

int main(int argc, char* argv[]){
	registerTestlibCmd(argc, argv);
	
}
```



在 testlib.h 里，可以使用 `ouf.xxx()` 从参赛者输出中执行某些函数，使用 `ans.xxx()` 从答案中执行某些函数，可以执行以下函数：

```cpp
ouf.readInt(); //从参赛者输出中读入一个 int 类型整数
ouf.readInt(min_value, max_value, variable_name); //从参赛者输出中读入一个 int 类型整数，且这个整数需要在 [min_value, max_value] 中，否则 Checker 会返回错误，即输入不合法。 variable_name 表示这个变量的名字。

ouf.readLong(); //从参赛者输出中读如一个 ll 类型整数
ouf.readLong(min_value, max_value, variable_name); //和 ouf.readInt() 一样，只不过读入的是一个 ll 类型整数

ouf.readChar(); //从参赛者输出中读入一个 char 类型字符
ouf.readChar(c); //从参赛者输出中读入一个 char 类型字符，且这个字符一定要是 c

ouf.readWord(); //从参赛者输出中读入一个 string 类字符串

ouf.seekEof(); //若已经到达参赛者输出的文件末尾 (即下一个字符为 EOF) 则返回 true

//ans 与 ouf 一样，只不过把 ouf 改成 ans 即可
```



在 testlib.h 里，可以使用 `quitf()` 来返回判题结果，以及返回判题信息

```cpp
quitf(status, message);
//其中 status 代表返回的状态，_ok 代表 Accepted，_wa 代表 Wrong Answer.
//message 代表评测机给用户返回的详细信息
```



若需要参赛者的输出和答案输出完全一样，可以参考一下模板：

```cpp
#include <bits/stdc++.h>
#include "testlib.h"
typedef long long ll;

using namespace std;

int main(int argc, char* argv[]){
	registerTestlibCmd(argc, argv);
	while(!ans.seekEof()){
		ll jans = ans.readLong();
		if(ouf.seekEof())
			quitf(_wa, "Too few numbers in output, expected more");
		ll pans = ouf.readLong();
		if(pans != jans)
			quitf(_wa, "Expected %lld but received %lld", jans, pans);
	}
	if(!ouf.seekEof())
		quitf(_wa, "Too many numbers in output");
	quitf(_ok, "You finish all the test!");
}
```

以上代码实现了判断 用户输出的数字个数 和 答案输出的数字个数 是否相同，以及数字是否相同，可以依据实际情况进行更改。



Checker 还可以作为 Special Judge (spj)，手动书写正确条件。

例如给定 $a, b, c$，只要 $a \times b \times c \equiv M \pmod {2025}$ 即返回 AC，可以参考以下代码：

```cpp
#include <bits/stdc++.h>
#include "testlib.h"

using namespace std;

int main(int argc, char *argv[]){
	registerTestlibCmd(argc, argv);
	int T = inf.readInt();
	std::vector<int> ok(2030);
	for(int a = 1; a <= 100; a++)
		for(int b = 1; b <= 100; b++)
			for(int c = 1; c <= 100; c++)
				ok[a * b * c % 2025] = 1;
	inf.readEoln();
	while(T--){
		int M = inf.readInt();
		// inf.readEoln();
		int x = ouf.readInt(-1, 100, "a");
		if(x == -1){
			if(ok[M])
				quitf(_wa, "Galaxy_Ivan can found a, b, c such that a * b * c % 2025 == M");
			// ouf.readEoln();
		}else if(!x){
			quitf(_wa, "Found input number 0 but input should be in 1~100 or -1");
		}else{
			int a = x;
			int b = ouf.readInt(a, 100, "b"), c = ouf.readInt(b, 100, "c");
			// ouf.readEoln();
			int pans = a * b * c % 2025;
			if(pans != M)
				quitf(_wa, "Wrong answer.Got %d, expected %d", pans, M);
		}
	}
	// ouf.readEof();
	quitf(_ok, "You solved all the solution! You are so smart!");
}
```



### Tests

#### Generator

Generator 用来生成数据，让出题者不需要手动输入数据，而是可以通过 polygon 直接生成。模板为

```cpp
#include <bits/stdc++.h>
#include "testlib.h"

using namespace std;

int main(int argc, char* argv[]){
	registerGen(argc, argv, 1);
	
}
```







#### Script





### 一些报错的解决方法

#### Verification Failed

首先先在 Validator 和 Checker 下面添加 tests，检查是否能够正确运行，如果不行则进行修改

再在 Tests 里面，如果使用了 Generator 生成数据，点开 Preview，查看输入和输出是否能正确被生成，如果输入没有被生成，则 Generator 或者 Validator 有问题，若输出有问题，则 Checker 有问题

若 Checker 报错 `Wrong output format: Unexpected end of file - int32 expected`

则是你的 Checker 循环逻辑出错，检查是否使用了 `ans.eof()` 或者 `ouf.eof()`，若有，请更改为 `ans.seekEof()` 或者 `ouf.seekEof()` 控制循环。 
