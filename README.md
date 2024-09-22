# AES 128/192/256 (C)


## Description 

This program is a C implementation of AES 128/192/256. I chose this cipher since it's one of the safest, and for the
256 variant, it's also quantum-resistant. It also can be expected to be used more widely in the next decades.
This uses a ECB mode. I hesitated with implementing other modes, safer ones, like CBC or CTS, but as it is my first
try with ciphers (minus the pset2) I thought it was a better choice to stick with ECB.

It contains one file, AES.c.
The AES.c file contains 6 functions : main, keyschedule128, keyschedule192, keyschedule 256, encrypt and decrypt.
You need two commands line arguments to start : 1 : "encrypt" or "decrypt" and 2 : "128" or "192" or "256".

In main, the program prompts the user for a key (of 16/24/32 characters depending of the 128/192/256 variant). It's
formated as a string. I thought about using hexadecimal values, and I wrote some versions of this program using
them, but as it was less practical for the user and just useful for debugging, I chose to use strings instead.
If the user doesn't enter any key, a default key, stored with 0s, will be used.

One of the key schedule function is now called : keyschedule128, keyschedule192, or keyschedule 256, depending of
the variant you decided to use. These functions take the master key, store it into an array, proceed to byte-wise
manipulations (expand it, basically), and store the results (all the round keys) into some output arrays.
I chose to make these 3 differents functions because of the non-linear characteristics of the 192 version, and the
slight differences of the 256 version. It was also clearer (at least, to me) to keep them into 3 differents functions
than make little functions for subbytes/shiftrows/mixcolumn/addkey and call them everytime, passing arguments around.

Keyschedule128 generates 11 subkeys for a 128bits master key. To generate a key, it takes the last word of the last
key, shifts its bytes to the left, make a subbyte step on them, and then xor the first byte of the word with a
constant RCON. Then, to generate the other words of the key, it xor the previous word with the word at the same place
but in the previous round. (And I sure hope I'm clear enough but without a schema I wouldn't bet on it. Sorry !)

Keyschedule192 generates 13 subkeys for a 192bits master key. It follows the same steps than Keyschedule128, except
it's not linear anymore : it creates 6 words of 4 bytes for keys of 4 by 4. For this reason, there is only 8 rounds.
Then, I stored the words into the keys by slicing them by four (ex : word 0: 0 to 5, word 1 : 6 to 11 is stored :
key0 : 0 to 3, key 1 : 4 to 7, key 2 : 8 to 11) We actually only need 52 words, but we create 54.

Keyschedule256 generates 15 subkeys for a 256bits master key. It follows the same steps than Keyschedule128, except
it's not linear either (one round generates 8 words, so 2 keys) and, there is a additional operation added in the
middle of a round, between word 3 and 4. For word 4, we use a subbyte version of word 3. We need 7 rounds for all
the keys (we need 60 words, but creates 64).

Back into main, the user is prompted to choose an input file, and then to choose a name for an output file.
The file's content is dynamically stored into 16 bytes blocks, which are padded with blank spaces if needed.
The blocks are made of four words of four bytes each.
You don't really need to choose a format for the output file if you're encrypting, since the file won't be readable
anyway, but if you are decrypting a file, to be able to read the file, you need to precise the format, the same one
as your input file (the plaintext before encryption). For remembering the format of the plaintext, it might be useful
to put the ciphertext file in the same format. Of course, it also gives information if someone want to try to
decrypt it, so it's up to you.
The content of the file is then stored into arrays, which are passed into one of the two functions encrypt or
decrypt.

The encrypt or decrypt functions dynamically adapt the number of rounds depending of the variant. It uses 10 rounds
for AES 128, 12 for AES 192, and 16 for AES 256.
It encrypts or decrypts the ouput with an encryption/decryption routine :

for each blocks :

add round key 0 with xor operations with the input
then for rounds 1 to - 10/12/14 :
subbytes : changes actual bytes values with theirs corresponding values in the SBOX (encrypting) or inverse
           SBOX(decrypting). I chose to use a look up table for efficiency, since it's not top secrets inputs here,
           but it's actually safer to implement directly into a GF(2power8) so it won't be vulnerable to cache attacks.
shiftrows : shifts bytes with a pattern : for encryption : no change for line 0, 1 byte to the left for line 1,
            2 to the left for line 2, 3 to the left to line 3. And same but to the right for decryption.
mixcolumn : For MixColumn, I copy the array into a temporary one, en then I multiply a word with a constant matrix,
            and xor the results. Again, for efficiency, I used look up tables. There are differents matrixes for
            encryption and decryption.
addkey : add round key with xor operations

 and then store the results in arrays for each blocks.

Back into main, the output arrays are written into the output file. For the ciphertext, it's not going to be readable
(and I didn't see the point to encode it in base64, since the whole point is to make informations hidden) but I let
the choice of printing the hexadecimal values on screen.
