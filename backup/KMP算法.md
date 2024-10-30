index
- 字符串相关概念
  - 前缀函数  
- KMP
  - next数组
  - next+
  - KMP匹配过程

前言：数据结构课的老师认为这门课【最难不过KMP】。所以专门为KMP写一篇。

kmp是用来解决【字符串匹配问题】的算法。给你2个字符串T、P，问T是否包含P。

由于问题涉及字符串，先理解一些字符串的概念。

# 字符串相关概念：

- 子串：`S[i...j], i <= j`
- 前缀：从第一个字母开始的子串。

  - 真前缀：不能是字符串本身
- 后缀：以最后一个字母结尾的子串。

  - 真后缀：同上。
- 子序列：提出若干元素，不改变相对位置，可以不连续

前缀函数(next函数)：一个长度为 字符串长度 的数组。p[i]为：子串s[0..i]的最长的相等的真前缀与真后缀的长度。

- 真前缀 = 真后缀。eg abcdabc：abc = abc
- 最长的。eg abababab：ababab = ababab
- 真前缀的长度。如果没有相等的真前后缀，为0。

问题：给定一个字符串，求前缀函数：

```cpp
int* prefix(string s){
    int* p = new int[N];
    int len = s.size();
    for(int i = 1; i < len; i++){
        int j = p[i - 1]; // 增加一个字符后，j最大为 p[i-1]+1
  
        // 若新增的字符s[i]和 上一轮循环的相等真前缀（也是目前最长的）的最后一个字符 不同，就将j退回到上上次循环
        while(j > 0 && s[i] != s[j]) j = p[j - 1]; // 为什么是p[j-1]而不是p[i--]
  
        // j若退回到0，且字符相等，则应为1；若j没有被退回到0，说明s[i]!=s[j]，if为假
        if(s[i] == s[j])j++; 
        p[i] = j;
    }
    return p;
}
```

# KMP

最直接的想法是将T、P的每一位进行匹配，失配时再从P的开头开始匹配。

当`P[i]`(假设`i>0`)失配时，我们其实得到了一个重要信息：`P[0, i - 1]`是匹配的。如果我们研究一下`P[0, i - 1]`这一段字符串的性质，是否就不用每次回到P的开头重新匹配呢？

例如：
```
T:  ababaeba
P:  abababa
idx 0123456
         x
```
`P[5]`失配，sad！但`P[0, 4]`是匹配的，即`P[0, 4] == T[0, 4]`。
而且在`str = P[0, 4] = "ababa"`中，`str[0, 2]`和`str[2, 4]`相等。我们不妨利用这一性质，将指向P的指针回退到`3`：
```
T:  ababa e ba
    01234 5 67
P:    aba b aba
      012 3 456
```
我们发现，由于`str[2,4]==P[2,4]==T[2,4]`且`str[0,2]==str[2,4]`，得到`str[0,2]==T[2,4]`，我们不用再重复去比较`str[0,2]`和`T[2,4]`，因为它们相等已经是确定的了！

这一位置我们用next数组来记录，如果我们计算出所有`P[i]`的这一位置并存储到next数组中，那么匹配时就不用回到P的开头重新匹配，而是直接回到`next[i]`开始匹配。

## next

> 定义：`next[i]`表示当`P[i]`失配时，`i`应该回退到的位置。

next的性质如下：`next[i]` = `P[0,i-1]`的最大相等真前缀、真后缀的长度。

<img width="702" alt="image" src="https://github.com/user-attachments/assets/1f99bd3c-f8c3-4323-b275-8eb9c8e7662b">


为什么失配时，令`i = next[i]`

<img width="714" alt="image" src="https://github.com/user-attachments/assets/1ec5e13b-39a1-4e09-a5ce-e2e87c536f5e">


如何计算next：
<img width="702" alt="image" src="https://github.com/user-attachments/assets/39dcb497-1642-4059-afa0-c3917c9f7780">


```cpp
void calc_next(string P){
   int i = 0, j = -1, len = P.size();
   ne[0] = -1;
   while(i < len){
      if(j == -1 || equal(P[j], P[i])){
         // 计算ne[i+1]
         i++, j++; 
         ne[i] = j;
      }
      else j = ne[j];
   }
}
```

## kmp匹配过程

```c
void kmp(string t, string p){
   int p1 = 0, p2 = 0, len_t = t.size(), len_p = p.size();
   while(p1 < len_t){
      while(p2 != -1 && t[p1] != p[p2]) p2 = ne[p2]; // 失配，回退P的指针
      if(p2 == -1 || t[p1] == p[p2]){ // 匹配，继续比较下一位置
         p1++, p2++;
      }
      if(p2 == len_p){
         cout << p1 - len_p << ' ';
         int back = p2 - 1 - ne[p2 - 1];
         p2 = ne[p2 - 1];
         p1 -= back; // 处理子串重叠的情况。e.g. aaa in aaaaaaa
      }
   }
}
```

## next+

我们还可以优化next数组。

例如，当P、next分别为为 `a a a a a a a b, next = {-1 0 1 2 3 4 5 6}` 时，若在`P[6] = a`失配，会重复进行 `i = ne[i]`，即`i = ne[6] = 5, i = ne[5] = 4..., i = ne[0] = -1`，才能到达第一个不是a的位置。

optimized_next: {-1 -1 -1 -1 -1 -1 -1 6}，使得p[6] = a匹配失败时可以跳过前面的 a，直达第一个不是a的位置。

再例如，`aba`, `p[2] = a`匹配失败, `ne[2] = 0`，`p[0] = a`，已经知道`a`一定失配，就不用再匹配了。可以将`ne[2]` 改为 `-1`

再例如：
```
          a   b  a  b   a  b  c  a
    ne    -1  0  0  1   2  3  4  0
    ne+   -1  0 -1  0  -1  0  4  -1 
```

## 时间复杂度

O(n+m)。


