# lAZY aDMIN Writeup

I started this challenge which prompted me to find a flag "to get some booze"....? okay? I was unsure of this being related to the solution or a random red-herring as usual.
The challenge provided a JSON object containing a `hidden_key` string that looked like garbage data and an image file.

The first thing that stood out to me was the filename of the image `KGkqMiktMjgK.jpg`. It did not look like a random hash or a standard descriptive name. It looked exactly like a Base64 encoded string so I decided to decode it immediately to see if it held any clues.

I ran the following command to decode it:
`echo "KGkqMiktMjgK" | base64 -d`

The output was `(*2)-28`.

This output suggested a mathematical operation. Since the challenge title refers to a "Lazy Admin" I assumed this was a very simple substitution cipher used to generate the hidden key.

If the encryption logic was `(plaintext * 2) - 28` then I simply needed to reverse the math to decrypt it. The decryption logic would therefore be `(ciphertext + 28) / 2`.

I did briefly check the image metadata and found a Base64 string in the EXIF data that decoded to "Minecraft hides a deep message within in" but the filename seemed like the more direct path to the solution.

I wrote a quick Python script to take the weird string from the JSON and apply the reverse formula to the ASCII value of each character.

```python
ciphertext = "vljÚv¢Ò¦ÀÌ¢¢¢¢ÊÂ¾®¢¨ÂÂØ®¢¬JL¬¨JL°Þ"
flag = ""

for char in ciphertext:
    # Get ascii value
    val = ord(char)
    # Apply the reverse formula: (Cipher + 28) / 2
    decoded_val = int((val + 28) / 2)
    flag += chr(decoded_val)

print(flag)
```
and the flag was captured indeed! still don't have the booze though
