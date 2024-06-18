# 环境搭建

## 安装wsl

```bash
wsl --install
```

## 在 Windows 上安装 Docker 桌面

**将Docker Desktop的默认安装路径安装路径移到D盘**

```bash
mklink /j “C:\Program Files\Docker” “D:\software\docker\install”
```

**安装docker desktop**

```bash
//进入安装包所在文件夹
start /w "" "Docker Desktop Installer.exe" install --installation-dir=D:\software\docker\install
```

## 拉取镜像

- 下载完成后直接在终端中运行这条语句即可
- `docker pull xieguochao/csapp`
- 注意可能会下载一段时间，大概有不到2G左右
- 之后点击docker界面左侧的Images即可看到我们下载好的镜像

## 容器设置

点击run后出现如下的界面

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210718214307267.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3JvbWFuX2J1X3RpYw==,size_16,color_FFFFFF,t_70)

containerName就是容器名字，随便起一个就好比如“csapp_lab”

LocalHost随便一串数字就行

HostPath是你想要在容器中包含的本地文件夹，**使用绝对路径，如D:\file\CS\CSAPP\labs\myLabs\bomb**

containerPath是你想要将本地文件夹同步在容器中的位置，**如/home/csapp/bomb**

[安装Docker在D盘](https://www.jianshu.com/p/1ae32787fb14)

# DataLab

## 位操作技巧

### 判断一个数某几位是否符合要求

```c
```

### 获取符号位方法

```c
//有符号数
 int a = x >> 31 & 0x1;
//无符号数
 int a = x >> 31;
```

### 使用+表示-

 ```c
 int res = y + (~x+1);
 ```

### 取反码

```c
x = (~x&sign) | (~sign&x);//负数则取反，非负数不变
```

## 1.1bitXor(x,y)

```c
/* 
 * bitXor - x^y using only ~ and & 
 *   Example: bitXor(4, 5) = 1
 *   Legal ops: ~ &
 *   Max ops: 14
 *   Rating: 1
 */
 /*
 *结果可以使用“非”和“与”计算不是同时为0情况和不是同时为1的情况进行位与
 */
int bitXor(int x, int y) {
  return ~(~x&~y)&~(x&y);
}
```

## 1.2tmin()

```c
/* 
 * tmin - return minimum two's complement integer 
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 4
 *   Rating: 1
 */
 /*
 *使用位运算获取补码的最小 int 值
 */
int tmin(void) {
  return 0x1<<31;
}
```

## 1.3isTmax

我们考虑四位的最大值`x=0111` 然后x+1之后就会变成`1000` 我们对`1000` 取非 `0111` 就会重新变回x值

这里要是可以用等于是不是直接完成了,但是不能用等于，在这可以用一个位运算的小技巧，我们知道自己与自己异或会得到0，也就是说我们可以用异或来判断等于`!((~(x+1)^x))` 判断这个是否为1即可判断是否为最大值

这里有一个例外就是`x=-1` 由于`-1=1111` 他利用上面的式子判断也符合，故要特判-1 利用`!!(x+1)` 这个操作-1和最大值并不相同

```c
/*
 * isTmax - returns 1 if x is the maximum, two's complement number,
 *     and 0 otherwise 
 *   Legal ops: ! ~ & ^ | +
 *   Max ops: 10
 *   Rating: 1
 */
int isTmax(int x) {
  /*return !(x^(0x1<<31)+1);
  *没发现不对的地方
  */
  //return !(~(x+1)^x)&!!(x+1);
  return !((x^(0x1<<31))+1);
}
```

## 1.4 allOddBits

先通过&保留所有奇数位，再通过^判断是否符合

```c
/* 
 * allOddBits - return 1 if all odd-numbered bits in word set to 1
 *   where bits are numbered from 0 (least significant) to 31 (most significant)
 *   Examples allOddBits(0xFFFFFFFD) = 0, allOddBits(0xAAAAAAAA) = 1
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 12
 *   Rating: 2
 */
int allOddBits(int x) {
  return !((0xAAAAAAAA&x)^0xAAAAAAAA);
}
```

## 1.5 negate

```c
/* 
 * negate - return -x 
 *   Example: negate(1) = -1.
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 5
 *   Rating: 2
 */
int negate(int x) {
  return ~x+1;
}
```

## 1.6 isAsciiDigit

```c
/* 
 * isAsciiDigit - return 1 if 0x30 <= x <= 0x39 (ASCII codes for characters '0' to '9')
 *   Example: isAsciiDigit(0x35) = 1.
 *            isAsciiDigit(0x3a) = 0.
 *            isAsciiDigit(0x05) = 0.
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 15
 *   Rating: 3
 */
int isAsciiDigit(int x) {
  return !((x>>3)^0x6) | !((x>>1)^0x1c);
}
/*
*A：0x30-0x37:0110 000-0110 111; B：0x38-0x39:011100 0 - 011100 1
*A情况只需要判断0110四位即可
*B情况只需要判断011100六位即可
*/
```

## 1.7 conditional

```c
/* 
 * conditional - same as x ? y : z 
 *   Example: conditional(2,4,5) = 4
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 16
 *   Rating: 3
 */
 /*使用+为什么返回为0x0
int conditional(int x, int y, int z) {
  return (~(!!x)+1)&y + (~(!x)+1)&z;
}
*/
/*
*~(!!x)之后为0xFFFFFFFF或者0xFFFFFFFD
+1之后，0xFFFFFFFF变为0x0，0xFFFFFFFD变为0xFFFFFFFF
因为当x不为0x0时，~(!!x)+1为0xFFFFFFFF，(~(!!x)+1)&y之后仍为y
当x为0x0时，~(!!x)+1为0x0，(~(!!x)+1)&y之后为0x0
z则相反，因此取~(~(!!x)+1)
*/
int conditional(int x, int y, int z) {
  //return (~(!!x)+1)&y | (~(!x)+1)&z;
  return (~(!!x)+1)&y | ~(~(!!x)+1)&z;
}
```

## 1.8 isLessOrEqual

```c
/* 
 * isLessOrEqual - if x <= y  then return 1, else return 0 
 *   Example: isLessOrEqual(4,5) = 1.
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 24
 *   Rating: 3
 */
 /*
 获取符号位方法
 int a = x >> 31 & 0x1;
 使用+表示-
 int res = y + (~x+1);
 */
int isLessOrEqual(int x, int y) {
  int a = x >> 31 & 0x1;
  int b = y >> 31 & 0x1;
  int c1 = (a&~b);
  int c2 = (~a&b);

  int res = y + (~x+1);
  int flag = res >> 31 & 0x1;
  return c1 | (!c2&!flag);
}
```

## 1.9 logicalNeg

```c
/* 
 * logicalNeg - implement the ! operator, using all of 
 *              the legal operators except !
 *   Examples: logicalNeg(3) = 0, logicalNeg(0) = 1
 *   Legal ops: ~ & ^ | + << >>
 *   Max ops: 12
 *   Rating: 4 
 */
 /*
 关键点是确定0
 0是唯一一个取反+1之后仍为0的数
 其余数字取反+1之后符号位会变化，|的结果为1
 通过~x+0x2可以将0x0、0x1的值调换
 */
int logicalNeg(int x) {
  int a = (~x+1) >> 31 & 0x1;
  int b = x >> 31 & 0x1;
  return ~(a | b) + 0x2;
}
```

## **1.10 howManyBits**

如果是一个正数，则需要找到它最高的一位（假设是n）是1的，再加上符号位，结果为n+1；如果是一个负数，则需要知道其最高的一位是0的（例如4位的1101和三位的101补码表示的是一个值：-3，最少需要3位来表示）

```c
/* howManyBits - return the minimum number of bits required to represent x in
 *             two's complement
 *  Examples: howManyBits(12) = 5
 *            howManyBits(298) = 10
 *            howManyBits(-5) = 4
 *            howManyBits(0)  = 1
 *            howManyBits(-1) = 1
 *            howManyBits(0x80000000) = 32
 *  Legal ops: ! ~ & ^ | + << >>
 *  Max ops: 90
 *  Rating: 4
 */
int howManyBits(int x) {
  int b16,b8,b4,b2,b1,b0;
  int sign=x>>31;
  x = (sign&~x)|(~sign&x);//如果x为正则不变，否则按位取反（这样好找最高位为1的，原来是最高位为0的，这样也将符号位去掉了）


// 不断缩小范围
  b16 = !!(x>>16)<<4;//高十六位是否有1
  x = x>>b16;//如果有（至少需要16位），则将原数右移16位
  b8 = !!(x>>8)<<3;//剩余位高8位是否有1
  x = x>>b8;//如果有（至少需要16+8=24位），则右移8位
  b4 = !!(x>>4)<<2;//同理
  x = x>>b4;
  b2 = !!(x>>2)<<1;
  x = x>>b2;
  b1 = !!(x>>1);
  x = x>>b1;
  b0 = x;
  return b16+b8+b4+b2+b1+b0+1;//+1表示加上符号位
}
```

## **1.11 floatScale2**

```c
/* 
 * floatScale2 - Return bit-level equivalent of expression 2*f for
 *   floating point argument f.
 *   Both the argument and result are passed as unsigned int's, but
 *   they are to be interpreted as the bit-level representation of
 *   single-precision floating point values.
 *   When argument is NaN, return argument
 *   Legal ops: Any integer/unsigned operations incl. ||, &&. also if, while
 *   Max ops: 30
 *   Rating: 4
 */
unsigned floatScale2(unsigned uf) {
  //取符号位
  unsigned sign = (uf >> 31) & 0x1;
  //取阶码
  unsigned exp = (uf<<1) >> 24;
  //取尾数
  unsigned frac = (uf<<9) >> 9;
  
  //NAN
  if(exp == 255)
    return uf;
  //非规格化，*2之后仍为非规格化
  else if(exp == 0){
    frac = frac << 1;
    return (sign<<31) | frac;
  }
  //规格化数
  else if(exp > 0 && exp <= 254){
    exp += 1;
    return (sign<<31) | (exp<<23) | frac;
  }
}
```

## **1.12 floatFloat2Int**

```c
/* 
 * floatFloat2Int - Return bit-level equivalent of expression (int) f
 *   for floating point argument f.
 *   Argument is passed as unsigned int, but
 *   it is to be interpreted as the bit-level representation of a
 *   single-precision floating point value.
 *   Anything out of range (including NaN and infinity) should return
 *   0x80000000u.
 *   Legal ops: Any integer/unsigned operations incl. ||, &&. also if, while
 *   Max ops: 30
 *   Rating: 4
 */
int floatFloat2Int(unsigned uf) {
  //取符号位
  int sign = uf >> 31;
  //取阶码
  unsigned exp = (uf<<1) >> 24;
  //unsigned exp = (uf&0x7f800000)>>23;
  //取尾数
  unsigned frac = (uf<<9) >> 9;
  //unsigned frac=uf&0x7FFFFF;
  //0值
  //求E
    unsigned E = exp - 127;
    if(E < 0)
    return 0;
  //越界或者无穷大、NAN
  else if(E >= 31)
    return 0x80000000;
  else{
    //规格化 
    //补1
    frac = frac | (0x1 << 23);
    //看是否需要舍入
    if(E < 23){
      frac = frac >> (23 - E);
    }
    else frac = frac << (E - 23);
    if(sign == 1)
      return -frac;
    else return frac;
  }
}
```

## **1.13 floatPower2**

```c
/* 
 * floatPower2 - Return bit-level equivalent of the expression 2.0^x
 *   (2.0 raised to the power x) for any 32-bit integer x.
 *
 *   The unsigned value that is returned should have the identical bit
 *   representation as the single-precision floating-point number 2.0^x.
 *   If the result is too small to be represented as a denorm, return
 *   0. If too large, return +INF.
 * 
 *   Legal ops: Any integer/unsigned operations incl. ||, &&. Also if, while 
 *   Max ops: 30 
 *   Rating: 4
 */
unsigned floatPower2(int x) {
    unsigned res = x;
  if(x > 127) return +INF;
  else if(x < -149) return 0;
  else if(x > -127){
    res = res + 127;
    res = res << 23;
    return res;
  }
  else if(x <= -127){
    return 0x00800000 >> (-(res+126));
  }
}
```

[CSAPP 之 DataLab详解，没有比这更详细的了](https://zhuanlan.zhihu.com/p/59534845)

[CSAPP:Lab1 -DataLab 超详解](https://zhuanlan.zhihu.com/p/339047608)

[超简便配置csapp lab环境，又少一个拖延不做的理由](https://blog.csdn.net/roman_bu_tic/article/details/118883365)
