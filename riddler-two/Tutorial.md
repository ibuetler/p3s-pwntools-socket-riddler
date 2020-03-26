
# Introduction
This challenge is about interacting with a web service in Python3 using [pwntools](http://docs.pwntools.com/en/stable/) and it's [sockets library](https://docs.pwntools.com/en/stable/tubes/sockets.html#). 

*Riddlers Legacy Two* follows the same principle as *Riddlers Legacy One* but with a variety of challenges. You'll write several functions solving different crypto puzzles similar to *Riddlers Legacy One*

## Learn how to ...
 - Use pwntools sockets library
 - Automate the interaction with a web service
 - Solve crypto puzzles programmatically

# Preparation
## Step 1
### Start the Resource
Start the *Riddlers Legacy Two* resource above.

![How to start the Riddler-service](../../assets/riddler-two/riddler-start.png?raw=true)

## Step 2
### Analyse the Riddler
#### Service Overview

![Riddler service diagram](../../assets/riddler-two/service-overview.png?raw=true)

#### Manual Connection
Before you start to write some code make yourself familiar with the responses of *Riddlers Legacy Two* by using telnet:

``` python
telnet [IP-Address] [Port]
```

The riddler will respond in the following format: 
```
[level];[challenge ID];[challenge]
```

In total *Riddlers Legacy Two* sends nine different challenge types which are identified by the **challenge ID**.

| Challenge ID | Decription | Example Challenge                                                                                                                                                                                               | Example Solution |
|:--------------:|:----------:|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|------------------|
|       0      |    ASCII   | 10112099108                                                                                                                                                                                                     | excl             |
|       1      |    BEALE   | aHR0cHM6Ly93d3cuaGFja2luZy1sYWIuY29tL2Nhc2VzLzcwMzct<br>cmlkZGxlcnMtbGVnYWN5LXR3by9mM2ExMjliYmNkYTgzMDEyM2JmYzRiYw==:19,7<br>https://www.hacking-lab.com/cases/7037-riddlers-legacy-two/f3a129bbcda830123bfc4bc | so               |
|       2      |   BASE64   | bG9ib3RvbWllcw==                                                                                                                                                                                                | lobotomies       |
|       3      |  CAESAR | 10:WKSXPBKWOC                                                                                                                                                                                                   | MAINFRAMES       |
|       4      |    MORSE   | - .-. .- -- .--. --- .-.. .. -. . ...                                                                                                                                                                           | TRAMPOLINES      |
|       5      |   REVERSE  | speeb                                                                                                                                                                                                           | beeps            |
|       6      |    FENCE   | 5:bsi0b0l0e0                                                                                                                                                                                                    | bibles           |
|       7      |  VIGENERE  | jdk:flbngc                                                                                                                                                                                                      | wireds           |
|       8      |     XOR    | j:BAMJAQYP                                                                                                                                                                                                      | nickle           |

# Challenge theory
All challenge types are explained independently. For each challenge type, you'll have to write a function that takes the specific challenge as a parameter and returns the solution to it. 

At the end of **Challenge theory** you're expected to have nine functions created which will be used in the chapter **Automation**.

## ASCII Challenge
The [*American Standard Code for Information Interchange*]([https://en.wikipedia.org/wiki/ASCII](https://en.wikipedia.org/wiki/ASCII)) is a 7-bit character encoding. ASCII defines 127 characters consisting of 33 unprintable and 95 printable characters.

You can solve the ASCII challenge by simply converting either two or three digits of the number to it's ASCII equivalent. You know if a character is encoded in three digits if the number starts with 1. 

![ASCII table](../../assets/riddler-two/ascii-table.png?raw=true)

### Example
``` python
asciiString = "10112099108"
# solution = excl
``` 
 1. Read the first digit
    - 'digit == 1': the ASCII character is encoded in three digits so read the first three digits
       ``` python
       encodedCharacter = asciiString[0:2]
       # encodedCharacter = '101'
	   ```
    - 'digit > 1': the ASCII character is encoded in two digits so read only two digits
       ``` python
       encodedCharacter = asciiString[6:7]
       # encodedCharacter = '99'
	   ```
 2. Convert the previously read digits to the corresponding character
    ``` python
	character = chr(101)
    # character = e
    ``` 
 4. Loop through *asciiString* until you've parsed all characters

### Python Function
Write your own *ASCII* function now which takes the challenge `10112099108` as an argument.
Your function is expected to return `excl`.
``` python
def ASCII(challenge):
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
Write your own *BEALE* function now which takes the challenge `8,5,5,1` as an argument.
Your function is expected to return `cool`.

**Hint:** The *BEALE*-text is static so you can hardcode it into your function. Just replace it with the real beale from [https://www.hacking-lab.com/cases/7037-riddlers-legacy-two/f3a129bbcda830123bfc4bc](https://www.hacking-lab.com/cases/7037-riddlers-legacy-two/f3a129bbcda830123bfc4bc) after you've tested your function.

``` python
def BEALE(challenge):
	# TODO: solve beale challenge
	beale = "Look at this example of the Beale Cipher" # Replace this with the real beale
```

## Base64 Challenge
Base64 is a method to encode 8-bit binary data to a string consisting of codepage-independent ASCII-characters only. The encoding bloats the space requirements in a data-stream by 33%.

ASCII uses the character `A-Z`, `a-z`, `0-9`, `+` and `/` as well as `=` at the end. For the encoding three bytes of the data-stream are separated in 6-bit-blocks. Each of these blocks represents a number from 0 to 63. These numbers then are converted in *printable ASCII characters* using the table below.

![Base64 table](../../assets/riddler-two/base64-table.png?raw=true)

### Example
As there already exists a base64 library in python use that library as writing a base64 decoder isn't a short task. So by using the library, the decoding is straight forward.

``` python
import base64

base = "bG9ib3RvbWllcw=="

solution = base64.b64decode(base).decode('ascii')

# solution = lobotomies
``` 

### Python Function
Write your own *BASE64* function now which takes the challenge `bG9ib3RvbWllcw==` as an argument.
Your function is expected to return `lobotomies`.
``` python
def BASE64(challenge):
	# TODO: solve base64 challenge
```

## Caesar Challenge
**Important:** If you've already solved *Riddlers Legacy One* you might want to head over to the next challenge theory as this description would be redundant.

The Caesar cipher is a monoalphabetic cipher which is based on a simple substitution of the characters by shifting each character the same amount over the alphabet.

![Caesar Shift](../../assets/riddler-two/caesar-shift.png?raw=true)

### Example
``` python
caesar = "10:WKSXPBKWOC"
# solution = MAINFRAMES
```
The *10* before the semicolon indicates a right shift over the alphabet by *10*. This means "W" in the cipher-text was an "M" initially and "S" was an "I". You can simply align the plain-text alphabet starting at "A" with the cipher-text alphabet starting at "K".

>
|**PlainText**  |A|B|C|D|E|F|G|H|I|J|K|L|M|N|O|P|Q|R|S|T|U|V|W|X|Y|Z
|-              |-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-
|**CipherText** |K|L|M|N|O|P|Q|R|S|T|U|V|W|X|Y|Z|A|B|C|D|E|F|G|H|I|J

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
Write your own *CAESAR* function now which takes the challenge `10:WKSXPBKWOC` as an argument.
Your function is expected to return `MAINFRAMES`.
``` python
def CAESAR(challenge):
	# TODO: solve caesar challenge
```

## Morse Challenge
Morse code is an encoding method used in telecommunication. Test characters are encoded as standardized sequences consisting of two different signal durations which are called `dots` and `dashes`.

![Morse Alphabet](../../assets/riddler-two/morse-alphabet.png?raw=true)

### Example
``` python
morse = "- .-. .- -- .--. --- .-.. .. -. . ..."
# solution = TRAMPOLINES
```
Each character in the morse code is separated by a whitespace.

1. Create a [dictionary](https://docs.python.org/3/tutorial/datastructures.html?highlight=dictionaries#dictionaries) for the morse alphabet, keep in mind that you have to switch the keys and values as you have to lookup by morse key
   ```python
   morseAlphabet = {
	".-": "A", 
	"-...": "B", 
	"-.-.": "C",
	... 
   }
   ```
2. Split the the morse code sequence
   ```python
   morseSeparated = morse.split()
   ```
3. Decode each morse code in *morseSeparated* and concatenate the solution
   ```python
   solution = ''
   for code in morseSeparated:
	   solution += morseAlphabet[code]
   ```

### Python Function
Write your own *MORSE* function now which takes the challenge `- .-. .- -- .--. --- .-.. .. -. . ...` as an argument.
Your function is expected to return `TRAMPOLINES`.
``` python
def MORSE(challenge):
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
reversed = reverse[::-1]
```
### Python Function
Write your own *REVERSE* function now which takes the challenge `speeb` as an argument.
Your function is expected to return `beeps`.
``` python
def REVERSE(challenge):
	# TODO: solve reverse challenge
```

## Rail Fence Challenge
The Rail fence cipher also called zigzag cipher is a transposition cipher. The plain text is written downwards and diagonally on "rails" of an imaginary fence, then moving up when the bottom rail is reached.

### Example
```
fence = "5:bsi0b0l0e0"
# solution = bibles
```
TODO: Fence written out example

TODO: Implementation of algorithm

### Python Function
Write your own *FENCE* function now which takes the challenge `5:bsi0b0l0e0` as an argument.
Your function is expected to return `bibles`.
``` python
def FENCE(challenge):
	# TODO: solve fence challenge
```

## Vigenere Challenge
The Vigenère cipher is a polyalphabetic substitution method. The cleartext is split into monograms which are then substituted by the secret-text character which is chosen with the aid of the *Vigenère table*.

![Vigenère square](../../assets/riddler-two/vigenere-square.png?raw=true)

### Example

``` python
vigenere = "jdk:flbngc"
# solution = wireds
```
The left-hand side of the string `jdk` is our key. Write out the cipher and write the key repeatedly below the cipher until their lengths match.

| Cipher  | f | l | b | n | g | c |
|---------|---|---|---|---|---|---|
| **Key** | j | d | k | j | d | k |

To decipher Vigenère simply mark the row (Y-axis) at the index of the matching key character `j`. Then follow this row horizontally until you reach the cipher character `f`. Now mark this column (X-axis) and voilá the index on the x-axis is your clear-text character `W`. Repeat the previous steps for all characters in your cipher.

![Vigenère square decipher](../../assets/riddler-two/vigenere-square-decipher.png?raw=true)

TODO: Implementation of algorithm

### Python Function
Write your own *VIGENERE* function now which takes the challenge `jdk:flbngc` as an argument.
Your function is expected to return `wireds`.
``` python
def VIGENERE(challenge):
	# TODO: solve vigenere challenge
```

## XOR Challenge
Exclusive or is a logical operation indicating that of two given inputs only one of them has the value `true`.

![XOR table](../../assets/riddler-two/xor-table.png?raw=true)

### Example
``` python
xor = "j:BAMJAQYP"
# solution = nickle
```
1. Extract the "key" `j`from *xor*
   ```python
   key = xor.split(':')[0]
   ```
2. Convert the *key* to an integer (unicode equivalent) using the function [ord](https://docs.python.org/3/library/functions.html?highlight=ord#ord)
   ```python
   unicodeKey = ord(key)
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
4. For each character run a bitwise exclusive or ( ^-operator) and convert it back to a character
   ```python
   solution = ''
   for character in decoded:
	   solution += chr(character ^ unicodeKey)
   ```

### Python Function
Write your own *XOR* function now which takes the challenge `j:BAMJAQYP` as an argument.
Your function is expected to return `nickle`.
``` python
def XOR(challenge):
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
   rawData = service.recvline()
   ```
3. The data received by the service isn't usable upfront, look at the following response: `b0;0;10112099108\n`. You are only interested in the portion **0;0;10112099108** but before you can extract the portion you first have to decode the *rawData* by using:
   ``` python
   decodedData = rawData.decode('latin-1')
   ```
4. Your *decodedData* will look like this `0;0;10112099108\n` remove the `\n` now
   ``` python
   decodedData = decodedData.replace('\n','')
   ```
5. From the format `[level];[challenge ID];[challenge]` we only need the *challenge ID* and the *challenge* itself as the level is only informative indicating how many challenges you've solved so far
   ``` python
   challengeID = decodedData.split(';')[1]
   challenge = decodedData.split(';')[2]
   ```
6. Now you have to call the correct challenge function based on your *challengeID* with the argument *challenge*. As python doesn't offer a `switch case` you can either go for an `if elif  elif` or you can use a dictionary with lambdas as follows:
   ``` python
   functions = {
	   0: lambda arg: ASCII(arg),
	   1: lambda arg: BEALE(arg),
	   ...
	}
	
   # call the lambda in functions
   solution = functions[challengeID](challenge)
   ```
   
7. Return the *solution* to the service by calling
   ```python
   service.sendline(solution)
   ```
   
8. As the service responds with the *partial solution* you've accomplished so far print out the response, that way you can follow your Python3 program solving *Riddlers Legacy One*.
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
