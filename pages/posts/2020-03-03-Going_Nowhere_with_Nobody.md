---
title: 'Going Nowhere with Nobody'
layout: post
---

I wanted to find good examples of linked lists in Python on the net. Nothing really stood out to me so I just wrote a more complete example based of some others:

```
import logging
from logging.config import fileConfig

fileConfig('logging.ini')
logger = logging.getLogger()

class Node:
    def __init__(self, cargo=None, next=None):
        self.cargo = cargo
        self.next = next

    def __str__(self):
        return str(self.cargo)

    def pretty_print(self):
        if self.next != None:
            node = self.next
            node.pretty_print()
        if ord(self.cargo) % 16:
            print(f"\t{self.cargo}\t", end="")
        else:
            print(f"\n{self.cargo}")


class LinkedList:
    def __init__(self):
        self.length = 0
        self.head = None

    def print_list(self):
        if self.head != None:
            self.head.pretty_print()

    def add_first(self, cargo):
        node = Node(cargo)
        node.next = self.head
        self.head = node
        self.length = self.length + 1

    def search(self, search_node):
        current = self.head
        node_count = 0
        logger.debug(f"Started here: {current}")
        while current != None:
            if current.cargo == search_node:
                print(f"Done! Saw {node_count} node(s))")
                return current
            current = current.next
            node_count += 1
    
    def delete(self, node):
        if node == None or node.next == None:
            return False
        next = node.next
        node.cargo = next.cargo
        node.next = next.next
        return True

L = LinkedList()

[LinkedList.add_first(L, i) for i in "abcdef"]

LinkedList.print_list(L)
node = LinkedList.search(L, "c")
LinkedList.delete(L, node)
LinkedList.print_list(L)
```

I hope this code makes more sense to readers wanting to better understand linked lists in Python.
Please note that this a singly linked list. 

[This](https://medium.com/@kojinoshiba/data-structures-in-python-series-1-linked-lists-d9f848537b4d) write up by Kojin is also pretty helpful for a deeper dive.

Note too the use of the `logging` module. Read up on that @ [PSF (python.org)](https://docs.python.org/3/howto/logging.html#logging-basic-tutorial) 
