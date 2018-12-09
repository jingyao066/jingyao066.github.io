---
title: IO
tags: java基础
date: 2018-12-09 22:46:16
---

### IO流总结

所有的IO操作都在java.io包之中进行定义，而且整个java.io包实际上就是五个类和一个接口：
• 五个类：File、InputStream、OutputStream、Reader、Wirter；
• 一个接口：Serializable。

###  文件操作类:File
```
File file = new File("D:/demo.txt"); // 文件的路径
    if (file.exists()) { // 文件存在
    file.delete(); // 删除文件
} else { // 文件不存在
    file.createNewFile(); // 创建新文件
}
```

在windows操作系统之中，使用“\”作为路径分隔符，而在linux系统下使用“/”作为路径的分隔符，
File.separator这个常量，windows中表示\，unix中表示/
所以写路径可以这么写
`File file = new File("D:" + File.separator + "demo.txt"); // 文件的路径`

更加完善的写法
```
File file = new File("D:" + File.separator + "hellodemo"
        + File.separator + "my" + File.separator + "test"
        + File.separator + "demo.txt"); // 文件的路径
    if (!file.getParentFile().exists()) { // 父路径不存在
    file.getParentFile().mkdirs(); // 创建目录
}
    if (file.exists()) { // 文件存在
    file.delete(); // 删除文件
} else { // 文件不存在
    file.createNewFile(); // 创建新文件
}
```
注：
mkdir()
只能在已经存在的目录中创建创建文件夹。
mkdirs()
可以在不存在的目录中创建文件夹。
		
除此之外，还有一些其他方法可以返回文件的基本信息
```
取得文件的名称：public String getName();
给定的路径是否是文件夹：public boolean isDirectory();
给定的路径是否是文件：public boolean isFile();
是否是隐藏文件：public boolean isHidden();
文件的最后一次修改日期：public long lastModified();
取得文件大小：public long length()，是以字节为单位返回的
```

其他方法:(都写在文档里)
	1.文件重命名
	2.列出指定目录内容

### 二、字节流和字符流
使用File类执行的所有操作都是针对于文件本身，但是却没有针对于文件的内容,要针对文件内容
字节操作流：OutputStream、InputStream
字符操作流：Writer、Reader
	
1.字节输出流：OutputStream
```
1.使用OutputStream向文件中输出数据:
//定义文件路径
File file = new File("D:"+File.separator+"test.txt");
    if (!file.exists()) {
    file.createNewFile();
}
OutputStream out = new FileOutputStream(file);//此处还可添加一个布尔值，表示是否续写,默认false
String data = "Hello world";
    //将数据变为字节数组输出，此处还可加两个参数,如0,5，表示从数据的0位置开始写，写到第五位
    out.write(data.getBytes());
    out.close();
```

2.字节输入流：InputStream
```
//从本地文件读数据(一次性全部读取)
    File file = new File("D:"+File.separator+"test.txt");
		if (file.exists()) {
        InputStream in = new FileInputStream(file);
        byte data[] = new byte[1024];
        int len = in.read(data);
        in.close();
        System.out.println(new String(data,0,len));
    }
```
3.字符输出流:Write
```
File file = new File("D:"+File.separator+"test.txt");
    if (!file.exists()) {
    file.createNewFile();
}
Writer out = new FileWriter(file);
String data = "Hello";
    out.write(data);
    out.close();
```
在Wirter类之中比OutputStream类最为方便的一点就是其可以直接使用String型数据输出，并且不再需要将其变为字节数组了

4.字符输入流:Reader
```
//字符比字节的好处就是在于字符串数据的支持上，而这个好处还只是在Writer类中体现
File file = new File("D:"+File.separator+"test.txt");
    if (file.exists()) {
    Reader in = new FileReader(file);
    char data[]  = new char[1024];
    int len = in.read(data);
    System.out.println(new String(data,0,len));
    in.close();
}
```
		
### 字节流和字符流的区别:
```
对于电脑磁盘或者是网络数据传输上，使用最多的数据类型都是字节数据，
包括图片、音乐、各种可执行程序也都是字节数据，很明显，字节数据要比字符数据更加的广泛，
但是在进行中文处理的过程之中，字符流又要比字节流方便许多，所以如果要使用的话，
首先考虑的是字节流（还有一个原因是因为字节流和字符流的代码形式类似），如果有中文处理的问题，才会考虑使用字符流。
```
```
主要的区别：
   • 字节流没有使用到缓冲区，而字符流使用了
   • 处理各种数据都可以通过字节流完成，而在处理中文的时候使用字符流会更好。
```

### IO流分类
```
根据处理数据类型的不同分为：字符流和字节流
根据数据流向不同分为：输入流和输出流
```

### 字符流和字节流
字符流的由来:因为数据编码的不同，而有了对数据进行高效操作的流对象，本质就是基于字节流读取时，去查了指定的码表。

区别:
读写单位的不同:字节流以字节为单位，字符流以字符为单位，根据码表映射字符，一次可能读多个字节
处理对象的不同:字节流能够处理所有类型的数据，而字符流只能处理字符类型的数据。

结论:只要处理纯文本数据，优先使用字符流，除此之外都使用字节流

输入流和输出流：
对输入流只能进行读操作，对输出流只能进行写操作，需要根据传输数据的不同特性而使用不通过的流

### java IO流对象
```
1.字节输入流：InputStream
	.InputStream是所有输入字节流的父类，它是一个抽象类
	.ByteArrayInputStream	-->byte[]数组
	.StringBufferInputStream-->StringBuffer
	.FileInputStream 		--->本地文件		
	.ObjectInputStream 和所有FilterInputStream 的子类都是装饰流（装饰器模式的主角）
```
```
2.字节输出流：OutputStream
	.OutputStream是所有输出字节流的父类，是一个抽象类
	.ByteArrayOutputStream -->byte[]数组
	.FileOutputStream  --->本地文件
	.ObjectOutputStream 和所有FilterOutputStream 的子类都是装饰流
```
```
字符输入流：Reader
	.Reader是所有字符输入流的父类，是一个抽象类
	.CharReader、StringReader 是两种基本的介质流，它们分别将Char 数组、String中读取数据。
	.BufferedReader 很明显就是一个装饰器，它和其子类负责装饰其它Reader 对象
	.FilterReader 是所有自定义具体装饰流的父类，其子类PushbackReader 对Reader 对象进行装饰，会增加一个行号
	.InputStreamReader 用于连接字节流和字符流，它将字节流转变为
```

### 字符流 
```
字符输出流:Writer
	.Writer是所有输出字符流的父类，是一个抽象类
	.CharArrayWriter、StringWriter 是两种基本的介质流，它们分别向Char 数组、String 中写入数据。
	.BufferedWriter 是一个装饰器为Writer 提供缓冲功能。
	.OutputStreamWriter 将OutputStream 转换为Writer,它的子类FileWriter就是实现此功能的具体类
```

1、判断文件是否存在，不存在创建文件
```
File file=new File("C:\\Users\\wangmeng\\Desktop\\1.txt");    
if(!file.exists())    
{    
    try {    
        file.createNewFile();    
    } catch (IOException e) {    
        // TODO Auto-generated catch block    
        e.printStackTrace();    
    }    
} 
```	

2、判断文件夹是否存在，不存在创建文件夹
```
File file =new File("C:\\Users\\wangmeng\\Desktop\\Dir");    
//如果文件夹不存在则创建    
if  (!file .exists()  && !file .isDirectory())      
{       
    System.out.println("//不存在");  
    file .mkdir();    
} else   
{  
    System.out.println("//目录存在");  
}
```

3.希望往这个文件内写入数据，需要获得输出流 
```
FileOutputStream outputStream = new FileOutputStream(file,true);
//输出内容
outputStream.write('c');
//写完后需要关闭流
outputStream.close();

//输入流的创建,输入流用来读取文件，输出流用来写入内容
FileInputStream inputStream = new FileInputStream(file);
for (int i = 0; i < file.length(); i++) {
    System.out.println((char)inputStream.read());
}
inputStream.close();
//我们每次读取时候，都只能读取一个字节  	接下来建立缓冲区，每次读取更多数据
byte[] bytes = new byte[1024];
FileInputStream inputStream2 = new FileInputStream("C:\\Users\\wangmeng\\Desktop\\1.txt");
int num= inputStream2.read(bytes);
System.out.println(num+"~~~"+Arrays.toString(bytes));
```

### 字符流读取数据
```
FileReader fileReader = new FileReader("/Users/zhangcheng/Desktop/12.txt");
int num = 0;
    while ((num = fileReader.read()) != -1) {
    System.out.println((char) num);
    // System.out.println(num);//第一个20170

}
fileReader.close();
```

### 字符流写入数据
```
FileWriter fileWriter = new FileWriter("/Users/zhangcheng/Desktop/12.txt", true);
fileWriter.write("今天雾霾很重，请大家注意健康\r\n");
fileWriter.close();
```

### 通过javaHttp链接将网络上的图片下载到本地
```
import java.io.ByteArrayOutputStream;  
import java.io.File;  
import java.io.FileOutputStream;  
import java.io.InputStream;  
import java.net.HttpURLConnection;  
import java.net.URL;  
/** 
 * @说明 从网络获取图片到本地 
 * 
 * @version 1.0 
 * @since 
 */  
public class GetImage {  

    /** 
     * 测试 
     * @param args 
     */  
    public static void main(String[] args) {  
        String url = "http://www.baidu.com/img/baidu_sylogo1.gif";  
        byte[] btImg = getImageFromNetByUrl(url);  
        if(null != btImg && btImg.length > 0){  
            System.out.println("读取到：" + btImg.length + " 字节");  
            String fileName = "123.gif";  
            writeImageToDisk(btImg, fileName);  
        }else{  
            System.out.println("没有从该连接获得内容");  
        }  
    }  
    /** 
     * 将图片写入到磁盘 
     * @param img 图片数据流 
     * @param fileName 文件保存时的名称 
     */  
    public static void writeImageToDisk(byte[] img, String fileName){  
        try {  
            File file = new File("D:\\test\\" + fileName);
            FileOutputStream fops = new FileOutputStream(file);  
            fops.write(img);  
            fops.flush();  
            fops.close();  
            System.out.println("图片已经写入");  
        } catch (Exception e) {  
            e.printStackTrace();  
        }  
    }  
    /** 
     * 根据地址获得数据的字节流 
     * @param strUrl 网络连接地址 
     * @return 
     */  
    public static byte[] getImageFromNetByUrl(String strUrl){  
        try {  
            URL url = new URL(strUrl);  
            HttpURLConnection conn = (HttpURLConnection)url.openConnection();  
            conn.setRequestMethod("GET");  
            conn.setConnectTimeout(5 * 1000);  
            InputStream inStream = conn.getInputStream();//通过输入流获取图片数据  
            byte[] btImg = readInputStream(inStream);//得到图片的二进制数据  
            inStream.close();
            return btImg;  
        } catch (Exception e) {  
            e.printStackTrace();  
        }  
        return null;  
    }  
    /** 
     * 从输入流中获取数据 
     * @param inStream 输入流 
     * @return 
     * @throws Exception 
     */  
    public static byte[] readInputStream(InputStream inStream) throws Exception{  
        ByteArrayOutputStream outStream = new ByteArrayOutputStream();  
        byte[] buffer = new byte[1024*1024];  
        int len = 0;  
        while( (len=inStream.read(buffer)) != -1 ){  
            outStream.write(buffer, 0, len);  
        }  
        inStream.close();  
        return outStream.toByteArray();  
    }  
}  
```
