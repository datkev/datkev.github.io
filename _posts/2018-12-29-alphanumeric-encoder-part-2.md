---
layout: article
title: Building an Alphanumeric Encoder Part 2 - Integer Partitions
permalink: /page/building-an-alphanumeric-encoder-part-2
key: page-article-header-overlay-background-image-HB
cover: /assets/images/thumbs/alphanumeric-encoder-part-2.png
header:
  theme: dark
  background: 'linear-gradient(135deg, rgb(15, 32, 39), rgb(44, 83, 100))'
article_header:
  type: overlay
  theme: dark
  background_color: '#0F2027'
  background_image:
    gradient: 'linear-gradient(135deg, rgba(15, 32, 39, .6), rgba(44, 83, 100, .6))'
    src: /assets/images/posts/alphanumeric-encoder-part-2/cover.png
---

Generating encoded values using integer partitions.

<!--more-->

<style>
  .page__header .header__brand path {
    fill: rgba(255, 255, 255, .95);
  }
</style>

## Objective
This is the second post of a multi-part series exploring the process of creating a custom alphanumeric shellcode encoder. In <a href="http://www.datkev.github.io/page/building-an-alphanumeric-encoder-part-1.html" target="_blank">Part 1</a>, we learned how some vulnerable applications mangle user input when fed bad characters, forcing attackers to use a workaround like an encoder. After examining the manual encoding process, we created an outline for a program that automates the process for us. In Part 2, we'll take the outline created in Part 1 and see how working with integer partitions can help us with the encoding process.



## Using GAP
Before reading any further, it is recommended to familiarize yourself with the the manual encoding process covered under the Manual Encoding section of Part 1 in this series. The process involved interpreting hex bytes as dividends and dividing those bytes by three to find a divisor. We used four of these divisor bytes to form a four-byte summand value. By subtracting three of these four-byte summand values from zero, the value stored in <code>eax</code> in our encoder, we could manipulate the value in <code>eax</code> to hold any value we desired. 

By adding <b>0x100</b> to any dividend less than <b>0x60</b>, we avoided divisors less than <b>\x20</b> (non-ASCII printable characters). For instance, if we had a divisor of <b>\x40</b>, dividing this by three would result in the non-ASCII printable character <b>\x15</b> (with remainder <b>\x01</b>). If we instead added <b>0x100</b> to <b>\x40</b> and divided our result, <b>0x140</b> by three, our dividend would be the ASCII-printable character <b>\x6a</b> (with remainder <b>\x02</b>). Of course, this function required us to go back and adjust the final summands to account for the extra <b>0x100</b>s we were adding to our dividend bytes. Luckily, these error-prone, tedious calculations can be avoided altogether with the right tools.

### GAP to the Rescue
<a href="https://www.gap-system.org/index.html" target="_blank">GAP</a> is an open-source system for computational discrete algebra used in research and teaching. The online repo contains a large library of functions and packages which can be referenced through their <a href="https://www.gap-system.org/Manuals/doc/ref/chap0.html#contents" target="_blank">manual</a>.

Under the <a href="https://www.gap-system.org/Manuals/doc/ref/chap16.html" target="_blank">Combinatorics</a> section, we can find a particular function called <b>Partitions</b> with the following description:

><b>Partitions(n[,k])</b><br>
returns the set of all (unordered) partitions of the positive integer n into sums with k summands. If k is not given it returns all unordered partitions of set for all k.
```
gap> Partitions( 7 );
[ [ 1, 1, 1, 1, 1, 1, 1 ], [ 2, 1, 1, 1, 1, 1 ], [ 2, 2, 1, 1, 1 ], 
  [ 2, 2, 2, 1 ], [ 3, 1, 1, 1, 1 ], [ 3, 2, 1, 1 ], [ 3, 2, 2 ], 
  [ 3, 3, 1 ], [ 4, 1, 1, 1 ], [ 4, 2, 1 ], [ 4, 3 ], [ 5, 1, 1 ], 
  [ 5, 2 ], [ 6, 1 ], [ 7 ] ]
gap> Partitions( 8, 3 );
[ [ 3, 3, 2 ], [ 4, 2, 2 ], [ 4, 3, 1 ], [ 5, 2, 1 ], [ 6, 1, 1 ] ]
```

We can automate the process of generating summands for our encoder by feeding individual hex byte values as the first parameter and the number "3" as the second parameter to the <b>Partitions</b> function, indicating that we want to return arrays or lists containing three summands. However, we would still have to convert the base 10 integers within these returned arrays to single hex bytes and check the hex bytes for bad characters. Once we find the first instance of an array with three single-byte summands that are not bad characters, we can use those single-byte values to construct our four-byte summands used to generate arbitrary values from <code>eax</code> (as mentioned under <b>Manual Encoding</b> in Part 1). The greater the value we feed into the <b>Partitions</b> function, the more partitions the function will generate. Using <b>NrPartitions</b>, we can find the number of total partitions for a given value. <b>NrPartitions(255,3)</b> shows us that hex value <b>\xff</b> generates 5,419 distinct partitions containing three summands each.

```
gap> NrPartitions(255,3);
5419
```

Having to scan up to thousands of partitions for bad characters for each hex byte in our encoder is not exactly an efficient solution to our problem. Instead, we can use:

><b>NrRestrictedPartitions( n, set[, k] ) ( function )</b><br>
returns the number of RestrictedPartitions(n,set,k).

```
gap> RestrictedPartitions(8,[2,3],3);
[ [ 3, 3, 2 ] ]
```

If we look at the output above, we can see that the function will only return partitions constructed from the values contained within the set passed in the second parameter. Essentially, we can think of this set as our good characters from which we will have to generate given a list of bad characters. Passing a set of good characters to each call to <b>RestrictedPartitions</b> will greatly reduce the overhead of finding valid single-byte summands. Rather than searching an array of up to thousands of partitions, we can simply return the first partition that <b>RestrictedPartitions</b> finds. If <b>RestrictedPartitions</b> cannot find a valid partition containing three summands from our good characters set, we know that it is impossible to encode our egghunter given the bad characters specified by the user. 


### Translating Code
If we check the <a href="https://github.com/gap-system/gap/blob/master/lib/combinat.gi" target="_blank">combinat.gi</a> file in the <b>lib</b> directory of the GAP repo, we can find the function definition of <b>RestrictedPartitionsK</b> as shown below:

```
RestrictedPartitionsK := function ( n, set, m, k, part, i )
    local   parts, l;
    if k = 1  then
        if n in set  then
            part := ShallowCopy(part);
            part[i] := n;
            parts := [ part ];
        else
            parts := [];
        fi;
    else
        part := ShallowCopy(part);
        parts := [ ];
        for l  in [1..m]  do
            if set[l]+(k-1)*set[1] <= n  and n <= k*set[l]  then
                part[i] := set[l];
                Append(parts,
                       RestrictedPartitionsK(n-set[l],set,l,k-1,part,i+1) );
            fi;
        od;
    fi;
    return parts;
end;
MakeReadOnlyGlobal( "RestrictedPartitionsK" );
```

Replacing the variables with meaningful names and annotating the output may help us understand how the function works. Below is a Python translation of the <b>RestrictedPartitionsK</b> function and a debugging version from which we can use to understand how the partitions are being generated. 

<b>partitions.py</b>
```py
#!/usr/bin/python

import copy

good_chars = [3,5,2,9,10,8]
good_chars.sort()

def restricted_partitions(n, good_chars, end_index, slots, part, i):

    # base case
    if (slots == 1):
        if (n in good_chars):
            part = copy.copy(part)
            if (i > len(part)-1): # append to part
                part.append(n)
            else: # replace in part
                part[i] = n
            parts = [part]
        else:
            parts = []
    # recursive case
    else:
        part = copy.copy(part)
        parts = []
        for j in range(end_index):
            if ((good_chars[j] + (slots-1)*good_chars[0] <= n) and (n <= slots*good_chars[j])):
                if (i > len(part)-1):
                    part.append(good_chars[j]) # append to part
                else:
                    part[i] = good_chars[j] # replace in part

                parts = parts + restricted_partitions(n-good_chars[j], good_chars, j+1, slots-1, part, i+1)
    return parts



print(restricted_partitions(21, good_chars, len(good_chars), 3, [], 0))
```

<b>partitions-dbg.py</b>
```py
#!/usr/bin/python

import copy

good_chars = [2,3,5,8,9,10]
good_chars.sort()

print("Sorted List: {}".format(good_chars))


def restricted_partitions(n, good_chars, end_index, slots, part, i):
    
    # base case
    if (slots == 1):
        print("\n[*] Base case reached!! n = {0}, end_index = {1}, slots = {2}, part = {3}, i = {4}".format(n,end_index,slots,part,i))
        if (n in good_chars):
            part = copy.copy(part)
            if (i > len(part)-1): # append to part
                part.append(n)
            else: # replace in part
                part[i] = n
            parts = [part]
            print("Partition formed: {}\n\n".format(parts))
        else:
            print("{} could not be found in good_chars\n\n".format(n))
            parts = []
    # recursive case
    else:
        print("\n[*] Recursive case {}:".format(3-slots+1))
        print("[*] part: {}".format(part))
        part = copy.copy(part)
        parts = []
        print("Looping through good_chars from index 0 to {}".format(end_index-1))
        for j in range(end_index):
            print(">>>In loop<<<")
            print("n: {0}, i: {1}, j: {2}".format(n,i,j))
            print("if ({0} + {1}*{2} <= {3}) and ({4} <= {5}*{6})".format(good_chars[j],slots-1,good_chars[0],n,n,slots,good_chars[j]))
            if ((good_chars[j] + (slots-1)*good_chars[0] <= n) and (n <= slots*good_chars[j])):
                print("SUCCESS!!")
                if (i > len(part)-1):
                    print("Appending {0} to {1}".format(good_chars[j], part))
                    part.append(good_chars[j])
                    print("Part is now: {}".format(part))
                else:
                    print("part[{0}] = {1}".format(i, good_chars[j]))
                    part[i] = good_chars[j]
                    print("Part is now: {}".format(part))

                print("[**] Entering recursion with:")
                print("parts = parts + restricted_partitions(n-good_chars[j], good_chars, j+1, slots-1, part, i+1)")
                print("where n = {0}, good_chars[j] = {1}, j = {2}, slots = {3}, part = {4}, i = {5}".format(n,good_chars[j],j,slots,part,i))
                parts = parts + restricted_partitions(n-good_chars[j], good_chars, j+1, slots-1, part, i+1)
    print("\n")
    return parts



print(restricted_partitions(21, good_chars, len(good_chars), 3, [], 0))    
```
Below is the console output when we run <b>partitions-dbg.py</b>.

```
# ./partitions-dbg.py
Sorted List: [2, 3, 5, 8, 9, 10]

[*] Recursive case 1:
[*] part: []
Looping through good_chars from index 0 to 5
>>>In loop<<<
n: 21, i: 0, j: 0
if (2 + 2*2 <= 21) and (21 <= 3*2)
>>>In loop<<<
n: 21, i: 0, j: 1
if (3 + 2*2 <= 21) and (21 <= 3*3)
>>>In loop<<<
n: 21, i: 0, j: 2
if (5 + 2*2 <= 21) and (21 <= 3*5)
>>>In loop<<<
n: 21, i: 0, j: 3
if (8 + 2*2 <= 21) and (21 <= 3*8)
SUCCESS!!
Appending 8 to []
Part is now: [8]
[**] Entering recursion with:
parts = parts + restricted_partitions(n-good_chars[j], good_chars, j+1, slots-1, part, i+1)
where n = 21, good_chars[j] = 8, j = 3, slots = 3, part = [8], i = 0

[*] Recursive case 2:
[*] part: [8]
Looping through good_chars from index 0 to 3
>>>In loop<<<
n: 13, i: 1, j: 0
if (2 + 1*2 <= 13) and (13 <= 2*2)
>>>In loop<<<
n: 13, i: 1, j: 1
if (3 + 1*2 <= 13) and (13 <= 2*3)
>>>In loop<<<
n: 13, i: 1, j: 2
if (5 + 1*2 <= 13) and (13 <= 2*5)
>>>In loop<<<
n: 13, i: 1, j: 3
if (8 + 1*2 <= 13) and (13 <= 2*8)
SUCCESS!!
Appending 8 to [8]
Part is now: [8, 8]
[**] Entering recursion with:
parts = parts + restricted_partitions(n-good_chars[j], good_chars, j+1, slots-1, part, i+1)
where n = 13, good_chars[j] = 8, j = 3, slots = 2, part = [8, 8], i = 1

[*] Base case reached!! n = 5, end_index = 4, slots = 1, part = [8, 8], i = 2
Partition formed: [[8, 8, 5]]






>>>In loop<<<
n: 21, i: 0, j: 4
if (9 + 2*2 <= 21) and (21 <= 3*9)
SUCCESS!!
part[0] = 9
Part is now: [9]
[**] Entering recursion with:
parts = parts + restricted_partitions(n-good_chars[j], good_chars, j+1, slots-1, part, i+1)
where n = 21, good_chars[j] = 9, j = 4, slots = 3, part = [9], i = 0

[*] Recursive case 2:
[*] part: [9]
Looping through good_chars from index 0 to 4
>>>In loop<<<
n: 12, i: 1, j: 0
if (2 + 1*2 <= 12) and (12 <= 2*2)
>>>In loop<<<
n: 12, i: 1, j: 1
if (3 + 1*2 <= 12) and (12 <= 2*3)
>>>In loop<<<
n: 12, i: 1, j: 2
if (5 + 1*2 <= 12) and (12 <= 2*5)
>>>In loop<<<
n: 12, i: 1, j: 3
if (8 + 1*2 <= 12) and (12 <= 2*8)
SUCCESS!!
Appending 8 to [9]
Part is now: [9, 8]
[**] Entering recursion with:
parts = parts + restricted_partitions(n-good_chars[j], good_chars, j+1, slots-1, part, i+1)
where n = 12, good_chars[j] = 8, j = 3, slots = 2, part = [9, 8], i = 1

[*] Base case reached!! n = 4, end_index = 4, slots = 1, part = [9, 8], i = 2
4 could not be found in good_chars




>>>In loop<<<
n: 12, i: 1, j: 4
if (9 + 1*2 <= 12) and (12 <= 2*9)
SUCCESS!!
part[1] = 9
Part is now: [9, 9]
[**] Entering recursion with:
parts = parts + restricted_partitions(n-good_chars[j], good_chars, j+1, slots-1, part, i+1)
where n = 12, good_chars[j] = 9, j = 4, slots = 2, part = [9, 9], i = 1

[*] Base case reached!! n = 3, end_index = 5, slots = 1, part = [9, 9], i = 2
Partition formed: [[9, 9, 3]]






>>>In loop<<<
n: 21, i: 0, j: 5
if (10 + 2*2 <= 21) and (21 <= 3*10)
SUCCESS!!
part[0] = 10
Part is now: [10]
[**] Entering recursion with:
parts = parts + restricted_partitions(n-good_chars[j], good_chars, j+1, slots-1, part, i+1)
where n = 21, good_chars[j] = 10, j = 5, slots = 3, part = [10], i = 0

[*] Recursive case 2:
[*] part: [10]
Looping through good_chars from index 0 to 5
>>>In loop<<<
n: 11, i: 1, j: 0
if (2 + 1*2 <= 11) and (11 <= 2*2)
>>>In loop<<<
n: 11, i: 1, j: 1
if (3 + 1*2 <= 11) and (11 <= 2*3)
>>>In loop<<<
n: 11, i: 1, j: 2
if (5 + 1*2 <= 11) and (11 <= 2*5)
>>>In loop<<<
n: 11, i: 1, j: 3
if (8 + 1*2 <= 11) and (11 <= 2*8)
SUCCESS!!
Appending 8 to [10]
Part is now: [10, 8]
[**] Entering recursion with:
parts = parts + restricted_partitions(n-good_chars[j], good_chars, j+1, slots-1, part, i+1)
where n = 11, good_chars[j] = 8, j = 3, slots = 2, part = [10, 8], i = 1

[*] Base case reached!! n = 3, end_index = 4, slots = 1, part = [10, 8], i = 2
Partition formed: [[10, 8, 3]]




>>>In loop<<<
n: 11, i: 1, j: 4
if (9 + 1*2 <= 11) and (11 <= 2*9)
SUCCESS!!
part[1] = 9
Part is now: [10, 9]
[**] Entering recursion with:
parts = parts + restricted_partitions(n-good_chars[j], good_chars, j+1, slots-1, part, i+1)
where n = 11, good_chars[j] = 9, j = 4, slots = 2, part = [10, 9], i = 1

[*] Base case reached!! n = 2, end_index = 5, slots = 1, part = [10, 9], i = 2
Partition formed: [[10, 9, 2]]




>>>In loop<<<
n: 11, i: 1, j: 5
if (10 + 1*2 <= 11) and (11 <= 2*10)




[[8, 8, 5], [9, 9, 3], [10, 8, 3], [10, 9, 2]]

```

#### Function Parameters
Using the console output from the debugging version, we can gather the following information about our function parameters:<br>

```py
restricted_partitions(n, good_chars, end_index, slots, part, i)
```

<b>n</b> = target sum. When entering a deeper level of recursion, this is value is reduced by the value of the last summand added to the partition.<br>
<b>good_chars</b> = list of summands to form partitions from<br>
<b>end_index</b> = index of <b>good_chars</b> beyond which we will not search for summands to add to partitions. As we recurse deeper, this will take the value of the index of the last summand added to the current partition.<br>
<b>slots</b> = number of summands left to add in our current partition<br>
<b>part</b> = current partition list, not to be confused with <b>parts</b> which will hold our final list of partitions.<br>
<b>i</b> = index of our current partition where we will place a summand

#### Recursion Check
Each time the recursive case is taken, our function makes two comparisons to evaluate the compatibility of the current summand.

```py
if ((good_chars[j] + (slots-1)*good_chars[0] <= n) and (n <= slots*good_chars[j])):
```

The first check returns True if the <b>target sum</b> >= <b>current summand + (slots - 1) * smallest summand</b>. If this check does not hold, all possible partitions formed using our current summand and any other summands will result in a value greater than the target sum.

The second check returns True if the <b>target sum</b> <= <b>slots * current summand</b>. If this check does not hold, all possible partitions formed using our current summand and any other summands will result in a value less than the target sum. This is due to the fact that the next recursive call will use the index of the current summand as the <b>end_index</b>. In other words, all subsequent summands added to a partition will be less than or equal to the current summand being evaluated at the check.

#### Final Optimization

Finally, we can see that the function returns a list of partitions, all of which are compatible with our <b>good_chars</b> set. While this may be helpful if a user does not have an accurate or comprehensive list of good characters for a particular exploit, when we build our program, we will assume that the user will provide an accurate list of all compatible good characters. As such, we will only need to return the first valid partition that the function finds. This can be accomplished in the recursive case by adding a simple check to see if the <b>parts</b> list contains a partition.

```py
# recursive case
else:
    part = copy.copy(part)
    parts = []
    for j in range(end_index):
        if (len(parts) == 1): # return first partition found
          return parts
        if ((good_chars[j] + (slots-1)*good_chars[0] <= n) and (n <= slots*good_chars[j])):
            if (i > len(part)-1):
                part.append(good_chars[j])
            else:
                part[i] = good_chars[j]

            parts = parts + restricted_partitions(n-good_chars[j], good_chars, j+1, slots-1, part, i+1)
```


## Testing the Partitions Function
Prior to running the <b>partitions</b> function, we assume we have converted a list of good character hex bytes to integer values.

Let's take a look at the 4-byte chunks of our egghunter from Part 1 in this series.

```
#NtAccessCheckAndAuditAlarm egghunter
#using w00t ("\x77\x30\x30\x74") tag

\x66\x81\xCA\xFF
\x0F\x42\x52\x6A
\x02\x58\xCD\x2E
\x3C\x05\x5A\x74
\xEF\xB8\x77\x30
\x30\x74\x8B\xFA
\xAF\x75\xEA\xAF
\x75\xE7\xFF\xE7
```

We had previously found the two's complement of <b>0xE7FFE775</b> to be <b>0x1800188B</b>. Let's assume we are able to pass the bytes that comprise this 
word into our <b>Partitions</b> function. For each byte, our function will return a partition containing three summands. We will need to distribute each summand into three separate lists. After each byte of the two's complement is passed into our function and we have distributed the summands returned from our partitions, we will have three lists containing four bytes each. These four bytes can then be concatenated to form values that when subtracted from zero, manipulate the value in the <code>eax</code> register.

Effectively, this is our translation of Step 5 from the Automated Solution outline we created in Part 1. 

<b>Encoding Loop:</b>
   * Point at the first 4 byte chunk (after reversing the chunk order)<br>
   * Take the two’s complement<br>
   * Generate a <a href="https://en.wikipedia.org/wiki/Partition_(number_theory)" _target="blank">partition</a> containing three operands comprised only of good characters<br>
   * Store operands<br>
   * Increment four bytes<br>
   * If we have not hit the end, loop back to second bullet.<br>



At last, the simple proof of concept below confirms that our <b>Partitions</b> function can be used in our encoder.

<b>partitions-test.py</b>
```py
#!/usr/bin/python

import copy
import struct

twos_comp = ['1800188b']
target_bytes = []
good_chars = []


'''
Finds integer partitions. Partitions used to construct summands to manipulate eax register
Input: Integers from hex bytes of two's complement egghunter chunks
'''
def restricted_partitions(n, good_chars, end_index, slots, part, i):  
    # base case
    if (slots == 1):
        if (n in good_chars):
            part = copy.copy(part)
            if (i > len(part)-1): # append to part
                part.append(n)
            else: # replace in part
                part[i] = n
            parts = [part]
      else:
          parts = []
    # recursive case
    else:
        part = copy.copy(part)
        parts = []
        for j in range(end_index):
            if (len(parts) == 1):
                return parts
            if ((good_chars[j] + (slots-1)*good_chars[0] <= n) and (n <= slots*good_chars[j])):
                if (i > len(part)-1):
                    part.append(good_chars[j]) # append to part
                else:
                    part[i] = good_chars[j] # replace in part

                parts = parts + restricted_partitions(n-good_chars[j], good_chars, j+1, slots-1, part, i+1)
    return parts



'''
Distributes byte_list into three summands.
Three bytes per list (each byte corresponds to a different summand)
Four lists per block, three summands per block.
Input: List of lists of bytes from restricted_partitions 
'''
def format_summands(byte_list):
    print("Integer partitions:\n{}".format(byte_list))
    for i in range(0,len(byte_list),4):
        summands = [["","",""]*(len(byte_list)/4)]
        for j in range(3):
            for k in range(3,-1,-1):
                summands[i][j] += "\\x{}".format(hex(byte_list[i+k].pop())[2:4]) # dec to hex string conversion
        
        i += 4
    
    print("Summands:\n{}".format(summands))



'''
Tests the restricted_partitions function
'''
def test_partitions():
    global twos_comp, target_bytes

    curr = []

    for i in range (0,8,2):
        target_bytes += [twos_comp[0][i:i+2]]
    
    print("Two's complement bytes:\n{}".format(target_bytes))

    good_chars = [i for i in range(20,128)]
    print("Good characters:\n{}".format(good_chars))

    carry = False

    for i in range(len(target_bytes)):
        target_i = int(target_bytes[i],16)
        if (carry):
            target_i -= 1
        if (target_i < 96):
            target_i += 256
            carry = True
        else:
            carry = False
      
        curr += restricted_partitions(target_i, good_chars, len(good_chars), 3, [], 0)
    
    format_summands(curr)

    

test_partitions()

```

Output of <b>partitions-test.py</b>:
```
# ./partitions-test.py
Two's complement bytes:
['18', '00', '18', '8b']
Good characters:
[20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36, 37, 38, 39, 40, 41, 42, 43, 44, 45, 46, 47, 48, 49, 50, 51, 52, 53, 54, 55, 56, 57, 58, 59, 60, 61, 62, 63, 64, 65, 66, 67, 68, 69, 70, 71, 72, 73, 74, 75, 76, 77, 78, 79, 80, 81, 82, 83, 84, 85, 86, 87, 88, 89, 90, 91, 92, 93, 94, 95, 96, 97, 98, 99, 100, 101, 102, 103, 104, 105, 106, 107, 108, 109, 110, 111, 112, 113, 114, 115, 116, 117, 118, 119, 120, 121, 122, 123, 124, 125, 126, 127]
Integer partitions:
[[47, 46, 46], [94, 93, 93], [85, 85, 85], [93, 93, 93]]
Summands:
[['\\x5d\\x55\\x5d\\x2e', '\\x5d\\x55\\x5d\\x2e', '\\x5d\\x55\\x5e\\x2f']]
```

We can check that when our summands are subtracted from 0, the result will be the last 4-byte chunk of our egghunter in little-endian format:<br>
<b>0</b> - <b>0x5d555d2e</b> - <b>0x5d555d2e</b> - <b>0x5d555e2f</b> = <b>0xe7ffe775</b>

Aside from taking the two's complement of 4-byte egghunter chunks, the rest of our program will consist of functions that handle the formatting of user input from stdin and output for stdout.

In the next post, we will take the entire Automated Solution outline created in Part 1 and translate it into Python code. When we are finished, we'll put our encoder to the test by generating an encoded egghunter exploit for the Eureka Email Client on a Windows XP box. 
