# Java基础系列-序列化与反序列化
## 序列化简介
&#160;&#160;&#160;&#160;&#160;&#160;&#160;在项目中有很多情况需要对实例对象进行序列化与反序列化，这样可以持久的保存对象的状态，甚至在各个组件之间进行对象传递和远程调用。序列化机制是项目中必不可少的常用机制。  
&#160;&#160;&#160;&#160;&#160;&#160;&#160;要想一个类拥有序列化、反序列化功能，最简单的方法就是实现java.io.Serializable接口，这个接口是一个标记接口（marker Interface），即其内部无任何字段与方法定义。  
&#160;&#160;&#160;&#160;&#160;&#160;&#160;当我们定义了一个实现Serializable接口的类之后，一般我们会手动在类内部定义一个private static final long serialVersionUID字段，用来保存当前类的序列版本号。这样做的目的就是唯一标识该类，其对象持久化之后这个字段将会保存到持久化文件中，当我们对这个类做了一些更改时，新的更改可以根据这个版本号找到已持久化的内容，来保证来自类的更改能够准确的体现到持久化内容中。而不至于因为未定义版本号，而找不到原持久化内容。  
&#160;&#160;&#160;&#160;&#160;&#160;&#160;当然如果我们不实现Serializable接口就对该类进行序列化与反序列化操作，那么将会抛出java.io.NotSerializableException异常。  
&#160;&#160;&#160;&#160;&#160;&#160;&#160;如下例子：
```java
public class Student implements Serializable {
    private static final long serialVersionUID = -3111843137944176097L;
    private String name;
    private int age;
    private String sex;
    private String address;
    private String phone;
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public int getAge() {
        return age;
    }
    public void setAge(int age) {
        this.age = age;
    }
    public String getSex() {
        return sex;
    }
    public void setSex(String sex) {
        this.sex = sex;
    }
    public String getAddress() {
        return address;
    }
    public void setAddress(String address) {
        this.address = address;
    }
    public String getPhone() {
        return phone;
    }
    public void setPhone(String phone) {
        this.phone = phone;
    }
}
```
## 序列化的使用
&#160;&#160;&#160;&#160;&#160;&#160;&#160;虽然要实现序列化只需要实现Serializable接口即可，但这只是让类的对象拥有可被序列化和反序列化的功能，它自己并不会自动实现序列化与反序列化，我们需要编写代码来进行序列化与反序列化。  
&#160;&#160;&#160;&#160;&#160;&#160;&#160;这就需要使用ObjectOutputStream类的writeObject()方法与readObject()方法，这两个方法分别对应于将对象写入到流中（序列化），从流中读取对象（反序列化）。  
&#160;&#160;&#160;&#160;&#160;&#160;&#160;Java中的对象序列化，序列化的是什么？答案是对象的状态、更具体的说就是对象中的字段及其值，因为这些值正好描述了对象的状态。  
&#160;&#160;&#160;&#160;&#160;&#160;&#160;下面的例子我们实现将Student类的一个实例持久化到本地文件“D:/student.out”中，并从本地文件中读到内存，这要借助于FileOutputStream和FileInputStream来实现:
```java
public class SerilizeTest {
    public static void main(String[] args) {
        serilize();
        Student s = (Student) deserilize();
        System.out.println("姓名：" + s.getName()+"\n年龄："+ s.getAge()+"\n性别："+s.getSex()+"\n地址："+s.getAddress()+"\n手机："+s.getPhone());
    }
    public static Object deserilize(){
        Student s = new Student();
        InputStream is = null;
        ObjectInputStream ois = null;
        File f = new File("D:/student.out");
        try {
            is = new FileInputStream(f);
            ois = new ObjectInputStream(is);
            s = (Student)ois.readObject();
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }finally{
            if(ois != null){
                try {
                    ois.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if(is != null){
                try {
                    is.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
        return s;
    }
    
    public static void serilize() {
        Student s = new Student();
        s.setName("张三");
        s.setAge(32);
        s.setSex("man");
        s.setAddress("北京");
        s.setPhone("12345678910");
//      s.setPassword("123456");
        OutputStream os = null;
        ObjectOutputStream oos = null;
        File f = new File("D:/student.out");
        try {
            os = new FileOutputStream(f);
            oos = new ObjectOutputStream(os);
            oos.writeObject(s);
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }finally{
            if(oos != null)
                try {
                    oos.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            if(os != null)
                try {
                    os.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
        }
    }
}
```
&#160;&#160;&#160;&#160;&#160;&#160;&#160;通过以上的代码就可以实现简单的对象序列化与反序列化。
&#160;&#160;&#160;&#160;&#160;&#160;&#160;执行结果:
```text
姓名：张三
年龄：32
性别：man
地址：北京
手机：12345678910
```
&#160;&#160;&#160;&#160;&#160;&#160;&#160;这里将writeObject的调用栈罗列出来：
```text
writeObject->writeObject0->writeOrdinaryObject->writeSerialData->defaultWriteFields->writeObject0->...
```
&#160;&#160;&#160;&#160;&#160;&#160;&#160;调用栈最后返回了writeObject0方法，这是使用递归的方式来遍历目标类的字段中所有普通实现Serializable接口的类型字段，将其全部写入流中，最后所有的写入都会在writeObject0方法中终结，这个方法会根据字段的类型来调用响应的write方法进行流写入。  
&#160;&#160;&#160;&#160;&#160;&#160;&#160;Java序列化的是对象的字段，但是这些字段并不一定都是简单的String、或者是Integer之类，可能也是很复杂的类型，一个实现了Serializable接口的类类型，这时候我们序列化的时候，就需要将这个内部的第二层次的对象进行递归序列化，这种嵌套可以有无数层，但是总会有个终结。
## 自定义序列化功能
&#160;&#160;&#160;&#160;&#160;&#160;&#160;上面的内容都是简单又简单，真正要注意的内容在这里，有关自定义序列化策略的内容才是序列化机制中最重要、最复杂的的内容。
### transient关键字的使用
&#160;&#160;&#160;&#160;&#160;&#160;&#160;正如上面所述，Java序列化的的是对象的非静态字段及其值。而transient关键字正是使用在实现了Serializable接口的目标类的字段中，凡是被该关键字修饰的字段，都将被序列化过滤掉，即不会被序列化。  
&#160;&#160;&#160;&#160;&#160;&#160;&#160;将上面的例子中Student类中的phone字段前面加上transient关键字：
```java
public class Student implements Serializable {
    //...
    private transient String phone;
    //...
}
```
&#160;&#160;&#160;&#160;&#160;&#160;&#160;执行结果变为：
```text
姓名：张三
年龄：32
性别：man
地址：北京
手机：null
```
&#160;&#160;&#160;&#160;&#160;&#160;&#160;可见由于phone字段添加了transient关键字，在序列化的时候，其值未进行序列化，反序列化回来之后其值将会是null。
### writeObject方法的使用
&#160;&#160;&#160;&#160;&#160;&#160;&#160;writeObject()是在ObjectOutputStream中定义的方法，使用这个方法可以将目标对象写入到流中，从而实现对象序列化。但是Java为我们提供了自定义writeObject()方法的功能，当我们在目标类中自定义writeObject()方法之后，将会首先调用我们自定义的方法，然后在继续执行原有的方法步骤（使用defaultWriteObject方法）。这样的功能为我们在对象序列化之前可以对对象的字段进行有一些附加操作，最为常用的就是针对一些需要保密的字段（比如密码字段），进行有效的加密措施，保证持久化数据的安全性。  
&#160;&#160;&#160;&#160;&#160;&#160;&#160;这里我对Student类添加password字段，和对应的set和get方法。
```java
public class Student implements Serializable {
    //...
    private String password;
    public String getPassword() {
        return password;
    }
    public void setPassword(String password) {
        this.password = password;
    }
}
```
&#160;&#160;&#160;&#160;&#160;&#160;&#160;然后在Student类中定义writeObject()方法：
```java
public class Student implements Serializable {
    //...
    private void writeObject(ObjectOutputStream oos) throws IOException{
        password = Integer.valueOf(Integer.valueOf(password).intValue() << 2).toString();
        oos.defaultWriteObject();
    }
}
```
&#160;&#160;&#160;&#160;&#160;&#160;&#160;这里我对密码字段的值以左移两位的方式进行简单加密，然后调用ObjectOutputStream中的defaultWriteObject()方法来返回原来的序列化执行步骤。具体的调用栈如下：
```text
writeObject->writeObject0->writeOrdinaryObject->writeSerialData->invokeWriteObject->invoke（调用自定义的writeObject）->defaultWriteObject->defaultWriteFields->writeObject0->...
```
&#160;&#160;&#160;&#160;&#160;&#160;&#160;在目标类中增加writeObject方法之后，我们通过上面的调用栈可以看到，调用顺序会在writeSerialData这里发生转折，执行invokeWriteObject方法，调用目标类中的writeObject方法，然后再经过defaultWriteObject方法重回原来的步骤，这表明自定义的writeObject方法操作会优先执行。  
&#160;&#160;&#160;&#160;&#160;&#160;&#160;这样设置之后，序列化完成后，保存到文件中的将会是加密后的密码值，我们结合下一个内容readObject方法进行测试。
### readObject方法的使用
&#160;&#160;&#160;&#160;&#160;&#160;&#160;该方法是与writeObject方法相对应的，是用于读取序列化内容的方法，用于反序列化过程中。类似于writeObject方法的自定义，我们进行readObject方法的自定义：
```java
public class Student implements Serializable {
    //...
    private void readObject(ObjectInputStream ois)throws IOException, ClassNotFoundException{
        ois.defaultReadObject();
        if(password != null)
            password = Integer.valueOf(Integer.valueOf(password).intValue() >> 2).toString();
    }
}
```
&#160;&#160;&#160;&#160;&#160;&#160;&#160;在测试程序中添加密码字段：
```java
public class SerilizeTest {
    public static void main(String[] args) {
        serilize();
        Student s = (Student) deserilize();
        System.out.println("姓名：" + s.getName()+"\n年龄："+ s.getAge()+"\n性别："+s.getSex()+"\n地址："+s.getAddress()+"\n手机："+s.getPhone()+"\n密码："+s.getPassword());
        
    }
    //...
    public static void serilize() {
        Student s = new Student();
        s.setName("张三");
        s.setAge(32);
        s.setSex("man");
        s.setAddress("北京");
        s.setPhone("12345678910");
        s.setPassword("123456");
        OutputStream os = null;
        ObjectOutputStream oos = null;
        File f = new File("D:/student.out");
        //...
    }
}
```
&#160;&#160;&#160;&#160;&#160;&#160;&#160;执行程序结果为：
```text
姓名：张三
年龄：32
性别：man
地址：北京
手机：null
密码：123456
```
&#160;&#160;&#160;&#160;&#160;&#160;&#160;这里的密码经过了序列化时的加密与反序列化时的加密操作，由于前后结果一致，无法看出变化，简单的做法就是将解密算法改变：
```java
public class Student implements Serializable {
    //...
    private void readObject(ObjectInputStream ois)throws IOException, ClassNotFoundException{
        ois.defaultReadObject();
        if(password != null)
            password = Integer.valueOf(Integer.valueOf(password).intValue() >> 3).toString();
    }
}
```
&#160;&#160;&#160;&#160;&#160;&#160;&#160;这里将解密的算法改为将目标值右移三位，这样就会导致最后获取到的密码值与原设置的“123456”不同。执行结果如下：
```text
姓名：张三
年龄：32
性别：man
地址：北京
手机：null
密码：61728
```
###  writeReplace方法的使用
&#160;&#160;&#160;&#160;&#160;&#160;&#160;Java的序列化并不是dead的，而是非常的灵活，我们甚至可以在序列化的时候改变目标的类型，这就需要writeReplace方法来操作。  
&#160;&#160;&#160;&#160;&#160;&#160;&#160;我们在目标类中自定义writeReplace方法，该方法用于返回一个Object类型，这个Object就是你改变之后的类型，序列化的过程中会判断目标类中是否存在writeObject方法，若存在该方法，就会实行调用，采用该方法返回的类型对象作为序列化的新目标对象。  
&#160;&#160;&#160;&#160;&#160;&#160;&#160;现在我们在Student类中自定义writeReplace方法：
```java
public class Student implements Serializable {
    //...
    private Object writeReplace() throws ObjectStreamException{
        StringBuffer sb = new StringBuffer();
        String s = sb.append(name).append(",").append(age).append(",").append(sex).append(",").append(address).append(",").append(phone).append(",").append(password).toString();
        return s;
    }
}
```
&#160;&#160;&#160;&#160;&#160;&#160;&#160;通过自定义的writeReplace方法将目标类中的数据整合转化为一个字符串，并将这个字符串作为新目标对象进行序列化。
&#160;&#160;&#160;&#160;&#160;&#160;&#160;执行之后会报错：
```text
Exception in thread "main" java.lang.ClassCastException: java.lang.String cannot be cast to xuliehua.Student
    at xuliehua.SerilizeTest.deSerilize(SerilizeTest.java:32)
    at xuliehua.SerilizeTest.main(SerilizeTest.java:20)
```
&#160;&#160;&#160;&#160;&#160;&#160;&#160;提示在反序列化时，字符串类型不能强转为Student类型，这说明，我们保存到文件中的序列化内容为字符串类型，也就是说我们自定义的writeReplace方法起作用了。  
&#160;&#160;&#160;&#160;&#160;&#160;&#160;现在我们来对反序列化方法进行些许修改，来准确的获取序列化的内容。
```java
public class SerilizeTest {
    public static void main(String[] args) {
        serilize();
        Student s = (Student) deserilize();
//        System.out.println("姓名：" + s.getName()+"\n年龄："+ s.getAge()+"\n性别："+s.getSex()+"\n地址："+s.getAddress()+"\n手机："+s.getPhone()+"\n密码："+s.getPassword());
        System.out.println(s);
    }
    //...
    public static String deSerilize(){
//      Student s = new Student();
        String s = "";
        InputStream is = null;
        ObjectInputStream ois = null;
        File f = new File("D:/student.out");
        try {
            is = new FileInputStream(f);
            ois = new ObjectInputStream(is);
//            s = (Student)ois.readObject();
            s = (String)ois.readObject();
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }finally{
            if(ois != null){
                try {
                    ois.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if(is != null){
                try {
                    is.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
        return s;
    }
}
```
&#160;&#160;&#160;&#160;&#160;&#160;&#160;再来执行一下：
```text
张三,32,man,北京,12345678910,123456
```
&#160;&#160;&#160;&#160;&#160;&#160;&#160;准确获取序列化内容。  
&#160;&#160;&#160;&#160;&#160;&#160;&#160;这里需要注意一点，当我们使用这种方式来改变目标对象类型后，原本类型中标识为transient的字段的过滤功能将会失效，因为我们序列化的目标发生的转移，自然原类型字段上设置的transient不会对新类型起任何作用，就比如此处的phone字段。
### readResolve方法的使用
&#160;&#160;&#160;&#160;&#160;&#160;&#160;与writeReplace方法对应的，我们也可以在反序列化的时候对目标的类型进行更改，这需要使用readResolve方法，使用方式是在目标类中自定义readResolve方法，该方法的返回值为Object对象，即转换的新类型对象。  
&#160;&#160;&#160;&#160;&#160;&#160;&#160;这里我们在3.3 的基础上进行代码修改，首先我们在Student类中自定义readResolve方法：
```java
public class Student implements Serializable {
    //...
//    private Object writeReplace() throws ObjectStreamException{
//        StringBuffer sb = new StringBuffer();
//        String s = sb.append(name).append(",").append(age).append(",").append(sex).append(",").append(address).append(",").append(phone).append(",").append(password).toString();
//        return s;
//    }
    private Object readResolve()throws ObjectStreamException{
        Map<String,Object> map = new HashMap<String,Object>();        
        map.put("name", name);
        map.put("age", age);
        map.put("sex", sex);
        map.put("address", address);
        map.put("phone", phone);
        map.put("password", password);
        return map;
    }
}
```
&#160;&#160;&#160;&#160;&#160;&#160;&#160;在这个方法中我们将获取的数据保存到一个Map集合中，并将这个集合返回。  
&#160;&#160;&#160;&#160;&#160;&#160;&#160;直接执行程序会报错：
```text
 Exception in thread "main" java.lang.ClassCastException: java.util.HashMap cannot be cast to xuliehua.Student
     at xuliehua.SerilizeTest.deSerilize(SerilizeTest.java:32)
     at xuliehua.SerilizeTest.main(SerilizeTest.java:20)
```
&#160;&#160;&#160;&#160;&#160;&#160;&#160;报错说明我们设置的readResolve方法被执行了，因为类型无法进行转化，所以报错，我们作如下修改：
```java
public class SerilizeTest {
    public static void main(String[] args) {
        serilize();
//        Student s = (Student) deserilize();
//        System.out.println("姓名：" + s.getName()+"\n年龄："+ s.getAge()+"\n性别："+s.getSex()+"\n地址："+s.getAddress()+"\n手机："+s.getPhone()+"\n密码："+s.getPassword());
        Map<String,Object> map = deSerilize();
        System.out.println(map);
    }
    //...
    @SuppressWarnings("unchecked")
    public static Map<String,Object> deSerilize(){
        Map<String,Object> map = new HashMap<String,Object>();
//        Student s = new Student();
        InputStream is = null;
        ObjectInputStream ois = null;
        File f = new File("D:/student.out");
        try {
            is = new FileInputStream(f);
            ois = new ObjectInputStream(is);
//            s = (Student)ois.readObject();
            map = (Map<String,Object>)ois.readObject();
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }finally{
            if(ois != null){
                try {
                    ois.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if(is != null){
                try {
                    is.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
        return map;
    }
}
```
&#160;&#160;&#160;&#160;&#160;&#160;&#160;执行结果：
```text
{phone=null, sex=man, address=北京, age=32, name=张三, password=61728}
```
&#160;&#160;&#160;&#160;&#160;&#160;&#160;可见我们可以准确获取到数据，而且是以改变后的类型。  
&#160;&#160;&#160;&#160;&#160;&#160;&#160;注意：writeObject方法与readObject方法可以同时存在，但是一般情况下writeReplace方法与readResolve方法是不同时使用的。因为二者均是基于原类型来进行转换，如果同时存在，那么两个新类型之间是无法进行类型转换的（当然如果这两个类型是存在继承关系的除外），功能无法实现。