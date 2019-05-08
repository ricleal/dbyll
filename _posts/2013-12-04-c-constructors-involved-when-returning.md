---
layout: post
title: 'C++ : Constructors involved when returning values from functions'
date: '2013-12-04T04:48:00.001-05:00'
author: ricleal
tags:
- Programming
- C++
excerpt: '...'
comments: true
---

Trying to figure out how to return huge objects when they are created inside a function or changed inside a function (i.e. avoiding copy constructors).

Code and output should be easy to understand.

Code:


```C++
#include <iostream>

using namespace std;

class Dummy {
    int x;

public:
    Dummy() : x(0) { cout << "\t -> Default Constructor" << endl; }
    virtual ~Dummy() { cout << "\t -> desctructor" << endl; }
    Dummy(int i) : x(i) { cout << "\t -> Parameter Constructor" << endl; }
    Dummy(const Dummy &other) : x(other.x) {
    cout << "\t -> Copy Constructor" << endl;
    }
    // Two existing objects!
    Dummy &operator=(const Dummy &other) {
    x = other.x;
    cout << "\t -> Copy Assignment Operator" << endl;
    return *this;
    }
    // C++11
    Dummy(Dummy &&other) {
    x = std::move(other.x);
    cout << "\t -> C++11 Move Constructor" << endl;
    }
    // C++11
    Dummy &operator=(Dummy &&other) {
    x = std::move(other.x);
    cout << "\t -> C++11 Move Operator" << endl;
    return *this;
    }

    void setX(int x) { this->x = x; }

    friend ostream &operator<<(ostream &out, const Dummy &m);
};

ostream &operator<<(ostream &out, const Dummy &m) {
    out << "\t   -> Dummy.x=" << m.x << endl;
    return out;
}

Dummy fRetClassValue() { return Dummy(); }

Dummy fRetClassValue(int x) {
    Dummy d = Dummy(x);
    return d;
}

// Cant't return references to local variables
// Dummy& fRetClassRef() {
// return Dummy();
//}
//
// Dummy& fRetClassRef(int x) {
// Dummy d = Dummy(x);
// return d;
//}

/**
    * This will return a reference for the class passed by param
    * No copy constructors involved
    */
Dummy &fRetClassRef(Dummy &d) { return d; }

Dummy &fRetClassRef(Dummy &d, int x) {
    d.setX(x);
    return d;
}

/**
    * This will return a copy for the class passed by param
    * Copy constructors involved!
    */
Dummy fRetClassValue(Dummy &d) { return d; }

Dummy fRetClassValue(Dummy &d, int x) {
    d.setX(x);
    return d;
}

Dummy &f3(Dummy &m) {
    m.setX(21);
    return m;
}

int main(void) {
    cout << "1........................." << endl;
    Dummy d1 = fRetClassValue();
    cout << d1;
    Dummy d2 = fRetClassValue(2);
    cout << d2;

    cout << "1_2......................... (Need const to work!)" << endl;
    const Dummy &d1_2 = fRetClassValue();
    cout << d1_2;
    const Dummy &d2_2 = fRetClassValue(22);
    cout << d2_2;

    cout << "2........................." << endl;
    Dummy d3 = fRetClassRef(d1);
    cout << d3;
    Dummy d4 = fRetClassRef(d1, 4);
    cout << d4;

    cout << "2_2........................." << endl;
    Dummy &d3_2 = fRetClassRef(d1);
    cout << d3_2;
    Dummy &d4_2 = fRetClassRef(d1, 42);
    cout << d4_2;

    cout << "3........................." << endl;
    Dummy &d5 = fRetClassRef(d1);
    cout << d5;
    Dummy &d6 = fRetClassRef(d1, 6);
    cout << "d1" << d1;
    cout << "d6" << d6;
    d6.setX(123);
    cout << "d1" << d1;
    cout << "d6" << d6;

    cout << "4........................." << endl;
    const Dummy &d7 = fRetClassValue(d1);
    cout << d7;
    const Dummy &d8 = fRetClassValue(d1, 7);
    cout << "d1" << d1;
    cout << "d8" << d8;
    d6.setX(1234);
    cout << "d1" << d1;
    cout << "d8" << d8;

    cout << "END ........................." << endl;

    return 0;
}
```

Output:

```
1.........................
  -> Default Constructor
    -> Dummy.x=0
  -> Parameter Constructor
    -> Dummy.x=2
1_2......................... (Need const to work!)
  -> Default Constructor
    -> Dummy.x=0
  -> Parameter Constructor
    -> Dummy.x=22
2.........................
  -> Copy Constructor
    -> Dummy.x=0
  -> Copy Constructor
    -> Dummy.x=4
2_2.........................
    -> Dummy.x=4
    -> Dummy.x=42
3.........................
    -> Dummy.x=42
d1    -> Dummy.x=6
d6    -> Dummy.x=6
d1    -> Dummy.x=123
d6    -> Dummy.x=123
4.........................
  -> Copy Constructor
    -> Dummy.x=123
  -> Copy Constructor
d1    -> Dummy.x=7
d8    -> Dummy.x=7
d1    -> Dummy.x=1234
d8    -> Dummy.x=7
END .........................
  -> desctructor
  -> desctructor
  -> desctructor
  -> desctructor
  -> desctructor
  -> desctructor
  -> desctructor
  -> desctructor
```