# String 
Immutable Class 

## 字符串拼接

### +
底层实现原理其实是 StringBuilder.append 

### s.concat 
```java
public String concat(String str) {
  int otherLen = str.length();
  if (otherLen == 0) {
    return this;
  }
  int len = value.length;
  char buf[] = Arrays.copyOf(value, len + otherLen);
  str.getChars(buf, len);
  return new String(buf, true);
}
```
首先创建了一个字符数组，长度是已有字符串和待拼接字符串的长度之和，再把两个字符串的值复制到新的字符数组中，并使用这个字符数组创建一个新的 String 对象并返回。

经过 concat 方法，其实是 new 了一个新的 String，体现了字符串的不变性。

### StringBuilder.append()

### StringBuffer.append()

### StringUtils.join

**循环体内，字符串的连接方式，使用 StringBuilder 的 append 方法进行扩展，而不要使用 +。**

