---
title: DS - pythonds3 - 6. Sorting and Searching
# author: Grace JyL
date: 2019-08-25 11:11:11 -0400
description:
excerpt_separator:
categories: [04CodeNote, PythonNote]
tags:
math: true
# pin: true
toc: true
# image: /assets/img/sample/devices-mockup.png
---

- [DS - pythonds3 - 6. Sorting and Searching](#ds---pythonds3---6-sorting-and-searching)
  - [Searching](#searching)
    - [The Sequential Search](#the-sequential-search)
      - [Sequential Search of an Unordered List](#sequential-search-of-an-unordered-list)
      - [Sequential Search of an ordered List](#sequential-search-of-an-ordered-list)
    - [The Binary Search](#the-binary-search)
    - [Hashing](#hashing)
      - [Hash Functions](#hash-functions)
      - [Collision Resolution](#collision-resolution)
      - [Collision Resolution with Linear Probing¶](#collision-resolution-with-linear-probing)
      - [Collision Resolution with Quadratic Probing](#collision-resolution-with-quadratic-probing)
      - [Collision Resolution with Chaining](#collision-resolution-with-chaining)
    - [Implementing the Map Abstract Data Type](#implementing-the-map-abstract-data-type)
      - [The complete hash table example](#the-complete-hash-table-example)
      - [Analysis of Hashing](#analysis-of-hashing)
  - [Sorting](#sorting)
    - [The Bubble Sort `𝑂(𝑛^2)`](#the-bubble-sort-𝑂𝑛2)
      - [bubbleSort in python](#bubblesort-in-python)
      - [analyze the bubble sort](#analyze-the-bubble-sort)
      - [the short bubble](#the-short-bubble)
    - [The Selection Sort `𝑂(𝑛^2)`](#the-selection-sort-𝑂𝑛2)
      - [Selection sort in python](#selection-sort-in-python)
      - [analyze the Selection sort](#analyze-the-selection-sort)
    - [The Insertion Sort `𝑂(𝑛^2)`](#the-insertion-sort-𝑂𝑛2)
      - [Insertion Sort in python](#insertion-sort-in-python)
      - [analyze the Insertion sort](#analyze-the-insertion-sort)
    - [The Shell Sort](#the-shell-sort)

---


# DS - pythonds3 - 6. Sorting and Searching

---

## Searching

Searching is the algorithmic process of finding a particular item in a collection of items.
- A search typically answers either `True` or `False` as to whether the item is present.
- On occasion it may be modified to return where the item is found.

In Python, there is a very easy way to ask whether an item is in a list of items.

use the `in` operator.

```py
>>> 15 in [3,5,2,4,1]
False
>>> 3 in [3,5,2,4,1]
True
>>>
```


many different ways to search for the item.

how these algorithms work and how they compare to one another.


### The Sequential Search

When data items are stored in a collection such as a list, they have a **linear or sequential relationship**.
- Each data item is stored in a position `relative to the others`.
- In Python lists, these relative positions are the `index` values of the individual items.
  - index values are ordered, possible for us to visit them in sequence.
  - This process gives rise to our first searching technique, the sequential search.

![seqsearch](https://i.imgur.com/crvHNDy.png)

- simply move from item to item,
- following the underlying sequential ordering until find it or run out of items.
- If run out of items, the item we were searching for was not present.

```py
def sequentialSearch(alist, item):
  pos = 0
  found = False

  while pos < len(alist) and not found:
    if alist[pos] == item:
      found = True
      else:
        pos = pos+1
  return found
testlist = [1, 2, 32, 8, 17, 19, 42, 13, 0]
print(sequentialSearch(testlist, 3))
print(sequentialSearch(testlist, 13))
```

#### Sequential Search of an Unordered List

![Screen Shot 2021-09-26 at 3.43.55 AM](https://i.imgur.com/hYcuowN.png)

To analyze searching algorithms, we need to decide on a **basic unit of computation**.
- typically the common step that must be repeated in order to solve the problem.
- For searching, it makes sense to count the number of comparisons performed.
- Each comparison may or may not discover the item we are looking for.
- In addition, we make another assumption here.
- The list of items is not ordered in any way.
- The items have been placed randomly into the list.
- In other words, the probability that the item we are looking for is in any particular position is exactly the same for each position of the list.

If the item is not in the list,
- the only way to know it is to **compare it against every item present**.
- If there are `𝑛` items, then the sequential search requires `𝑛` comparisons to discover that the item is not there.
- In the case where the item is in the list, the analysis is not so straightforward.
- There are actually three different scenarios that can occur.
  - best case, find the item in the first place, need only one comparison.
  - worst case, not discover the item until the very last comparison, the `nth` comparison.
  - the average case
    - On average, we will find the item about halfway into the list;
    - we will compare against `n/2` items.
    - however, that as `n` gets large, the coefficients, no matter what they are, become insignificant in our approximation,
    - so the complexity of the sequential search, is `𝑂(𝑛)`.
    - Table 1 summarizes these results.


#### Sequential Search of an ordered List


We assumed earlier that the items in our collection had been randomly placed, no relative order between the items.
- What would happen to the sequential search if the items were ordered in some way?
- Would we be able to gain any efficiency in our search technique?

Assume that the list of items was constructed so that the items were in ascending order, from low to high.
- If the item we are looking for is present in the list, the chance of it being in any one of the `n` positions is still the same as before.
- We will still have the same number of comparisons to find the item.
- However, if the item is not present there is a slight advantage.
  - the algorithm does not have to continue looking through all of the items to report that the item was not found. It can stop immediately.

![Screen Shot 2021-09-26 at 3.44.52 AM](https://i.imgur.com/FS7vkQZ.png)

- the best case we might discover that the item is not in the list by looking at only one item.
- On average, we will know after looking through only 𝑛/2 items.
- However, this technique is still 𝑂(𝑛).


In summary, a sequential search is improved by ordering the list only in the case where we do not find the item.

![Screen Shot 2021-09-26 at 3.43.55 AM](https://i.imgur.com/hYcuowN.png)
![Screen Shot 2021-09-26 at 3.44.52 AM](https://i.imgur.com/FS7vkQZ.png)

---


### The Binary Search

take greater advantage of the ordered list
- In the sequential search, compare against the first item, there are at most `𝑛−1` more items to look through if the first item is not what we are looking for.
- Instead of searching the list in sequence, a binary search will start by examining the middle item.
- If that item is the one we are searching for, we are done.
- If it is not the correct item, we can use the ordered nature of the list to eliminate half of the remaining items.
- If the item we are searching for is greater than the middle item, the entire lower half of the list as well as the middle item can be eliminated from further consideration.
- The item, if it is in the list, must be in the upper half.

We can then repeat the process with the upper half.
- Start at the middle item and compare it against what we are looking for.
- Again, we either find it or split the list in half, therefore eliminating another large part of our possible search space.

![binsearch](https://i.imgur.com/137Zfsb.png)

```py
def binarySearch(alist, item):
  first = 0
  last = len(alist)-1
  found = False

  while first<=last and not found:
    midpoint = (first + last)//2
    if alist[midpoint] == item:
      found = True
    else:
      if item < alist[midpoint]:
        last = midpoint-1
      else:
        first = midpoint+1
  return found

testlist = [0, 1, 2, 8, 13, 17, 19, 32, 42,]
print(binarySearch(testlist, 3))
print(binarySearch(testlist, 13))
```

great example of a `divide and conquer` strategy.
- Divide and conquer
- divide the problem into smaller pieces, solve the smaller pieces in some way, and then reassemble the whole problem to get the result.

When we perform a binary search of a list,
- we first check the middle item.
- If the item we are searching for is less than the middle item, we can simply perform a binary search of the left half of the original list.
- Likewise, if the item is greater, we can perform a binary search of the right half.
- Either way, this is a recursive call to the binary search function passing a smaller list.

```py
def binarySearch(alist, item):
  if len(alist) == 0:
    return False
  else:
    midpoint = len(alist)//2
    if alist[midpoint]==item:
      return True
    else:
      if item<alist[midpoint]:
        return binarySearch(alist[:midpoint],item)
      else:
        return binarySearch(alist[midpoint+1:],item)
testlist = [0, 1, 2, 8, 13, 17, 19, 32, 42,]
print(binarySearch(testlist, 3))
print(binarySearch(testlist, 13))
```


To analyze the binary search algorithm, we need to recall that each comparison eliminates about half of the remaining items from consideration.
- What is the maximum number of comparisons this algorithm will require to check the entire list?
- If we start with n items, about `𝑛/2` items will be left after the first comparison.
- After the second comparison, there will be about 𝑛/4. Then 𝑛/8, 𝑛/16, and so on.
- How many times can we split the list
- When we split the list enough times, we end up with a list that has just one item. Either that is the item we are looking for or it is not. Either way, we are done.
- The number of comparisons necessary to get to this point is i where `n/2^i = 1`.
- Solving for `i` gives us `𝑖=log𝑛`.
- The maximum number of comparisons is logarithmic with respect to the number of items in the list.
- Therefore, the binary search is `𝑂(log𝑛)`.

One additional analysis issue needs to be addressed. In the recursive solution shown above, the recursive call,
- uses the slice operator to create the left half of the list that is then passed to the next invocation (similarly for the right half as well).
- The analysis that we did above assumed that the slice operator takes `constant time`.
- However, we know that the slice operator in Python is actually `O(k)`.
- This means that the binary search using slice will not perform in strict logarithmic time.
- Luckily this can be remedied by passing the list along with the starting and ending indices.  

Even though a binary search is generally better than a sequential search, it is important to note that
- **for small values of n, the additional cost of sorting is probably not worth it.**
- In fact, we should always consider whether it is cost effective to take on the extra work of `sorting` to gain `searching` benefits.
- If we can sort once and then search many times, the cost of the sort is not so significant.
- However, for large lists, `sorting even once can be so expensive` that simply performing a sequential search from the start may be the best choice.


---


### Hashing

- Binary Search: make improvements by taking advantage of information about where items are stored in the collection with respect to one another.
  - For example, by knowing that a list was ordered, we could search in logarithmic time using a binary search.
- In this section we will attempt to go one step further by building a `data structure` that can be searched in `𝑂(1)` time.
- This concept is referred to as `hashing`.

In order to do this, we will need to know even more about where the items might be when we go to look for them in the collection.
- If every item is where it should be, then the search can use a single comparison to discover the presence of an item.
  - We will see, however, that this is typically not the case.

A hash table is a collection of items which are stored in such a way as to make it easy to find them later.
- Each position of the hash table, often called a `slot`, can hold an item and is named by an integer value starting at 0.
- For example, we will have a slot named 0, a slot named 1, a slot named 2, and so on.
- Initially, the hash table contains no items so every slot is empty.
- We can implement a hash table by using a list with each element initialized to the special Python value `None`.  

![hashtable](https://i.imgur.com/kBYvacm.png)

`The mapping between an item and the slot` where that item belongs in the hash table is called the `hash function`.
- The hash function will take any item in the collection and return an integer in the range of slot names, between 0 and m-1.
- Assume that we have the set of integer items 54, 26, 93, 17, 77, and 31.
- Our first hash function, sometimes referred to as the “remainder method,” simply takes an item and divides it by the table size, returning the remainder as its hash value (ℎ(𝑖𝑡𝑒𝑚)=𝑖𝑡𝑒𝑚%11).
- Table 4 gives all of the hash values for our example items.
- Note that this remainder method (modulo arithmetic) will typically be present in some form in all hash functions, since the result must be in the range of slot names.

- Once the hash values have been computed, insert each item into the hash table at the designated position
- Note that 6 of the 11 slots are now occupied.
- This is referred to as the `load factor`, and is commonly denoted by `𝜆=𝑛𝑢𝑚𝑏𝑒𝑟𝑜𝑓𝑖𝑡𝑒𝑚𝑠 / 𝑡𝑎𝑏𝑙𝑒𝑠𝑖𝑧𝑒`. For this example, `𝜆=6/11`.

- to search for an item,
  - simply use the hash function to compute the slot name for the item
  - and then check the hash table to see if it is present.
  - This searching operation is 𝑂(1),
  - since a constant amount of time is required to `compute the hash value` and then `index the hash table at that location`.
  - If everything is where it should be, we have found a constant time search algorithm.

You can probably already see that this technique is going to work only if each item maps to a unique location in the hash table.
- For example, if the item 44 had been the next item in our collection, it would have a hash value of 0 (44%11==0).
- Since 77 also had a hash value of 0, we would have a problem.
- According to the hash function, two or more items would need to be in the same slot.
- This is referred to as a `collision/clash`


#### Hash Functions
- Given a collection of items, a hash function that `maps each item into a unique slot` is referred to as a perfect hash function.
- If we know the items and the collection will never change, then it is possible to construct a perfect hash function (refer to the exercises for more about perfect hash functions).
- Unfortunately, given an arbitrary collection of items, there is no systematic way to construct a perfect hash function.
- Luckily, we do not need the hash function to be perfect to still gain performance efficiency.

to always have a perfect hash function
- `increase the size of the hash table` so that each possible value in the item range can be accommodated.
- This guarantees that each item will have a unique slot.
- Although this is practical for small numbers of items, it is not feasible when the number of possible items is large.
- For example, if the items were nine-digit Social Security numbers, this method would require almost one billion slots.
- If we only want to store data for a class of 25 students, we will be wasting an enormous amount of memory.

Our goal is to create a hash function that
- `minimizes the number of collisions`,
- easy to compute,
- evenly distributes the items in the hash table.



**remainder method**
- The folding method for constructing hash functions begins by dividing the item into `equal-size pieces` (the last piece may not be of equal size).
- These pieces are then added together to give the resulting hash value.
- For example
- item was the phone number 436-555-4601.
- we would take the digits and divide them into groups of 2 (43,65,55,46,01).
- After the addition, 43+65+55+46+01, we get 210.
- If we assume our hash table has 11 slots, then we need to perform the extra step of dividing by 11 and keeping the remainder.
- In this case 210 % 11 is 1, so the phone number 436-555-4601 hashes to slot 1.
- Some folding methods go one step further and reverse every other piece before the addition.
- For the above example, we get 43+56+55+64+01=219 which gives 219 % 11=10.

**mid-square method**
- Another numerical technique for constructing a hash function is called the `mid-square method`.
- We first square the item, and then extract some portion of the resulting digits.
- For example, if the item were 44, we would first compute 44^2=1,936.
- By extracting the middle two digits, 93, and performing the remainder step, we get 5 (93 % 11).


the remainder method and the mid-square method.
- You should verify that you understand how these values were computed.

![Screen Shot 2021-09-26 at 4.37.38 AM](https://i.imgur.com/9OrXzlU.png)

We can also create hash functions for character-based items such as strings.
- The word “cat” can be thought of as a sequence of ordinal values.

```py
>>> ord('c')
99
>>> ord('a')
97
>>> ord('t')
116
```

- take these three ordinal values,
- add them up,
- and use the `remainder` method to get a hash value


![stringhash](https://i.imgur.com/yJZZee2.png)

```py
def hash(astring, tablesize):
    sum = 0
    for pos in range(len(astring)):
        sum = sum + ord(astring[pos])
    return sum%tablesize
```

It is interesting to note that when using this hash function,
- anagrams will always be given the same hash value.
- To remedy this, we could use the position of the character as a weight.
- Figure 7 shows one possible way to use the positional value as a weighting factor.
- The modification to the hash function is left as an exercise.

![stringhash2](https://i.imgur.com/hCZwbS8.png)

You may be able to think of a number of additional ways to compute hash values for items in a collection.
- The important thing to remember is that the hash function has to be `efficient` so that it does not become the dominant part of the storage and search process.
- If the hash function is too complex, then it becomes more work to compute the slot name than it would be to simply do a basic sequential or binary search as described earlier.
- This would quickly defeat the purpose of hashing.



#### Collision Resolution

When two items hash to the same slot,
- we must have a systematic method for placing the second item in the hash table.
- This process is called `collision resolution`.
- if the hash function is perfect, collisions will never occur.
- However, since this is often not possible, collision resolution becomes a very important part of hashing.

One method for resolving collisions

#### Collision Resolution with Linear Probing¶
- looks into the hash table and tries to find another open slot to hold the item that caused the collision.
- `start at the original hash value position` and then `move in a sequential manner through the slots until we encounter the first slot that is empty`.
- Note that we may need to go back to the first slot (circularly) to cover the entire hash table.
- This collision resolution process is referred to as open addressing in that it tries to find the next open slot or address in the hash table.
- By systematically visiting each slot one at a time, we are performing an open addressing technique called linear probing.


![hashtable2](https://i.imgur.com/5s3n6tV.png)

![linearprobing1](https://i.imgur.com/f3odMnr.png)

an extended set of integer items under the simple `remainder` method hash function (54,26,93,17,77,31,44,55,20).
- When we attempt to place 44 into slot 0, a collision occurs.
- Under linear probing, look sequentially, slot by slot, until we find an open position slot 1.
- Again, 55 should go in slot 0 but must be placed in slot 2 since it is the next open position.
- The final value of 20 hashes to slot 9. Since slot 9 is full, do `linear probing`, finally find an empty slot at position 3.


**search for items**
Once we have built a hash table using open addressing and linear probing, it is essential that we utilize the same methods to search for items.

to look up the item 93.
- When we compute the hash value, we get 5.
- Looking in slot 5 reveals 93, and we can return True.

looking for 20?
- Now the hash value is 9, and slot 9 is currently holding 31.
- We cannot simply return False since we know that there could have been collisions.
- We are now forced to do a sequential search, starting at position 10, looking until either we find the item 20 or we find an empty slot.


**clustering**
- A disadvantage to linear probing is the `tendency` for `clustering` 聚集;
- items become clustered in the table.
- This means that if many collisions occur at the same hash value, a number of surrounding slots will be filled by the linear probing resolution.
- This will have an impact on other items that are being inserted, as we saw when we tried to add the item 20 above.
- A cluster of values hashing to 0 had to be skipped to finally find an open position


- One way to deal with clustering
- extend the linear probing technique, instead of looking sequentially for the next open slot, we skip slots, thereby more evenly distributing the items that have caused collisions.
- This will potentially reduce the clustering that occurs.
- Figure 10 shows the items when collision resolution is done with a “plus 3” probe.
- This means that once a collision occurs, we will look at every third slot until we find one that is empty.

![linearprobing2](https://i.imgur.com/HdixzUR.png)


**rehashing**
- The general name for this process of looking for another slot after a collision is rehashing.
- With simple `linear probing`, the rehash function is
  - newhashvalue = rehash(oldhashvalue)
  - rehash(pos) = (pos+1)%table_size
- The “plus 3” rehash can be defined as
  - rehash(pos) = (pos+3)%table_size
- In general
  - rehash(pos) = (pos+skip)%table_size
- It is important to note that the size of the “skip” must be such that all the slots in the table will eventually be visited.
- Otherwise, part of the table will be unused.
- To ensure this, it is often suggested that the table size be a prime number.
- This is the reason we have been using 11 in our examples.


#### Collision Resolution with Quadratic Probing
- A variation of the linear probing idea is called quadratic probing.
- Instead of using a constant “skip” value, we use a `rehash function` that `increments the hash value by 1, 3, 5, 7, 9, and so on`.
- This means that if the first hash value is `h`, the successive values are `ℎ+1, ℎ+4, ℎ+9, ℎ+16`, and so on.
- In general, the i will be `i^2 𝑟𝑒ℎ𝑎𝑠ℎ(𝑝𝑜𝑠)=(ℎ+𝑖^2)`.
- In other words, quadratic probing uses a skip consisting of successive perfect squares.
- Figure 11 shows our example values after they are placed using this technique.

![quadratic](https://i.imgur.com/jUuUlbe.png)


#### Collision Resolution with Chaining

- An alternative method for handling the collision problem is to allow each slot to hold a reference to a collection (or chain) of items.
- Chaining allows many items to exist at the same location in the hash table.
- When collisions happen, the item is still placed in the proper slot of the hash table.
- As more and more items hash to the same location, the difficulty of searching for the item in the collection increases.

![chaining](https://i.imgur.com/tC4Hhw8.png)

When we want to search for an item, we use the hash function to generate the slot where it should reside.
- Since each slot holds a collection, we use a searching technique to decide whether the item is present.
- The advantage is that on the average there are likely to be many fewer items in each slot, so the search is perhaps more efficient. We will look at the analysis for hashing at the end of this section.

---


### Implementing the Map Abstract Data Type

dictionary is an associative data type where you can store key–data pairs.
- The key is used to look up the associated data value.
- We often refer to this idea as a map.

The map abstract data type is defined as follows.
- The structure is an unordered collection of associations between a key and a data value.
- The keys in a map are all unique so that there is a one-to-one relationship between a key and a value.
- The operations are given below.

- `Map()`
  - Create a new, empty map. It returns an empty map collection.

- `put(key,val)`
  - Add a new key-value pair to the map. If the key is already in the map then replace the old value with the new value.

- `get(key)`
  - Given a key, return the value stored in the map or None otherwise.

- `del`
  - Delete the key-value pair from the map using a statement of the form del map[key].

- `len()`
  - Return the number of key-value pairs stored in the map.
- `in`
  - Return True for a statement of the form key in map, if the given key is in the map, False otherwise.



One of the great benefits of a dictionary is the fact that given a key, we can look up the associated data value very quickly.
- In order to provide this fast look up capability, we need an implementation that supports an efficient search.
- We could use a list with `sequential or binary search` but it would be even better to use a hash table as described above since looking up an item in a hash table can approach `𝑂(1)` performance.

use two lists to create a `HashTable` class that implements the `Map abstract data type`.
- One list, called `slots`, will hold the key items
- a parallel list, called `data`, will hold the data values.
- When we look up a key, the corresponding position in the data list will hold the associated data value.
- We will treat the key list as a hash table using the ideas presented earlier.
- Note that the initial size for the hash table has been chosen to be 11.
- Although this is arbitrary, it is important that the size be a prime number so that the collision resolution algorithm can be as efficient as possible.


```py
class HashTable:
    def __init__(self):
        self.size = 11
        self.slots = [None] * self.size
        self.data = [None] * self.size
```

hashfunction implements the simple `remainder` method.
- The `collision resolution technique` is `linear probing` with a `“plus 1”` rehash function.
- The `put` function
  - assumes that there will eventually be an empty slot unless the key is already present in the `self.slots`.
  - It computes the original hash value
  - and if that slot is not empty, iterates the rehash function until an empty slot occurs.
  - If a nonempty slot already contains the key, the old data value is replaced with the new data value.
  - Dealing with the situation where there are no empty slots left is an exercise.


```py
def put(self,key,data):
  hashvalue = self.hashfunction(key,len(self.slots))

  if self.slots[hashvalue] == None:
    self.slots[hashvalue] = key
    self.data[hashvalue] = data
  else:
    if self.slots[hashvalue] == key:
      self.data[hashvalue] = data  #replace
    else:
      nextslot = self.rehash(hashvalue,len(self.slots))
      while self.slots[nextslot] != None and \
                      self.slots[nextslot] != key:
        nextslot = self.rehash(nextslot,len(self.slots))

      if self.slots[nextslot] == None:
        self.slots[nextslot]=key
        self.data[nextslot]=data
      else:
        self.data[nextslot] = data #replace

def hashfunction(self,key,size):
     return key%size

def rehash(self,oldhash,size):
    return (oldhash+1)%size
```

- get function
  - begins by computing the initial hash value.
  - If the value is not in the initial slot, rehash is used to locate the next possible position.
  - Notice that line 15 guarantees that the search will terminate by checking to make sure that we have not returned to the initial slot.
  - If that happens, we have exhausted all possible slots and the item must not be present.

The final methods of the HashTable class provide additional dictionary functionality.
- We overload the `__getitem__` and `__setitem__` methods to allow access using``[]``.
- This means that once a HashTable has been created, the familiar index operator will be available.
- We leave the remaining methods as exercises.


```py
def get(self,key):
  startslot = self.hashfunction(key,len(self.slots))

  data = None
  stop = False
  found = False
  position = startslot
  while self.slots[position] != None and not found and not stop:
     if self.slots[position] == key:
       found = True
       data = self.data[position]
     else:
       position=self.rehash(position,len(self.slots))
       if position == startslot:
           stop = True
  return data

def __getitem__(self,key):
    return self.get(key)

def __setitem__(self,key,data):
    self.put(key,data)
```


The following session shows the HashTable class in action.
- First we will create a hash table and store some items with integer keys and string data values.

```py
>>> H=HashTable()
>>> H[54]="cat"
>>> H[26]="dog"
>>> H[93]="lion"
>>> H[17]="tiger"
>>> H[77]="bird"
>>> H[31]="cow"
>>> H[44]="goat"
>>> H[55]="pig"
>>> H[20]="chicken"
>>> H.slots
[77, 44, 55, 20, 26, 93, 17, None, None, 31, 54]
>>> H.data
['bird', 'goat', 'pig', 'chicken', 'dog', 'lion', 'tiger', None, None, 'cow', 'cat']

# Next we will access and modify some items in the hash table.
# Note that the value for the key 20 is being replaced.

>>> H[20]
'chicken'
>>> H[17]
'tiger'
>>> H[20]='duck'
>>> H[20]
'duck'
>>> H.data
['bird', 'goat', 'pig', 'duck', 'dog', 'lion', 'tiger', None, None, 'cow', 'cat']
>> print(H[99])
None
```

#### The complete hash table example

```py
class HashTable:
    def __init__(self):
        self.size = 11
        self.slots = [None] * self.size
        self.data = [None] * self.size

    def put(self,key,data):
      hashvalue = self.hashfunction(key,len(self.slots))

      if self.slots[hashvalue] == None:
        self.slots[hashvalue] = key
        self.data[hashvalue] = data
      else:
        if self.slots[hashvalue] == key:
          self.data[hashvalue] = data  #replace
        else:
          nextslot = self.rehash(hashvalue,len(self.slots))
          while self.slots[nextslot] != None and \
                          self.slots[nextslot] != key:
            nextslot = self.rehash(nextslot,len(self.slots))

          if self.slots[nextslot] == None:
            self.slots[nextslot]=key
            self.data[nextslot]=data
          else:
            self.data[nextslot] = data #replace

    def hashfunction(self,key,size):
         return key%size

    def rehash(self,oldhash,size):
        return (oldhash+1)%size

    def get(self,key):
      startslot = self.hashfunction(key,len(self.slots))

      data = None
      stop = False
      found = False
      position = startslot
      while self.slots[position] != None and  \
                           not found and not stop:
         if self.slots[position] == key:
           found = True
           data = self.data[position]
         else:
           position=self.rehash(position,len(self.slots))
           if position == startslot:
               stop = True
      return data

    def __getitem__(self,key):
        return self.get(key)

    def __setitem__(self,key,data):
        self.put(key,data)

H=HashTable()
H[54]="cat"
H[26]="dog"
H[93]="lion"
H[17]="tiger"
H[77]="bird"
H[31]="cow"
H[44]="goat"
H[55]="pig"
H[20]="chicken"
print(H.slots)
print(H.data)

print(H[20])

print(H[17])
H[20]='duck'
print(H[20])
print(H[99])
```

---


#### Analysis of Hashing

best case hashing would provide a `𝑂(1)`, constant time search technique.
- However, due to collisions, the number of comparisons is typically not so simple.
- Even though a complete analysis of hashing is beyond the scope of this text, we can state some well-known results that approximate the number of comparisons necessary to search for an item.

The most important piece of information we need to analyze the use of a hash table is the `load factor`, `𝜆`.
- Conceptually, if `𝜆` is small, then there is a lower chance of collisions, meaning that items are more likely to be in the slots where they belong.
- If `𝜆` is large, meaning that the table is filling up, then there are more and more collisions.
- This means that collision resolution is more difficult, requiring more comparisons to find an empty slot.
- With chaining, increased collisions means an increased number of items on each chain.

Result for both a successful and an unsuccessful search.
- For a successful search using `open addressing` with `linear probing`
  - the average number of comparisons is approximately `1/2(1+1/(1−𝜆))`
  - and an unsuccessful search gives `1/2 (1+ (1/(1−𝜆))^2 )`
- If we are using `chaining`,
  - the average number of comparisons is `1+𝜆/2` for the successful case,
  - and simply `𝜆` comparisons if the search is unsuccessful.

---


## Sorting
Sorting
- the process of placing elements from a collection in some kind of order.
  - For example,
  - a list of words could be sorted alphabetically or by length.
  - A list of cities could be sorted by population, by area, or by zip code.
- We have already seen a number of algorithms that were able to benefit from having a sorted list (recall the final anagram example and the binary search).

There are many, many sorting algorithms that have been developed and analyzed. This suggests that sorting is an important area of study in computer science.
- Sorting a large number of items can take a substantial amount of computing resources.
- Like searching, the efficiency of a sorting algorithm is related to the number of items being processed.
- For small collections, a complex sorting method may be more trouble than it is worth. The overhead may be too high.
- On the other hand, for larger collections, we want to take advantage of as many improvements as possible.


the operations to analyze a sorting process.
- First, it will be necessary to compare two values to see which is smaller (or larger).
  - In order to sort a collection, it will be necessary to have some systematic way to compare values to see if they are out of order.
  - The `total number of comparisons` will be the most common way to measure a sort procedure.
- Second, when values are not in the correct position with respect to one another, it may be necessary to exchange them.
  - This exchange is a costly operation and the total number of exchanges will also be important for evaluating the overall efficiency of the algorithm.



---


### The Bubble Sort `𝑂(𝑛^2)`

The bubble sort makes multiple passes through a list.
- It compares adjacent items and exchanges those that are out of order.
- Each pass through the list places the next largest value in its proper place.
- In essence, each item “bubbles” up to the location where it belongs.

![bubblepass](https://i.imgur.com/IYs8zcP.png)

first pass:
- n items in the list, `𝑛−1` pairs of items that need to be compared
- It is important to note that once the largest value in the list is part of a pair, it will continually be moved along until the pass is complete.

second pass:
- the largest value is now in place.
- There are `𝑛−1` items left to sort, meaning that there will be `𝑛−2` pairs.

Since each pass places the next largest value in place, the total number of passes necessary will be `𝑛−1`.

After completing the 𝑛−1 passes, the smallest item must be in the correct position with no further processing required.


#### bubbleSort in python
- It takes the list as a parameter, and modifies it by exchanging items as necessary.

```py
def bubbleSort(alist):
    for passnum in range(len(alist)-1,0,-1):

        for i in range(passnum):
            if alist[i]>alist[i+1]:
                temp = alist[i]
                alist[i] = alist[i+1]
                alist[i+1] = temp

alist = [54,26,93,17,77,31,44,55,20]
bubbleSort(alist)
print(alist)
```

The exchange operation, “swap,”

```py
temp = alist[i]
alist[i] = alist[j]
alist[j] = temp
```

- is slightly different in Python than in most other programming languages.
- Typically, swapping two elements in a list requires a `temporary storage location (an additional memory location)`.
- A code fragment will exchange the ith and jth items in the list.
- Without the temporary storage, one of the values would be overwritten.

In Python, it is possible to perform **simultaneous assignment**.
- The statement `a,b=b,a` will result in two assignment statements being done at the same time
- Using simultaneous assignment, the exchange operation can be done in one statement.

---

#### analyze the bubble sort

> the number of comparisons for each pass.

![Screen Shot 2021-09-27 at 1.54.47 AM](https://i.imgur.com/i0u3HYx.png)

> the sum of the first `n` integers: `n+(n-1)+(n-2)+...+1`
> - $$= \frac{n}{2} *(n+1)$$
> - $$= \frac{1}{2} *n^2 + \frac{1}{2} * n$$
> - $$= \frac{1}{2} n^{2} + \frac{1}{2} n$$

the sum of the first `n-1` integers: `(n-1)+(n-2)+...+1`
- $$= \frac{1}{2} n^{2} + \frac{1}{2} n - n$$
- $$= \frac{1}{2} n^{2} - \frac{1}{2}n$$


**Time**
- **worse case**:
  - every comparison will cause an exchange.
  - regardless of how the items are arranged in the initial list,
  - `𝑛−1` passes is needs to sort a list of size `n`.
  - The total number of **comparisons** is the sum of the first `𝑛−1 integers`.
  - $$= \frac{1}{2} n^{2} - \frac{1}{2}n$$
  - still <font color=red> 𝑂(𝑛&2) </font> comparisons,
- **best case**,
  - the list is already ordered,
  - no exchanges will be made.
- On average, we exchange half of the time.

bubble sort is often considered the **most inefficient sorting** method
- **inefficient**
  - since it must exchange items before the final location is known.
  - These “wasted” exchange operations are very costly.
- However, because the bubble sort **makes passes through the entire unsorted portion of the list**, it has the capability to do something most sorting algorithms cannot.
  - In particular, if during a pass there are no exchanges, then we know that the list must be sorted.
  - A bubble sort can be modified to stop early if it finds that the list has become sorted. This means that for lists that require just a few passes, a bubble sort may have an advantage in that it will recognize the sorted list and stop.

---

#### the short bubble

```py
def shortBubbleSort(alist):
    exchanges = True
    passnum = len(alist)-1
    while passnum > 0 and exchanges:
       exchanges = False
       for i in range(passnum):
           if alist[i]>alist[i+1]:
               exchanges = True
               temp = alist[i]
               alist[i] = alist[i+1]
               alist[i+1] = temp
       passnum = passnum-1

alist=[20,30,40,90,50,60,70,80,100,110]
shortBubbleSort(alist)
print(alist)
```

---

### The Selection Sort `𝑂(𝑛^2)`

- The selection sort improves on the bubble sort by `making only one exchange for every pass through the list`.
  - selection sort `looks for the largest value` as it makes a pass
  - and, after completing the pass, places it in the proper location.
- As with a bubble sort, after the first pass, the largest item is in the correct place.
- After the second pass, the next largest is in place.
- This process continues and requires `𝑛−1` passes to sort n items, since the final item must be in place after the (𝑛−1) st pass.

![selectionsortnew](https://i.imgur.com/ninnEUi.png)

![0_J2ta_c6YA870MKor](https://i.imgur.com/vIUtyfF.jpg)

#### Selection sort in python
```py
def selectionSort(alist):
   for fillslot in range(len(alist)-1,0,-1):
       positionOfMax=0
       for location in range(1,fillslot+1):
           if alist[location]>alist[positionOfMax]:
               positionOfMax = location
       temp = alist[fillslot]
       alist[fillslot] = alist[positionOfMax]
       alist[positionOfMax] = temp

alist = [54,26,93,17,77,31,44,55,20]
selectionSort(alist)
print(alist)
```


#### analyze the Selection sort

You may see that the selection sort makes the same number of comparisons as the bubble sort and is therefore also `𝑂(𝑛^2)`.
- However, due to the reduction in the number of exchanges, the selection sort typically executes faster in benchmark studies.
- In fact, for our list, the bubble sort `makes 20 exchanges`, while the selection sort `makes only 8`.
- `make change != compare change`

---


### The Insertion Sort `𝑂(𝑛^2)`

- It always maintains a sorted sublist in the lower positions of the list.
- Each new item is then “inserted” back into the previous sublist such that the sorted sublist is one item larger.

![insertionsort](https://i.imgur.com/96gXhrx.png)

- We begin by assuming that a list with one item (position 0) is already sorted.
- On each pass, one for each item 1 through `𝑛−1`, the current item is checked against those in the already sorted sublist.
- we shift those items that are greater to the right. When we reach a smaller item or the end of the sublist, the current item can be inserted.

![insertionpass](https://i.imgur.com/s9Wm7lx.png)

the fifth pass in detail.
- At this point in the algorithm, a sorted sublist of five items consisting of 17, 26, 54, 77, and 93 exists.
- insert 31 back into the already sorted items.
- The first comparison against 93 causes 93 to be shifted to the right.
- 77 and 54 are also shifted.
- When the item 26 is encountered, the shifting process stops and 31 is placed in the open position.
- Now we have a sorted sublist of six items.


#### Insertion Sort in python

```py
def insertionSort(alist):
  for index in range(1,len(alist)):
    currentvalue = alist[index]
    position = index
    prevalue = alist[position-1]

    while position>0 and alist[position-1]>currentvalue:
      alist[position]=alist[position-1]
      position = position-1
    alist[position]=currentvalue

alist = [54,26,93,17,77,31,44,55,20]
insertionSort(alist)
print(alist)
```


#### analyze the Insertion sort

- `𝑛−1` passes to sort `n` items.
- The iteration starts at position 1 and moves through position `𝑛−1`, as these are the items that need to be inserted back into the sorted sublists.
- Line 8 performs the shift operation that moves a value up one position in the list, making room behind it for the insertion. Remember that this is not a complete exchange as was performed in the previous algorithms.

<font color=red> worse case </font>:

- The maximum number of comparisons for an insertion sort is the sum of the `first 𝑛−1 integers`. 𝑂(𝑛^2).

<font color=red> best case </font>:

- However, in the best case, only `one comparison needs to be done on each pass`.
- This would be the case for an already sorted list.

One note about `shifting` versus `exchanging` is also important.
- In general, a shift operation requires approximately a third of the processing work of an exchange since only one assignment is performed.
- In benchmark studies, <font color=red> insertion sort will show very good performance </font>.


---

### The Shell Sort

- sometimes called the “diminishing increment sort,”
- improves on the insertion sort by
  - breaking the original list into a number of smaller sublists,
  - each of which is sorted using an insertion sort.
- The unique way that these sublists are chosen is the key to the shell sort. Instead of breaking the list into sublists of contiguous items, the shell sort uses an increment i, sometimes called the gap, to create a sublist by choosing all items that are i items apart.

This can be seen in Figure 6. This list has nine items. If we use an increment of three, there are three sublists, each of which can be sorted by an insertion sort. After completing these sorts, we get the list shown in Figure 7. Although this list is not completely sorted, something very interesting has happened. By sorting the sublists, we have moved the items closer to where they actually belong.

../_images/shellsortA.png
Figure 6: A Shell Sort with Increments of Three

../_images/shellsortB.png
Figure 7: A Shell Sort after Sorting Each Sublist

Figure 8 shows a final insertion sort using an increment of one; in other words, a standard insertion sort. Note that by performing the earlier sublist sorts, we have now reduced the total number of shifting operations necessary to put the list in its final order. For this case, we need only four more shifts to complete the process.

../_images/shellsortC.png
Figure 8: ShellSort: A Final Insertion Sort with Increment of 1

../_images/shellsortD.png
Figure 9: Initial Sublists for a Shell Sort

We said earlier that the way in which the increments are chosen is the unique feature of the shell sort. The function shown in ActiveCode 1 uses a different set of increments. In this case, we begin with 𝑛2 sublists. On the next pass, 𝑛4 sublists are sorted. Eventually, a single list is sorted with the basic insertion sort. Figure 9 shows the first sublists for our example using this increment.

The following invocation of the shellSort function shows the partially sorted lists after each increment, with the final sort being an insertion sort with an increment of one.







.
