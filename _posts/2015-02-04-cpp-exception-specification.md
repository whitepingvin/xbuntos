---
layout: post
title: C++ exception specification
permalink: cpp-exception-specification
tags: C++
---

О тонкостях использования ключевого слова **throw**.

---

Краткое содержание статьи, что бы вообще ее не читать: не используйте **dynamic exception specification**, начиная со стандарта С++11 фича помечена как [deprecated](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2010/n3051.html).

---

Все, кто хоть как-то знаком с ООПшным подходом обработки ошибок в С++, знакомы с ключевым словом **throw**. Самое тривиальное использование - это генерация/выбрасывания исключения. Также достаточно часто используется **throw** само по себе внутри блока catch для повторного выброса текущего исключения:
{% highlight c++ %}
class my_exception : public std::exception {};
....
try {
    throw my_exception();
} catch(const my_exception& e) {
    std::cout << e.what() << '\n';
//  throw e; // copy-initializes a new exception 
             // object of type my_exception
    throw;   // rethrows the exception object of type my_exception
}
//need catch and handle exception again here
{% endhighlight %}
Также **throw** может применяться как спецификатор функции/метода, т.е. указывать какие типы исключений функция вообще может генерировать(и может ли вообще). Эта штука в стандарте С++ называется **dynamic exception specification**. Реализована следующим образом: на любой функции может быть указан throw(_exception-type-list_), где _exception-type-list_ — список из нуля или более типов или `...`. Если на функции стоит спецификатор **throw**, это означает, что функцию могут покидать исключения только указанных типов. Если спецификатор не указан, это эквивалентно `throw(...)`, т.е. функцию могут покидать исключения любого типа. Звучит вроде классно: программист может явно определять какие исключения ф-ция может генерировать(читай - какие ошибки могут возникать) или вообще запретить генерацию исключений.
{% highlight c++ %}
#include <iostream>

class X {};
class Z : public X {};
class W {};

void f(int n) throw(int, X) {
    switch(n) {
        case 0: throw n;   // OK
        case 1: throw X(); // OK
        case 2: throw Z(); // also OK
        default: throw W(); // will call std::unexpected()
    }
}

int main(){
    try {
        f(0);
    } catch(int) {
        std::cout << "int type throw" << std::endl; // OK
    }

    try {
        f(1);
    } catch(X) {
        std::cout << "X type throw" << std::endl;   // OK
    }

    try {
        f(2);
    } catch(Z) {
        std::cout << "Z type throw" << std::endl;   // OK
    }

    try {
        f(3);   //oops...crash :(
    } catch(W) {
        std::cout << "W type throw" << std::endl;
    }
   return 0;
}
{% endhighlight %}
Если скомпилировать вышеприведенный код g++(о mvsc речь будет чуть позже, там поведение отличается) и запустить программу, то она упадет. Из-за того что ф-ция попыталась сгенерировать исключение, тип которого отсутствует в списке _exception-type-list_, т.е. в данном случае `W`.

Указатель на функцию можно объявить соответственно, а вот с typedef не прокатит:
{% highlight c++ %}
void f() throw(int); // OK
void (*fp)() throw (int); // OK
typedef int (*pf)() throw(int); // ill-formed
{% endhighlight %}
Также интересный момент с **throw**, наследованием и виртуальными ф-циями:
{% highlight c++ %}
class A {
    virtual void f(int x) throw(int, long, std::logic_error) {
        switch(x) {
            case 0: throw long(0);
            case -1: throw std::logic_error("wtf?");
            default: throw(x);
        }
    }
};

class B: public A {
    void f(int x) throw(std::logic_error);    // OK
    //  сужать список исключений можно

    void f(int x) throw(int, long, bool); // COMPILE ERROR
    // нельзя расширять список исключений, т.к. данный метод может
    // быть вызван как метод базового класса.
    // была бы ф-ция не virtual - ошибки не было бы
    
    void f(int x);  // COMPILE ERROR
    // тоже самое что и с предыдущим случаем
    // расширение списка исключений - throw(...) 
    
    void f(int x) throw (std::logic_error) {
        int *p = new int[5]; // RUNTIME ERROR
        // new может бросить std::bad_alloc, а это не допускается
        // спецификацией исключений этого метода
        // упадет в рантайме

        try {
             int *p = new int[5];  // OK
             // т.к. std::bad_alloc не покидает метод.
        } catch(std::bad_alloc &e) {
            throw std::logic_error(e.what());
        } catch(...) {// Совсем OK, т.к. в реальности на new стоит
                      // throw(...) и он теоретически может бросить
                      // что угодно.
            throw std::logic_error("shit happens...");
        }
    }
};
{% endhighlight %}
Вообще использование exception specification считается не очень хорошей практикой, потому как использование может дать больше проблем, чем решить их. Очень подробно о этом [пишет](http://www.gotw.ca/publications/mill22.htm) Herb Sutter.

Теперь о mvsc - он вообще игнорит exception specification. Компилятор(по крайней мере так было до msvc2010 включительно) парсит синтаксис, успешно компилирует и лишь выдает подобного рода предупреждения:
    
    warning C4290: C++ exception specification ignored except to indicate a function is not __declspec(nothrow)
    
Если же хочется/необходимо сделать, так что бы явно запретить/указать что ф-ция не генерирует исключений, то можно воспользоваться нововведением стандарта С++11: ключевое слово **noexecpt**.
{% highlight c++ %}
// the function f() does not throw
void f() noexcept;
// fp points to a function that may throw
void (*fp)() noexcept(false);
// g takes a pointer to function that doesn't throw
void g(void pfa() noexcept);
typedef int (*pf)() noexcept; // error
{% endhighlight %}
