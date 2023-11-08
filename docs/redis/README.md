## Redis 笔记
全局哈希表，NIO，epoll_wait（timeout） 超时模型

```java
import { defaultTheme, defineUserConfig } from 'vuepress'

export default defineUserConfig({
  title: '你好， VuePress',

  theme: defaultTheme({
    logo: 'https://vuejs.org/images/logo.png',
  }),
})
```

## Redis的数据结构与对象 <Badge text="测试" />

### 动态字符串

+ Redis 并没有直接使用C语言的字符串格式(即 '\0' 的风格) 而是自己构建可一个名为简单动态字符串的抽象类型，需要注意的是，SDS 仅仅在KV中被使用，在代码的日志等常量字符串中依旧使用的是字符串字面量

  比如：

  ```sh
  redis> SET msg "hello, world"
  ```

  在Redis 的底层中 msg 也是一个SDS，"hello, world" 也是一个SDS

  

+ SDS 的数据结构如下 

  ```c
  struct sdshdr{
    // 记录buf中已使用的字节的数量
    int len;
    
    // 记录buf中未使用的字节的数量
    int free;
    
    // 字节数组用于保存字符串, buf的最后一位仍然是 '\0'
    char buf[];
  }
  ```

+ 除了用于保存数据库中的字符串之外，SDS还被用于缓冲区(buffer) 以及AOF模块中的缓冲区

+ SDS 中的buf的结尾处依然遵循C语言的传统，其尾部是 '\0' 这样做的好处是可以直接复用 C 语言的相关函数，而无需为SDS编写专门的函数，比如

  ```c
  printf("%s",s->buf)
  ```

+ C 语言的字符串并不适用于Redis的原因:

  > 1. C 字符串并不保存字符串的长度信息，获取其长度的时间复杂度为O(n) 而SDS的时间复杂度则为O(1)
  >2. 除了获取字符串长度的时间复杂度高之外，C语言字符串容易带来缓冲区溢出的风险，比如在合并字符串 strcat 之前需要确保字符串的空间足够，否则就会出现缓冲区溢出的问题，但SDS则不会出现这个问题，当SDS不满足空间大小要求的时间，其会自动拓展至所需要的空间大小，然后才执行操作
  > 3. 减少修改字符串时带来的内存重分配问题，如果执行的C字符串增长操作，则必须先通过内存重分配来手动拓展空间，否则会出现缓冲区溢出的风险；如果执行的是缩短操作，那么进行内存重分配缩短空间，否则可能会出现内存泄漏的问题，而每次执行内存重分配则是一个非常耗时的操作。且频繁的进行内存重分配对于速度要求苛刻的Redis来说显然是不合适。
  >4. 二进制安全性, 在使用Redis存储二进制数据的时候，不能强制要求使用 '\0' 作为结束符号，常见的文件，图片以及音频视频等等就不能，通过二进制安全，使得Redis的SDS可以保存二进制数据



### 链表

+ 



