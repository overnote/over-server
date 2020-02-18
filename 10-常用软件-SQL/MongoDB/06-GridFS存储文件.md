## 一 GridFS简介

GridFS是MongoDB存储大型文件的规范。在Mongo中，以BSON对象存储对数据的大小是有限制的，GridFS规范提供了将文件分块的标准，可以将一个大型文件分割成多个较小的文档，然后可以方便的存储如视频、高清图片等大型文件。在该标准中，每个文件都会在文件集合中保存一个元数据对象，一个或多个chunk块对象可以被组合保存在一个chunk块集合中。这样就可以实现大数据文件的海量存储了。  

GridFS存储文件时会分两个集合来存储：
- files包含元数据对象
- chunks包含其他一些想换信息的二进制块

如果需要将多GridFS命名为一个单一数据库，可以给文件和块加一个前缀，默认情况下，前缀为fs，所以在默认情况下，GridFS存储将包含命名空间fs.files和fs.chunks。  
任何第三方驱动都可以改变这个前缀，可以尝试设置另一个GridFS命名空间来存储照片，它的具体位置包括：phpotos.files和photos.chunks。

GridFS是建立在普通MongoDB文档的基础上的轻量级文件存储规范，MongoDB服务器实际上对GridFS的相关工作都由客户端驱动或者工具来完成的。  

GridFS应用场景：
- 大量的图片上传场景
- 大文件存储
- 文件备份，故障转移和恢复
  

GridFS局限性：
- GridFS文件越多，就会越干扰Mongo的内存工作集，建议将GridFS存储在不同的Mongo服务器上
- 文件服务性能慢于直接从Web服务器获取文件，但是这样也带来了管理优势
- GridFS没有提供对文件的院子更新方式。

## 二 mongofiles工具
mongofiles是从命令行操作GridFS的工具。  
```
mongofiles put test.txt         // 将test.txt保存到数据库中                 
mongofiles list                 // 查看数据库中有哪些GridFS
mongofiles get test.txt         // 取出test.txt文件并存储到硬盘中

依次输入
mongo
show collections                // 输出fs.chunks  fs.files
db.fs.files.find()              // 输出 存储test.txt的文档的相关信息，这里存储的是基础的元数据信息
db.fs.chunks.find()             // 输出结果中 n 代表chunks序号，从0开始，这里存储的是具体的数据信息
```