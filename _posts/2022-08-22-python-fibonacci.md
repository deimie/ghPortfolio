---
layout: single
title:  "Fibonacci Sequence"
date:   2024-08-22
categories: Python
---

Fibonacci sequence generator in Python.

{% highlight python %}

n = int(input('Type the nth term in the fibonacci sequence: '))

def fibonacci(n):
  a=0
  b=1
  fibList = [0]

  # there are no negative nth terms allowed
  if n <=0:
    print('Input must be greater than 0.')
    fibonacci() # restart the program

  # instead of using logic for the whole sequence, it's easier to make the 1st term always return 0.
  elif n == 1:
    print(fibList)
    return
  
  else:
    fibList.append(1)
    
    # for loop will go through each number of the sequence and append them into the list.
    for i in range(2,n):
      c = a + b
      a = b
      b = c
      fibList.append(c)
  print(fibList) # print final result
      
fibonacci(n)
{% endhighlight %}
