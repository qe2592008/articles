# Java基础系列-时间日期API
## 一、概述
Java提供了有关时间的类和API，可以很方便的处理日期时间。

JDK 1.8之前使用的是Date和Calendar，JDK 1.8之后使用DateTime，前者毛病较多，后者更加适用。
## 二、基础知识
### 2.1 时区
我们都知道，地球是圆的，并且在不停的自转与公转，自转一圈是为一天，公转一圈是为一年。

我们这里关注自转的部分。

圆形的地球总是存在面向太阳的半球与背向太阳的半球，面向太阳的半球为白天，背向太阳的半球为黑夜，边界区域为凌晨和傍晚。

那么在这些处于地球不同位置的地区的时间也怎么分配的呢？

如果我们规定一个统一的时间，让全球共用。这样的话，0点时分，全球将拥有各种不同的景象，有的在凌晨，有的在早晨，有的在中午，有的在下午，有的在晚上等等，这也无关大雅。

但是当一个人从一个地方到另外一个地方后，他自认为的时间与光景将不再对应，这就是时差。

举个例子，比如规定中国晚上0点为统一的0点时间，而中国和美国都是在白天工作，晚上休息，但是中国是白天的时候，美国正好是晚上，一个人从中国到达美国之后，看到的就是人们在0点上班，12点休息，这与人的行为习惯相悖!

所以有了时区的概念，将全球以经线均分为24个时区，相邻时区的时间相差一个小时，当你跨越时区时，只要将你的手表调快或调慢一个小时即可，还是满足你的0点休息，12点上班的行为习惯。

实际上我们并没有绝对的使用标准的时区来划分时间，比如中国，横跨5个时区，为了国内的时间统一，我们规定统一使用北京时间为中国的时间。
### 2.2 格林威治时间
有了时区的概念，那么就有一个标准时间的问题了，以哪个地方为标准来定义时间呢？

那就是格林威治时间了，即GMT（Greenwich Mean Time）。

GMT表示的是格林威治平时，是指位于英国伦敦郊区的皇家格林尼治天文台当地的平太阳时，因为本初子午线被定义为通过那里的经线。

自1924年2月5日开始，格林尼治天文台负责每隔一小时向全世界发放调时信息。国际天文学联合会于1928年决定，将由格林威治平子夜起算的平太阳时作为世界时，也就是通常所说的格林威治时间。

比如中国北京位于东八区，所以北京时间即为GMT+8，即北京时间比格林威治平时快8个小时。
### 2.3 时间戳
时间戳是指格林威治时间1970年01月01日00时00分00秒起至现在的总秒数。

这个值是全球固定值，全球通用，虽然各地的时间不同，但是时间戳是一致的。

有了时间戳，我们就是实现不同地区时间的转换。
### 2.4 
## 三、Date/Calendar/SimpleDateFormat
### 3.1 Date
关于Date，请查看[java基础系列--Date类](https://www.cnblogs.com/V1haoge/p/7126930.html)
### 3.2 Calendar
关于Calendar，请查看[java基础系列--Calendar类](https://www.cnblogs.com/V1haoge/p/7136575.html)
### 3.3 SimpleDateFormat
SimpleDateFormat是JDK提供的用于实现Date与String之间相互转换的工具类，成为时间格式化工具。

我们来看一下简单的格式化方式：
```java
public class SimpleDateFormatTest {
    public static void main(String[] args) throws ParseException {
        test();
    }

    public static void test() throws ParseException {
        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        // 格式化-将Date格式化输出
        Date date = new Date();
        System.out.println(sdf.format(date));
        // 解析-将String解析为Date
        String dateStr = "2019-01-01 12:12:12";
        System.out.println(sdf.parse(dateStr));
    }
}
```
执行结果为：
```text
2019-02-20 18:01:30
Tue Jan 01 12:12:12 CST 2019
```
最后输出了“Tue Jan 01 12:12:12 CST 2019”格式的字符串是因为使用System.out.println()方式输出自动调用了Date的toString方法。

上面的东西就是基础，重点关注的是下面的内容。

SimpleDateFormat是非线程安全的类，如果将其作为共享变量使用，在多线程环境中将会发生线程安全问题。

下面是推荐的处理办法：
1. 作为局部变量使用
2. 加锁
3. 使用ThreadLocal，这也是阿里巴巴java手册中推荐的方式。
```java
public static ThreadLocal<SimpleDateFormat> sdfThreadLocal = new ThreadLocal<SimpleDateFormat>(){
    @Override
    protected SimpleDateFormat initialValue() {
        return new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
    }
};
```
4. 更彻底的方式是使用DateTimeFormatter来替换。

这里只补充一点关于时区的内容：

Date其实保存的就是时间戳，我们可以通过设定时区来获取不同地点的当前时间。

```java
public class DateTest {
    public static void main(String[] args) {
        Date date1 = new Date(0);// 格林威治1970年1月1日0点0分0秒
        System.out.println("格林威治0时的时间戳：" + date1.getTime());
        System.out.println("格林威治0时的中国时区时间：" + date1);
        Date date2 = new Date();// 当前时间戳
        System.out.println("date2的时间戳：" + date2.getTime());
        System.out.println("date2表示的中国时区时间:" + date2);
        TimeZone.setDefault(TimeZone.getTimeZone("America/Los_Angeles"));// 设置时区为美国洛杉矶时区
        System.out.println("date2表示的美国洛杉矶时区时间：" + date2);
        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        sdf.setTimeZone(TimeZone.getTimeZone("America/New_York"));// 设置为美国纽约时区
        System.out.println("date2表示的美国纽约时区时间：" + sdf.format(date2));
    }
}
```
执行结果为：
```text
格林威治0时的时间戳：0
格林威治0时的中国时区时间：Thu Jan 01 08:00:00 CST 1970
date2的时间戳：1550656106538
date2表示的中国时区时间:Wed Feb 20 17:48:26 CST 2019
date2表示的美国洛杉矶时区时间：Wed Feb 20 01:48:26 PST 2019
date2表示的美国纽约时区时间：2019-02-20 04:48:26
```
可见中国时区的时间确实是比格林威治时间快8个小时，而美国洛杉矶时区的时间则是比格林威治时间慢8个小时，美国纽约时区的时间又比洛杉矶快3个小时。

如果不指定时区，则自动获取操作系统的默认时区。
## 四、DateTime/DateTimeFormatter

## 五、总结

参考：
- [关于Java中的时间处理，你真的了解吗？](https://www.hollischuang.com/archives/3082)