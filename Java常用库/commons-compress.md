[toc]

# commons-compress的使用

## 基础知识

commons-compress用于打包、压缩

### 依赖

```
	<dependency>
      <groupId>org.apache.commons</groupId>
      <artifactId>commons-compress</artifactId>
      <version>1.21</version>
    </dependency>
```

## 解压

两种解压：顺序解压和随机访问解压，由zip格式本身决定

ZipFile：适用于zip文件在硬盘里或内存里的情况，可以随机访问

ZipArchiveInputStream：适用于通过网络IO或其他只能顺序读取zip的情况，只能顺序访问

### ZipFile随机访问

1. 解压出zip包中的单个文件

   根据名字解压特定文件，比如将/root/test.zip文件中，名称为targetFile的文件，解压为/tmp/output/targetFile文件

   ```
   ZipFile zipFile = new ZipFile("/root/test.zip");
   ZipArchiveEntry entry = zipFile.getEntry("targetFile"); // 我们可以根据名字，直接找到要解压的文件
   try (InputStream inputStream = zipFile.getInputStream(entry)) {
       // 这里inputStream就是一个正常的IO流，按照正常IO流的方式读取即可，这里简单给个例子
       byte[] buffer = new byte[4096];
       File outputFile = new File("/tmp/output/targetFile");
       try (FileOutputStream fos = new FileOutputStream(outputFile)) {
           while (inputStream.read(buffer) > 0) {
               fos.write(buffer);
           }
       } catch (FileNotFoundException e) {
           e.printStackTrace();
       } catch (IOException e) {
           e.printStackTrace();
       }
   }
   ```

    

2. 解压全部文件

   ```
   ZipFile zipFile = new ZipFile(new File("/root/test.zip"));
   byte[] buffer = new byte[4096];
   ZipArchiveEntry entry;
   Enumeration<ZipArchiveEntry> entries = zipFile.getEntries(); // 获取全部文件的迭代器
   InputStream inputStream;
   while (entries.hasMoreElements()) {
       entry = entries.nextElement();
       if (entry.isDirectory()) {
           continue;
       }
   
       File outputFile = new File("/tmp/output/" + entry.getName());
   
       if (!outputFile.getParentFile().exists()) {
           outputFile.getParentFile().mkdirs();
       }
   
       inputStream = zipFile.getInputStream(entry);
       try (FileOutputStream fos = new FileOutputStream(outputFile)) {
           while (inputStream.read(buffer) > 0) {
               fos.write(buffer);
           }
       } catch (FileNotFoundException e) {
           e.printStackTrace();
       } catch (IOException e) {
           e.printStackTrace();
       }
   }
   ```

### ZipArchiveInputStream顺序访问

一个文件一个文件的读取，你在使用时可以决定解压或是不解压遍历到的文件

```
File file = new File("/root/xxxxx/xx.zip");
// 这里还是用文件流举例，实际可以是各种InputStream
try (ZipArchiveInputStream zipInputStream = new ZipArchiveInputStream(new FileInputStream(file))) {
    ZipArchiveEntry entry = zipInputStream.getNextZipEntry();
    entry.getName(); // 这里同样可以获取包括名字在内的许多文件信息
    entry = zipInputStream.getNextZipEntry(); // 如果我们不需要读取第一个文件，可以直接跳到下一个文件
    byte[] buffer = new byte[4096];
    int read;
    while ((read = zipInputStream.read(buffer)) > 0) {
        XXXX
    }
}
```

## 压缩

```
File archive = new File("/root/xx.zip");
try (ZipArchiveOutputStream outputStream = new ZipArchiveOutputStream(archive)) {
    ZipArchiveEntry entry = new ZipArchiveEntry("testdata/test1.xml");
    // 可以设置压缩等级
    outputStream.setLevel(5);
    // 可以设置压缩算法，当前支持ZipEntry.DEFLATED和ZipEntry.STORED两种
    outputStream.setMethod(ZipEntry.DEFLATED);
    // 也可以为每个文件设置压缩算法
    entry.setMethod(ZipEntry.DEFLATED);
    // 在zip中创建一个文件
    outputStream.putArchiveEntry(entry);
    // 并写入内容
    outputStream.write("abcd\n".getBytes(Charset.forName("UTF-8")));
    // 完成一个文件的写入
    outputStream.closeArchiveEntry();

    entry = new ZipArchiveEntry("testdata/test2.xml");
    entry.setMethod(ZipEntry.STORED);
    outputStream.putArchiveEntry(entry);
    outputStream.write("efgh\n".getBytes(Charset.forName("UTF-8")));
    outputStream.closeArchiveEntry();
}
```

