# Java 的数据类型

## Primitive data type（基本数据类型）
- Integer types 
  - byte
  - short 
  - int 
  - long
- Floating point types 
  - float 
  - double 
  

| Data Type | Size | Range |
|------- | ------- | ---------|
byte |  1 byte | -128 ~ 127
short | 2 bytes | -32,768 ~ 32,767
int  | 4 bytes | -2,147,483,648 ~ 2,147,483,647  
long | 8 bytes | -9,223,372,036,854,775,807 ~ 9,223,372,036,854,775,807
float | 4 bytes | 精度范围为小数点后 6~7 位  
double | 8 bytes | 精度范围为小数点后 15 位
boolean | 1 bit | true/false
char | 2 bytes | character/letter or ASCII values

### Type Casting 
- Widening Casting (automatically) - coverting a smaller type to a larger type size
  - byte -> short -> char -> int -> long -> float -> double
- Narrowing Casting (manually) - converting a larger type to a smaller type size
  - double -> float -> long -> int -> char -> short -> byte

### Precision Loss
![Precision Loss](/images/TypeCasting.png)

实线代表合法转换即无信息丢失的转换，虚线表示转换可能存在精度丢失问题。


## Non-primitive data type（引用数据类型）
- String
- Arrays 
- Classes 
