# Introduction
This python3 programming exercise is about interacting with a `socket` service using [pwntools](http://docs.pwntools.com/en/stable/) and it's [sockets library](https://docs.pwntools.com/en/stable/tubes/sockets.html#). 

```
+-----------------+            +-----------------------+
| python3 program +----------->+ riddler 2 on port 443 |
+-----------------+            +-----------------------+
```

*Riddlers Legacy Two* follows the same principle as *Riddlers Legacy One* but with more than just solving a caesar cipher. You'll need to write several functions solving different crypto puzzles similar to *Riddlers Legacy One*


## Learn how to ...
 - Use pwntools sockets library
 - Automate the interaction with a web service

## Tasks
* Task 1: Please analyze the Riddler Service on port `443`
* Task 2: Write a Python3 solver for the Riddler One Service

# Analyzing Riddler
## Step 1

### Resource
* Get the name of the service from `RESOURCES`
* the riddler service is running on port `443`


## Step 2
### Analysis of Riddler Two Service
Riddler is a network services that is listening on port `443`. It is **not** https - but still running on port `443`!

![service-overview.png](/media/challenge/png/e6baaba7-65b1-45bd-ad73-bff3ccb11de9.png)


#### Manual Connection

Before we start to write some code, we should make ourself familiar with the responses of *Riddler Two* by using `telnet` or `netcat`:

``` python
telnet [IP-Address] [Port]
telnet riddler.vm.vuln.land 443
```

``` python
nc -v [IP-Address] [Port]
nc -v riddler.vm.vuln.land 443
```

LOG
```
========= 1 ==========
nc riddler.vm.vuln.land 443
0;8;r:EB0TABY=

========= 2 ==========
nc riddler.vm.vuln.land 443
0;8;c:AQwWDQAGBw==

========= 3 ==========
nc riddler.vm.vuln.land 443
0;2;Ym9ndXM=

========= 4 ==========
nc riddler.vm.vuln.land 443
0;4;..-. .- .-. .- -.. .. --.. . -.. 
```

The riddler 2 will respond in the following format: 

```
[level];[challenge ID];[challenge]
```

In total *Riddlers Legacy Two* sends nine different challenge types which are identified by the **challenge ID**.

| Challenge ID |      Decription      | Example Challenge                                            | Example Solution |
| :----------: | :------------------: | ------------------------------------------------------------ | ---------------- |
|      0       |        ASCII         | 10112099108                                                  | excl             |
|      1       |        Beale         | aHR0cHM6Ly93d3cuaGFja2luZy1sYWIuY29tL2Nhc2VzLzcwMzct<br>cmlkZGxlcnMtbGVnYWN5LXR3by9mM2ExMjliYmNkYTgzMDEyM2JmYzRiYw==:19,7<br>https://www.hacking-lab.com/cases/7037-riddlers-legacy-two/f3a129bbcda830123bfc4bc | so               |
|      2       |        Base64        | bG9ib3RvbWllcw==                                             | lobotomies       |
|      3       |        Caesar        | 10:WKSXPBKWOC                                                | MAINFRAMES       |
|      4       |        Morse         | - .-. .- -- .--. --- .-.. .. -. . ...                        | TRAMPOLINES      |
|      5       |       Reverse        | speeb                                                        | beeps            |
|      6       | Transposition Cipher | 8:pordise0s0t0h0o0                                           | priesthoods      |
|      7       |       Vigenere       | jdk:flbngc                                                   | wireds           |
|      8       |         XOR          | j:BAMJAQYP                                                   | nickle           |

# Challenge Theory
All nine challenge types are explained independently. For each challenge type, you'll need to write a function that takes the specific challenge as a parameter and returns the solution to it. At the end of **Challenge Theory** you're expected to have nine functions created which will be used in the chapter **Automation**.

## ASCII Challenge
The [*American Standard Code for Information Interchange*]([https://en.wikipedia.org/wiki/ASCII](https://en.wikipedia.org/wiki/ASCII)) is a 7-bit character encoding. ASCII defines 127 characters consisting of 33 unprintable and 95 printable characters.

You can solve the ASCII challenge by simply converting either two or three digits of the number to it's ASCII equivalent. You know if a character is encoded in three digits if the number starts with 1. 

![ASCII table](/media/challenge/png/a2a9c202-2ab9-4266-a23f-fa22acf0afa6.png)

### Example

``` python
ascii_string = "10112099108"
# solution = excl
```

 1. Read the first digit

    - 'digit == 1': the ASCII character is encoded in three digits so read the first three digits

      ``` python
      encoded_character = ascii_string[0:2]
      # encoded_character = '101'
      ```

    - 'digit > 1': the ASCII character is encoded in two digits so read only two digits

      ``` python
      encoded_character = ascii_string[6:7]
      # encoded_character = '99'
      ```

 2. Convert the previously read digits to the corresponding character

    ``` python
    character = chr(101)
    # character = e
    ```

 3. Loop through *ascii_string* until you've parsed all characters

### Python Function

Write your own *solve_ascii* function now which takes the challenge `10112099108` as an argument.
Your function is expected to return `excl`.

``` python
def solve_ascii(challenge):
	# TODO: solve ascii challenge
```

## Beale Challenge

The [Beale-Cipher](https://en.wikipedia.org/wiki/Beale_ciphers) is a set of three cipher texts. Only one of them has been solved so far. 

The Beale Challenge in *Riddlers Legacy Two* is tricky, in a response you'll get the following ciphertext as a challenge:

```
HR0cHM6Ly93d3cuaGFja2luZy1sYWIuY29tL2Nhc2VzLzcwMzct
cmlkZGxlcnMtbGVnYWN5LXR3by9mM2ExMjliYmNkYTgzMDEyM2JmYzRiYw==:19,7
```

The left-hand side of the **semicolon** is a base64 encoded URL.  
If you encode that URL you'll get: [https://www.hacking-lab.com/cases/7037-riddlers-legacy-two/f3a129bbcda830123bfc4bc](https://www.hacking-lab.com/cases/7037-riddlers-legacy-two/f3a129bbcda830123bfc4bc)

Now head over to that URL and you'll see the following text (cutout only for readability):

> I have deposited in the county of Bedford, about four miles from
> Buford's, in an excavation or vault, six feet below the surface of the
> ground, the following articles, belonging jointly to the parties whose
> names are given in number three, herewith...

For every *Beale-Cipher* challenge you can use that text statically assigned to a variable in your python code.

### Example

```python
beale = "Look at this example of the Beale Cipher:8,5,5,1"
# solution = cool
```

The numbers `8, 5, 5, 1` are indices pointing at the word in the array above - but the array starts at 1. To decrypt the *Beale-Cipher* you simply read the first digit of the word in the array at the position of  `number - 1`.

    beale[8] = c
    beale[5] = o
    beale[5] = o
    beale[1] = l

 1. Extract the indices from *beale*

    ``` python
    indices = beale.split(':')[1]
    # indices = "8,5,5,1"
    ```

2. Split the text by each whitespace and save it to a list

   ``` python
    text = beale.split(':')[0]
    words = text.split()
    # words = ['look', 'at', 'this', 'example', 'of', 'the', 'beale', 'cipher']
   ```

3. Extract the first letter of the words using *indices* as the index for the word

   ``` python
   solution = ''
   for index in indices:
       solution += words[int(index) - 1][0]
   ```

### Python Function

Write your own *solve_beale* function now which takes the challenge `8,5,5,1` as an argument.
Your function is expected to return `cool`.

**Hint:** The *Beale*-text is static so you can hardcode it into your function. Just replace it with the real beale from [https://www.hacking-lab.com/cases/7037-riddlers-legacy-two/f3a129bbcda830123bfc4bc](https://www.hacking-lab.com/cases/7037-riddlers-legacy-two/f3a129bbcda830123bfc4bc) after you've tested your function.

``` python
def solve_beale(challenge):
	# TODO: solve beale challenge
	beale = "Look at this example of the Beale Cipher" # Replace this with the real beale
```

## Base64 Challenge

Base64 is a method to encode 8-bit binary data to a string consisting of codepage-independent ASCII-characters only. The encoding bloats the space requirements in a data-stream by 33%.

ASCII uses the character `A-Z`, `a-z`, `0-9`, `+` and `/` as well as `=` at the end. For the encoding three bytes of the data-stream are separated in 6-bit-blocks. Each of these blocks represents a number from 0 to 63. These numbers then are converted in *printable ASCII characters* using the table below.

![Base64 table](/media/challenge/png/d2c78722-58f9-448b-8258-c9c5751d6dd3.png?raw=true)

### Example

As there already exists a base64 library in python use that library as writing a base64 decoder isn't a short task. So by using the library, the decoding is straight forward.

``` python
import base64

base = "bG9ib3RvbWllcw=="

solution = base64.b64decode(base).decode('ascii')

# solution = lobotomies
```

### Python Function

Write your own *solve_base64* function now which takes the challenge `bG9ib3RvbWllcw==` as an argument.
Your function is expected to return `lobotomies`.

``` python
def solve_base64(challenge):
	# TODO: solve base64 challenge
```

## Caesar Challenge

**Important:** If you've already solved *Riddlers Legacy One* you might want to head over to the next challenge theory as this description would be redundant.

The Caesar cipher is a monoalphabetic cipher which is based on a simple substitution of the characters by shifting each character the same amount over the alphabet.

![Caesar Shift](/media/challenge/png/6625944d-4350-4fac-887f-54d30e085b51.png?raw=true)

### Example

``` python
caesar = "10:WKSXPBKWOC"
# solution = MAINFRAMES
```

The *10* before the semicolon indicates a right shift over the alphabet by *10*. This means "W" in the cipher-text was an "M" initially and "S" was an "I". You can simply align the plain-text alphabet starting at "A" with the cipher-text alphabet starting at "K".

>|**PlainText**  |A|B|C|D|E|F|G|H|I|J|K|L|M|N|O|P|Q|R|S|T|U|V|W|X|Y|Z
>|-              |-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-
>|**CipherText** |K|L|M|N|O|P|Q|R|S|T|U|V|W|X|Y|Z|A|B|C|D|E|F|G|H|I|J

You can now decrypt the cipher `WKSXPBKWOC` by simply mapping the cipher-text characters to the plain-text characters.

```
W = M
K = A
S = I
X = N
P = F
B = R
K = A
W = M
O = E
C = S
```

1. Extract the shift from *caesar*

   ```python
   shift = caesar.split(':')[0]
   ```

2. Extract the cipher from *caesar*

   ``` python
   cipher = caesar.split(':')[1]
   ```

3. Create a mapping between the plain-text alphabet and the cipher-text alphabet using [string.maketrans](https://docs.python.org/2/library/string.html?highlight=maketrans#string.maketrans)

   ```python
   import string ascii_uppercase as UC
   
   mapping = str.maketrans(UC, UC[shift:] + UC[:shift])
   ```

 4. Decrypt the cipher using the mapping table and [string.translate](https://docs.python.org/2/library/string.html?highlight=translate#string.translate)

    ``` python
    cipher.translate(mapping)
    ```

### Python Function

Write your own *caesar* function now which takes the challenge `10:WKSXPBKWOC` as an argument.
Your function is expected to return `MAINFRAMES`.

``` python
def solve_caesar(challenge):
	# TODO: solve caesar challenge
```

## Morse Challenge

Morse code is an encoding method used in telecommunication. Test characters are encoded as standardized sequences consisting of two different signal durations which are called `dots` and `dashes`.

![Morse Alphabet](/media/challenge/png/87900f23-f307-4eba-b54b-6359debc5a46.png?raw=true)

### Example

``` python
morse = "- .-. .- -- .--. --- .-.. .. -. . ..."
# solution = TRAMPOLINES
```

Each character in the morse code is separated by a whitespace.

1. Create a [dictionary](https://docs.python.org/3/tutorial/datastructures.html?highlight=dictionaries#dictionaries) for the morse alphabet, keep in mind that you have to switch the keys and values as you have to lookup by morse key

   ```python
   morse_alphabet = {
   ".-": "A", 
   "-...": "B", 
   "-.-.": "C",
   ... 
   }
   ```

2. Split the the morse code sequence

   ```python
   morse_separated = morse.split()
   ```

3. Decode each morse code in *morse_separated* and concatenate the solution

   ```python
   solution = ''
   for code in morse_separated:
      solution += morse_alphabet[code]
   ```

### Python Function

Write your own *morse* function now which takes the challenge `- .-. .- -- .--. --- .-.. .. -. . ...` as an argument.
Your function is expected to return `TRAMPOLINES`.

``` python
def solve_morse(challenge):
	# TODO: solve morse challenge
```

## Reverse Challenge

The reverse challenge is fairly easy.  Simply reverse the received challenge.

### Example

``` python
reverse = "speeb"
# solution = beeps
```

To reverse a string in python you can apply a method called *slicing*. Python offers the ability to access characters in a string using `encoded[0] -> s`, in addition you can specify a range by `encoded[0:1] -> sp`.  To reverse the given string just provide the stepping value as follows:

```python
solution = reverse[::-1]
```

### Python Function

Write your own *solve_reverse* function now which takes the challenge `speeb` as an argument.
Your function is expected to return `beeps`.

``` python
def solve_reverse(challenge):
	# TODO: solve reverse challenge
```

## Caesar Box / Columnar Transposition Challenge

The Caesar Box is a **columnar transposition cipher** in which the plain text is written out in rows and then reading off the cipher-text in columns.

### Example

```
transposition = "8:pordise0s0t0h0o0"
# solution = priesthoods
```

The *8* indicates the number of *columns* used in the cipher. The trailing **zeros** are just used as a padding.

```
|       |  0   |  1   |  2   |  3   |  4   |  5   |  6   |  7   |
| :---: | :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: |
| **0** |  p   |  r   |  i   |  e   |  s   |  t   |  h   |  o   |
| **1** |  o   |  d   |  s   |  0   |  0   |  0   |  0   |  0   |
```
To get the cipher-text eg. encrypt just read column by column from the above example `pordise0s0t0h0o0`.



To decrypt the cipher use the key *8* as the number of rows. By dividing the cipher's length by two you can calculate the number of columns needed. `16 / 8 = 2`

```
|       |  0   |  1   |
| :---: | :--: | :--: |
| **0** |  p   |  o   |
| **1** |  r   |  d   |
| **2** |  i   |  s   |
| **3** |  e   |  0   |
| **4** |  s   |  0   |
| **5** |  t   |  0   |
| **6** |  h   |  0   |
| **7** |  o   |  0   |
```


1. Extract the *key* from *transposition*

   ```python
   key = transposition.split(':')[0]
   ```

2. Extract the *cipher* from *transposition*

   ```python
   cipher = transposition.split(':')[1]
   ```

3. Calculate the columns which indicate how many times we have to iterate over the string

   ``` pyt
   steps = len(cipher) // int(key)  # "//" operator for automatic rounding
   ```

4. Loop *steps*-times over the *cipher* and extract at each iteration the characters which are a multiple of *steps* using [itertools islice](https://docs.python.org/3/library/itertools.html?highlight=islice#itertools.islice), you can use a lambda within join to **filter** characters

   ```python
   from itertools import islice
   
   for step in range(steps):
       it = islice(cipher, step, None, steps)
       solution += ''.join(filter(lambda char: char != "0", it)
   ```

### Python Function

Write your own *solve_transposition function now which takes the challenge `8:pordise0s0t0h0o0` as an argument.
Your function is expected to return `priesthoods`.

``` python
def solve_transposition(challenge):
	# TODO: solve transposition challenge
```

## Vigenere Challenge

The Vigenère cipher is a polyalphabetic substitution method. The cleartext is split into monograms which are then substituted by the secret-text character which is chosen with the aid of the *Vigenère table*.

![Vigenère square](/media/challenge/png/3dc5f164-b002-4a68-b7f1-6c242332ed76.png?raw=true)

### Example

``` python
vigenere = "jdk:flbngc"
# solution = wireds
```

The left-hand side of the string `jdk` is our key. Write out the cipher and write the key repeatedly below the cipher until their lengths match.

| Cipher  | f    | l    | b    | n    | g    | c    |
| ------- | ---- | ---- | ---- | ---- | ---- | ---- |
| **Key** | j    | d    | k    | j    | d    | k    |

To decipher Vigenère simply mark the row (Y-axis) at the index of the matching key character `j`. Then follow this row horizontally until you reach the cipher character `f`. Now mark this column (X-axis) and voilá the index on the x-axis is your clear-text character `W`. Repeat the previous steps for all characters in your cipher.

![Vigenère square decipher](/media/challenge/png/a68a3ae2-5b84-4302-afa9-bd5c19852a12.png?raw=true)

### Example

```python
vigenere = "jdk:flbngc"
# solution = wireds
```

1. Extract the *key* `jdk`from *vigenere*

   ```python
   key = vigenere.split(':')[0]
   ```

2. Extract the *cipher* `flbngc` from *vigenere*

   ```python
   cipher = vigenere.split(':')[1]
   ```

3. You have to extend the *key* to match the *cipher* length, in other words you have to repeatedly access the content of *key* circularly

   ```python
   from itertools import cycle
   key_cycle = cycle(key)
   
   for char in cipher:
       current_key = next(key_cycle)  # current_key holds the element
   ```

4. Calculate the needed shift of the character

   ```py
   shift = (ord(char) - ord(current_key) + 26) % 26
   ```

5. Convert the *shift* back to a character, you have to define a starting-point with `ord('a')`

   ```python
   deciphered_char = chr(ord('a') + shift)
   ```

6. Add *deciphered_char* to your solution

   ```python
   solution += deciphered_char
   ```


### Python Function

Write your own *solve_vigenere* function now which takes the challenge `jdk:flbngc` as an argument.
Your function is expected to return `wireds`.

``` python
def solve_vigenere(challenge):
	# TODO: solve vigenere challenge
```

## XOR Challenge

Exclusive or is a logical operation indicating that of two given inputs only one of them has the value `true`.

![XOR table](/media/challenge/png/d2c5d20a-6502-416e-959d-cbc621bea319.png?raw=true)

### Example

``` python
xor = "j:BAMJAQYP"
# solution = nickle
```

1. Extract the *key* `j`from *xor*

   ```python
   key = xor.split(':')[0]
   ```

2. Convert the *key* to an integer (unicode equivalent) using the function [ord](https://docs.python.org/3/library/functions.html?highlight=ord#ord)

   ```python
   unicode_key = ord(key)
   ```

3. Extract the encoded cipher `BAMJAQYP` from *xor*

   ```python
   cipher = xor.split(':')[1]
   ```

 4. As the cipher is encoded in base64 decode it

    ```python
    import base64
    decoded = base64.b64decode(cipher)
    ```

5. For each character run a bitwise exclusive or ( ^-operator) and convert it back to a character

   ```python
   solution = ''
   for character in decoded:
      solution += chr(character ^ unicode_key)
   ```

### Python Function

Write your own *solve_xor* function now which takes the challenge `j:BAMJAQYP` as an argument.
Your function is expected to return `nickle`.

``` python
def solve_xor(challenge):
	# TODO: solve xor challenge
```

# Automation

As the service expects your answer very quickly there's no way to manually solve the challenges and respond with the correct solution. Therefore you'll have to automate the whole process using Python3.

## Step 1

As all your functions from chapter **Challenge theory** work flawlessly you can now start to automate the requests.

To start using *pwntools* you can just import `pwn` as follows:

``` python
#!/usr/bin/env python
from pwn import *
```

## Step 2

### Interaction with the service

1. Create a TCP connection using [sockets](https://docs.pwntools.com/en/stable/tubes/sockets.html) and extend the above script with:

   ``` python
   service = remote('[Target IP-Address]', [Target Port])
   ```

2. To receive data from the service you use the *service* object from above:

   ``` python
   raw_data = service.recvline()
   ```

3. The data received by the service isn't usable upfront, look at the following response: `b0;0;10112099108\n`. You are only interested in the portion **0;0;10112099108** but before you can extract the portion you first have to decode the *rawData* by using:

   ``` python
   decoded_data = raw_data.decode('latin-1')
   ```

4. Your *decoded_data* will look like this `0;0;10112099108\n` remove the `\n` now

   ``` python
   decoded_data = decoded_data.replace('\n','')
   ```

5. From the format `[level];[challenge ID];[challenge]` we only need the *challenge ID* and the *challenge* itself as the level is only informative indicating how many challenges you've solved so far

   ``` python
   challenge_id = decoded_data.split(';')[1]
   challenge = decoded_data.split(';')[2]
   ```

6. Now you have to call the correct challenge function based on your *challengeID* with the argument *challenge*. As python doesn't offer a `switch case` you can either go for an `if elif  elif` or you can use a dictionary with lambdas as follows:

   ``` python
   functions = {
      0: lambda arg: solve_ascii(arg),
      1: lambda arg: solve_beale(arg),
      ...
   }
   
   # call the lambda in functions
   solution = functions[challenge_id](challenge)
   ```

7. Return the *solution* to the service by calling

   ```python
   service.sendline(solution)
   ```

8. As the service responds with the *partial solution* you've accomplished so far print out the response, that way you can follow your Python3 program solving *Riddlers Legacy Two.

   ``` python
   print(service.recvline())
   ```

9. Keep in mind that you have to continue sending new solutions and receiving new challenges until the service responds with `CONGRATULATION! YOU DID IT!`, as an alternative run `try: except EOFError:` within your loop to stop as soon as the service closes its connection.

   ``` python
   while True:
      try:
   	   # your code goes here
   	   
      except EOFError:
   	   print("Connection closed")
   	   break 
   ```

## Step 3
### Finalize your Solver
Please finalize your work. Write python3 software that is connecting the riddler service using `pwntools` and solving the puzzles using your `challenge solvers`  until the riddler service is sending the final string `CONGRATULATION! YOU DID IT!`

