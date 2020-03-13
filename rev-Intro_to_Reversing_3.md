# Writeup
Like in "Intro to Reversing 1" and "Intro to Reversing 2",  we were given two files: A text file named `flag` and an executable named `rev3`, which again asks the user for a password.

Once again I recompiled `rev3` using ghidra, to get some insights on how the password is checked and just as in "Intro to Reversing 2", it reads in the password, modifies it character by character, using a while-loop and compares it to another constant, predefined string. Here is the loop after some cleanup:

```
i = 0;
while (i < (int)input_length + -1) {
  input_buffer[i] = input_buffer[i] ^ (char)i + 10U;
  input_buffer[i] = input_buffer[i] + -2;
  i = i + 1;
}
```
It iterates over every character and XORs it with the character position `+ 10`. So the first character is XORed with `0 + 10`, the second character with `1 + 10`, etc.
After that, it subtracts 2 from it. Both of these operations are reversible. XOR with XOR and the subtraction with an addition. To do this I wrote a short C Program:
```
#include <stdio.h>
#include <string.h>

int main() {
  char password_obfuscated[] = "lp`7a<qLw\x1ekHopt(f-f*,o}V\x0f\x15J";
  
  int i = 0;
  while (i < strlen(password_obfuscated)) {
    password_obfuscated[i] = password_obfuscated[i] + 2;
    password_obfuscated[i] = password_obfuscated[i] ^ i + 10;
    i++;
  }

  printf("%s\n", password_obfuscated);
  return 0;
}
```
Running it resulted in the password `dyn4m1c_k3y_gen3r4t10n_y34h` which again made the program print out the flag.

## Security Risk
Like in "Intro to Reversing 2" the obfuscation only used operations, which were easy to reverse, so the password could be easily extracted out of the predefined string.

## Possible Fix
Again it could be prevented by the use of one-way functions, like hash-functions, which are easy to calculate, but hard to reverse.