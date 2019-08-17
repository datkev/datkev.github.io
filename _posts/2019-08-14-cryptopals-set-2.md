---
layout: article
title: Cryptopals - Set 2
permalink: /page/cryptopals-set-2
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

We'll be importing the following modules and packages to implement our solutions.

```python
import sys
import secrets
import base64
import string
import urllib.parse
from Crypto.Cipher import AES
from Crypto.Util.Padding import pad, unpad

from set1 import *
from tinydb import TinyDB, Query, where
```

## Challenge 9

### Implement PKCS#7 padding

A block cipher transforms a fixed-sized block (usually 8 or 16 bytes) of plaintext into ciphertext. But we almost never want to transform a single block; we encrypt irregularly-sized messages.

One way we account for irregularly-sized messages is by padding, creating a plaintext that is an even multiple of the blocksize. The most popular padding scheme is called PKCS#7.

So: pad any block to a specific block length, by appending the number of bytes of padding to the end of the block. For instance,
```
"YELLOW SUBMARINE"
```

... padded to 20 bytes would be:

```
"YELLOW SUBMARINE\x04\x04\x04\x04"
```


#### Solution
```python
"""
Takes plaintext string as input and returns plaintext padded according to PKCS#7 spec
"""
def pad_pkcs7(data, block_size=16):
  val = block_size-(len(data)%block_size)
  if val == 0: val = block_size
  hex_val = bytes([val])
  return data + hex_val.decode()*val
```

Here, the number of padding bytes is <b>len(data) % block_size</b>. If we are already byte aligned, we need to add a full block of padding bytes of <b>block_size</b> length.

```python
>>> pad_pkcs7("YELLOW SUBMARINE",20)
'YELLOW SUBMARINE\x04\x04\x04\x04'
```


## Challenge 10

### Implement CBC mode

CBC mode is a block cipher mode that allows us to encrypt irregularly-sized messages, despite the fact that a block cipher natively only transforms individual blocks.

In CBC mode, each ciphertext block is added to the next plaintext block before the next call to the cipher core.

The first plaintext block, which has no associated previous ciphertext block, is added to a "fake 0th ciphertext block" called the initialization vector, or IV.

Implement CBC mode by hand by taking the ECB function you wrote earlier, making it encrypt instead of decrypt (verify this by decrypting whatever you encrypt to test), and using your XOR function from the previous exercise to combine them.

<a href="https://cryptopals.com/static/challenge-data/10.txt" target="_blank">The file here</a> is intelligible (somewhat) when CBC decrypted against "YELLOW SUBMARINE" with an IV of all ASCII 0 (\x00\x00\x00 &c)

<b>Don't cheat.</b>
```
Do not use OpenSSL's CBC code to do CBC mode, even to verify your results. What's the point of even doing this stuff if you aren't going to learn from it?
```


#### Solution
```python
with open("10.txt","r") as f:
  data = f.readlines()
s = "".join(data).replace("\n","").encode() # argument should be a bytes-like object or ASCII string

key = b"YELLOW SUBMARINE"
iv = b"\x00"*AES.block_size
cipher = AES.new(key, AES.MODE_CBC, iv)
ct = base64.b64decode(s)
pt = unpad(cipher.decrypt(ct), AES.block_size).decode("utf-8") # AES block size is 16 bytes
print(f"Plaintext:\n{pt}")
```

Here, we are using the Pycryptodome package which has an implementation of the AES cipher and all of its modes. We choose CBC mode specifically and feed the cipher a key of <b>"YELLOW SUBMARINE"</b> and IV of "\x00"*16. The block size for AES is 16 bytes.

When we execute <b>challenge10.py</b> we get the following output. The plaintext below is truncated for brevity.
```
(cryptopals) E:\cryptopals\set2>python challenge10.py
Plaintext:
I'm back and I'm ringin' the bell
A rockin' on the mike while the fly girls yell
In ecstasy in the back of me
Well that's my DJ Deshay cuttin' all them Z's
Hittin' hard and the girlies goin' crazy
Vanilla's on the mike, man I'm not lazy.
...
```


## Challenge 11

### An ECB/CBC detection oracle
Now that you have ECB and CBC working:

Write a function to generate a random AES key; that's just 16 random bytes.

Write a function that encrypts data under an unknown key --- that is, a function that generates a random key and encrypts under it.

The function should look like:
```
encryption_oracle(your-input)
=> [MEANINGLESS JIBBER JABBER]
```

Under the hood, have the function <i>append</i> 5-10 bytes (count chosen randomly) <i>before</i> the plaintext and 5-10 bytes <i>after</i> the plaintext.

Now, have the function choose to encrypt under ECB 1/2 the time, and under CBC the other half (just use random IVs each time for CBC). Use rand(2) to decide which to use.

Detect the block cipher mode the function is using each time. You should end up with a piece of code that, pointed at a block box that might be encrypting ECB or CBC, tells you which one is happening.

#### Solution

```python
"""
Generate sequence of random bytes using secrets module
Returns random bytes
"""
def gen_key(length=16):
  return secrets.token_bytes(length)


"""
Encrypts data under an unknown key using ECB or CBC
Can append 5-10 bytes before data and after data
Retuns result in base64
"""
def encryption_oracle(data, key=None, iv=None, mode="random", padding=True, format="ascii"):
  if key == None:
    key = gen_key()

  if format == "ascii":
    data = data.encode()
  elif format == "hex":
    data = bytes.fromhex(data)

  if padding == True:
    pad_len = secrets.randbelow(6)+5
    data = secrets.token_bytes(pad_len)+data+secrets.token_bytes(pad_len)

  mode = mode.lower()
  if mode == "random":
    mode = "cbc" if secrets.randbits(1) else "ecb"
  if mode == "cbc":
    if not iv:
      iv = gen_key()
    cipher = AES.new(key, AES.MODE_CBC, iv)
  elif mode == "ecb":
    cipher = AES.new(key, AES.MODE_ECB)
    
  #ct_bytes = cipher.encrypt(pad(data, AES.block_size)) # AES block size is 16 bytes
  ct_bytes = cipher.encrypt(pad(data, len(key)))
  ct = base64.b64encode(ct_bytes).decode('utf-8')
  return ct


"""
Finds number of repeating blocks in a given plaintext string
This can be used to detect AES in ECB mode
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

Our first function above simply generates a random sequence of 16 bytes which we will use to generate an AES key or initialization vector. The second function is our encryption oracle which, by default, randomly chooses to encrypt plaintext with AES in CBC mode or ECB mode with equal probabilities. The oracle also appends and prepends 5-10 random bytes before the plaintext. 

We want to detect AES in ECB mode, which means we want to look for any occurences of repeated ciphertext blocks. We can guarantee the generation of repeated ciphertext blocks if we can control the input. The question is, "How do we account for randomly padded bytes that are prepended to our plaintext?" In the following solution, we assume we don't know how many bytes are being prepended -- only that the additional bytes exist. To detect AES in CBC mode, we don't actually need to account for any bytes that might be <i>appended</i> to our plaintext as we will see shortly.

To figure out the bytes we need to inject in order to detect AES in ECB mode, we'll need to first define some variables:

* <b>block_size</b> = 16
* <b>num_prepended_bytes</b> - some random number between 1 and 16
* <b>front_padding</b> - user-controlled junk bytes that we will use to block-align the first block
* <b>block_1</b> - user-controlled block of 16 bytes that we will send twice in order to detect a repeated pattern of ciphertext blocks
* <b>block_2</b> - user-controlled copy of block_1

Say the oracle prepends 5 random bytes. Then, the plaintext bytes will look like the following:

* Block 1: ((block_size) - (num_prepended_bytes))\*front_padding
* Block 2: block_1
* Block 3: block_2

In ECB mode, encryption of each block is done independently of other blocks. The plaintext is fed into the encryption algorithm along with the key and a ciphertext block is generated. You can read more about ECB mode <a href="https://en.wikipedia.org/wiki/Block_cipher_mode_of_operation#Electronic_Codebook_(ECB)" target="_blank">here</a>.

<img src="/assets/images/posts/cryptopals/ecb_encryption.png" alt="" class="center" alt="" class="center">

<img src="/assets/images/posts/cryptopals/ecb_decryption.png" alt="" class="center" alt="" class="center">

Let's view a concrete example of our plaintext as described above, assuming that the oracle prepends at least 5 random bytes.

* Block 1: (16 - 5)*"A"
* Block 2: "A"*16
* Block 3: "A"*16

XXXXXAAAAAAAAAAA | AAAAAAAAAAAAAAAA | AAAAAAAAAAAAAAAA

We assume that the X's represent the random bytes that are prepended to our plaintext. If we know that at <i>least</i> 5 random bytes are being prepended, then we know we need at <i>most</i> (block_size - num_prepended_bytes) total bytes of padding. This is just 16 - 5 = 12 bytes of A's in our case. From there, we need two identical blocks of plaintext. Again, we use two blocks of A's for this purpose. Since ciphertexts produced by ECB mode encryption are not dependent on any other ciphertext blocks, we know that the second and third blocks of ciphertext generated by ECB encryption for this plaintext input would be the same. 

Using this strategy, if we know that there is at least one random byte being prepended to our plaintext, then we will need 16 - 1 = 15 bytes of padding for our first block, followed by two identical blocks of plaintext. This means a block of 15 + 16 + 16 = 47 A's will guarantee ECB mode detection. If two identical ciphertext blocks are produced, then we can be fairly certain ECB mode is being used. Otherwise, we will assume CBC mode is being used. 


```python
# minimum number of repeated bytes to guarantee two repeated blocks under AES
s = "A"*47

ct = encryption_oracle(s)
decoded = base64.b64decode(ct)
print_blocks(decoded)
if find_repeating_blocks(decoded):
  print(f"AES in ECB mode detected.")
else:
  print(f"AES in CBC mode likely.")
```

```
(cryptopals) E:\cryptopals\set2>python challenge11.py
Block 0: b'k\x90f\xf5J\xd1Y\xa9i}6\xfe\x7f\xef\nV'
Block 1: b'=\xde\xa2\xe8\xd4\xc4\x02\xda\xbc\x9f\xd1\xf9g\x85\xed\xa8'
Block 2: b'=\xde\xa2\xe8\xd4\xc4\x02\xda\xbc\x9f\xd1\xf9g\x85\xed\xa8'
Block 3: b'\xb3}6\xcf`8?\xaf*{(,\xbdaI)'
AES in ECB mode detected.

(cryptopals) E:\cryptopals\set2>python challenge11.py
Block 0: b'\xa6\xdb\x9c\n\nb\xcc\x04a\x91V\xc5p\xbb\xb8F'
Block 1: b'\xc9\x11\xb2u#\xa8Y{F6EA\xe7\xc3W"'
Block 2: b'\xa2\x12c\xc0"3\x1f\x10J\x8b\xcb\xad\x8a\xc6\xf6\x04'
Block 3: b'\n\xaa+\x80\xf8\x8c^,~\xb8$\x06\xae\xf0\xcaV'
AES in CBC mode likely.
```



## Challenge 12

### Byte-at-a-time ECB decryption (Simple)
Copy your oracle function to a new function that encrypts buffers under ECB mode using a consistent but unknown key (for instance, assign a single random key, once, to a global variable).

Now take that same function and have it append to the plaintext, BEFORE ENCRYPTING, the following string:
```
Um9sbGluJyBpbiBteSA1LjAKV2l0aCBteSByYWctdG9wIGRvd24gc28gbXkg
aGFpciBjYW4gYmxvdwpUaGUgZ2lybGllcyBvbiBzdGFuZGJ5IHdhdmluZyBq
dXN0IHRvIHNheSBoaQpEaWQgeW91IHN0b3A/IE5vLCBJIGp1c3QgZHJvdmUg
YnkK
```

<b>Spoiler alert.</b>

Do not decode this string now. Don't do it.

Base64 decode the string before appending it. <i>Do not base64 decode the string by hand; make your code do it.</i> The point is that you don't know its contents.

What you have now is a function that produces:
```
AES-128-ECB(your-string || unknown-string, random-key)
```

It turns out: you can decrypt "unknown-string" with repeated calls to the oracle function!

Here's roughly how:

1. Feed identical bytes of your-string to the function 1 at a time --- start with 1 byte ("A"), then "AA", then "AAA" and so on. Discover the block size of the cipher. You know it, but do this step anyway.
2. Detect that the function is using ECB. You already know, but do this step anyways.
3. Knowing the block size, craft an input block that is exactly 1 byte short (for instance, if the block size is 8 bytes, make "AAAAAAA"). Think about what the oracle function is going to put in that last byte position.
4. Make a dictionary of every possible last byte by feeding different strings to the oracle; for instance, "AAAAAAAA", "AAAAAAAB", "AAAAAAAC", remembering the first block of each invocation.
5. Match the output of the one-byte-short input to one of the entries in your dictionary. You've now discovered the first byte of unknown-string.
6. Repeat for the next byte.
 
<b>Congratulations.</b>
This is the first challenge we've given you whose solution will break real crypto. Lots of people know that when you encrypt something in ECB mode, you can see penguins through it. Not so many of them can decrypt the contents of those ciphertexts, and now you can. If our experience is any guideline, this attack will get you code execution in security tests about once a year.


#### Solution
```python
"""
Helper function to fuzz encryption oracle with identical bytes of data
"""
def fuzz_helper(data="A", start=0, end=float('infinity')):
  i = start
  while i < end:
    yield data*i, i
    i+=1
```

Above, we have created a function to assist us in fuzzing an encryption oracle. We'll use this function to incrementally increase the size of the plaintext byte-by-byte. When we detect an increase in blocksize, the blocksize of the encryption algorithm will be the current blocksize - previous blocksize.

Once we determine the algorithm's blocksize, we can then go on to detect AES-ECB. If we detect AES-ECB (see Challenge 11), we'll attempt to crack it.

In this problem, we're attempting to decode unknown plaintext defined as <b>unknown_decoded</b> below:

```python
unknown_s = "Um9sbGluJyBpbiBteSA1LjAKV2l0aCBteSByYWctdG9wIGRvd24gc28gbXkgaGFpciBjYW4gYmxvdwpUaGUgZ2lybGllcyBvbiBzdGFuZGJ5IHdhdmluZyBqdXN0IHRvIHNheSBoaQpEaWQgeW91IHN0b3A/IE5vLCBJIGp1c3QgZHJvdmUgYnkK"
unknown_decoded = base64.b64decode(unknown_s).decode('utf-8')
```

The idea here is to iterate through each character of <b>unknown_decoded</b>. For each unknown character, we'll want to pad the first 15 bytes of a block with some known plaintext and let that unknown character be the 16th or last byte of the block. 

For example, the block below shows 15 user-controlled A's and the first unknown character of <b>unknown_decoded</b> to be the first block of plaintext for AES-ECB. The X's represent unknown characters of <b>unknown_decoded</b>:

AAAAAAAAAAAAAAAX | XXXXXXXXXXXXXXXX | XXXXXXXXXXXXXXXX | ...


Since we are working to decrypt an AES-ECB encrypted ciphertext, we know that the algorithm is deterministic. In other words, a plaintext block encrypted under AES-ECB will always generate the same ciphertext. Using this knowledge to our advantage, we can encrypt every possible pattern matching 15 bytes of A's followed by an ASCII-printable character. For each pattern of plaintext, we'll map the plaintext to the ciphertext that was generated using AES-ECB.

For example:
```
{
  AAAAAAAAAAAAAAAa: <ciphertext_1>,
  AAAAAAAAAAAAAAAb: <ciphertext_2>,
  AAAAAAAAAAAAAAAc: <ciphertext_3>,
  ...
}
```

After we have done this, we are ready to decrypt the first byte of <b>unknown_decoded</b>. We know that if we successfully align the first character of <b>unknown_decoded</b> to the last byte of a plaintext block, the resulting ciphertext block will match one of the ciphertext blocks we generated earlier using a pattern of 15 bytes of A's and some ASCII-printable character. Since we have already mapped all plaintext blocks conforming to this pattern with their corresponding ciphertext blocks, we can identify the last letter of the plaintext block or the first unknown character of <b>unknown_decoded</b>.

We continue this strategy for each unknown character of <b>unknown_decoded</b>. For example, decrypting the second character of <b>unknown_decoded</b> will require a plaintext block that looks like the following:

AAAAAAAAAAAAAAXX | XXXXXXXXXXXXXXXX | XXXXXXXXXXXXXXXX | ...



The code that does this is shown below. <b>decoded</b> will be the final decoded string and <b>decoder</b> will be a temporary dictionary of plaintext patterns with their corresponding ciphertexts under AES-ECB. We will need to generate a new <b>decoder</b> for every unknown character in <b>unknown_decoded</b>. We also need to keep track of which block we are trying to decrypt as we iterate through each character. This is done with the variable <b>block_num</b>. After the final character of unknown_decoded has been decrypted, we may hit PCKS7 padding in which case our decoder will not contain a matching ciphertext block. 

```python
global_key = gen_key()

"""
Break AES-ECB
"""
if __name__ == "__main__":
  unknown_s = "Um9sbGluJyBpbiBteSA1LjAKV2l0aCBteSByYWctdG9wIGRvd24gc28gbXkgaGFpciBjYW4gYmxvdwpUaGUgZ2lybGllcyBvbiBzdGFuZGJ5IHdhdmluZyBqdXN0IHRvIHNheSBoaQpEaWQgeW91IHN0b3A/IE5vLCBJIGp1c3QgZHJvdmUgYnkK"
  unknown_decoded = base64.b64decode(unknown_s).decode('utf-8')

  """
  Find block size of encryption algorithm
  Feed identical bytes of string to a function 1 at a time
  For example: start with 1 byte ("A"), then "AA", then "AAA" and so on
  """
  ct = base64.b64decode(encryption_oracle(unknown_decoded, global_key, None, "ecb", False))
  ct_size = len(ct)
  for fuzz, i in fuzz_helper("A",1):
    new_ct_size = len(base64.b64decode(encryption_oracle(fuzz+unknown_decoded, global_key, None, "ecb", False)))
    if new_ct_size > ct_size:
      block_size = new_ct_size - ct_size
      break
  
  """
  If AES-ECB detected, try to crack it
  """
  decoded = ""
  if detect_aes_ecb(base64.b64decode(encryption_oracle("A"*47+unknown_decoded, global_key, None, "ecb", False))):
    for i in range(1,ct_size+1):
      decoder = {}
      block_num = i//block_size
      prepend = "A"*(block_size-(i%block_size))
      # set start and end indices to calibrate location of block
      start = block_num*block_size
      end = block_num*block_size+block_size
      # build decoder
      for printable in string.printable:  
        data = prepend+decoded+printable
        ct = base64.b64decode(encryption_oracle(data, global_key, None, "ecb", False))
        curr_block = ct[start:end]
        # add block to decoder
        if curr_block not in decoder:
          decoder[curr_block] = printable
      # find ct[block_num] in decoder
      data = prepend+unknown_decoded
      ct = base64.b64decode(encryption_oracle(data, global_key, None, "ecb", False))
      curr_block = ct[start:end]
      # we will no longer be able to find curr_block in decoder once hit padding bytes
      if curr_block not in decoder:
        print(f"Broke at i={i}/{ct_size+1}. Likely hit PCKS7 padding.\n")
        break
      else:
        decoded += decoder[curr_block]
    print(decoded)
```


```
(cryptopals) E:\cryptopals\set1>python challenge12.py
Broke at i=139/145. Likely hit PCKS7 padding.

Rollin' in my 5.0
With my rag-top down so my hair can blow
The girlies on standby waving just to say hi
Did you stop? No, I just drove by
```



## Challenge 13

### ECB cut-and-paste
Write a k=v parsing routine, as if for a structured cookie. The routine should take:
```
foo=bar&baz=qux&zap=zazzle
```
... and produce:
```
{
  foo: 'bar',
  baz: 'qux',
  zap: 'zazzle'
}
```
(you know, the object; I don't care if you convert it to JSON).

Now write a function that encodes a user profile in that format, given an email address. You should have something like:
```
profile_for("foo@bar.com")
```
... and it should produce:
```
{
  email: 'foo@bar.com',
  uid: 10,
  role: 'user'
}
```
... encoded as:
```
email=foo@bar.com&uid=10&role=user
```
Your "profile_for" function should not allow encoding metacharacters (& and =). Eat them, quote them, whatever you want to do, but don't let people set their email address to "foo@bar.com&role=admin".

Now, two more easy functions. Generate a random AES key, then:

A. Encrypt the encoded user profile under the key; "provide" that to the "attacker".
B. Decrypt the encoded user profile and parse it.
Using only the user input to profile_for() (as an oracle to generate "valid" ciphertexts) and the ciphertexts themselves, make a role=admin profile.


#### Solution

For the first task, our solution is as follows. The function splits the string on all instances of the separator, "&" into a list of strings which should each contain an "=" if the string was formatted properly. Under this assumption, if we split each string on "=" into a list of strings, the first string should be our key and the second string should be our value. For instance, on the first split, "foo=bar&baz=qux&zap=zazzle" should become:
```
[[foo=bar],[baz=qux],[zap=zazzle]]
```

On the second split, we will have:
```
[[foo,bar],[baz,qux],[zap,zazzle]]
```

After which, we will build a dictionary using the first item of each list object as our key and second item as our value:
```
{
  foo: bar,
  baz: qux,
  zap: zazzle
}
```

After the first split, if an "=" sign is not found in one of the strings in our newly separated list, it must mean that there was an "&" in one of the values (the "v" in "k=v") and that we accidentally split a value. For example,
"foo=b&ar&baz=qux&zap=zazzle" would become:
```
[[foo=b],[ar],[baz=qux],[zap=zazzle]]
```

We need to rejoin the values and "quote" the separator so it is not interpreted literally again. Here, by "quote", we really mean to url-encode the separator. So, what we want to produce is:
```
[[foo=b%26ar],[baz=qux],[zap=zazzle]]
```

This way, our second split will produce:
```
{
  foo: b%26ar,
  baz: qux,
  zap: zazzle
}
```

We make a final note that we will need to url-encode "=" signs as well.

```python
"""
k=v parsing routine
"""
def k_v_parser(data, separator="&"):
  parsed = {}
  data = data.split(separator)

  for i in range(0, len(data)):
    # split on first occurence of "="
    data[i] = data[i].split("=",1)
    # if no "=" found, then there was a separator in the user input
    # we need to put the separator back into prev data and quote the separator
    # then combine current data with prev data
    if len(data[i]) == 1:
      parsed[data[i-1][0]]+=f"{urllib.parse.quote(separator)}{data[i][0]}"
      continue
    else:
      k,v = data[i][0],data[i][1].replace("=",urllib.parse.quote("="))
      parsed[k] = v
  return parsed
```

We then build a function to create a user with email, uid, and role parameters. Additionally, we'll include an encoder that can retrieve a user from a databse by email and encode the parameters in a string. <a href="https://tinydb.readthedocs.io/en/latest/" target="_blank">Tinydb</a> is a simple document-oriented database that will be suitable for our needs.


```python
"""
Quotes out occurences of specific characters
Characters to remove are provided in a list
"""
def url_quote(data, remove=[]):
  if remove == []:
    return data
  for r in remove:
    data = data.replace(r,urllib.parse.quote(r))
  return data


"""
Encode user data formatted as:

{
  email: 'foo@bar.com',
  uid: 10,
  role: 'user'
}

into:

"email=foo@bar.com&uid=10&role=user"
"""
def k_v_encoder(data):
  encoded = ""
  for k, v in data.items():
    if k == "role":
      encoded += k + "=" + v
    else:
      encoded += k + "=" + v + "&"
  return encoded


"""
Add a user to the db
"""
def add_user(email, role="user"):
  # zero-padded uid to three digits
  uid = f"{len(db):03}"
  email = url_quote(email,['&','='])
  db.insert({'email': email, 'uid': str(uid), 'role': role})


"""
Remove a user from the db
"""
def remove_user(email):
  user = Query()
  doc_id = db.get(user.email == email).doc_id
  db.remove(doc_ids=[doc_id])


"""
Queries the db for a user by email
Returns encoded string if user is found
"""
def profile_for(email):
  user = Query()
  user = db.get(user.email == email)
  if user is None:
    print(f"User with email, {email}, was not found.")
    return
  return k_v_encoder(user)
```

Finally, we get to the meat of the problem. We first generate a key and determine the block size of our encryption algorithm. Technically, we already know we are using AES-ECB with a block size of 16, but implement this check anyway.

```python
  key = gen_key()

  """
  Find block size of encryption algorithm
  """
  add_user("A@test.com")
  ct_size = len(base64.b64decode(encryption_oracle("A@test.com", key, None, "ecb", False)))
  remove_user("A@test.com") # to avoid filling up our db
  for fuzz, i in fuzz_helper("A",2):
    new_user = fuzz+"@test.com"
    add_user(new_user)
    new_ct_size = len(base64.b64decode(encryption_oracle(new_user, key, None, "ecb", False)))
    remove_user(new_user)
    if new_ct_size > ct_size:
      block_size = new_ct_size - ct_size
      break
  print(f"Block size: {block_size}")
```

We then detect if AES-ECB is being used. See Challenge 11 for details on how this is done.

If we detect AES-ECB, we can proceed with our attack. Our goal here is to be able to manipulate a ciphertext which will decrypt to "...role=user" + possible padding into a ciphertex that will decrypt to "...role=admin" + possible padding.

We know that the plaintext will decrypt to something along the lines of:
```
email=foo@bar.com&uid=000&role=user
```

Since we have control of the email value and know that the rest of the plaintext has fixed length, we can manipulate the block alignment to our liking. We have three tasks to accomplish.

<b>Task 1:</b> Identify what "admin\x0b\x0b\x0b\x0b\x0b\x0b\x0b\x0b\x0b\x0b\x0b" looks like when encrypted under AES-ECB by aligning it within its own block. We want to generate something along the lines of:
```
First block: "email=aaaa@a.com"
Second block: "admin\x0b\x0b\x0b\x0b\x0b\x0b\x0b\x0b\x0b\x0b\x0b"
Third block: "uid=000&role=use"
Fourth block: "r\x0f\x0f\x0f\x0f\x0f\x0f\x0f\x0f\x0f\x0f\x0f\x0f\x0f\x0f\x0f"
```

  This can be done by adding a user with email "aaaa@a.comadmin\x0b\x0b\x0b\x0b\x0b\x0b\x0b\x0b\x0b\x0b\x0b"

  The second ciphertext block will be the block we are interested in.

<b>Task 2:</b> Align the "user" value to the beginning of a block. For example:
```
First block: "email=aaaaaa@a.c"
Second block: "om&uid=000&role="
Third block: "user..."
```

  This can be done by adding a user with email "aaaaaa@a.com".

<b>Task 3:</b> Identify the "user\x0c\x0c\x0c\x0c\x0c\x0c\x0c\x0c\x0c\x0c\x0c\x0c" ciphertext block and replace it with a ciphertext block corresponding to "admin\x0b\x0b\x0b\x0b\x0b\x0b\x0b\x0b\x0b\x0b\x0b".


```python
if __name__ == "__main__":
  key = gen_key()

  """
  Find block size of encryption algorithm
  """
  add_user("A@test.com")
  ct_size = len(base64.b64decode(encryption_oracle("A@test.com", key, None, "ecb", False)))
  remove_user("A@test.com") # to avoid filling up our db
  for fuzz, i in fuzz_helper("A",2):
    new_user = fuzz+"@test.com"
    add_user(new_user)
    new_ct_size = len(base64.b64decode(encryption_oracle(new_user, key, None, "ecb", False)))
    remove_user(new_user)
    if new_ct_size > ct_size:
      block_size = new_ct_size - ct_size
      break
  print(f"Block size: {block_size}")

  """
  Fuzzing with "A"*block_size*3 guarantees detection of AES-ECB even if padding
  is being prepended to the data
  """
  new_user = "A"*block_size*3+"@test.com"
  add_user(new_user)
  prof = profile_for(new_user)
  print(f"{prof} added to the database")
  encrypted_data = base64.b64decode(encryption_oracle(prof, key, None, "ecb", False))
  remove_user(new_user)
  print(f"{prof} removed")


  """
  If we detect AES_ECB, proceed with attack
  """
  if detect_aes_ecb(encrypted_data):
    print("AES_ECB detected. Proceeding with attack.")
    # align first block
    new_user = "aaaa@a.com"
    # craft replacement ciphertext
    new_user += "admin\x0b\x0b\x0b\x0b\x0b\x0b\x0b\x0b\x0b\x0b\x0b"

    # determine what the "admin\x0b..." block looks like
    add_user(new_user)
    prof = profile_for(new_user)
    print(f"{prof} added to the database")
    encrypted_data = base64.b64decode(encryption_oracle(prof, key, None, "ecb", False))
    remove_user(new_user)
    admin_block = encrypted_data[block_size:block_size*2]
    print(f"{admin_block} is the admin+padding block.")

    """
    Create a user so that "role=..." is aligned to the end of a block

    First block: "email=aaaaaa@a.c"
    Secondblock: "om&uid=000&role="
    Third block: "admin\x0b\x0b\x0b\x0b\x0b\x0b\x0b\x0b\x0b\x0b\x0b"

    After oracle generates ciphertext, replace last block with our admin_block
    """
    new_user = "aaaaaa@a.com"
    add_user(new_user)
    prof = profile_for(new_user)
    print(f"{prof} added to the database")
    encrypted_data = base64.b64decode(encryption_oracle(prof, key, None, "ecb", False))
    remove_user(new_user)
    print(f"{prof} removed")
    print(f"Old payload: {encrypted_data}")

    ct_size = len(encrypted_data)
    payload = encrypted_data[:ct_size-block_size] + admin_block
    print(f"New payload: {payload}")

    # test if generated user is an admin
    print(decrypt_aes_ecb(payload, key, block_size))
    # b'email=aaaaaa@a.com&uid=000&role=admin'
```


## Challenge 14

### Byte-at-a-time ECB decryption (Harder)
Take your oracle function from #12. Now generate a random count of random bytes and prepend this string to every plaintext. You are now doing:
```
AES-128-ECB(random-prefix || attacker-controlled || target-bytes, random-key)
```
Same goal: decrypt the target-bytes.

<b>Stop and think for a second.</b>
What's harder than challenge #12 about doing this? How would you overcome that obstacle? The hint is: you're using all the tools you already have; no crazy math is required.

Think "STIMULUS" and "RESPONSE".


#### Solution

The only difference between this and challenge 12 is that we are now prepending a number of random bytes between 1 and <b>block_size</b> long. We need to pad this to a length of <b>block_size</b> to block-align the rest of our blocks. The blocks that we are now interested start from block 1 (instead of block 0) and increase from there.

We can use the same strategy we used in challenge 12 to tackle this challenge. For a more in-depth explanation, see challenge 12.

```python
global_key = gen_key()

"""
Break AES-ECB
"""
if __name__ == "__main__":
  unknown_s = "Um9sbGluJyBpbiBteSA1LjAKV2l0aCBteSByYWctdG9wIGRvd24gc28gbXkgaGFpciBjYW4gYmxvdwpUaGUgZ2lybGllcyBvbiBzdGFuZGJ5IHdhdmluZyBqdXN0IHRvIHNheSBoaQpEaWQgeW91IHN0b3A/IE5vLCBJIGp1c3QgZHJvdmUgYnkK"
  unknown_decoded = base64.b64decode(unknown_s).decode('utf-8')

  """
  Find block size of encryption algorithm
  Feed identical bytes of string to a function 1 at a time
  For example: start with 1 byte ("A"), then "AA", then "AAA" and so on
  """
  ct = base64.b64decode(encryption_oracle(unknown_decoded, global_key, None, "ecb", False))
  ct_size = len(ct)
  for fuzz, i in fuzz_helper("A",1):
    new_ct_size = len(base64.b64decode(encryption_oracle(fuzz+unknown_decoded, global_key, None, "ecb", False)))
    if new_ct_size > ct_size:
      block_size = new_ct_size - ct_size
      break
  
  """
  If AES-ECB detected, try to crack it
  """
  decoded = ""
  if detect_aes_ecb(base64.b64decode(encryption_oracle("A"*47+unknown_decoded, global_key, None, "ecb", False))):
    for i in range(1,ct_size+1):
      decoder = {}
      block_num = i//block_size
      prepend = "A"*(block_size-(i%block_size))
      # set start and end indices to calibrate location of block
      start = block_num*block_size
      end = block_num*block_size+block_size
      # build decoder
      for printable in string.printable:  
        data = prepend+decoded+printable
        ct = base64.b64decode(encryption_oracle(data, global_key, None, "ecb", False))
        curr_block = ct[start:end]
        # add block to decoder
        if curr_block not in decoder:
          decoder[curr_block] = printable
      # find ct[block_num] in decoder
      data = prepend+unknown_decoded
      ct = base64.b64decode(encryption_oracle(data, global_key, None, "ecb", False))
      curr_block = ct[start:end]
      # we will no longer be able to find curr_block in decoder once hit padding bytes
      if curr_block not in decoder:
        print(f"Broke at i={i}/{ct_size+1}. Likely hit PCKS7 padding.\n")
        break
      else:
        decoded += decoder[curr_block]
    print(decoded)
```

```
(cryptopals) E:\cryptopals\set1>python challenge14.py
Broke at i=139/145. Likely hit PCKS7 padding.

Rollin' in my 5.0
With my rag-top down so my hair can blow
The girlies on standby waving just to say hi
Did you stop? No, I just drove by
```


## Challenge 15

### PKCS#7 padding validation
Write a function that takes a plaintext, determines if it has valid PKCS#7 padding, and strips the padding off.

The string:
```
"ICE ICE BABY\x04\x04\x04\x04"
```
... has valid padding, and produces the result "ICE ICE BABY".

The string:
```
"ICE ICE BABY\x05\x05\x05\x05"
```
... does not have valid padding, nor does:
```
"ICE ICE BABY\x01\x02\x03\x04"
```
If you are writing in a language with exceptions, like Python or Ruby, make your function throw an exception on bad padding.

Crypto nerds know where we're going with this. Bear with us.

#### Solution

Our function will need to check the value of the last byte of plaintext to determine the number of padding bytes that should be present. It then takes the last pad_num bytes of the plaintext and checks to see if it is equivalent
to <b>value_of_last_byte</b> * <b>pad_num</b>.

```python
"""
Validates whether plaintext has valid PKCS7 padding
"""
def validate_pkcs7(plaintext):
  pad_num = ord(plaintext[-1])
  all_padding = plaintext[-pad_num:]
  if all_padding != plaintext[-1]*pad_num:
    raise Exception("Invalid PKCS7 padding")
  else:
    return True
```

```bash
>>> validate_pkcs7("ICE ICE BABY\x04\x04\x04\x04")
True
>>> validate_pkcs7("ICE ICE BABY\x01\x02\x03\x04")
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "E:\Kevin\Documents\NCC\cryptopals\set2\set2.py", line 193, in validate_pkcs7
    raise Exception("Invalid PKCS7 padding")
Exception: Invalid PKCS7 padding
```


## Challenge 16

### CBC bitflipping attacks
Generate a random AES key.

Combine your padding code and CBC code to write two functions.

The first function should take an arbitrary input string, prepend the string:
```
"comment1=cooking%20MCs;userdata="
```
.. and append the string:
```
";comment2=%20like%20a%20pound%20of%20bacon"
```
The function should quote out the ";" and "=" characters.

The function should then pad out the input to the 16-byte AES block length and encrypt it under the random AES key.

The second function should decrypt the string and look for the characters ";admin=true;" (or, equivalently, decrypt, split the string on ";", convert each resulting string into 2-tuples, and look for the "admin" tuple).

Return true or false based on whether the string exists.

If you've written the first function properly, it should not be possible to provide user input to it that will generate the string the second function is looking for. We'll have to break the crypto to do that.

Instead, modify the ciphertext (without knowledge of the AES key) to accomplish this.

You're relying on the fact that in CBC mode, a 1-bit error in a ciphertext block:

* Completely scrambles the block the error occurs in
* Produces the identical 1-bit error(/edit) in the next ciphertext block.
<b>Stop and think for a second.</b>
Before you implement this attack, answer this question: why does CBC mode have this property?


#### Solution


It will be useful to familiarize yourself with <a href="https://en.wikipedia.org/wiki/Block_cipher_mode_of_operation" target="_blank">CBC mode</a> encryption and decryption.

<img src="/assets/images/posts/cryptopals/cbc_encryption.png" alt="" class="center" alt="" class="center">

<img src="/assets/images/posts/cryptopals/cbc_decryption.png" alt="" class="center" alt="" class="center">

We can approach this problem in two different ways. The first is as follows.

Without user input, our blocks are aligned as shown below:
```
Block 1: "comment1=cooking"
Block 2: "%20MCs;userdata="
Block 3: ";comment2=%20lik"
Block 4: "e%20a%20pound%20"
Block 5: "of%20bacon..."
```


The user input will begin with the first index of the third block and shift all corresponding bytes down as each new byte is introduced. We want the beginning of the third block to read, ";admin=true". This is a total of 11 bytes.

We will need to input 11 bytes -- say, ":admin;true". We use ":" and "<" as our initial bytes to be modified since their bit values are somewhere in the neighborhood of ";" and "=", respectively.

We then need to flip bits in bytes 0 and 6 of the second block so that we alter the bits in bytes 0 and 6 of our third block. This is because each ciphertext block depends on the previous ciphertext. We will need to check each resulting decrypted string to see if we've managed
to modify bytes in position 0 and 6 of the third block to ";" and "=" to obtain ";admin=true".

If so, our parser should see a new key called 'admin' set to "true".

```python
  key = gen_key()
  iv = gen_key()
  cipher = AES.new(key, AES.MODE_CBC, iv)
   
  print("=====Approach 1=====")
  user_input = ":admin<true"
  ct = build_userdata(user_input, key, iv)
  ct = bytearray(ct)
  print("Old ciphertext. Check bytes 0 and 6 of block 1:")
  print_blocks(ct)
  # flip bits of first byte in second block to modify first byte in third block
  for new_ct in cbc_flipper(ct, 16):
    pt = unpad(cipher.decrypt(new_ct), AES.block_size)
    if bytes([pt[32]]) == b";":
      ct = new_ct
      break
  # flip bits of first byte in second block to modify first byte in third block
  for new_ct in cbc_flipper(ct, 22):
    pt = unpad(cipher.decrypt(new_ct), AES.block_size)
    if bytes([pt[38]]) == b"=":
      ct = new_ct
      break
  print("New ciphertext. Check bytes 0 and 6 of block 1:")
  print_blocks(ct)
  passed = parse_userdata(ct, key, iv)
  print(passed)
```

```
(cryptopals) E:\cryptopals\set1>python challenge16.py
=====Approach 1=====
Old ciphertext. Check bytes 0 and 6 of block 1:
Block 0: bytearray(b's [=\x059<\x1880\xdc\xab Lq\xa4')
Block 1: bytearray(b'\xae\xf7p\x80\xe7|,\xe4\xfcL\xdc\n*Yz!')
Block 2: bytearray(b' \x00\x18`IB\xdcK6\xd2\x89\xbfx\x9e\x7f\x19')
Block 3: bytearray(b'Z\x8c\xe5\x93\x8a\x08\x00\xff \x8f\x1d\xf9FV\x1b\xdb')
Block 4: bytearray(b"y\'\xd9L\xc1\\\xae\x08 {\x1d\xd5\x01r\x92\xf5")
Block 5: bytearray(b'\x840\x95w"\x98\xc9\x97\x96\xff\xbd\xa6\xe6\x14\n\xef')
New ciphertext. Check bytes 0 and 6 of block 1:
Block 0: bytearray(b's [=\x059<\x1880\xdc\xab Lq\xa4')
Block 1: bytearray(b'\xaf\xf7p\x80\xe7|-\xe4\xfcL\xdc\n*Yz!')
Block 2: bytearray(b' \x00\x18`IB\xdcK6\xd2\x89\xbfx\x9e\x7f\x19')
Block 3: bytearray(b'Z\x8c\xe5\x93\x8a\x08\x00\xff \x8f\x1d\xf9FV\x1b\xdb')
Block 4: bytearray(b"y\'\xd9L\xc1\\\xae\x08 {\x1d\xd5\x01r\x92\xf5")
Block 5: bytearray(b'\x840\x95w"\x98\xc9\x97\x96\xff\xbd\xa6\xe6\x14\n\xef')
Plaintext was b"@<gQ(B\xeb\xe2\xa0\x0c'W`\xc2\x88\xe2Ic\x1a\x84\x1e\x12\xb9\x0cn<\x08\xcc\xbe\x04\x87(;admin=true;comment2=%20like%20a%20pound%20of%20bacon"
True
```


Our second approach will not involve guessing and checking.

During decryption, we know that a previous ciphertext block is xor'ed with the current post-decryption ciphertext block to derive the current plaintext block. If this is unclear, it may be helpful to reference the diagram above.

The following two statements can be derived from each other and just reiterate the point in the paragraph above.
```
(prev_ct ^ curr_post_decrypt) == curr_pt
```
is equivalent to
```
(prev_ct ^ curr_pt) == curr_post_decrypt
```

In fact,
```
curr_post_encrypt == curr_pre_encrypt
```

<b>curr_post_encrypt</b> refers to the intermediate state of the block when undergoing encryption. This is equivalent to the intermediate state of the block when undergoing decryption, which we refer to as <b>curr_post_decrypt</b>for clarity.

We want our curr_pt to be ":admin&lttrue;comm". Using this, We can derive <b>curr_pre_encrypt</b> with:
```
pre_mod_prev_ct ^ curr_pt == curr_pre_encrypt
```

<i>Note: The prev_ct above refers to the prev_ct pre-modification and is different from the prev_ct post-modification which we seek to replace it with.</i>

Our goal is to find a previous ciphertext block (post-modification) which when xor'ed with <b>curr_pre_encrypt</b> which we just derived, generates a <b>curr_pt</b> of ":admin&lttrue;comm".

We can do this by xor'ing <b>curr_pt</b> and <b>curr_pre_encrypt</b>:
```
curr_pre_encrypt ^ curr_pt == post_mod_prev_ct
```


Where
```
<b>curr_pt</b> = ":admin;true"
<b>curr_pre_encrypt</b> = pre_mod_prev_ct ^ curr_pt
```


We use the following helper functions in this challenge:
```python
"""
Takes arbitrary input string
Prepends "comment1=cooking%20MCs;userdata="
Appends ";comment2=%20like%20a%20pound%20of%20bacon"
Quotes out the ";" and "=" characters
Pad out the input to the 16-byte AES block length and encrypt it under the random AES key
"""
def build_userdata(data, key=None, iv=None):
  assert key is not None, f"Need valid AES key and IV"
  assert iv is not None, f"Need valid AES IV"
  comment_1 = "comment1=cooking%20MCs;userdata="
  comment_2 = ";comment2=%20like%20a%20pound%20of%20bacon"
  data = comment_1 + url_quote(data,[';','=']) + comment_2
  encrypted_data = base64.b64decode(encryption_oracle(data, key, iv, "cbc", False))
  return encrypted_data


"""
Takes AES-encrypted output of build_userdata() as input
Decrypts the string, parses keys, and looks for 'admin': "true"
"""
def parse_userdata(ciphertext, key=None, iv=None):
  assert key is not None, f"Need valid AES key and IV"
  assert iv is not None, f"Need valid AES IV"
  cipher = AES.new(key, AES.MODE_CBC, iv)
  data = {}
  try:
    pt = unpad(cipher.decrypt(ciphertext), AES.block_size).decode()
    print(f"Plaintext was {pt}")
    data = k_v_parser(pt,";")
    print(f"Data was {data}")
    return data.get('admin', "0").lower() == "true" 
  except:
    pt = unpad(cipher.decrypt(ciphertext), AES.block_size)
    print(f"Plaintext was {pt}")
    return b";admin=true;" in pt


"""
Performs xor on the bits of ciphertext[pos] with values 0 to 255
to achieve desired plaintext output
"""
def cbc_flipper(ciphertext, pos):
  for i in range(1,255):
    ciphertext[pos] = ciphertext[pos] ^ i
    yield ciphertext


"""
Xor two byte arrays
"""
def byte_array_xor(arr_1, arr_2):
  # zero extend the smaller byte array
  len_1, len_2 = len(arr_1), len(arr_2)
  if len_1 >= len_2:
    byte_diff = len_1-len_2
    arr_2 = b'\x00'*byte_diff + arr_2
  else:
    byte_diff = len_2-len_1
    arr_1 = b'\x00'*byte_diff + arr_1
  return bytearray(arr_1[i] ^ arr_2[i] for i in range(max(len_1,len_2)))


"""
Prints data block by block
"""
def print_blocks(data, block_size=16):
  for i in range(len(data)//block_size):
    print(f"Block {i}: {data[i*block_size:i*block_size+block_size]}")
```

The solution is then implemented with the following code:
```python
  print("=====Approach 2=====")
  user_input = ":admin<true"
  ct = build_userdata(user_input, key, iv)
  ct = bytearray(ct)
  print("Old ciphertext. Check bytes 0 and 6 of block 1:")
  print_blocks(ct)
  # second block pre-modification
  pre_mod_prev_ct = ct[16:32]
  # intermediate state
  curr_pre_encrypt = byte_array_xor(pre_mod_prev_ct, bytearray(b":admin<true;comm"))
  # derive desired ciphertext of previous block
  post_mod_prev_ct = byte_array_xor(curr_pre_encrypt, bytearray(b";admin=true;comm"))
  # replace old previous ciphertext block with new previous ciphertext block
  new_ct = ct[0:16] + post_mod_prev_ct + ct[32:]
  print("New ciphertext. Check bytes 0 and 6 of block 1:")
  print_blocks(new_ct)
  passed = parse_userdata(new_ct, key, iv)
  print(passed)
```

```
(cryptopals) E:\cryptopals\set1>python challenge16.py
=====Approach 2=====
Old ciphertext. Check bytes 0 and 6 of block 1:
Block 0: bytearray(b's [=\x059<\x1880\xdc\xab Lq\xa4')
Block 1: bytearray(b'\xae\xf7p\x80\xe7|,\xe4\xfcL\xdc\n*Yz!')
Block 2: bytearray(b' \x00\x18`IB\xdcK6\xd2\x89\xbfx\x9e\x7f\x19')
Block 3: bytearray(b'Z\x8c\xe5\x93\x8a\x08\x00\xff \x8f\x1d\xf9FV\x1b\xdb')
Block 4: bytearray(b"y\'\xd9L\xc1\\\xae\x08 {\x1d\xd5\x01r\x92\xf5")
Block 5: bytearray(b'\x840\x95w"\x98\xc9\x97\x96\xff\xbd\xa6\xe6\x14\n\xef')
New ciphertext. Check bytes 0 and 6 of block 1:
Block 0: bytearray(b's [=\x059<\x1880\xdc\xab Lq\xa4')
Block 1: bytearray(b'\xaf\xf7p\x80\xe7|-\xe4\xfcL\xdc\n*Yz!')
Block 2: bytearray(b' \x00\x18`IB\xdcK6\xd2\x89\xbfx\x9e\x7f\x19')
Block 3: bytearray(b'Z\x8c\xe5\x93\x8a\x08\x00\xff \x8f\x1d\xf9FV\x1b\xdb')
Block 4: bytearray(b"y\'\xd9L\xc1\\\xae\x08 {\x1d\xd5\x01r\x92\xf5")
Block 5: bytearray(b'\x840\x95w"\x98\xc9\x97\x96\xff\xbd\xa6\xe6\x14\n\xef')
Plaintext was b"@<gQ(B\xeb\xe2\xa0\x0c'W`\xc2\x88\xe2Ic\x1a\x84\x1e\x12\xb9\x0cn<\x08\xcc\xbe\x04\x87(;admin=true;comment2=%20like%20a%20pound%20of%20bacon"
True
```

