---
title: "几种使用 Java 操作 Zip 包的方法"
date: 2022-05-20
tags: ["java", "zip"]
description : "该篇文章主要是讲述几种 Zip 包的方法和怎么使用 Java Jdk 操作 Zip 包的。"
---

该篇文章主要是讲述几种 Zip 包的方法和怎么使用 Java Jdk 操作 Zip 包的。

## 几种方法
这里汇总了可以使用的工具类，开箱即用
- [Hutool ZipUtil](https://hutool.cn/docs/#/core/%E5%B7%A5%E5%85%B7%E7%B1%BB/%E5%8E%8B%E7%BC%A9%E5%B7%A5%E5%85%B7-ZipUtil)
- [Ant Zip](https://ant.apache.org/manual/Tasks/zip.html)
- [Apache Commons compress](https://commons.apache.org/proper/commons-compress/zip.html)
- JDK 的方式

当然还有其他的方式。下面以 JDK 的方式为例讲述怎么实现文件的压缩和解压缩过程。

## JDK 方式

### 压缩
核心的类位于 java.util.zip 包中，其中有个 ZipEntry， 这个是表示了 Zip 每个节点，包含了所有文件夹和文件。比如下面的文件路径

```log
dir222
    |-- file1.md
    |-- file2.md
    |-- dir223
        |- file3.md
```
如果将 dir222 压缩成 zip 包。 大体思路如下

- 获取合法路径下的所有文件， 可以递归获取
- 根据要保存的 zip 包，声明文件输出流和 Zip 输出流。文件输出流是 zip 包的实体，而 Zip 输出流是 zip 包的内容。有点类似于 docker 中的 image 和 container。
- 遍历所有文件信息，生成相应的 zipEntry， zipEntry 的名称就是文件名。将每个文件的输入流写入到 Zip 包的输出流中。 一个文件对应一个 ZipEntry， 每个 ZipEntry 写完后都需要关闭。

使用技巧： `try() {}` 语法，该语法会自动关闭相应的流。

具体代码如下：

```java
/**
 * 递归获取对应的文件 * * @param file
 * @return
 */
private static void getAllFile(File file, List<File> af) {
        if (file.isDirectory()) {
            File[] files = file.listFiles();
            if (Objects.nonNull(files)) {
                for (File f : files) {
                    getAllFile(f, af);
                }
            }
        }

        if (file.isFile()) {
            af.add(file);
        }
    }


    /**
     * 将需要目录中的文件压缩成 zip
     *
     * @param filePath 需要压缩的路径
     * @param zipFile  压缩后的路径
     * @return
     */
    public static void zip(String filePath, String zipFile) throws IOException {
        File file = new File(filePath);

        // 如果路径不存在或者不是路径直接返回
        if (!file.exists() || !file.isDirectory()) {
            System.out.println("文件不存在或者不是目录");
            return;
        }

        List<File> allFiles = new ArrayList<>();
        getAllFile(file, allFiles);

        try (
                // 文件输出流
                FileOutputStream outputStream = new FileOutputStream(zipFile);
                // zip 包输出流
                ZipOutputStream zipOutputStream = new ZipOutputStream(outputStream)) {
            for (File f : allFiles) {
                byte[] buffer = new byte[2048];
                String name = f.getAbsolutePath().substring(file.getAbsolutePath().length() + 1);
                System.out.println(f.getAbsolutePath() + "=====" + name);
                // 设置 entry 节点
                ZipEntry zipEntry = new ZipEntry(name);
                zipOutputStream.putNextEntry(zipEntry);

                // 写入数据流
                try (FileInputStream fileInputStream = new FileInputStream(f)) {
                    int read;
                    while ((read = fileInputStream.read(buffer, 0, buffer.length)) > 0) {
                        zipOutputStream.write(buffer, 0, read);
                    }
                }

                zipOutputStream.closeEntry();
            }
        }
    }
```



### 解压
大体思路如下：
1.  使用 zip 包生成 zipInputStream， 遍历 ZipInputStream 直到没有 nextEntry， 这样通过 getNextEntry 获取到 ZipEntry。
2.  根据 entry 中的名称，创建相应的文件夹。
3.  将 当前 entry 的 ZipInputStream 读取到 buffer 中。 将 buffer 中的内容写入到 file 中。


具体代码如下

```java

public static void unzip(String zipFile, String decompressPath) throws IOException {
    // 这里可以优化，目前只支持一层的外部路径
    Path path = Paths.get(decompressPath);
    File decompressPathFile = path.toFile();
    if (!decompressPathFile.exists()) {
        decompressPathFile.mkdirs();
    }

    try (
    FileInputStream fileInputStream = new FileInputStream(zipFile);
    ZipInputStream zipInputStream = new ZipInputStream(fileInputStream)) {

        ZipEntry zipEntry;
        byte[] buffer = new byte[2048];
        while ((zipEntry = zipInputStream.getNextEntry()) != null) {
            Path formatPath = path.resolve(zipEntry.getName());
            if (zipEntry.getName().contains("/")) {
                mkdirAllDir(zipEntry.getName(), decompressPath);
            }
            try (
                    // 文件输出流
                    FileOutputStream fos = new FileOutputStream(formatPath.toFile());
                    // 内容输出流
                    BufferedOutputStream bos = new BufferedOutputStream(fos, buffer.length)) {
                int read;
                if ((read = zipInputStream.read(buffer, 0, buffer.length)) > 0) {
                    bos.write(buffer, 0, read);
                }
            }
        }
    }

}

private static void mkdirAllDir(String name, String decompressPath) {
    String[] paths = name.split("/");
    StringBuilder allPath = new StringBuilder(decompressPath + "/");
    for (int i = 0; i < paths.length - 1; i++) {
        allPath.append(paths[0]);
        if (!new File(allPath.toString()).mkdirs()) {
            System.out.println("创建文件失败！！");
            return;        }
    }
}
```

## 总结

通过上述的📦压缩包和解压缩包来看，几个比较重要的类：
- 压缩 ：FileOutputStream、ZipOutputStream、ZipEntry、FileInputStream
- 解压缩：FileInputStream、ZipInputStream、ZipEntry、FileOutputStream、BufferedOutputStream

上述实现的还不够灵活，hutool 实现的比较全。如果可以使用的话比较推荐它。代码详情：[ZipUtil](https://github.com/plutokaito/case-demo/blob/main/my-util/src/main/java/com/kaitoshy/utils/ZipUtil.java)