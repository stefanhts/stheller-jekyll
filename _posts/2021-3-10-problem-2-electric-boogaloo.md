---
layout: post
categories: posts
title: "Problem 2 Electric Boogaloo"
tags: [thesequelnoonesawcoming]
date-string: March 10 2021
---
# As promised, I have returned with the second coding problem
Not much has changed for me in the last day. I've attempted a couple Minecraft speedruns, and suffered from some pretty significant allergies.
Life is a pleasure as always.

## Anyways, on to the problem

## Problem 2:
> Good morning! Here's your coding interview problem for today.
This problem was asked by Uber.
Given an array of integers, return a new array such that each element at index i of the new array is the product of all the numbers in the original array except the one at i.
For example, if our input was [1, 2, 3, 4, 5], the expected output would be [120, 60, 40, 30, 24]. If our input was [3, 2, 1], the expected output would be [2, 3, 6].
<br/>
Follow-up: what if you can't use division?

## My solution:
Again we find ourselves with a fairly simple problem, although today's was tagged as [Hard] hm...
Anyway, we are being given an array and need to return an array which at each element contains the product of each of the
elements from the input array excluding the item at the current index. Once again we want to split the problem into 3 pieces.

### Input:
An array of integers we'll call ```arr```

### Goal:
Return an array with the product of the entire input array, excluding the element at the current index.

### Constraints:
Can we do this without division?

### Time to write some code
Anytime I see a challenge constraint I immediately try to solve the problem including that approach first, but for the sake of explanation I will start with
the easier version first. The first thing that comes to mind is to iterate through the list and then use nested iteration at each element to get the product. There
are a couple key issues with the approach. The first is that this over complicates the problem, we need to use our heads here and not just charge at the problem with
for-loops brandished. The second issue is that this algorithm would then have a time complexity of **O(n^2)** which, in my book, is much to large for this problem.

#### Let's start with approaching this from a math perspective
What is the easiest way to get the product excluding a particular element? If we really want to simplify our work we can just multiply each of the elements and then
everytime we want to get the value we simply divide the product by the element in question. The implementation should look something like this:
<br/>
First I'll just define my array
``` const arr = [3, 2, 1] ```
Next I'll define my product and division functions with arrow functions
``` javascript
// vvvvv Division vvvvv
const product = (lst) => { // define product function
  let prod = 1 // our product variable
  for (let num of lst) // loop through values in the array
    prod *= num // scale product
  return prod //return the product
}

const division = (lst) => { // define the division function
  let output = [] //define an output array
  const prod = product(lst) // get the total product by calling our previous function
  for (let num of lst) // increment through values in the array
    output.push(prod / num) // add product/element to output array
  return output //return array
}
console.log(`With division: ${division(arr)}`) // output: 2,3,6

```
The product function will take in our array and multiply each of the elements to give us a total product. Division will divide the product by the element at the current
index and add that value to the output array.

#### A little bit harder...
Now we want to try to do this without using division. I have not yet come up with a solution better than **O(n^2)**, but I'll keep it in the back of my mind and publish an
edit when I come up with one.
<br/><br/>
Let's figure out how we are going to structure this answer... We need to write a function which will take an array and and index to ignore when multiplying the values in
the array. That doesn't sound too hard, so let's do it.

``` javascript
const multiply = (lst, index) => { //define a function which takes an array and an index to ignore
  let prod = 1; //define a product value
  for (let i in lst) { //loop through indexes
    if (i === index) //if ignored index, don't multiply
      continue //skip loop iteration
    prod *= lst[i] //increase product
  }
  return prod //return product
}
```
Here we have a function which will loop through the array and anytime it is at the passed index it will simply skip that step and continue with
the process of multiplying the values. It will then return the product. Notice I made use of the ```continue``` keyword here, much to the chagrin
of all my past CS professors, but they can't take off style points here. HAHA! This could also have been accomplished by using an if statement or
even a ternary statement if you are really feeling feisty, but this is the implementation I chose. Anyway, we now just need a function which adds
each of these values to an output array, much like the first implementation.

``` javascript
const noDivision = (lst) => {
  let output = [] //define an empty output array
  for (let i in lst) //loop through indexes
    output.push(multiply(lst, i)) //append the result of multiply()
  return output //return our new array
}
console.log(`Answer without division: ${noDivision(arr)}`) // output: 2,3,6
```
And there we have it. A solution to the challenge portion of the problem, which returns the same values as the simpler implementation.
I guess I'll call this a succes if this is supposedly a **[Hard]** problem

So yeah... I'll see you in the next one

## Edit:
It has been brought to my attention that the challenge part of this question *can* be completed in **O(n)** by looping through the array
twice. The first time from front to back, assigning the value before calculating the product, and then doing the same thing again from
back to front. It should look something like this:

```javascript
const arr = [3,2,1]
const firstPass = (lst) => {
  let product = 1
  let output = []
  for(let num of lst){
    output.push(product) // add product to array
    product*=num // scale product
  }
  return output // return output
}

const secondPass = (lst) => {
  let product = 1
  for(let i = lst.length-1; i > -1; i--){ // loop through array backwards
    lst[i]*=product // multiply current value by the product
    product=arr[i] // scale the product
  }
  return lst //return outpur
}
console.log(secondPass(firstPass(arr))) // outputL 2,3,6
```

I can't claim credit for this algorithm, as I did not come up with it on my own, but I said I would update this post if
I found an **O(n)** solution, so here it is.

See ya in the next one...

