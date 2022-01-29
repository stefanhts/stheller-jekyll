---
layout: post
categories: posts
title: "And We're Back"
tags: [thecomebackstory]
date-string: March 9 2021
---
# So yeah, I was gone, but now I'm back
I have been pretty busy with video games and school and external projects, but I think I'm back now. I plan on
making some pretty significant changes to this site in the near future, so stay tuned... :)

## A new series
I recently subscribed to a service [linked here](https://www.dailycodingproblem.com) which sends me a coding problem everyday. I've decided to blog about them here.
The way the service works, you start easy and it gets progressively harder, so for a while the problems are going to
be pretty easy, but I will be implementing all the solutions in JavaScript, which is a fairly new language for me, so this
will be a fun exercise.

## Problem 1:
> Good morning! Here's your coding interview problem for today. This problem was recently asked by Google. Given a list of numbers and a number k, return whether any two numbers from the list add up to k. For example, given [10, 15, 3, 7] and k of 17, return true since 10 + 7 is 17.

> Bonus: Can you do this in one pass?

## My solution:
To solve this fairly trivial problem I first want to break the problem down. What is the input? What is the output? What are the constraints?

### Input:
An array of integers we'll call ```arr```
An integer tarket we'll call ```k```

### Goal:
Return true if 2 numbers in ```arr``` add to ```k```, else return false

### Constraints:
Do this in one pass, and be as efficient as possible

### Time to write some code
I decided that my optimal solution would be one in which we only make 1 pass, as that would make the complexity **O(n)** which is acceptable.
I think the best way to do this is to use a JavaScript Set, which is the JS implementation of a hash set. We love hash sets because of their **O(1)**
lookup times.

First I'll just define my array
``` const arr = [10, 15, 3, 7] ```
Next I'll define my results function using a fancy JavaScript arrow function
``` javascript
const arr = [10, 15, 3, 7] //define array
const k = 17 // define constant

const results = (lst, k) => {
    let lookup = new Set() // define a set
    for(let num of lst){ // loop through array
        if(lookup.has(num)) // check set for value
            return true
        lookup.add(k-num) // add the missing value to set
    }
    return false
}

console.log(results(arr, k)) // output results
```
What I've done here is iterate through the array and check to see if the value in the list is in our set, if it is we return true.
But why does this work? It works because we are not adding the number to the set, instead we are adding (k-num) which means that when
we look at the set, we are only checking to see if our current value in the array is the value we need to add up to k.

And thus we have a **O(n)**, one pass algorithm written and done in Javascript.

On to the next...

