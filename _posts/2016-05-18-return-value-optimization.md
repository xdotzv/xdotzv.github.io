---
layout: post
title: 返回值优化 Return Value Optimization
---

先看一段C++代码。

```C++
struct C {
  C() {}
  C(const C&) { std::cout << "A copy was made." << endl; }
};

C f() {
  return C();
}

int main() {
  cout << "hello world" << endl;
  C obj = f();
}
```

这段代码会输出什么？在一个月之前，我肯定会说它调用了两次拷贝构造函数，输出如下内容。

```
hello world
A copy was made. 
A copy was made. 
```
并且还颇为得意自己很懂C++。事实证明C++这个坑原比我想象的大。

绝大多数的现代编译器编译出来的程序，最后都只会输出hello wordl，不会调用拷贝构造函数。我们很容易看出来，拷贝构造函数在上面的函数中都是不必要的，因为原来生成的都是临时对象，在拷贝完成后就直接没用了。现代编译器通常在这里会进行返回值优化(Return Value Optimization)，不通过拷贝，而是在需要返回值的地方**直接构造**。C++标准允许了编译器这样做，即使某些拷贝构造函数是有side-effects。

### Named Return Value Optimization

在我知道了上述事实后，我写代码使用过类似下图的方式来试图让编译器进行返回值优化。

```C++
Foo fun(){
  Foo ret;
  /* ... */
  return std::move(ret);
}
```
但实际上又是自作聪明了，编译器对于返回一个函数内的局部对象也会进行优化。同样也是当构造局部对象时，在需要返回值的地址那里**直接进行构造**。这种类型的优化被称作NRVO, "named return value optimization"。

### 参考

<http://en.cppreference.com/w/cpp/language/copy_elision>
<http://stackoverflow.com/questions/12953127/what-are-copy-elision-and-return-value-optimization>

