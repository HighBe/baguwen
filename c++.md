# c++

## c++11
### 右值引用
#### 左值、右值
- 左值：可以放到等号左边，可以取地址且有名字的是左值
  左值一般有：
  + 函数名和变量名
  + 返回左值引用的函数调用
  + 前置自增自减表达式 ++i --i
  + 由赋值表达式或赋值运算符链接的表达式（如a=b,a+=b）
  + 解引用表达式*p
- 右值：不可以放到等号左边，不能取地址没有名字的是右值
  右值一般有：
  + 除字符串字面值外的字面值
  + 返回非引用类型的函数调用
  + 后置自增自减表达式 i++ i--
  + 算术表达式(a+b,a*b,a&&b,a==b)
  + 取地址表达式等(&a)

#### 三种引用
c++11中包含三种引用：
- 左值应用
- 常量左值引用(不希望被修改)
- 右值引用

c++11中引入了右值引用，因为右值的声明周期很短，右值引用的引入，使得可以延长右值的生命周期。在c++中规定，右值引用是&&表示，左值引用是&表示。右值引用的作用是为了绑定右值。**右值引用本身是左值**
```
    string s = "abc";
    string &str = s;  //左值引用

    const int &a = 10; // 常量左值引用

    int &&b = 10; // 右值引用
```
#### 移动语义
可以理解为转移所有权，将一块资源转为自己所有，别人不会再拥有也不会再使用通过移动语义，可以省去很多拷贝负担。通过移动构造函数实现移动语义。
```
class A{
private:
    int *data_;
    int size_;
public:
    A(int size) : size_(size){
        data_ = new int[size];
    }
    A(){}
    A(const A& a){
        size_ = a.size_;
        data_ = new int[size_]_;
        // 将a.data中的内容复制到新申请的空间中
        cout << "copy" << endl;
    }
    A(A&& a){
        this->data_ = a.data_;
        a.data_ = nullptr;
        cout << "move" << endl;
    }
    ~A(){
        if(data_ != nullptr){
            delete []data_;
        }
    }
}
int main(){
    A a(10);
    A b(a); //调用拷贝构造函数
    A c = std::move(a); // 调用移动构造函数
    return 0;
}
```



## bind

    void test3()
    {
        int number = 10;
        //bind采用的是值传递
        //ref = reference, 引用的包装器
        //cref = const reference, 引用的包装器
        auto f = bind(func4, 1, std::placeholders::_3, std::placeholders::_1, std::cref(number), number);
        number = 30;
        f(50, 40, 60, 70, 90, 80);//没有用到的参数就丢掉
        /*
        x1 = 1 直接传入的参数
        x2 = 60 实参列表第三位
        x3 = 50 实参列表第一位
        x4 = 30 number的引用，后续更改为30
        x5 = 10 传参时number的值,相当于绑定一个常量
        */
    }

---
## 响应状态码
|状态码分类|说明|
|----|-------|
|1XX|**响应中** 临时状态码，表示请求已经接受，告诉客户端应该继续请求或者如果它已经完成则忽略它|
|2XX|**成功** 表示请求已经被成功接受，处理已完成|
|3XX|**重定向**  重定向到其他地方（让客户端再发起一个请求以完成整个处理）|
|4XX|**客户端错误** 处理发生错误，责任在客户端，如：客户端的请求一个不存在的资源，客户端未被授权，禁止访问等|
|5XX|**服务器端错误** 处理发生错误，责任在服务器端，如：服务端抛出异常，路由出错，HTTP版本不支持等|

|状态码|英文描述|解释|
|-----|------|-----|
|200|OK|客户端请求成功，即处理成功|
|302|Found|只是所请求的资源已移动到由Location响应头给定的URL，浏览器会自动重新访问到这个页面|
|304|Not Modified|告诉客户端，你请求的资源至上次取得后，服务端并未更改，你直接用你本地缓存吧。隐式重定向
|400|Bad Request|客户端请求有语法错误，不能被服务器所理解
|403|Forbidden|服务器收到请求，但是拒绝提供服务，比如：没有权限访问相关资源
|404|Not Found|请求资源不存在，一般是URL输入有误，或者网站资源被删除了
|428|Precondition Required|服务器要求有条件的请求，告诉客户端要想访问资源，必须携带特定的请求头
|429|Too Many Requests|太多请求，可以限制客户端请求某个资源的数量，配合Retry-After（多长时间后可以请求）响应头一起使用
|431|Request Header Fields Too Large|请求头太大，服务器不愿意处理请求，因为它的头部字段太大。请求可以在减少请求头域的大小后提交
|405|Method Not Allowed|请求方式有误，比如：应该用GET请求方式的资源，而用了POST
|500|Internal Server Error|服务器发生不可预期的错误，表示服务器出异常了
|503|Service Unavailable|服务器尚未准备好处理请求，比如：服务器刚刚启动，还未初始化好
|511|Network Authentication Required|客户端需要进行身份验证才能获得网络访问权限
---
## 结构体和类的大小
### 有虚函数的类
1. 带有虚函数的类的sizeof大小，实际上和虚函数的个数不相关，相关的是虚函数指针。
2. 一个类中若有虚函数，（不论是自己的虚函数，还是继承而来的），那么类中就有一个成员变量：虚函数指针，这个指针指向一个虚函数表，虚函数表的第一项是类的typeinfo信息，之后的项为此类的所有虚函数的地址。
3. 如果一个类继承自两个含有虚函数的类，那么这个类有两个虚函数指针。

### #pragma pack() 设置结构体的对其方式
1. 各成员变量存放的起始地址相对于结构的起始地址的偏移量必须为该变量的类型所占用的字节数的倍数。  
2. 各成员变量在存放的时候会按照在结构体中出现的顺序依次申请空间，同时确保结构体的大小为字节边界数（即该结构体中占用最大空间的类型所占用的字节数）的倍数。  
3. 对于一个结构或类来说，其数据成员按照指定大小与成员本身大小的最小值对齐，而这个类本身则按照类的最大成员大小和指定大小的最小值对齐。如果类的数据成员也是一个类，其是按照成员类中最大成员大小和指定大小的最小值对齐。

### static
#### 静态成员变量
静态成员变量是属于整个类所有，对类的所有对象只有一份拷贝  
静态成员变量初始化应该在类外进行。  注意：不能再头文件中初始化，只能再cpp中初始化。  
```
    class A{
    private:
        static int a;
    };
    int A::a = 1;
```

### 零碎
1. 变量定义不区分大小写  
2. 一个int型变量频繁调用，最好定义为register。  register int a = 0;  
register的作用是请求编译器让变量a直接放在寄存器中，运算速度更快
3. 以下代码中哪个是正确的
```
    int var,*ptr;  
    *ptr = var; // ptr没有初始化，应该先分配一个地址再使用
    ptr = NULL; // 正确
```
4. fseek ftell 的功能
   fseek用来将定位文件指针
```
   fseek(FILE* stream,long offset,int origin);
   第一个参数为文件流，第二个参数为偏移量，第三个参数为文件指针定位的位置。  
   第三个参数有如下内容：
   SEEK_SET 文件开始；
   SEEK_CUR 文件指针当前位置；
   SEEK_END文件末尾
```
   ftell 返回文件指针相对于起始位置的偏移量
   下列代码，结合fseek和ftell用来计算文件的大小
```
    FILE* fstream = open("/path/file","r");
    fseek(fstream,0,SEEK_SET);
    int fset = ftell(fstream);
    fseek(fstream,0,SEEK_END);
    int fend = ftell(fstream);
    int flength = fend - fset;
```
# 计算机网络
## 滑动窗口
### 为什么要有滑动窗口？
如果发送方发送一个包，接收方接收到并返回一个ACK，发送方收到ACK再给接收方发送下一个包，这样效率太低。  
所以采用批量发送，批量应答的情况。即发送方，不断的发包，不需要收到应答再发送下一个。这样还有一个好处，如果ACK600丢了，但是收到了ACK700那么说明接收方700以前的数据都收到了。  
但这样会有一个问题，如果接收方系统繁忙或资源紧张，来不及处理这么多报文，那么就会全部丢弃，使得效率降低。为解决这种现象就提出了**滑动窗口**，让发送方根据接收方的接收能力控制发送量。


# Linux
## 常用Linux命令
### 文件操作
#### 常用

    cd /home    进入'/home'目录
    cd ..   返回上一级目录
    pwd     显示工作路径
    ls      查看目录中的文件
    ls -l   显示文件的详细资料
    tree    显示由根目录开始的树形结构
    mkdir dirname   创建名为dirname的目录
    mkdir -p /tmp/dir1/dir2     创建一个目录树
    rm -f file1     删除名为file1的文件
    rmdir dir1      删除名为dir1的目录
    rm -rf dir1     删除名为dir1的目录及该目录下的所有内容
    mv dir new_dir  移动/重命名一个目录
    cp file1 file2  复制一个文件
    cp dir/*        复制一个目录下的所有文件到当前工作目录
    cp -a dir1 dir2 复制一个目录
    cp -r dir1 dir2 复制一个目录及其子目录
    touch -t YYMMDDhhmm file1       修改一个文件的时间戳
    iconv -l        列出已知的编码

#### 文件授权

    ls -lh      显示授权
    chmod ugo+rwx dir1      设置目录的所有人(u)、群组(g)以及其他人(o)以读(r)、写(r)和执行(x)权限。
    chown user1 file1       改变一个文件的所有人属性
    chgrp group1 file1      改变文件的群组
    chmod u+s /bin/file1    设置一个二进制文件的SUID位 - 运行该文件的用户也被赋予和所有者同样的权限
    chmod u-s /bin/file1    禁用要给二进制文件的SUID位
    chmod g+s /home/dir1    设置一个目录的SGID位 - 类似SUID，不过这个针对目录
    chmod o+t /home/dir1    设置一个目录的STIKY位 - 只允许合法所有人删除目录

#### chmod的使用

    chmod [可选项] <mode> <file...>
    可选项：
        -c
        -f
        -v
        -R
    mode:
        [ugoa...][[+-=][rwxX]...][,...]
        [ugoa...]
        u表示该文件的拥有者，g表示与该文件所有者同一个群组的用户，o表示其他以外的人，a表示所有人(包括以上三者)。
        [+-=]
        +表示增加权限，-表示取消权限，=表示唯一设定权限
        [rwxX]
        r表示可读，w表示可写，x表示可执行，X表示只有当该档案是个子目录或者该档案已经被设定过为可执行。
        数字4 2 1分别表示读 写 执行

#### 文件搜索

    find / -name file1      从'/'开始搜索文件和目录
    find / -user user1      搜索属于用户user1的文件
    find /home -name \*.bin 在目录/home中搜索以.bin结尾的文件


#### 文件特殊属性

    文件的特殊属性 - 使用 '+' 设置属性，使用 '-' 取消。
    chatter [+-][属性] file1
    a   只允许以追加方式写文件
    c   允许这个文件能被内核自动压缩/解压
    d   在进行文件系统备份时，dump程序将忽略这个文件
    i   设置成不可变的文件，不能被删除、修改、重命名或链接
    s   允许一个文件被安全的删除
    S   一旦应用程序对这个文件进行了写操作，使系统立刻把修改的结果写到磁盘
    u   若文件被删除，系统会允许你在以后恢复这个被删除的文件

    lsatter 显示特殊的属性

#### 


### 操作系统
#### 系统信息

    arch    显示处理器架构
    uname -r    显示正在使用的内核版本
    cat /proc/cpuinfo           显示CPU信息
    cat /proc/interrupts        显示中断
    cat /proc/meminfo           校验内存使用
    cat /proc/swaps             显示哪些swap被使用
    cat /proc/version           显示内核版本
    cat /proc/net/dev           显示网络适配器及统计
    cat /proc/mounts            显示已加载的文件系统
    lspci -tv                   显示PCI设备
    lsusb -tv                   显示USB设备
    data                        显示系统日期
    cal 2007                    显示2007年的日历表

#### 用户和群组

    groupadd group_name         创建一个新的用户组
    groupdel group_name         删除一个用户组
    groupmod -n new_name old_name       重命名用户组
    useradd user1               创建一个新用户
    password                    修改口令
    password user1              修改一个用户的口令(只允许root执行)

#### 磁盘空间

    df -h                       显示已挂载的分区列表
    ls -lSr |more               已尺寸大小排列文件和目录
    du -sh dir1                 估算目录dir1使用的磁盘空间

### 系统进程
#### linux查看端口占用
    
    netstat -tunlp | grep 端口号    查看端口进程
    lsof -i:端口号       查看端口占用
    ss -tln             查看tcp的listen端口
    ss -tlnp            查看哪些进程使用了监听端口

#### ps命令

    a   显示先行终端机下所有程序，包括其他用户程序
    A   显示所有进程
    c   列出程序时，显示每个程序真正的指令名称
    e   列出程序时，显示每个程序所使用的环境变量
    f   用ASCII字符显示树状结构，表达程序间的相互关系 
    -H  显示树状结构，表达程序间的相互关系
    u   以用户为主的格式来显示程序状况
    x   显示所有程序，不以终端机来区分
常用指令组合：ps aux，然后通过grep命令过滤查找特定的进程，然后再对特定的进程进行操作。

    ps aux | grep prom_filter_word,ps-ef | grep tomcat
    ps -ef | grep java | grep -v grep  显示出所有的Java进程，去除掉当前的grep进程
