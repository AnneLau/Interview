[](https://www.cnblogs.com/xxjcai/)

## [JS中substr与substring的区别](https://www.cnblogs.com/xxjcai/p/10865321.html)

js中substr和substring都是截取字符串中子串，非常相近，可以有一个或两个参数。

语法：substr(start [，length]) 第一个字符的索引是0，start必选 length可选

　　　substring(start [, end]) 第一个字符的索引是0，start必选 end可选

相同点：当有一个参数时，两者的功能是一样的，返回从start指定的位置直到字符串结束的子串

var str = "hello Tony";

str.substr(6); //Tony

str.substring(6); //Tony

 

不同点：有两个参数时

（1）substr(start,length) 返回从start位置开始length长度的子串

“goodboy”.substr(1,6);  //oodboy

【注】当length为0或者负数，返回空字符串

（2）substring(start,end) 返回从start位置开始到end位置的子串（不包含end）

“goodboy”.substring(1,6); //oodbo

【注】:

（1）substring 方法使用 start 和 end 两者中的较小值作为子字符串的起始点

（2）start 或 end 为 NaN 或者负数，那么将其替换为0