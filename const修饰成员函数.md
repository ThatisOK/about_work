### const修饰成员函数

1. 作用
> void test() const; 函数中不能有对会造成成员变量改动的操作。包括对string ,int类型的直接修改，对vector,map等类型的增删改操作。__尤其是stl中的一些数据结构中的非const方法，如[]操作符，可以使用c++11中的at()替代。__

2. const可以重载
> void test();
> void test() const;

3. const成员函数不能访问非const成员函数
> 因为非const成员函数可以修改成员变量，这与const的功能相违背。

4. 示范案例
```c++
***person.h***

#pragma once

#include <string>
#include <map>

class Person {
public:
    Person();
    Person(const std::string& name, unsigned int age);
    Person GetFather() const;
    void AddRelationship(const std::string& r, const Person& p);
    void Print() const;
private:
    std::string name_;
    unsigned int age_;
    std::map<std::string, Person> relationship_;
};

***person.cc***

#include "Person.h"
#include <iostream>

Person::Person()
{
    
}

Person::Person(const std::string& name, unsigned int age)
            : name_(name),
              age_(age)
{

}

Person Person::GetFather() const
{
    if (relationship_.find("father") == relationship_.end()) {
        return Person("no father", 0);
    }
    return relationship_["father"];
}

void Person::AddRelationship(const std::string& r, const Person& p)
{
    relationship_[r] = p;
}

void Person::Print() const
{
    std::cout<< name_ << "," << age_;
}

int main()
{
    Person son("zhangsan", 12);
    Person father("zhanger", 38);
    son.AddRelationship("father", father);
    son.GetFather().Print();
}
```

g++编译会报一下错误
```shell
person.cc: In member function ‘Person Person::GetFather() const’:
person.cc:21:34: error: passing ‘const std::map<std::basic_string<char>, Person>’ as ‘this’ argument of ‘std::map<_Key, _Tp, _Compare, _Alloc>::mapped_type& std::map<_Key, _Tp, _Compare, _Alloc>::operator[](std::map<_Key, _Tp, _Compare, _Alloc>::key_type&&) [with _Key = std::basic_string<char>; _Tp = Person; _Compare = std::less<std::basic_string<char> >; _Alloc = std::allocator<std::pair<const std::basic_string<char>, Person> >; std::map<_Key, _Tp, _Compare, _Alloc>::mapped_type = Person; std::map<_Key, _Tp, _Compare, _Alloc>::key_type = std::basic_string<char>]’ discards qualifiers [-fpermissive]
     return relationship_["father"];
```
查看std::map的[]运算符，如下：
```c++
mapped_type& operator[] (const key_type& k);
mapped_type& operator[] (key_type&& k);
```
这两个函数都不是const函数，说明他们会修改std::map的成员。
查看其说明：
> Access element
If k matches the key of an element in the container, the function returns a reference to its mapped value.
If k does not match the key of any element in the container, the function inserts a new element with that key and returns a reference to its mapped value. Notice that this always increases the container size by one, even if no mapped value is assigned to the element (the element is constructed using its default constructor).
A similar member function, map::at, has the same behavior when an element with the key exists, but throws an exception when it does not.

[]会在存在k的情况下返回引用，没有则在最后插入一个，正是这一点，修改了std::map的成员。

```c++
//只需要将
return relationship_["father"];
//替换为
return relationship_.at("father");
//at()会在key不存在的情况下跑出 out_of_range 异常。