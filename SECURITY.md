1. Caesar Cipher
from Crypto.Cipher import AES
def caesar(text, shift):
    result = ""
    for char in text:
        if char.isalpha():
            base = ord('A') if char.isupper() else ord('a')
            result += chr((ord(char) - base + shift) % 26 + base)
        else:
            result += char
    return result
text = "HELLO"
encrypted = caesar(text, 3)
decrypted = caesar(encrypted, -3)
print("Original :", text)
print("Encrypted:", encrypted)
print("Decrypted:", decrypted)

Output:
Original : HELLO
Encrypted: KHOOR
Decrypted: HELLO

=============================================================================================================================================================================================================
2. Playfair Cipher
def generate_matrix(key):
    key = key.upper().replace("J", "I")
    seen = []
    for ch in key + "ABCDEFGHIKLMNOPQRSTUVWXYZ":
        if ch not in seen:
            seen.append(ch)
    return [seen[i*5:(i+1)*5] for i in range(5)]
def find_pos(matrix, ch):
    for r, row in enumerate(matrix):
        if ch in row:
            return r, row.index(ch)
def playfair_encrypt(text, key):
    matrix = generate_matrix(key)
    text = text.upper().replace("J", "I").replace(" ", "")
    pairs = []
    i = 0
    while i < len(text):
        a = text[i]
        b = text[i+1] if i+1 < len(text) else 'X'
        if a == b:
            pairs.append((a, 'X'))
            i += 1
        else:
            pairs.append((a, b))
            i += 2
    result = ""
    for a, b in pairs:
        r1,c1 = find_pos(matrix, a)
        r2,c2 = find_pos(matrix, b)
        if r1 == r2:
            result += matrix[r1][(c1+1)%5] + matrix[r2][(c2+1)%5]
        elif c1 == c2:
            result += matrix[(r1+1)%5][c1] + matrix[(r2+1)%5][c2]
        else:
            result += matrix[r1][c2] + matrix[r2][c1]
    return result
print("Encrypted:", playfair_encrypt("HELLO", "KEY"))

Output:
Encrypted: DAXGM

============================================================================================================================================================================================================
3. Hill Cipher
import numpy as np
def hill_encrypt(text, key_matrix):
    text = text.upper()
    nums = [ord(c) - ord('A') for c in text]
    vector = np.array(nums).reshape(-1, 1)
    result = np.dot(key_matrix, vector) % 26
    return ''.join(chr(int(n) + ord('A')) for n in result.flatten())

key = np.array([[3, 3], [2, 5]])
text = "HI"
encrypted = hill_encrypt(text, key)
print("Original :", text)
print("Encrypted:", encrypted)

Output:
Original : HI
Encrypted: DL

===============================================================================================================================================================================================================
4. Rail Fence Cipher
def rail_fence_encrypt(text, rails):
    fence = [[] for _ in range(rails)]
    rail, direction = 0, 1
    for char in text:
        fence[rail].append(char)
        if rail == 0:
            direction = 1
        elif rail == rails - 1:
            direction = -1
        rail += direction
    return ''.join(''.join(row) for row in fence)
def rail_fence_decrypt(cipher, rails):
    n = len(cipher)
    pattern = []
    rail, direction = 0, 1
    for i in range(n):
        pattern.append(rail)
        if rail == 0: direction = 1
        elif rail == rails - 1: direction = -1
        rail += direction
    indices = sorted(range(n), key=lambda x: pattern[x])
    result = [''] * n
    for i, ch in zip(indices, cipher):
        result[i] = ch
    return ''.join(result)
text = "HELLOWORLD"
enc = rail_fence_encrypt(text, 3)
dec = rail_fence_decrypt(enc, 3)
print("Original :", text)
print("Encrypted:", enc)
print("Decrypted:", dec)

Output:
Original : HELLOWORLD
Encrypted: HOLELWRDLO
Decrypted: HELLOWORLD

===============================================================================================================================================================================
5. Columnar Transposition Cipher
def columnar_encrypt(text, key):
    text = text.replace(" ", "").upper()
    cols = len(key)
    rows = -(-len(text) // cols)          # ceil division
    text = text.ljust(rows * cols, 'X')   # pad with X
    grid = [text[i*cols:(i+1)*cols] for i in range(rows)]
    order = sorted(range(cols), key=lambda x: key[x])
    return ''.join(''.join(row[c] for row in grid) for c in order)
def columnar_decrypt(cipher, key):
    cols = len(key)
    rows = len(cipher) // cols
    order = sorted(range(cols), key=lambda x: key[x])
    grid = {}
    idx = 0
    for c in order:
        grid[c] = list(cipher[idx:idx+rows])
        idx += rows
    return ''.join(''.join(grid[c][r] for c in range(cols)) for r in range(rows))
text = "HELLOWORLD"
key  = [3, 1, 4, 2]
enc  = columnar_encrypt(text, key)
dec  = columnar_decrypt(enc, key)
print("Original :", text)
print("Encrypted:", enc)
print("Decrypted:", dec)

Output:
Original : HELLOWORLD
Encrypted: ELXLHXORWD
Decrypted: HELLOWORLD

===========================================================================================================================================================================
Digital Signature Standard (DSS) using Python

from Crypto.PublicKey import DSA
from Crypto.Signature import DSS
from Crypto.Hash import SHA256
key = DSA.generate(2048)
private_key = key
public_key = key.publickey()
message = "Hello, this is a digital signature example"
msg_bytes = message.encode()
hash_obj = SHA256.new(msg_bytes)
signer = DSS.new(private_key, 'fips-186-3')
signature = signer.sign(hash_obj)
print("Signature:", signature)
verifier = DSS.new(public_key, 'fips-186-3')
try:
    verifier.verify(hash_obj, signature)
    print("Signature is valid")
except ValueError:
    print("Signature is not valid")
    
Output (Example)
Signature: b'...binary data...'
Signature is valid



==========================================================================================================================================================================================

📦 Install Required Library
bashpip install pycryptodome numpy


PYTHON PROGRAM PROCEDURES (Pycrypt Libraries / Logic)
1. Caesar Cipher
         Take plaintext and key (shift value).
         Convert each letter to its ASCII value.
         Shift the letter by key value.
         Wrap around if it exceeds alphabet range.
         Convert back to character → ciphertext.
         For decryption, shift in reverse.
2. Playfair Cipher
         Remove duplicate letters from key.
         Create 5×5 matrix (combine I/J).
         Split plaintext into letter pairs.
         If same letters in pair → insert ‘X’.
         Apply rules:
         Same row → shift right
         Same column → shift down
         Rectangle → swap columns
         Combine letters → ciphertext.
3. Hill Cipher
         Choose key matrix (n×n).
         Convert plaintext letters → numbers (A=0…Z=25).
         Split into blocks of size n.
         Multiply block with key matrix.
        Take mod 26 of result.
        Convert numbers → ciphertext letters.
4. Rail Fence Cipher
        Choose number of rails (rows).
         Write plaintext in zigzag pattern.
         Read row-wise to get ciphertext.
         For decryption, reconstruct zigzag.
5. Columnar Transposition
         Choose a keyword.
        Write plaintext in rows under keyword.
        Number columns based on alphabetical order of key.
        Read columns in sorted order.
        Combine → ciphertext.
        TOOL-BASED PROCEDURES (CrypTool / Snort)
6. DES Algorithm (CrypTool)
         Open CrypTool.
         Enter plaintext.
         Select DES algorithm.
         Enter key (56-bit).
         Click encrypt → ciphertext generated.
         Use same key for decryption.
7. AES Algorithm (CrypTool)
       Open CrypTool.
       Enter plaintext.
      Choose AES (128/192/256-bit).
      Enter key.
      Click encrypt → ciphertext.
      Decrypt using same key.
8. RSA Algorithm (CrypTool)
       Open CrypTool.
       Generate public & private keys.
       Enter plaintext.
       Encrypt using public key.
       Decrypt using private key.
9. Diffie-Hellman Key Exchange
      Choose prime number (p) and base (g).
      User A selects private key → computes public key.
      User B selects private key → computes public key.
      Exchange public keys.
      Compute shared secret key using formula.
      Both get same key.
10. SHA-1 Message Digest
      Open CrypTool.
      Enter message.
      Select SHA-1 hashing.
      Generate hash value (160-bit).
      Output is fixed-length digest.
11. MD5 Message Digest
      Open CrypTool.
      Enter message.
      Select MD5 hashing.
      Generate hash value (128-bit).
      Used for integrity checking.
12. Digital Signature (DSS)
     Import required modules (DSA, SHA256, DSS).
     Generate DSA key pair (private + public key).
     Take the message to be signed.
     Convert message into bytes.
     Create hash of the message using SHA-256.
     Sign the hash using the private key.
     Send message + signature.
     Verify signature using public key.
     Display whether signature is valid or not.
13. Intrusion Detection System (Snort)
     Install Snort tool.
     Configure rules file.
     Start Snort in monitoring mode.
      Capture network packets.
     Detect suspicious activity.
       Generate alerts/logs.



























# Security Policy

## Supported Versions

Use this section to tell people about which versions of your project are
currently being supported with security updates.

| Version | Supported          |
| ------- | ------------------ |
| 5.1.x   | :white_check_mark: |
| 5.0.x   | :x:                |
| 4.0.x   | :white_check_mark: |
| < 4.0   | :x:                |

## Reporting a Vulnerability

Use this section to tell people how to report a vulnerability.

Tell them where to go, how often they can expect to get an update on a
reported vulnerability, what to expect if the vulnerability is accepted or
declined, etc.
