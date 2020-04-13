# Introduction
This python3 exercise is about interacting with a `socket` service using the commonly known [pwntools](http://docs.pwntools.com/en/stable/) and it's [sockets library](https://docs.pwntools.com/en/stable/tubes/sockets.html#). 

```
+-----------------+            +---------------------+
| python3 program +----------->+riddler 1 on port 80 |
+-----------------+            +---------------------+
```

## Learn how to ...
 - Use pwntools sockets library
 - Automate the interaction with a web service

## Tasks
* Task 1: Please analyze the Riddler Service on port `80`
* Task 2: Write a Python3 solver for the Riddler One Service

# Analyzing Riddler
## Step 1

### Resource
* Get the name of the service from `RESOURCES` aka `riddler.vm.vuln.land`
* the `riddler one` service is running on port `80`


## Step 2
### Analysis of Riddler One Service
Riddler is a network services that is listening on port `80`. It is **not** http - but still running on port `80`!

![service-overview.png](/media/challenge/png/e6baaba7-65b1-45bd-ad73-bff3ccb11de9.png)


#### Testing manually the Connection
Before we start to write some code, we should make ourself familiar with the responses of *Riddler 1* by using `telnet` or `netcat`:

``` python
telnet [IP-Address] [Port]
telnet riddler.vm.vuln.land 80
```

``` python
nc -v [IP-Address] [Port]
nc -v riddler.vm.vuln.land 80
```

The riddler will respond with a "puzzle". 

``` python
nc -v riddler.vm.vuln.land 80
Ncat: Version 7.80 ( https://nmap.org/ncat )
Ncat: Connected to 152.96.6.243:80.
19:MKTPEXW

nc -v riddler.vm.vuln.land 80
Ncat: Version 7.80 ( https://nmap.org/ncat )
Ncat: Connected to 152.96.6.243:80.
14:SZJWGVSG

nc -v riddler.vm.vuln.land 80
Ncat: Version 7.80 ( https://nmap.org/ncat )
Ncat: Connected to 152.96.6.243:80.
16: DQDEJUSXDEBEWYUI
```

As you can see in the three riddler responsese above, the rsponse is looks encrypted. Let's start to investigate the cipher furthermore using CrypTools.

## Step 3
### Analysis Riddler One Response
#### CrypTools
Start CrypTools, create a new file with *"File > New"* and paste the cipher `DQDEJUSXDEBEWYUI` into that new file. Fiddle around and try to solve it using the statistic functions of CrypTool.

#### Spoiler
The Riddler is sending its challenges encrypted using a symmetric encryption algorithm called **Caesar cipher**.

The Caesar cipher is a monoalphabetic cipher which is based on a simple substitution of the characters by shifting each character the same amount over the alphabet.

![service-overview.png](/media/challenge/png/6625944d-4350-4fac-887f-54d30e085b51.png)

By using *"Analysis > Symmetric Encryption (classic) > Ciphertext-Only > Caesar"* decrypting is straight forward and leads to the key **Q** that correlates with a right-shift over the alphabet by **16**. You can simply align the plain-text alphabet starting at "A" with the cipher-text alphabet starting at "Q".
This means "Q" in the cipher-text was an "A" initially and "D" was an "N".
>
|**PlainText**  |A|B|C|D|E|F|G|H|I|J|K|L|M|N|O|P|Q|R|S|T|U|V|W|X|Y|Z
|-              |-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-
|**CipherText** |Q|R|S|T|U|V|W|X|Y|Z|A|B|C|D|E|F|G|H|I|J|K|L|M|N|O|P

You can now decrypt the cipher `DQDEJUSXDEBEWYUI` by simply mapping the cipher-text characters to the plain-text characters.

```
D = N
Q = A
D = N
E = O 
J = T 
U = E
S = C
X = H
D = N
E = O
B = L
E = O
W = G
Y = I
U = E
I = S
```
## Step 4
### Write your Caesar Decrypt Function
Write your own python3 *CAESAR* function, which takes the challenge `6:IUTMXGZARGZOUTY` as an argument.
Your function is expected to return `CONGRATULATIONS`.

``` python
caesar = "6:IUTMXGZARGZOUTY"
# solution = CONGRATULATIONS

def CAESAR(challenge):
	# TODO: solve caesar challenge
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

# Connection to the Riddler Service
As the service expects your responses to the riddler puzzles very quickly, there's no way to manually decrypt the cipher and respond with the solution. Therefore you'll have to automate the whole process. Please write a python3 programm that is doing this job for you. 

**Hint:** The service always responds in **UPPERCASE** and the alphabet is **Latin**, therefore the alphabet includes **ABCDEFGHIJKLMNOPQRSTUVWXYZ**. Keep this in mind when you are writing your function.

## Step 1
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
3. The data received by the service isn't usable upfront, look at the following response: `'b7:ALEA\n'`. You are only interested in the portion **7:ALEA** but before you can extract the portions you first have to decode the *rawData* by using:
   ``` python
   decodedData = rawData.decode('latin-1')
   ```
4. Besides, you should consider **replacing** the *'\n'* from *decodedData* with **''** as this could interfere with your *CAESAR* function.
   ``` python
   decodedData = decodedData.replace('\n', '')
   ```

5. After you've called your *CAESAR* function with the "cleaned" string you now have to send the encrypted text back to the service:
   ``` python
   service.sendline(decrypted)
   ```
6. As the service responds with the *partial solution* you've accomplished so far print out the response, that way you can follow your Python3 program solving *Riddlers Legacy One*.
   ``` python
   print(service.recvline())
   ```
7. Keep in mind that you have to continue sending new solutions and receiving new challenges until the service responds with `CONGRATULATION! YOU DID IT!`, as an alternative run `try: except EOFError:` within your loop to stop as soon as the service closes its connection.

   ``` python
   while True:
      try:
      # your code goes here
      
   except EOFError:
      print("Connection closed")
      break 
   ```

## Write a Python3 solver for the Riddler One Service
Please finalize your work. Write python3 software that is connecting the riddler service using `pwntools` and solving the puzzles using `ceasar` shifts until the riddler service is sending the final string `CONGRATULATION! YOU DID IT!`

