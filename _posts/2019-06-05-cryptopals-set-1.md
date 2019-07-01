---
layout: article
title: Cryptopals - Set 1
permalink: /page/cryptopals-set-1
key: page-article-header-overlay-background-image-HB
cover: /assets/images/thumbs/cryptopals.png
header:
  theme: dark
  background: 'linear-gradient(135deg, rgb(34, 193, 195), rgb(253, 187, 45))'
article_header:
  type: overlay
  theme: dark
  background_color: '#0F2027'
  background_image:
    gradient: 'linear-gradient(135deg, rgba(34, 193, 195, .6), rgba(253, 187, 45, .6))'
    src: /assets/images/posts/cryptopals/enigma.png
---

Solutions for NCC Group's Crypto Challenges.

<!--more-->

<style>
  .page__header .header__brand path {
    fill: rgba(255, 255, 255, .95);
  }
</style>

The code for this set can be accessed here:<br>
<a href="https://github.com/datkev/cryptopals" target="_blank">https://github.com/datkev/cryptopals</a>


## Imports

We'll be importing the following modules to implement our solutions.

```python
import base64
```

## Challenge 1

### Convert hex to base64
The string:
```
49276d206b696c6c696e6720796f757220627261696e206c696b65206120706f69736f6e6f7573206d757368726f6f6d
```
Should produce:
```
SSdtIGtpbGxpbmcgeW91ciBicmFpbiBsaWtlIGEgcG9pc29ub3VzIG11c2hyb29t
```
So go ahead and make that happen. You'll need to use this code for the rest of the exercises.

Cryptopals Rule
```
Always operate on raw bytes, never on encoded strings. Only use hex and base64 for pretty-printing.
```

#### Solution
```python
"""
Convert hex string to base64
Returns a base64 encoded string
"""
def hex_to_b64(hex_str):
  data = bytearray.fromhex(hex_str)
  return base64.b64encode(data).decode()
```

<b>hex_to_b64</b> takes advantage of the base64 module encoding and decoding functions. If we want to return a string, we can use the <b>.decode()</b> method to decode the bytes object into a string.

```python
>>> s = "49276d206b696c6c696e6720796f757220627261696e206c696b65206120706f69736f6e6f7573206d757368726f6f6d"
>>> hex_to_b64(s)
'SSdtIGtpbGxpbmcgeW91ciBicmFpbiBsaWtlIGEgcG9pc29ub3VzIG11c2hyb29t'
```


## Challenge 2

### Fixed XOR
Write a function that takes two equal-length buffers and produces their XOR combination.

If your function works properly, then when you feed it the string:
```
1c0111001f010100061a024b53535009181c
```
... after hex decoding, and when XOR'd against:
```
686974207468652062756c6c277320657965
```
... should produce:
```
746865206b696420646f6e277420706c6179
```


#### Solution
```python
"""
Xor two hex strings
Returns result in hex, no prefix
"""
def xor_hex(hex1, hex2):
  return hex(int(hex1,16)^int(hex2,16)).replace("0x","")
```

Here, we simply convert the strings to their integer representations and perform a bitwise xor on the two ints. We then return the hexadecimal representations. We don't want to prepend the "0x", so we remove that from the return value.

```python
>>> s1 = "1c0111001f010100061a024b53535009181c"
>>> s2 = "686974207468652062756c6c277320657965"
>>> xor_hex(s1,s2)
'746865206b696420646f6e277420706c6179'
```

<b>hex_to_b64</b> takes advantage of the base64 module encoding and decoding functions. If we want to return a string, we can use the <b>.decode()</b> method to decode the bytes object into a string.


## Challenge 3

### Single-byte XOR cipher
The hex encoded string:
```
1b37373331363f78151b7f2b783431333d78397828372d363c78373e783a393b3736
```
... has been XOR'd against a single character. Find the key, decrypt the message.

You can do this by hand. But don't: write code to do it for you.

How? Devise some method for "scoring" a piece of English plaintext. Character frequency is a good metric. Evaluate each output and choose the one with the best score.

#### Solution
```python

# frequency taken from http://en.wikipedia.org/wiki/Letter_frequency
FREQ_TABLE = {'E': 12.70, 'T': 9.06, 'A': 8.17, 'O': 7.51, 'I': 6.97, 'N': 6.75, 'S': 6.33, 'H': 6.09, 'R': 5.99, 'D': 4.25, 'L': 4.03, 'C': 2.78, 'U': 2.76, 'M': 2.41, 'W': 2.36, 'F': 2.23, 'G': 2.02, 'Y': 1.97, 'P': 1.93, 'B': 1.29, 'V': 0.98, 'K': 0.77, 'J': 0.15, 'X': 0.15, 'Q': 0.10, 'Z': 0.07}

# default table for building an alphabetic histogram from a document
DEF_TABLE = {'A': 0, 'B': 0, 'C': 0, 'D': 0, 'E': 0, 'F': 0, 'G': 0, 'H': 0, 'I': 0, 'J': 0, 'K': 0, 'L': 0, 'M': 0, 'N': 0, 'O': 0, 'P': 0, 'Q': 0, 'R': 0, 'S': 0, 'T': 0, 'U': 0, 'V': 0, 'W': 0, 'X': 0, 'Y': 0, 'Z': 0}

# used to hold candidates for single key xors
CHARS = ['0x01', '0x02', '0x03', '0x04', '0x05', '0x06', '0x07', '0x08', '0x09', '0x0a', '0x0b', '0x0c', '0x0d', '0x0e', '0x0f', '0x10', '0x11', '0x12', '0x13', '0x14', '0x15', '0x16', '0x17', '0x18', '0x19', '0x1a', '0x1b', '0x1c', '0x1d', '0x1e', '0x1f', '0x20', '0x21', '0x22', '0x23', '0x24', '0x25', '0x26', '0x27', '0x28', '0x29', '0x2a', '0x2b', '0x2c', '0x2d', '0x2e', '0x2f', '0x30', '0x31', '0x32', '0x33', '0x34', '0x35', '0x36', '0x37', '0x38', '0x39', '0x3a', '0x3b', '0x3c', '0x3d', '0x3e', '0x3f', '0x40', '0x41', '0x42', '0x43', '0x44', '0x45', '0x46', '0x47', '0x48', '0x49', '0x4a', '0x4b', '0x4c', '0x4d', '0x4e', '0x4f', '0x50', '0x51', '0x52', '0x53', '0x54', '0x55', '0x56', '0x57', '0x58', '0x59', '0x5a', '0x5b', '0x5c', '0x5d', '0x5e', '0x5f', '0x60', '0x61', '0x62', '0x63', '0x64', '0x65', '0x66', '0x67', '0x68', '0x69', '0x6a', '0x6b', '0x6c', '0x6d', '0x6e', '0x6f', '0x70', '0x71', '0x72', '0x73', '0x74', '0x75', '0x76', '0x77', '0x78', '0x79', '0x7a', '0x7b', '0x7c', '0x7d', '0x7e', '0x7f', '0x80', '0x81', '0x82', '0x83', '0x84', '0x85', '0x86', '0x87', '0x88', '0x89', '0x8a', '0x8b', '0x8c', '0x8d', '0x8e', '0x8f', '0x90', '0x91', '0x92', '0x93', '0x94', '0x95', '0x96', '0x97', '0x98', '0x99', '0x9a', '0x9b', '0x9c', '0x9d', '0x9e', '0x9f', '0xa0', '0xa1', '0xa2', '0xa3', '0xa4', '0xa5', '0xa6', '0xa7', '0xa8', '0xa9', '0xaa', '0xab', '0xac', '0xad', '0xae', '0xaf', '0xb0', '0xb1', '0xb2', '0xb3', '0xb4', '0xb5', '0xb6', '0xb7', '0xb8', '0xb9', '0xba', '0xbb', '0xbc', '0xbd', '0xbe', '0xbf', '0xc0', '0xc1', '0xc2', '0xc3', '0xc4', '0xc5', '0xc6', '0xc7', '0xc8', '0xc9', '0xca', '0xcb', '0xcc', '0xcd', '0xce', '0xcf', '0xd0', '0xd1', '0xd2', '0xd3', '0xd4', '0xd5', '0xd6', '0xd7', '0xd8', '0xd9', '0xda', '0xdb', '0xdc', '0xdd', '0xde', '0xdf', '0xe0', '0xe1', '0xe2', '0xe3', '0xe4', '0xe5', '0xe6', '0xe7', '0xe8', '0xe9', '0xea', '0xeb', '0xec', '0xed', '0xee', '0xef', '0xf0', '0xf1', '0xf2', '0xf3', '0xf4', '0xf5', '0xf6', '0xf7', '0xf8', '0xf9', '0xfa', '0xfb', '0xfc', '0xfd', '0xfe', '0xff']


"""
Determine which single character a hex string was xor'ed with
Returns the most likely candidate based on agreement with 'normal' english histogram distribution
"""
def find_xor_chr(hex_str):
  candidates = []
  for key in CHARS:
    score = 0
    desired_len = len(hex_str)
    candidate_key = key.replace("0x","") * (desired_len//2)
    candidate_str = xor_hex(hex_str, candidate_key)

    if len(candidate_str) < desired_len:
      candidate_str = "0" * (desired_len-len(candidate_str)) + candidate_str
    try:
      candidate_str = bytes.fromhex(candidate_str).decode("utf-8")
    except:
      continue

    hist = DEF_TABLE
    # build a histogram of alphabetic freq values for candidate_str
    for c in candidate_str:
      if c.upper() in hist:
        hist[c.upper()] += 1
      elif c == " ":
        continue
      else:
        score += 1

    # normalize freq values
    # penalize by subtracting resultant freq from target freq
    denom = len(candidate_str)
    for k,v in hist.items():
      hist[k] /= denom
      score += abs(FREQ_TABLE[k]-hist[k])

    candidates.append((candidate_str, candidate_key[:2], score))
  
  if len(candidates) > 0:
    #print(sorted(candidates,key=lambda x:x[2]))
    return sorted(candidates,key=lambda x:x[2])[0]
  else:
    return 0
```

Here, we store the candidate 1-byte keys in the CHARS list. For each key, we store the result of xor'ing each byte in our 'ciphertext' with a candidate key. This result is candidate_str. We then perform a frequency analysis of the characters in candidate_str. In other words, we convert each byte of the resultant string into its equivalent ASCII representation. For each English character, the frequency of occurence in the string is recorded. We then compare each string's distribution of English letters to a model distribution of English letters in a typical English document. The closest distribution to this model is our top candidate string. We will hold the candidate string, candidate key, and 'score' in a tuple. For each string, the lower the score, the closer we were to matching the model frequency distribution.

```python
>>> s = "1b37373331363f78151b7f2b783431333d78397828372d363c78373e783a393b3736"
>>> find_xor_chr(s)
("Cooking MC's like a pound of bacon", '58', 100.00802851494042)
```


## Challenge 4

### Detect single-character XOR
One of the 60-character strings in this <a href="https://cryptopals.com/static/challenge-data/4.txt" target="_blank">file</a> has been encrypted by single-character XOR.

Find it.

(Your code from #3 should help.)

#### Solution
```python
from set1 import *

with open("4.txt", "r") as f:
  data = f.readlines()
data = [line.strip() for line in data]

for i,line in enumerate(data):
  res = find_xor_chr(line)
  if res:
    print(f"{i}: {res}")
```

We're just taking advantage of our <b>find_xor_chr</b> function and assuming that the only string that was single-character XOR'ed with some key will result in an ASCII-readable string. We will then look through the results and find that string. From our results, it looks like line 170 holds our desired string with 0x35 as the key.

```bash
(cryptopals) E:\cryptopals\set1>python challenge4.py
35: ('JVC\x04FQM\r\x05MyFn\x0fnIw(\x19eg]J\x16\x08lZfNo', '52', 108.09701519659114)
149: ('B;+PYl!CFN<\x15T&f VK+OzQHKgYn|/\x12', '78', 109.1682745876717)
165: ("Ri$o+C5uh| [Jfw\x1aZ%l #\x10'#cTyn&u", '7e', 111.2693882517414)
170: ('Now that the party is jumping\n', '35', 99.99467556558034)
195: ('Ea NEy2HcAoF2Um\x7fCUxe%s)Sv69KQL', '7a', 106.06475474679073)
219: ('TooV\x19yqcJ\x13\x1fjFUᇧ\x03\x19]I]vx-Y\x19\rlJ', '43', 110.19301279949227)
225: ('Q7]Kl(X4wQL\\gU*;7ej  XQ0OjkhtD', '66', 108.16705363847498)
230: ('tsVpxdmmp#E;a.bW1)yQBkxDi={vFG', '61', 106.02701272891545)
289: ('Iu<HKzhxpik6TxkaNqwS;ad\x1exTBXik', '67', 102.92682383851282)
295: ('x\x07oQoJ\x18KPgAU\x02\x16\x03I\x07lBfEd\x1e\x1dQhgO\x1ep', '4e', 108.09701277276382)
```



## Challenge 5

### Implement repeating-key XOR
Here is the opening stanza of an important work of the English language:

```
Burning 'em, if you ain't quick and nimble
I go crazy when I hear a cymbal
```
Encrypt it, under the key "ICE", using repeating-key XOR.

In repeating-key XOR, you'll sequentially apply each byte of the key; the first byte of plaintext will be XOR'd against I, the next C, the next E, then I again for the 4th byte, and so on.

It should come out to:
```
0b3637272a2b2e63622c2e69692a23693a2a3c6324202d623d63343c2a26226324272765272
a282b2f20430a652e2c652a3124333a653e2b2027630c692b20283165286326302e27282f
```

Encrypt a bunch of stuff using your repeating-key XOR function. Encrypt your mail. Encrypt your password file. Your .sig file. Get a feel for it. I promise, we aren't wasting your time with this.

#### Solution
```python
"""
Convert ascii string to hex string
Returns hex string, no prefix
"""
def ascii_to_hex(str):
  res = ""
  for c in str:
    res += "{0:02x}".format(ord(c))
  return res


"""
Xor plaintext with key
Returns result
"""
def repeating_key_xor(data, key, format="plain"):
  res = ""
  len_key = len(key)
  key_it = 0
  if format == "plain": 
    for c in data:
      res += xor_hex(ascii_to_hex(c), ascii_to_hex(key[key_it])).zfill(2)
      key_it+=1
      key_it = key_it%len_key
  elif format == "hex":
    for j in range(0,len(data),2):
      res += xor_hex(data[j:j+2],key[key_it:key_it+2]).zfill(2)
      key_it+=2
      key_it = key_it%len_key
  return res
```

We implement a function that allows the user to input a string, key, and specify an input format of either "plain" or "hex". Plain may be a slight misnomer because plaintext may be hex-encoded, but here we are assuming plain to be ASCII-printable. The only difference between the two formats is that hex-formatted text will need to be XOR'ed in two-character chunks instead of character-by-character. The modulo operator cycles through the characters of our key once we reach the end.

```python
>>> s = "Burning 'em, if you ain't quick and nimble I go crazy when I hear a cymbal"
>>> k = "ICE"
>>> from set1 import *
>>> repeating_key_xor(s, k)
'0b3637272a2b2e63622c2e69692a23693a2a3c6324202d623d63343c2a26226324272765272a282b2f20690a652e2c652a3124333a653e2b2027630c692b20283165286326302e27282f'
```


## Challenge 6

### Break repeating-key XOR
It is officially on, now. This challenge isn't conceptually hard, but it involves actual error-prone coding. The other challenges in this set are there to bring you up to speed. This one is there to qualify you. If you can do this one, you're probably just fine up to Set 6.

There's a file <a href="https://cryptopals.com/static/challenge-data/6.txt" target="_blank">here</a>. It's been base64'd after being encrypted with repeating-key XOR.

Decrypt it.

Here's how:

1. Let KEYSIZE be the guessed length of the key; try values from 2 to (say) 40.
2. Write a function to compute the edit distance/Hamming distance between two strings. The Hamming distance is just the number of differing bits. The distance between:
```
this is a test
```
and
```
wokka wokka!!!
```
is 37. Make sure your code agrees before you proceed.
3. For each KEYSIZE, take the first KEYSIZE worth of bytes, and the second KEYSIZE worth of bytes, and find the edit distance between them. Normalize this result by dividing by KEYSIZE.
4. The KEYSIZE with the smallest normalized edit distance is probably the key. You could proceed perhaps with the smallest 2-3 KEYSIZE values. Or take 4 KEYSIZE blocks instead of 2 and average the distances.
5. Now that you probably know the KEYSIZE: break the ciphertext into blocks of KEYSIZE length.
6. Now transpose the blocks: make a block that is the first byte of every block, and a block that is the second byte of every block, and so on.
7. Solve each block as if it was single-character XOR. You already have code to do this.
8. For each block, the single-byte XOR key that produces the best looking histogram is the repeating-key XOR key byte for that block. Put them together and you have the key.

This code is going to turn out to be surprisingly useful later on. Breaking repeating-key XOR ("Vigenere") statistically is obviously an academic exercise, a "Crypto 101" thing. But more people "know how" to break it than can actually break it, and a similar technique breaks something much more important.


#### Solution
```python
from set1 import *

with open("6.txt","r") as f:
  data = f.readlines()
s = "".join(data).replace("\n","")
s = b64_to_hex(s)

 # contains keysize guess and avg hamming distance between two blocks
hams = {}

# keysizes need to be in multiples of two since we are reading hex
for size in range(4,82,2):
  num_slices = len(s)//size
  score = 0
  iterations = 0
  for i in range(num_slices-1):
    block_1 = s[i*size:i*size+size]
    block_2 = s[i*size+size:i*size+size*2]
    score += ham_dist(block_1,block_2,"hex")/size
    iterations += 1
  hams[size//2] = score/iterations

# keysizes sorted by shortest average hamming distance between two blocks
sorted_hams = sorted(hams.items(), key=lambda x:(x[1],x[0]))

# item[0] is keysize, item[1] is hamming dist
for item in sorted_hams[:3]:
  test_blocks = data_to_blocks(s, int(item[0]))
  transposed_blocks = transpose_blocks(test_blocks)
  
  print(f"Finding candidate key for keysize: {item[0]}")
  key = ""
  for block in transposed_blocks:
    block_len = len(block)
    key+=find_xor_chr(block)[1]

  pkey = bytes.fromhex(key).decode("utf-8")
  ptext = bytes.fromhex(repeating_key_xor(s, key, "hex")).decode("utf-8")

  print(f"key: {pkey}\nplaintext: {ptext}")

```

Here we take each line of the file and decode the base64 to hex. We then make guesses of values 2-40 for the keysize. Note, when dealing with hex strings, this actually means 4-80 characters since each hex byte is two characters.

For each keysize guess, we split the line into blocks of keysize length. We then take the hamming distance between each consecutive block. This means we'll compute the normalized hamming distance between block 1 and block 2, block 2 and block 3, block 3 and block 4, ... etc. We then record the total score for each keysize and store them in a dict with keysize as the key and score as the value. We'll average the score for each keysize by dividing by the number of hamming distance calculations we took. If you look at the code, we add to a keysize's score each time we take a hamming distance calculation, so larger keysizes will have lower scores. Averaging scores by the number of calculations made should eliminate bias.

We then take the three keysizes with lowest scores and split the data into chunks of keysize length. We'll then do the recommended transposition of these blocks and find each XOR byte one by one by using our <b>find_xor_chr</b> function from earlier. We print the results from the top 3 keysizes and find that the desired keysize length is 29.


```bash
(cryptopals) E:\cryptopals\set1>python challenge6.py

Finding candidate key for keysize: 29
key: Terminator X: Bring the noise
plaintext: I'm back and I'm ringin' the bell
A rockin' on the mike while the fly girls yell
In ecstasy in the back of me
Well that's my DJ Deshay cuttin' all them Z's
Hittin' hard and the girlies goin' crazy
Vanilla's on the mike, man I'm not lazy.

I'm lettin' my drug kick in
It controls my mouth and I begin
To just let it flow, let my concepts go
My posse's to the side yellin', Go Vanilla Go!

Smooth 'cause that's the way I will be
And if you don't give a damn, then
Why you starin' at me
So get off 'cause I control the stage
There's no dissin' allowed
I'm in my own phase
The girlies sa y they love me and that is ok
And I can dance better than any kid n' play

Stage 2 -- Yea the one ya' wanna listen to
It's off my head so let the beat play through
So I can funk it up and make it sound good
1-2-3 Yo -- Knock on some wood
For good luck, I like my rhymes atrocious
Supercalafragilisticexpialidocious
I'm an effect and that you can bet
I can take a fly girl and make her wet.

I'm like Samson -- Samson to Delilah
There's no denyin', You can try to hang
But you'll keep tryin' to get my style
Over and over, practice makes perfect
But not if you're a loafer.

You'll get nowhere, no place, no time, no girls
Soon -- Oh my God, homebody, you probably eat
Spaghetti with a spoon! Come on and say it!

VIP. Vanilla Ice yep, yep, I'm comin' hard like a rhino
Intoxicating so you stagger like a wino
So punks stop trying and girl stop cryin'
Vanilla Ice is sellin' and you people are buyin'
'Cause why the freaks are jockin' like Crazy Glue
Movin' and groovin' trying to sing along
All through the ghetto groovin' this here song
Now you're amazed by the VIP posse.

Steppin' so hard like a German Nazi
Startled by the bases hittin' ground
There's no trippin' on mine, I'm just gettin' down
Sparkamatic, I'm hangin' tight like a fanatic
You trapped me once and I thought that
You might have it
So step down and lend me your ear
'89 in my time! You, '90 is my year.

You're weakenin' fast, YO! and I can tell it
Your body's gettin' hot, so, so I can smell it
So don't be mad and don't be sad
'Cause the lyrics belong to ICE, You can call me Dad
You're pitchin' a fit, so step back and endure
Let the witch doctor, Ice, do the dance to cure
So come up close and don't be square
You wanna battle me -- Anytime, anywhere

You thought that I was weak, Boy, you're dead wrong
So come on, everybody and sing this song

Say -- Play that funky music Say, go white boy, go white boy go
play that funky music Go white boy, go white boy, go
Lay down and boogie and play that funky music till you die.

Play that funky music Come on, Come on, let me hear
Play that funky music white boy you say it, say it
Play that funky music A little louder now
Play that funky music, white boy Come on, Come on, Come on
Play that funky music

...
```


## Challenge 7

### AES in ECB mode 
The Base64-encoded content in this file has been encrypted via AES-128 in ECB mode under the key
```
"YELLOW SUBMARINE".
```
(case-sensitive, without the quotes; exactly 16 characters; I like "YELLOW SUBMARINE" because it's exactly 16 bytes long, and now you do too).

Decrypt it. You know the key, after all.

#### Solution
```python
from Crypto.Cipher import AES
from Crypto.Util.Padding import unpad


with open("7.txt","r") as f:
  data = f.readlines()
s = "".join(data).replace("\n","").encode() # argument should be a bytes-like object or ASCII string

key = b"YELLOW SUBMARINE"
cipher = AES.new(key, AES.MODE_ECB)
ct = base64.b64decode(s)
pt = unpad(cipher.decrypt(ct), AES.block_size).decode("utf-8") # AES block size is 16 bytes
print(f"Plaintext:\n{pt}")
```

We can use the PyCryptodome package for this challenge. Read more about it at <a href="https://pycryptodome.readthedocs.io/en/latest/index.html" target="_blank">https://pycryptodome.readthedocs.io/en/latest/index.html</a>.

```bash
(cryptopals) E:\cryptopals\set1>python challenge7.py
Plaintext:
I'm back and I'm ringin' the bell
A rockin' on the mike while the fly girls yell
In ecstasy in the back of me
Well that's my DJ Deshay cuttin' all them Z's
Hittin' hard and the girlies goin' crazy
Vanilla's on the mike, man I'm not lazy.

I'm lettin' my drug kick in
It controls my mouth and I begin
To just let it flow, let my concepts go
My posse's to the side yellin', Go Vanilla Go!

Smooth 'cause that's the way I will be
And if you don't give a damn, then
Why you starin' at me
So get off 'cause I control the stage
There's no dissin' allowed
I'm in my own phase
The girlies sa y they love me and that is ok
And I can dance better than any kid n' play

Stage 2 -- Yea the one ya' wanna listen to
It's off my head so let the beat play through
So I can funk it up and make it sound good
1-2-3 Yo -- Knock on some wood
For good luck, I like my rhymes atrocious
Supercalafragilisticexpialidocious
I'm an effect and that you can bet
I can take a fly girl and make her wet.

I'm like Samson -- Samson to Delilah
There's no denyin', You can try to hang
But you'll keep tryin' to get my style
Over and over, practice makes perfect
But not if you're a loafer.

You'll get nowhere, no place, no time, no girls
Soon -- Oh my God, homebody, you probably eat
Spaghetti with a spoon! Come on and say it!

VIP. Vanilla Ice yep, yep, I'm comin' hard like a rhino
Intoxicating so you stagger like a wino
So punks stop trying and girl stop cryin'
Vanilla Ice is sellin' and you people are buyin'
'Cause why the freaks are jockin' like Crazy Glue
Movin' and groovin' trying to sing along
All through the ghetto groovin' this here song
Now you're amazed by the VIP posse.

Steppin' so hard like a German Nazi
Startled by the bases hittin' ground
There's no trippin' on mine, I'm just gettin' down
Sparkamatic, I'm hangin' tight like a fanatic
You trapped me once and I thought that
You might have it
So step down and lend me your ear
'89 in my time! You, '90 is my year.

You're weakenin' fast, YO! and I can tell it
Your body's gettin' hot, so, so I can smell it
So don't be mad and don't be sad
'Cause the lyrics belong to ICE, You can call me Dad
You're pitchin' a fit, so step back and endure
Let the witch doctor, Ice, do the dance to cure
So come up close and don't be square
You wanna battle me -- Anytime, anywhere

You thought that I was weak, Boy, you're dead wrong
So come on, everybody and sing this song

Say -- Play that funky music Say, go white boy, go white boy go
play that funky music Go white boy, go white boy, go
Lay down and boogie and play that funky music till you die.

Play that funky music Come on, Come on, let me hear
Play that funky music white boy you say it, say it
Play that funky music A little louder now
Play that funky music, white boy Come on, Come on, Come on
Play that funky music
```


## Challenge 8

### Detect AES in ECB mode
In this <a href="https://cryptopals.com/static/challenge-data/8.txt" target="_blank">file</a> are a bunch of hex-encoded ciphertexts.

One of them has been encrypted with ECB.

Detect it.

Remember that the problem with ECB is that it is stateless and deterministic; the same 16 byte plaintext block will always produce the same 16 byte ciphertext.

```python
"""
Finds number of repeating blocks in a given plaintext string
Returns number of repeats
"""
def find_repeating_blocks(data, blocksize=16):
  block_set = set()
  res=0
  for i in range(len(data)//blocksize-1):
    curr = data[i*blocksize:i*blocksize+blocksize]
    if curr in block_set:
      res+=1
    else:
      block_set.add(curr)
  return res
```

```python
from Crypto.Cipher import AES
from Crypto.Util.Padding import unpad
from set1 import *

data = []
with open("8.txt","r") as f:
  for line in f:
    data.append(line.replace("\n",""))

res = {}
for line in data:
  res[line] = find_repeating_blocks(line)

print(sorted(res.items(), key=lambda x:(x[1], x[0]), reverse=True)[0])
```

Here, we just use the naive solution of finding how many repeated blocks there are in the ciphertext. We assume the ciphertext with the most repeated blocks has been encoded with ECB. 

```bash
(cryptopals) E:\cryptopals\set1>python challenge8.py
('d880619740a8a19b7840a8a31c810a3d08649af70dc06f4fd5d2d69c744cd283e2dd052f6b641dbf9d11b0348542bb5708649af70dc06f4fd5d2d69c744cd2839475c9dfdbc1d46597949d9c7e82bf5a08649af70dc06f4fd5d2d69c744cd28397a93eab8d6aecd566489154789a6b0308649af70dc06f4fd5d2d69c744cd283d403180c98c8f6db1f2a3f9c4040deb0ab51b29933f2c123c58386b06fba186a', 6)
```

