# Java基础系列-数据类型
## 概述
Java有8中基本数据类型：
- byte
- short
- char
- int
- long
- float
- double
- boolean
## byte(字节型)
    byte为8位的有符号整数，范围：-128到127（-2的7次方到2的7次方-1）。
    其包装类型为Byte，其中的缓存ByteCache中可以直接获取byte所有的值。
## short(短整形)
    short为16位的有符号整数，即占用两个字节，范围：-32768到32767（-2的15次方到2的15次方-1）。
    其包装类型为Short，其中的缓存ShortCache中可直接获取-128到127之内的值。
## char(字符型)
    char为16位的无符号整数，代表的是Unicode字符，这些字符正好与0-65535对应，遂用整数来表示这些字符。
    其包装类型为Character，这个类很重要。