---
layout: post
title:  "Designing a python app to use plugins"
date:   2018-07-22 15:56:56 +0200
categories: python plugins
---

* TOC
{:toc}


# Intro

After solving a technical problem, I was asked to do it using plugins (it made sense). 
I was not able to give a good solution, I had some ideas, but I wasn't sure if any of them was going to work.

After that, I decided to research and learn how to do it. Using plugins make sense in some scenarios, it is not a  silver 
bullet, so be careful applying it. Most of the information here is from [André Roberge blog][André_Roberge-blog] 
and [lkubuntu blog][lkubuntu-blog]

The code can be found [here][github-code-sample]

# First approach (calculator without plugins)

We are going to do a console app (a calculator) and we are going to transform into a plugable calculator.

A basic class to simulate a calculator could be `calculator.py`: 


{% highlight python %}
class Calculator(object):

    def __init__(self, num_1, num_2, op):
        self.__num_1 = num_1
        self.__num_2 = num_2
        self.__op = op

    def __add(self):
        return self.__num_1 + self.__num_2

    def __subtract(self):
        return self.__num_1 - self.__num_2

    def __multiply(self):
        return self.__num_1 * self.__num_2

    def __divide(self):
        return self.__num_1 / self.__num_2

    def calc(self):
        if "+" == self.__op:
            return self.__add()
        elif "-" == self.__op:
            return self.__subtract()
        elif "*" == self.__op:
            return self.__multiply()
        elif "/" == self.__op:
            return self.__divide()
        else:
            return "Operation not supported"
{% endhighlight %}


This class receives 3 arguments, the first argument for the first number, the second argument for the second number 
and the third for the operation.

After the constructor we have all the methods related with the calculations per se.

Also we have a method calc to return the result.

{% highlight python %}
from calculator import Calculator

if __name__ == "__main__":
    print("*****Calculator*****")
    num_1 = float(input("First number: "))
    num_2 = float(input("Second number: "))
    op = input("Operator: ")

    calc = Calculator(num_1, num_2, op)
    print("Result {}".format(calc.calc()))
{% endhighlight %}

Main method gets the first and sencond operands, and the operator. After that it prints the result.

So this is a quite common way to solve this problem, but what happens if we want to add the square root? we have to 
add that operation as a new method, and handle it in the calc method.


# Second approach (calculator with plugins)


Another approach to solve this could be have a class to handle plugins, and each operation is a new pluggin, so we leave
the main code untouched.

For this approach we are going to use a base class `Plugin` that can be found inside the file `plugin.py` 

{% highlight python %}
class Plugin(object):
    op = None
{% endhighlight %}


All our plugins are going to inherit from that base class like this one, that can be found in `addition_plugin.py` :

{% highlight python %}
from plugins.plugin import Plugin


class AdditionPlugin(Plugin):
    op = '+'

    @staticmethod
    def calc(num_1, num_2):
        return num_1 + num_2
{% endhighlight %}

A basic class to simulate a calculator using plugins could be `plugable_calculator.py`: 
 
{% highlight python %}
from configparser import ConfigParser
from importlib import import_module
from os import listdir
from sys import path
from plugins.plugin import Plugin


class Calculator(object):

    @staticmethod
    def __get_plugins():
        config_parser = ConfigParser()
        config_parser.read('properties.ini')
        plugin_path = config_parser.get("paths", "plugins")
        plugin_files = [plugin_file[:-3] for plugin_file in listdir(plugin_path) if plugin_file.endswith(".py")]
        path.insert(0, plugin_path)
        for plugin_file in plugin_files:
            import_module(plugin_file)

    def __register_plugin(self):
        for plugin in Plugin.__subclasses__():
            self.__operations[plugin.op] = plugin

    def __init__(self, num_1, num_2, op):
        self.__operations = {}
        self.__num_1 = num_1
        self.__num_2 = num_2
        self.__op = op
        self.__get_plugins()
        self.__register_plugin()

    def calc(self):
        try:
            return self.__operations[self.__op].calc(self.__num_1, self.__num_2)
        except KeyError:
            return "Operation not supported"
{% endhighlight %}
 
 Here we have an auxiliary method which is going to read the configuration in order to get the folder containing the 
 plugins and loading them with the import_module method.
 
 After we have all modules we can scan for the ones that are inheriting from he Plugin class and we store it in a 
 dictionary to use them when necessary.
 
 Here the calc method is using rhe dictionary which the key is the operation and the value is the plugin, which 
 implements the calc method with the two operands.
 
 
{% highlight python %}
 from configparser import ConfigParser
 from importlib import import_module
 from os import listdir
 from sys import path
 from plugins.plugin import Plugin
 
 
 class Calculator(object):
 
     @staticmethod
     def __get_plugins():
         config_parser = ConfigParser()
         config_parser.read('properties.ini')
         plugin_path = config_parser.get("paths", "plugins")
         plugin_files = [plugin_file[:-3] for plugin_file in listdir(plugin_path) if plugin_file.endswith(".py")]
         path.insert(0, plugin_path)
         for plugin_file in plugin_files:
             import_module(plugin_file)
 
     def __register_plugin(self):
         for plugin in Plugin.__subclasses__():
             self.__operations[plugin.op] = plugin
 
     def __init__(self, num_1, num_2, op):
         self.__operations = {}
         self.__num_1 = num_1
         self.__num_2 = num_2
         self.__op = op
         self.__get_plugins()
         self.__register_plugin()
 
     def calc(self):
         try:
             return self.__operations[self.__op].calc(self.__num_1, self.__num_2)
         except KeyError:
             return "Operation not supported"
{% endhighlight %}

So now, each time we want to add a new operation, we only have to  add a new plugin to the folder using the same 
inheriting from the `Plugin` class and implementing the calc method.


The main method is the same we saw previously excep for the import statement.

{% highlight python %}
from plugable_calculator import Calculator

if __name__ == "__main__":
    print("*****Calculator*****")
    num_1 = float(input("First number: "))
    num_2 = float(input("Second number: "))
    op = input("Operator: ")

    calc = Calculator(num_1, num_2, op)
    print("Result {}".format(calc.calc()))
{% endhighlight %}




[André_Roberge-blog]: https://aroberge.blogspot.com/2008/12/plugins-part-1-application.html
[lkubuntu-blog]:      https://lkubuntu.wordpress.com/2012/10/02/writing-a-python-plugin-api/
[github-code-sample]: https://github.com/dionisioC/python_plugin_example