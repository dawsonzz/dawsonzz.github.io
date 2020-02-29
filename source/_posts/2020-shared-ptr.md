---
title: boost::asio中shared_ptr的作用
date: 2020-01-23 14:26:25
tags: [c++, shared_ptr, lambda]
description: <center>对boost异步通讯服务器中共享指针应用的理解</center>
---

## std::shared_ptr概述

使用std::shared_ptr的好处优于new的地方在于它自身维护一个引用计数器，当进行拷贝时使得引用计数加1,离开作用域时引用计数减1,当引用计数为0时，对管理的对象进行析构

## boost案例

```c++
//
// async_tcp_echo_server.cpp
// ~~~~~~~~~~~~~~~~~~~~~~~~~
//
// Copyright (c) 2003-2019 Christopher M. Kohlhoff (chris at kohlhoff dot com)
//
// Distributed under the Boost Software License, Version 1.0. (See accompanying
// file LICENSE_1_0.txt or copy at http://www.boost.org/LICENSE_1_0.txt)
//

#include <cstdlib>
#include <iostream>
#include <memory>
#include <utility>
#include <boost/asio.hpp>

using boost::asio::ip::tcp;

class session
  : public std::enable_shared_from_this<session>
{
public:
  session(tcp::socket socket)
    : socket_(std::move(socket))
  {
  }

  void start()
  {
    do_read();
  }

private:
  void do_read()
  {
    auto self(shared_from_this());
    socket_.async_read_some(boost::asio::buffer(data_, max_length),
        [this, self](boost::system::error_code ec, std::size_t length)
        {
          if (!ec)
          {
            do_write(length);
          }
        });
  }

  void do_write(std::size_t length)
  {
    auto self(shared_from_this());
    boost::asio::async_write(socket_, boost::asio::buffer(data_, length),
        [this, self](boost::system::error_code ec, std::size_t /*length*/)
        {
          if (!ec)
          {
            do_read();
          }
        });
  }

  tcp::socket socket_;
  enum { max_length = 1024 };
  char data_[max_length];
};

class server
{
public:
  server(boost::asio::io_context& io_context, short port)
    : acceptor_(io_context, tcp::endpoint(tcp::v4(), port))
  {
    do_accept();
  }

private:
  void do_accept()
  {
    acceptor_.async_accept(
        [this](boost::system::error_code ec, tcp::socket socket)
        {
          if (!ec)
          {
            std::make_shared<session>(std::move(socket))->start();
          }

          do_accept();
        });
  }

  tcp::acceptor acceptor_;
};

int main(int argc, char* argv[])
{
  try
  {
    if (argc != 2)
    {
      std::cerr << "Usage: async_tcp_echo_server <port>\n";
      return 1;
    }

    boost::asio::io_context io_context;

    server s(io_context, std::atoi(argv[1]));

    io_context.run();
  }
  catch (std::exception& e)
  {
    std::cerr << "Exception: " << e.what() << "\n";
  }

  return 0;
}
```

## 过程分析

### 共享指针初始化

在server::do_accpect()中使用了make_shared创建了共享指针，此时引用计数为1。当共享指针离开了`if (!ec){...}`作用域时，共享指针引用计数减1,因此为了保证session的存在，start函数必须将共享指针传递下去。

```c++
void do_accept()
  {
    acceptor_.async_accept(
        [this](boost::system::error_code ec, tcp::socket socket)
        {
          if (!ec)
          {
            std::make_shared<session>(std::move(socket))->start();
          }

          do_accept();
        });
  }
```

为什么不能在session构造的时候将共享指针传递下去？因为session继承于std::enable_shared_from_this<session> ，构造的时候共享指针还未创建，因此共享指针并不存在。

### 共享指针传递

紧接着`session.start()`中调用了`do_read()`,一进来就运行了`auto self(shared_from_this());`使得引用计数加1,此时引用计算为2，这时候就算共享指针离开了`server::do_accept`,引用计数变为1，也不会使得`session`的实例析构，这条语句保证了`session`实例在函数`do_read()`始终存在。

```c++
void do_read()
  {
    auto self(shared_from_this());
    socket_.async_read_some(boost::asio::buffer(data_, max_length),
        [this, self](boost::system::error_code ec, std::size_t length)
        {
          if (!ec)
          {
            do_write(length);
          }
        });
  }
```

接下来调用了lambda表达式`[this, self](){}`中值拷贝的形式，调用了拷贝构造函数，使得引用计数加1,在lambda表达式范围内，共享指针引用计数不为0。如果现在没有离开`server::do_accept`和`session::do_read`的作用域，此时引用计算为3，如果离开了，此时引用计数为1，`async_read_some`是异步的，所以会离开`session::do_read`的作用域。

类似的，`session::do_write`与`session::do_read`相似，保证了共享指针的引用计数始终不为0。只有当发生错误时，才不继续执行这两个函数，引用计数为0，session实例析构。