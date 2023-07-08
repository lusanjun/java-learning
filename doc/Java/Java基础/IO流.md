## IO流



### 1.流

流，任何有能力产出数据的数据源或者是有能力接受数据的接收者，两者之间的数据传输，就是流。

字节流：数据流中最小的数据单元是字节。

字符流：数据流中最小的数据单元是字符。Java中的字符是Unicode编码，1个字符占用2个字节。

输入流：程序从输入流读取数据源。数据源包括文件、网络等。

输出流：程序向输出流写入数据。程序中的数据输出到外界，显示器、打印机、文件、网络等。

输入（Input）和输出（Output）。

### 2. Java IO类

字符流：

- Reader：BufferedReader、InputStreamReader、StringReader、PipedReader、ByteArrayReader、FilterReader
- Writer：BufferedWriter、OutputStreamWriter、StringWriter、PrinterWriter、CharArrayWriter、FilterWriter

字节流：

- InputStream：FileInputStream、FilterInputStream、ObjectInputStream、PipedInputStream、SequenceInputStream、StringBufferInputStream、ByteArrayInputStream
- OutputStream：FileOutputStream、FilterOutputStream、ObjectOutputStream、PipedOutputStream、ByteArrayOutputStream

### 3. File 类

通过 File 类对文件进行基本操作。

File 类对象代表一个具体的文件路径，可以是绝对路径，也可以是相对路径。也可以是目录。

```java
 File f1 = new File("d:\\a\\b.txt");
 File f1 = new File("b.txt");
 File f1 = new File("d:\\a);
```

#### 3.1 创建文件

在 dir 目录下新建 fil.txt

```java
 File file = new File(new File("dir"),"file.txt");
 file.createNewFile();
```

新建文件：/home/dir/file.txt

```java
 File file = new File(new URI("/home/dir/file.txt"));
 file.createNewFile();
```

#### 3.2 创建目录

相对路径：

```java
 new File("dir").mkdir();
```

绝对路径：

```java
 new File("/home/dir").mkdir();
```

#### 3.3 删除文件

如果删除的是目录，则目录必须为空，如果不为空，要首先删除该目录下的所有文件和目录。

递归删除每一个文件。

```java
   public static void delete (File file){
         File[] listFiles = file.listFiles();
         if (listFiles != null){
             for(File f: listFiles){
                 if(f.isDirectory()){
                     delete(f);
                 }else {
                     f.delete();
                 }
             }
         }
         file.delete();
  }
```

### 4. 字节流

### 4.1. InputStream 和 OutputStream

InputStream 字节输入流，所有输入流的基类。

OutputStream 字节输出流，所有输出流的基类。

#### 4.2. FileInputStream 和 FileOutputStream

以字节为单位读取文件，常用于读二进制文件，图片、声音、影像等。

通过 byte 数组暂存内容。

```java
 public static void readByBytes(String inFile, String outFile) throws IOException {
         File file = new File(inFile);
         InputStream inputStream = null;
         OutputStream outputStream = null;
         byte[] bytes = new byte[100];
         int byteread = 0;
         inputStream = new FileInputStream(file);
         outputStream = new FileOutputStream(outFile);
         while ((byteread = inputStream.read(bytes)) != -1) {
             outputStream.write(bytes, 0, byteread);
         }
         inputStream.close();
         outputStream.close();
     }
```

#### 4.3 DataInputStream和DataOutputStream

DataInputStream 数据输入流，继承于 FilterInputStream。

DataOutputStream 数据输出流，继承于 FilterOutputStream。

允许应用程序以与机器无关方式向底层写入基本Java数据类型。

#### 4.4 BufferedInputStream 和 BufferedOutputStream

BufferedInputStream 缓冲输入流，继承于 FilterInputStream，默认缓冲区是 8M，提高文件读取性能。

BufferedOutputStream 缓冲输出流，继承于 FilterOutputStream，能提高文件的写入性能。

本质上通过一个内部缓冲区数组实现。新建某个流对应的 BufferedInputStream 后，通过 read 方法读取输入流的数据，会将数据分批填入到缓冲区中。每当缓冲区数据读完后，输入流再填充，反复直到读完输入流数据。

```java
 public static void readAndWrite(String inFile,String outFile) throws IOException {
         BufferedInputStream bis = new BufferedInputStream(new FileInputStream(inFile));
         BufferedOutputStream bos = new BufferedOutputStream(new FileOutputStream(outFile));
         byte[] bytes = new byte[1024];
         int len = 0;
         while (-1!=(len = bis.read(bytes,0,bytes.length))){
             bos.write(bytes,0,len);
         }
         bis.close();
         bos.close();
     }
```

#### 4.5 ByteArrayInputStream 和 ByteArrayOutputStream

ByteArrayInputStream 字节组输入流，继承于 InputStream。

ByteArrayOutputStream 字节组输出流，继承于 OutputStream。

从内存中的字节数组中读取数据，数据源是字节数组。

```java
public static byte[] readFromByteArray(byte[] dataSource) throws IOException {
        ByteArrayInputStream in = new ByteArrayInputStream(dataSource);
        ByteArrayOutputStream out = new ByteArrayOutputStream();
        int len = 0;
        byte[] bytes = new byte[1024];
        while (-1!=(len = in.read(bytes,0,bytes.length))){
            out.write(bytes,0,len);
        }
        return out.toByteArray();
    }
```

### 5. 字符流

#### 5.1 Reader 和 Writer

Reader 和 Writer 都是一个抽象类，是以字符为单位的输入流和输出流的基类。用于文本文件，可以读取字符。

#### 5.2 InputStreamReader 和 OutputStreamWriter

InputStreamReader 和 OutputStreamWriter ，将字节流转换为字符流。内部是一个 inputStream 和 outputStream 对象。可以为字节流指定字符集，默认为平台默认的字符集。

```java
    public static void readByChars(String file) throws IOException {
        Reader reader = new InputStreamReader(new FileInputStream(new File(file)));
        //1次读1个字符
        int temchar;
        while (-1!=(temchar=reader.read())){
            System.out.println((char)temchar);
        }
        //1次读多个字符
        char[] chars = new char[50];
        int temchar2;
        while (-1!=(temchar2=reader.read(chars))){
            for(char c:chars){
                System.out.println((char)c);
            }
        }
    }
```

#### 5.3 FileReader 和 FileWriter

FileReader 和 FileWriter分别继承自 InputStreamReader 和 OutputStreamWriter，读取文件。

```java
public static void readAndWrite(String inFile, String outFile) throws IOException {
        FileReader fr = new FileReader(inFile);
        FileWriter fw = new FileWriter(outFile);
        //一次读1个字符，写1个字符
        int c = 0;
        while (-1 != (c = fr.read())) {
            fw.write(c);
        }
        //一次读多个字符，写多个字符
        char[] chars = new char[50];
        int len = 0;
        while (-1 != (len = fr.read(chars))) {
            fw.write(chars, 0, len);
        }
        fr.close();
        fw.close();
    }
```

#### 5.4 BufferedReader 和 BufferedWriter

BufferedReader 缓冲读取流，BufferedWriter缓冲写入流，默认缓冲区是 8M。

BufferedReader 读取文件时，读入字符数据放入缓冲区。read 方法读取时，先从缓冲区读取，缓冲区数据不足时，再读取文件。

BufferedWriter 写入文件时，写入数据先放入缓冲区，缓冲区数据满了，才一次性写入文件。

### 6. IO 管道

IO 中的管道为 JVM 中两个线程提供通信的能力。

线程1使用 PipedOutputStream 输出流，输出数据。线程2使用 PipedInputStream 读取流，读取数据。

```java
    public void test() throws IOException {
        PipedInputStream pipedInputStream = new PipedInputStream();
        PipedOutputStream pipedOutputStream = new PipedOutputStream(pipedInputStream);
        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    pipedOutputStream.write("abc".getBytes());
                    pipedOutputStream.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }).start();
        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    byte[] arr = new byte[8];
                    while (pipedInputStream.read(arr) != -1) {
                        System.out.println(Arrays.toString(arr));
                    }
                    pipedInputStream.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }).start();
    }
```

### 7. RandomAccessFile

RandomAccessFile 不属于输入流和输出流体系的，是一个完全独立的类，既可以读文件，也可以写文件。

RandomAccessFile 的基本功能有：getFilePointer 定位，seek 在文件里移动，length 判断文件大小，skipBytes 跳过多少字节数。

RandomAccessFile 构造方法可以指定模式：

- r，只读方式打开文件；
- rw，读写方式打开文件；
- rws，读写方式打开文件，对内容或元数据的更新都同步写入底层存储设备；
- rwd，读写方式打开文件，对内容的更新同步写入底层存储设备 。

```java
    public static void readFileByRandomAccessFile(String file) throws IOException{
        RandomAccessFile raf = new RandomAccessFile(file,"rw");
        long length = raf.length();
        raf.seek(0);
        byte[] bytes= new byte[50];
        int byteread = 0;
        while (-1!=(byteread=raf.read(bytes))){
            raf.write(bytes,byteread,0);
        }
        raf.close();
    }
```