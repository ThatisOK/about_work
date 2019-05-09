在一次开发中，使用了boost::shared_ptr，出core了。
core的信息为assert(px != 0)检测未通过，代码如下
```c++
T * operator->() const
{
    BOOST_ASSERT( px != 0 );
    return px;
}
```
出core的代码为：
```c++
ServiceAccessor::ServiceAccessor(boost::shared_ptr<etcd::EtcdClient> etcd_client) : Accessor(etcd_client)
{
    LOG(INFO) << etcd_client_->Test();
}
```
这个core的意思应该是这个shared_ptr为空，也就是use_count = 0。
很纳闷，因为出core的地方在一个构造函数，仅仅是将传进来的指针赋值到成员变量上，没有什么其他的操作，而且验证了构造函数的变量etcd_client的use_count!=0；

那为什么etcd_client_的use_count=0呢？

简单的代码如下：
```c++
class Accessor {
public:
    explicit Accessor(boost::shared_ptr<etcd::EtcdClient> etcd_client);
protected:
    boost::shared_ptr<etcd::EtcdClient> etcd_client_;
};

class ServiceAccessor : public Accessor{
public:
    explicit ServiceAccessor(boost::shared_ptr<etcd::EtcdClient> etcd_client);
private:
    boost::shared_ptr<etcd::EtcdClient> etcd_client_;

}
```
从这个代码可以看出，core出在了ServiceAccessor中也有一个etcd_client_，而Accessor(etcd_client)只会将Accessor中的etcd_client_=etcd_client，而ServiceAccessor中的etcd_cleint_还是为空的状态。在ServiceAccessor的构造函数中使用的etcd_client_属于ServiceAccessor，并没有与传进来的etcd_client做任何操作，所以还是空的状态。

而在子类ServiceAccessor操作etcd_client_是操作的其本身的etcd_client_，并不是父类accessor的etcd_client_，对其进行->操作，势必会触发assert，导致出core。

其实这个bug应该很容易发现，但是因为下面的代码和上面的代码不在一个文件中，下面的在service.h，上面的在service.cpp，所以只看service.cpp是很难发现在ServiceAccessory中多了一个etcd_client_。

