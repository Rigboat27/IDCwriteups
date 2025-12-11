The executable presents a 10,000 round math challenge where each round asks for i^2 under severe timing constraints. Since completing this legitimately is unrealistic, the approach is to locate the code path taken when the program considers the game cleared and follow what happens next. That path references two static data objects named perm.1 and enc_flag.0. perm.1 is a 21-byte permutation table describing where each output byte should be placed; enc_flag.0 is a 21-byte array containing the encrypted flag bytes. Exporting these two blobs from the binary allows reconstructing the plaintext without executing the interactive gauntlet.

The decryptor initializes a 32-bit state to 0xC0FFEE12, then repeatedly transforms the state using shifts, XORs and a multiply/XOR mix. Each iteration yields one keystream byte from the low eight bits of the transformed state. That keystream byte is XORed with the corresponding encrypted byte, and the result is written into the output buffer at the index specified by the permutation table. After 21 iterations the output buffer contains the recovered flag.

dumping perm.1 and enc_flag.0 from the binary (21 bytes each) and running this custom script,

```python

p = open("perm.bin","rb").read()
e = open("enc_flag.bin","rb").read()

def f(s):
 x = (s ^ ((s << 7) & 0xffffffff)) & 0xffffffff
 x ^= (x >> 5)
 return ((x * 8) ^ x) & 0xffffffff

s = 0xc0ffee12
o = [0]*21
for i in range(21):
 s = f(s)
 o[p[i]] = e[i] ^ (s & 0xff)

print(bytes(o))

```
yields the flag.
The encryption is simply a process that involves XORing each encrypted byte with a state-derived keystream byte and stores results according to a small permutation table.
