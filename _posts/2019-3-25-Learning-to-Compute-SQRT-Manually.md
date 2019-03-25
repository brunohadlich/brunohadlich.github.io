---
layout: post
title: Learning to Compute SQRT Manually
---

This post has the objective of explaining the solution I developed for a problem designed by a recruitment company recently,
basically the problem explained me one possible way of calculating square root by an iteractive method and then asked for
such method to be implemented in the language of my preference, I chose python for the sake of simplicity. Here is the code I wrote with
comments to make understanding easier:

```python
def sqrt(x):
        precision = 0.0000000001                        #The lower the precision number the more precise the result will be
        lower_bound = 0                                 #lower_bound is the lowest possible value for our result, 0 because the result will never be lower than that
        upper_bound = x                                 #upper_bound is the highest possible value for our result, the same as the entry as the square root will never be higher than that
        interval = upper_bound - lower_bound            #Calculate the difference between the lowest and highest possible values
        while interval > precision:                     #In case the difference between the possible lowest and highest values is higher than the precision, keep iterating
                result = lower_bound + (interval / 2)   #Find the number that is right between lowest and highest values
                power = result * result                 #Calculate the possible result to the power of 2
                if power > x:                           #In case it is greater than the entry number
                        upper_bound = result            #Use this result as upper_bound
                else:                                   #Otherwise
                        lower_bound = result            #Use this result as lower_bound
                interval = upper_bound - lower_bound    #Recalculate the interval
        return result

if __name__ == "__main__":
        while True:
                value = input()
                if value == "":
                        break
                print(round(sqrt(int(value)), 9))       #Round the result otherwise it may appear like 6.999999999987267 for entry 49
```
