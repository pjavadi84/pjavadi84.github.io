---
layout: post
title:      "SELF in different contexts"
date:       2020-03-30 18:28:47 +0000
permalink:  self_in_different_contexts
---


A subject of self has been somewhat confusing to me and to many. The "self" is simply a reference or an object that is being used to locate the code; By location, we mean in what scope the self is defined. If I have a blank page and type self, and try to print out self in command line, I will receive main. "main" is highest scope possible. Think about a world that is limited to earth. Earth is the highest level of scope; there is nothing higher than earth in this example. Now, self is earth. But then, if we have a continent method (def continent) and put self inside the continent method, the self is continent. Another example is, within a context of a module with bunch of methods inside of it (if you don't know about what module is, it is pretty act like a class):

```
module company
   self

   class accounting_department
	      self
	 end
	 
	 class marketing_department
	 
	 end

```

if we put self inside the class accounting_department, the self referencing to company::accounting_department and self inside the module company is referencing company. 

So, self can be helpful when you want your code and how you want your code needs to be scoped so it can be used in a proper operation. 
