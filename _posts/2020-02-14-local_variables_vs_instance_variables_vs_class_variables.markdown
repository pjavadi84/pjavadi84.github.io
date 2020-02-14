---
layout: post
title:      "Local Variables vs. Instance Variables vs. Class Variables"
date:       2020-02-14 18:27:59 -0500
permalink:  local_variables_vs_instance_variables_vs_class_variables
---



When we are talking about different types of variable, one of the main things about this subject is to in what scope these variables are declared and can be accessible to do some work for us.  We have three types of variables: Local, Instance, and Class variables:


**Local Variable**, in short, is a variable that is defined inside a function definition and is scoped into only that ONE particular function. 

```
def sample_function_1
    x = 10;
end

def sampe_function_2
   puts x
end
```

in the above sampe_function method, x=1 ONLY can be defined within that function. sample_function_2 cannot access the sample_function_1 to prints the value of x that is previously defined in sampe_function_1. That makes it a local variable. 

The **instance variable** is variable that is accessible to all function definitions within that particular class.
```
def initialize
    @x= 25
end 

def function_example
    puts "#{@x}
end
```

The "@" sign inside the initialize method makes the variable x accessible to function_example method and when it is called it can prints 25 in the command line.

The **global variable** goes one step above instance variable and makes that variable accessible inside the entire program, classes of classes. It's like a universal resource accessible by all the objects and things that want to access it:

```
entire program:

class sample1 
    @@all = [1,2,3]
end

class sample2
   puts @@all
end

class sample3
   puts @@all
end
```

in the above, code @@ sign makes the "all" inside class sample1 accessible to class sample2, class sample 3, and more.


In conclusion, when we talking about these types of variables, one important consideration is where our variables can be used and where we don't want to mix them with other part of the operations inside of program in its entirity. 

