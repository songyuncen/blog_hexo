---
title: 敏捷设计书摘  
date: 2016-10-26 16:49:50
categories:
- 软件
- 面向对象设计
tags: 
- 读书笔记 
- 软件设计
---

![Cover](images/agile-design.jpg)

敏捷设计是一个过程，它是一个持续的应用原则、模式以及实践来改进软件的结构和可读性，避免软件腐化的过程。

# 软件的腐化
- 僵化性（Rigidity）：很难对系统进行改动，因为每个改动都会迫使许多对系统其他部分的其他改动。
- 脆弱性（Fragility）：对系统的改动会导致系统中和改动的地方在概念上无关的许多地方出现问题。
- 牢固性（Immobility）：很难解开系统的纠结，使之成为一些可在其他系统中重用的组件。
- 粘滞性（Viscosity）：做正确的事情比作错误的事情要困难。
- 不必要的复杂性（Needless Complexity）：设计中包含有不具任何直接好处的基础结构。
- 不必要的重复（Needless Repetition）：设计中包含有重复的结构，而该重复的结构本可以使用单一的抽象进行统一。
- 晦涩性（Opacity）：很难阅读、理解。没有很好地表现出意图。

# 原则

## 单一职责原则（SRP）

就一个类而言，应该仅有一个引起它变化的原因。在SRP中，我们把职责定义为“变化的原因”。如果你能够想到多于一个的动机去改变一个类，那么这个类就具有多于一个的职责。当需求发生改变时，该变化会反映为类的职责的变化。如果一个类承担了多于一个的职责，那么对引起变化的职责的修改，有可能会影响到其他职责的功能。

## 开放-封闭原则（OCP）

软件实体（类、模块、函数等等）应该是可以扩展的，但不可以修改，它们对于扩展是开放的，对于更改是封闭的。当需求改变时，不应该修改已有的模块的源代码和二进制代码，而是通过扩展模块的功能来满足需求的变更。

实现OCP原则的关键工具是**抽象**。对于行为容易发生改变的部分，利用抽象体进行连接。模块之间在行为容易改变的地方通过相对固定的抽象体相互连接，利用抽象的多态性实现行为的扩展。

两个例子：

通过抽象体Drawable实现绘制形状，当有新的形状时，paint函数和已有的shape都不需要修改（封闭），只需要增加（扩展）新的形状类。

```c++
class Drawable
{
public:
    virtual void Draw() = 0;
}

void Window::paint()
{
    for (Drawable *s : shapes_)
    {
        s->Draw();
    }
}

class Rectangle : public Drawable
{
public:
    void Draw()
    {
        // draw Rectangle
    }
}

class Circle : public Drawable
{
public:
    void Draw()
    {
        // draw Circle
    }
}

```

为了使得上面例子的paint函数支持绘制形状的顺序性，同时使得顺序性的变化具有可扩展性，需要将顺序变化部分抽象出来以供扩展（开放），并将按照顺序画形状这一行为固化（封闭）。

```c++
class Drawable
{
public:
    virtual void Draw() = 0;
    bool operator<(const Drawable &) const = 0;
}

void Window::paint()
{
    vector<Drawable*> ordered_list = shapes_;
    sort(ordered_list.begin(), ordered_list.end());

    for (Drawable *s : ordered_list)
    {
        s->Draw();
    }
}

class Rectangle : public Drawable
{
public:
    void Draw()
    {
        // draw Rectangle
    }
    bool operator<(const Drawable &s) const 
    {
        // order
    }
}

class Circle : public Drawable
{
public:
    void Draw()
    {
        // draw Circle
    }
    bool operator<(const Drawable &s) const 
    {
        // order
    }
}

```

## Liskov替换原则（LSP）

子类型必须能够替换掉它们的基类型。Liskov 原则是使得OCP成为可能的主要原则和保证。子类型对于基类性的替代是一种IS-A的关系，而IS-A并不是概念上的，而应该是行为上的，不能说正方形属于矩形，就一定要让正方形继承自矩形。行为方式才是软件真正关注的问题。

这种行为上的一致，可以用基于契约的设计来指导，比如每个方法对于前置条件、后置条件的约束在子类中都需要得到保证。基于契约的设计常常通过单元测试来检测和实现。

## 依赖倒置原则（DIP）

- 高层模块不应该依赖于低层模块。二者都应该依赖于抽象。
- 抽象不应该依赖于细节。细节应该依赖于抽象。

在层次化的构架设计中，应该由高层次的模块拥有抽象接口，低层次模块实现该抽象接口。这样高层次模块并不知道低层次模块的细节，只是通过抽象接口向低层次模块提出了服务的需求。这和通常的想法有些不同，通常是由低层次的模块设计和拥有对外接口。不反对低层次模块具有调用接口，但是在层次构架中，低层次的模块需要根据高层次模块提出的接口需要，组合自己的功能，实现高层次模块的接口需求。

同时在高层次模块中不应该保存具体的低层次模块的指针或引用，而是应该保存抽象接口的指针或引用。


## 接口隔离原则（ISP）

不应该强迫客户依赖于他们不用的方法。使用委托或是多重继承等方法，将接口进行分离。当客户在使用一个类时，如果使用的是该类的一个职责，就只将该职责对应的接口提供给客户。如果强迫客户程序依赖于他们不使用的方法，那么这些客户就面临着由于这些未使用方法的改变所带来的变更。
