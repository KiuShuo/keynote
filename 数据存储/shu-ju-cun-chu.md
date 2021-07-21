#### 参考资料
[最全iOS数据存储方法介绍：FMDB，SQLite3 ，Core Data，Plist，Preference偏好设置，NSKeyedArchiver归档，Realm](https://www.jianshu.com/p/e88880be794f)  

#### 数据存储

1. UserDefaults 通过key-value配对的形式进行数据存储。至于`func synchronize()`立即保存函数原本是为了在写入数据的时候，App突然被杀掉了，系统注释中也明确说了，已经弃用不需要再调用，将来会明确标记出来。一般保存App的基本参数。     
2. Plist文件，它是一个XML文件，读写分别通过`contensOfFile`和`writeToFile`实现；一般也用来保存App的基本参数。  
3. NSKeyedArchiver 序列化反序列化的方式。遵从NSCoding协议的对象就可以实现序列化。NSCoding中有两个方法，即归档`initWithCoder`和解归档`encodeWithCoder`。通过NSKeyedArchiver的`archiveRootObject:toFile`来实现数据存储，通过NSKeyedArchiver的`unarchiveObjectwithFile`来实现读取。  
4. 通过数据库实现，如SQLit、CoreData、FMDB、Realm实现。前面的几种方法，都是覆盖式存储，修改数据时要读取整个文件，然后再重新覆盖写入，十分不适合大量数据的存储。  

5. 将一些需用应用删除重装之后仍想获取到的信息保存在钥匙串里面。

#### 沙盒机制

* Documents: 保存应⽤运行时生成的需要持久化的数据，iTunes同步设备时会备份该目录。
* tmp: 保存应⽤运行时所需的临时数据，使⽤完毕后再将相应的文件从该目录删除。应用没有运行时，系统也可能会清除该目录下的文件。iTunes同步设备时不会备份该目录。
* Library/Caches: 保存应用运行时⽣成的需要持久化的数据，iTunes同步设备时不会备份该目录。一般存储体积大、不需要备份的非重要数据，比如网络数据缓存存储到Caches下。


