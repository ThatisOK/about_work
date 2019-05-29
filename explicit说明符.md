### explicit说明符

现有两段代码如下:
```c++
***service.h***
#include <string>

class Service {
public:
    Service();
    Service(const std::string& name);                                              
    ~Service();
    inline std::string get_name() const {return name_;}
    inline int get_num() const {return num_;};

private:
    std::string name_;
    int num_;
};

***service.cc**
#include "service.h"
#include <iostream>

Service::Service()
{

}

Service::Service(const std::string& name)
    : name_(name),
      num_(0)
{

}

Service::~Service()
{

}

void PrintService(const Service& service)
{
    std::cout << service.get_name() << service.get_num() << "\n";
}

int main()
{
    std::string service = "test";
    PrintService(service);
    return 0;
}
```
使用g++编译不会报错
```shell 
g++ service.cc service.h -std=c++11
```
运行得到如下：
> test0

很明显在PrintService中，我们需要的是一个Service类型的引用，但是传进去一个string类型的数据居然不报错！且能运行成功。通过输出可以看出是调用service的Service(const std::string& name)构造函数。然后将name_初始化为test，num_为0。这种情况下有时候会引起bug。在构造函数中，对num_赋值0，如果不赋值num_会是什么就完全是未知的，可能会引起很严重的后果，所以对于单参数的构造函数，都会显式的声明为explicit函数。也就是
```c++
explicit Service(const std::string& name);
```
现在再编译就会报错
```shell
service.cc: In function ‘int main()’:
service.cc:30:25: error: invalid initialization of reference of type ‘const Service&’ from expression of type ‘std::string {aka std::basic_string<char>}’
     PrintService(service);
                         ^
service.cc:21:6: error: in passing argument 1 of ‘void PrintService(const Service&)’
 void PrintService(const Service& service)
 ```
 可以看到
 > invalid initialization of reference of type ‘const Service&’ from expression of type ‘std::string {aka std::basic_string<char>}’

 已经是无效的初始化。表明已经不能隐式的使用string初始化了，必须
 ```c++
 Service s("test");
 ```
 
 总结：
 > explicit的作用是指定构造函数或转换函数 (C++11 起)为显式，即它不能用于隐式转换和复制初始化（从另一个对象初始化对象）。对于只包含一个参数的构造函数，需添加explicit说明符。
