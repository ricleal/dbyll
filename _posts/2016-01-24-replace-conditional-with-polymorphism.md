---
layout: post
title: Replace Conditional With Polymorphism
excerpt: Replacing if/then/else with Polymorphism
categories: [Dev]
tags: [cleancode]
comments: true
---

Trying to convince my team to give up on if/then/else & case...

Let's play with some code!

### First the the non Object Oriented / procedural version:

{% highlight c++ %}
/*
 * non_oop.cpp
 *
 *  Created on: Sep 4, 2015
 *      Author: ferrazlealrm@ornl.gov
 *
 *  Non OOP example of URL:
 *  scheme:[//domain[:port]][/]path
 */

#include <iostream>
#include <string>

std::string domain = "ornl.gov";
std::string http_port = "80";
std::string https_port = "443";
std::string ftp_port = "20";

std::string build_url(const std::string &scheme,
  const std::string &domain,
  const std::string &port) {
 return scheme + "://" + domain + ":" + port;
}

int main(int argc, char* argv[]) {
 if (argc != 2) {
  std::cerr << "Use " + std::string(argv[0]) + " [http, https, ftp]"<<std::endl;
  exit(-1);
 }
 std::string scheme = argv[1];

 if (scheme == "http"){
  std::cout << build_url(scheme,domain,http_port) <<std::endl;
 }
 else if (scheme == "https"){
  std::cout << build_url(scheme,domain,https_port) <<std::endl;
 }
 else if (scheme == "ftp"){
  std::cout << build_url(scheme,domain,ftp_port) <<std::endl;
 }
 else {
  std::cerr << "Scheme not valid. Use of these: http, https, ftp"<<std::endl;
 }
 return 0;
}
{% endhighlight %}

### Finally the Object Oriented version:

{% highlight c++ %}
/*
 * oop.cpp
 *
 *  Created on: Sep 4, 2015
 *      Author: ferrazlealrm@ornl.gov
 *
 *  OOP example of URL:
 *  scheme:[//domain[:port]][/]path
 */

#include <iostream>
#include <string>
#include <map>

// Abstract class: cannot be instantiated
class URL {
protected:
 std::string domain = "ornl.gov";
 std::string port;
 std::string scheme;
 std::string build_url(const std::string &scheme,
   const std::string &port) const {
  return scheme + "://" + domain + ":" + port;
 }
public:
 // Pure virtual function: i.e. must be overridden by a derived class
 virtual std::string build_url() const = 0;
 ~URL() {};
};

class Http: public URL {
public:
 Http() {port = "80"; scheme = "http";}
 std::string build_url() const {
  return URL::build_url(scheme, port);
 }
};

class Https: public URL {
public:
 Https() {port = "443"; scheme = "https";}
 std::string build_url() const {
  return URL::build_url(scheme, port);
 }
};

class Ftp: public URL {
public:
 Ftp() {port = "20"; scheme = "ftp";}
 std::string build_url() const {
  return URL::build_url(scheme, port);
 }
};

/**
 * Main function
 */
std::string build_url(const URL &url) {
 return url.build_url();
}

int main(int argc, char* argv[]) {
 if (argc != 2) {
  std::cerr << "Use " + std::string(argv[0]) + " [http, https, ftp]"
    << std::endl;
  exit(-1);
 }
 std::string scheme = argv[1];

 // UGLY!!!
 // In real world I would have here some sort of Dependency Injection (e.g. factory)
 // This just shows that we can get an object given a string
 Http http;
 Https https;
 Ftp ftp;
 std::map<std::string, URL*> choices;
 choices.insert(std::make_pair("http", &http));
 choices.insert(std::make_pair("https", &https));
 choices.insert(std::make_pair("ftp", &ftp));

 if (scheme == "http" or scheme == "https" or scheme == "ftp") {
  std::cout <<  build_url(*choices[scheme]) << std::endl;
 } else {
  std::cerr << "Scheme not valid. Use of these: http, https, ftp"
    << std::endl;
 }
 return 0;
}
{% endhighlight %}

I know it looks complicated, but imagine you want to add a new functionality. Let's for example add a path to the HTTP url:

{% highlight c++ %}
/**
 * Let's imagine that I want to add a path to the URL: http://ornl.gov:80/
 * I don't need to modify the code! Open/closed principle.
 * Just extend it, i.e., add new code.
 */

class HttpWithPath: public Http {
protected:
 std::string path;
public:
 HttpWithPath(std::string path) : Http(), path(path) {}
 std::string build_url() const {
  std::string default_url = URL::build_url(scheme, port);
  return default_url + "/" + path;
 }
};
{% endhighlight %}



It does not violate the open/closed principle, which states:

>  "software entities should be open for extension, but closed for modification".
