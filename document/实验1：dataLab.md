# 实验1：dataLab

- [ 1.bitXor](#head1)
- [ 2.tmin](#head2)
- [ 3.isTmax](#head3)
- [ 4.allOddBits](#head4)
- [ 5.negate](#head5)
- [ 6.isAsciiDigit](#head6)
- [ 7.conditional](#head7)
- [ 8.isLessOrEqual](#head8)
- [ 9.logicalNeg](#head9)
- [ 10.howManyBits](#head10)
- [ 11.floatScale2](#head11)
- [ 12.floatFloat2Int](#head12)
- [ 13.floatPower2](#head13)
- [ 参考资料](#head14)


- 阅读`readme` 完成配置

~~~text
```
shell    To compile and run the btest program, type:
unix> make btest
unix> ./btest [optional cmd line args]
```
~~~

make btest 进行编译

./btest运行

每次更改bits.c文件后都要重新编译btest。如果需要检查单个函数的正确性，可以使用-f标志：

```
./btest -f bitXor
```

`dlc`程序可以检测我们有没有违规，如果运行没有输出则没有问题

```
./bits.c
```

可以采用vscode进行代码编写，进入容器编译运行



### <span id="head1"> 1.bitXor</span>

```c
int bitXor(int x, int y) {
  return ~(x&y)&~(~x&~y);
}
```

根据表格推导可得到结果

![image-20211019160020922](实验1：dataLab.assets/20211021211627.png)

### <span id="head2"> 2.tmin</span>

```c
int tmin(void) {
  return 1 << 31;
}
```

### <span id="head3"> 3.isTmax</span>

首先，Tmax + 1 会得到Tmin，  并且通过~Tmin，重新得到Tmax。 根据这一特性，配合x ^ x = 0  的特点，可以判断是否为Tmax   即 ~ (x + 1) ^ x  如果为0则x为最大值。

特殊的，由于0xffffffff也会得到相同的结果，其中关键的特点在于0xffffffff + 1 = 0x00000000，而其他值得到的都是非零值，利用!!的特点，0值会得到0，而非零值会得到1，过滤掉0xffffffff的情况，即!!(x + 1)

只要满足这两种条件，即能得到正确答案。

```c
int isTmax(int x) {
  return !(~(x+1) ^ x)&!!(x + 1);
}
```

### <span id="head4"> 4.allOddBits</span>

分别判断偶数位是否为1，暴力求解

```c
int allOddBits(int x) {
  return 1&(x >> 1) & 
         1&(x >> 3) & 
         1&(x >> 5) & 
         1&(x >> 7) & 
         1&(x >> 9) & 
         1&(x >> 11) & 
         1&(x >> 13) & 
         1&(x >> 15) & 
         1&(x >> 17) & 
         1&(x >> 19) & 
         1&(x >> 21) & 
         1&(x >> 23) & 
         1&(x >> 25) & 
         1&(x >> 27) & 
         1&(x >> 29) & 
         1&(x >> 31);
}
```

方法一超过了操作数12，所以应该采用下述方法。

方法二：通过构造0xAAAAAAAA来求解

```c
int allOddBits(int x) {
  int a = 0xA << 4 | 0xA;
  int b = a << 8 | a;
  int c = b << 16 | b;
  return !((c & x)^ c);
}
```

方法三：利用&运算符“归一性”

```c
int allOddBits(int x) {	
	x = (x>>16) & x;	
	x = (x>>8) & x;		
	x = (x>>4) & x;		
	x = (x>>2) & x;		
	return (x>>1)&1;	
	// &运算符的“归一性”
	//1010 1010 1010 1010 1010 1010 1010 1010
	//0000 0000 0000 0000 1010 1010 1010 1010
	//0000 0000 0000 0000 0000 0000 1010 1010
	//0000 0000 0000 0000 0000 0000 0000 1010
	//0000 0000 0000 0000 0000 0000 0000 0010 
	// 可以反推理解：后四位四次翻转得第一行
	// 只要倒数第二位为1成立，反推后所有的奇数位都为1
}
```



### <span id="head5"> 5.negate</span>

补码常识，取反加1

```c
int negate(int x) {
  return ~x + 1;
}
```

### <span id="head6"> 6.isAsciiDigit</span>

如果5、6位全1 且 （4位为0或4位为1，2、3位为0）

```c
int isAsciiDigit(int x) {
    int a = ((x >> 3) & 1);
  	return  !(x >> 4 ^ 0x3) & (!a | (a & ((x >> 2) ^ 1) & ((x >> 1) ^ 1) ));
}
```

通过抽取相同计算的作为变量，达到减少操作数的效果

### <span id="head7"> 7.conditional</span>

```c
int conditional(int x, int y, int z) {
  int a = !!x;  //先判断是不是为0
  int b =  (a << 31) >> 31;  //转化为0xffffffff或0x00000000
  return (~b & z)|(b & y);  //根据~0x00000000或0xffffffff来选择y或z
}
```

### <span id="head8"> 8.isLessOrEqual</span>

```c
int isLessOrEqual(int x, int y) {
  int Sx = (x >> 31) & 1; // 取出x的符号
  int Sy = (y >> 31) & 1; // 取出y的符号
  int z = x + ~y + 1;     // z = x-y
  int d = ((z >> 31) & 1) | !(z ^ 0x0); 
  // 同号下如果z的符号位为1，或者z全为0，说明x<=y。若x为负数，y为正数 x<=y也成立
  return !(Sx ^ Sy) & d | (Sx & !Sy); 
}
```

### <span id="head9"> 9.logicalNeg</span>

```c
int logicalNeg(int x) {
  // |运算符的"分裂性"
  x |= x >> 16;
  x |= x >> 8;
  x |= x >> 4;
  x |= x >> 2;
  x |= x >> 1;
  x ^= 1; // 此时可能为 0xfffffffe  
  return x & 1;
}
```

方法二：由于非零数相反数的符号位相反，采用 x | ~x + 1   则正数或负数最后得到的符号位都为1

将其右移31位后  正负数结果为-1，而零的结果为0，向上+1得到结果

```c
int logicalNeg(int x) {
  	return ((x | ~x + 1) >> 31) + 1;
}
```

### <span id="head10"> 10.howManyBits</span>

负数取反的目的：去掉符号位，计算所需要的位数

最大位数取决于最大1所在的位置。所以不断二分，直至找不到1为止。

```c
int howManyBits(int x) {
  int s = (x >> 31) & 1;  // 计算符号位，用于判断是否需要取反
  x = ((s << 31) >> 31) ^ x;   // 负数取反，正数不变，  目的是去掉符号位进行位数计算。
  s = !!(x >> 16);    //判断高16位是否有1
  int c1 = s << 4;     //若有，低16位存在，结果+16
  x = x >> c1;         //高c1位移动,  注意  这里要用c1而不是16，因为如果高16位没有1，就不需要计算和进行移动，直接从低16位开始
  s = !!(x >> 8);
  int c2 = s << 3;     
  x = x >> c2;          //同理，高8位存在1，则结果加8，移动8位。
  s = !!(x >> 4);
  int c3 = s << 2;     
  x = x >> c3;          //同理
  s = !!(x >> 2);
  int c4 = s << 1;     
  x = x >> c4;          //同理
  s = !!(x >> 1);
  int c5 = s;     
  x = x >> c5;          //同理
  int c6 = !!x ;
  return c1 + c2 + c3 + c4 + c5 + c6 + 1;
}
```

### <span id="head11"> 11.floatScale2</span>

![image-20211019200620809](实验1：dataLab.assets/20211021211628.png)

情况1：当 exp  == 255  

frac == 0 得到NaN

frac !=0  得到无穷大

return uf;

情况2：当exp == 0

frac == 0; 值为0

frac != 0； 值为frac

return frac << 1;

情况3： exp != 255 && exp != 0;

exp++;  其余不变

```c
unsigned floatScale2(unsigned uf) {
  //计算各个位置的值
  int exp = (uf & 0x7f800000) >> 23;
  int sign = uf >> 31 & 0x1;
  int frac = uf & 0x007fffff;

  if(exp == 255) {       //NaN或无穷大
    return uf;
  } else if (exp == 0){  //零值或frac值
    frac <<= 1;
    return sign << 31 | frac;
  } else {               //正常值，将exp+1即可
    exp++;
    return sign << 31 | exp << 23 | frac;
  }
}
```

### <span id="head12"> 12.floatFloat2Int</span>

```c
int floatFloat2Int(unsigned uf) {
    //计算各个位置的值
  int exp = (uf & 0x7f800000) >> 23;
  int sign = uf >> 31 & 0x1;
  int frac = uf & 0x007fffff;
  int E = exp - 127; //指数

  if(E < 0) {    //小于1， 舍入0
    return 0;
  } else if (E >=31){   //超过31  超出整数长度，出现溢出，返回0x80000000u
    return 0x80000000u;
  } else {
    frac = frac | 1 << 23;  // frac最高位补充被省略的1
    if(E < 23) {            //E小于23，则存在小数位，保留整数位即可，小数位舍入。
      frac = frac >>(23 - E);
    } else {                //E大于23，可以继续扩大。
      frac = frac <<(E - 23);
    }
  }
  //判断正负数
  if(sign){
    return -frac;
  }else {
    return frac;
  }
}
```

### <span id="head13"> 13.floatPower2</span>

分情况讨论：

情况一，都是0：

……

2^-150的二进制表示为0|00000000|00000000000000000000000



情况二，阶码0:

\-------------------------------------------------------------------

2^-149的二进制表示为0|00000000|00000000000000000000001

2^-148的二进制表示为0|00000000|00000000000000000000010

2^-147的二进制表示为0|00000000|00000000000000000000100

……

2^-129的二进制表示为0|00000000|00100000000000000000000

2^-128的二进制表示为0|00000000|01000000000000000000000

2^-127的二进制表示为0|00000000|10000000000000000000000



情况三，阶码非0：

\-------------------------------------------------------------------

2^-126的二进制表示为0|00000001|00000000000000000000000

2^-125的二进制表示为0|00000010|00000000000000000000000
2^-124的二进制表示为0|00000011|00000000000000000000000
……

2^0 的二进制表示为0|01111111|00000000000000000000000

2^1 的二进制表示为0|10000000|00000000000000000000000

……

2^127 的二进制表示为0|11111110|00000000000000000000000



情况四，无穷大：

\-----------------------------------------------------------------

2^128 的二进制表示为0|11111111|00000000000000000000000
2^129 的二进制表示为0|11111111|00000000000000000000000



```c
unsigned floatPower2(int x) {
    int exp = x + 127;
    if(exp >= 255){   		//情况4：无穷大或NaN
      return 0xff <<23;
    } else if(exp <= -23) {  //情况1：全0
      return 0;
    } else if(exp > 0){		//情况3：规格化
      return exp << 23;
    }else {					//情况2：非规格化
      return (1 << 23) >> exp;
    }
}
```

### <span id="head14"> 参考资料</span>

[CSAPP:Lab1 -DataLab 超详解](https://zhuanlan.zhihu.com/p/339047608)

[CSAPP：位操作实现基本运算](https://www.cnblogs.com/ustca/p/11740382.html)
