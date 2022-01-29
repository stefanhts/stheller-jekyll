---
layout: post
categories: posts
title: "Today I take advantage of a cheeky bugger called JSON"
tags: [JSONforthewin]
date-string: March 11 2021
---
# Day 3 Problem 3
Today's problem takes a turn away from the past 2. Now, instead of having a problem involving array manipulation, we need to deserialize
and serialize methods for a binary tree. Here's the problem they give us:
> Given the root to a binary tree, implement serialize(root), which serializes the tree into a string, and deserialize(s), which deserializes the string back into the tree.
For example, given the following Node class

``` python
class Node:
    def __init__(self, val, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right
```
The following test should pass:
``` python
node = Node('root', Node('left', Node('left.left')), Node('right'))
assert deserialize(serialize(node)).left.left.val == 'left.left'
```

As I mentioned before, the goal of what I am doing with these problems is to sharpen my JavaScript skills, so while I could answer this problem in
Python (which is the language the class is given in), I would prefer to transfer it to JavaScript. So here is our linked list class redefined in
JavaScript:
``` javascript
class Node{
  constructor(value, left, right) {
    this.val = value;
    this.left = left;
    this.right = right;
  }
}
```
I am actually going to make some modifications to this declaration in order to solve the problem, but we'll get to that. In summary what we have here
is a basic binary search tree with each node having a value, left node, and a right node. Now it's time to break the problem down.

## What's the goal?
Be able to serialize and deserialize a tree with no loss of information

## What's the input?
The input is a tree which has been predefined for us, and we want to match it to the output of deserialize after it has been serialized

## How are we going to solve this?
Well, the way I would solve this if, I wasn't working in JavaScript, would be to recurse through the tree, adding the
"value" from the current node in the tree to a string with some character to seperate the values. I would the use a Regular
Expression to read the string and create the appropriate nodes, thus deserializing the string and recreating the tree.
But... because we're working in JavaScript, the language of reb requests and serialization, we can just use the built in
JSON functions.
<br/>
So let's take advantage of the JSON functions stringify() and parse. We can add a toString() function to the Node class
to make the code cleaner. The implamentation should look something like this:
``` javascript
class Node{ //define our node class
  constructor(value, left, right) {
    this.val = value;
    this.left = left;
    this.right = right;
  }

  toString() { //define a toString method which uses built in JSON.stringify
    return JSON.stringify(this)
  }
}

const serialize = (node) => node.toString() // serialize simply calls JSON.stringify on the object
const deserialize = (serializedNode) => JSON.parse(serializedNode) // deserialize calls JSON.parse() on the serialized string

let node = new Node( //create the input node
  "root",
  new Node("left", new Node("left.left")),
  new Node("right")
);

console.log(deserialize(serialize(node)).left.left.val) // outputs left.left
```

So what's happening here? We add a toString() to Node which simply returns the JSON.stringify() when called on our node.
We define a serialize const which is equal to the output of the toString() called on the node. We then define a const
deserialize which equals an arrow function which returns the result of JSON.parse() on the serialized string. This simply
returns a Node object. We can then create an instance of our Node class with the params given by the prompt. We log the output
and find that we get the exact output we want.

### Acknowledge some issues
This may seem like a bit of a cop out... We didn't actually write anything complex to handle this **[Medium]** problem, but
these functions are an integral part of JavaScript and learning JavaScript is the driving factor for this series, so I'm going
to allow it.

I'll see you in the next one... Maybe I'll actually write some code for that one.
