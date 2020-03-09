# Writeup
The given zip again contained two files. A text file named `flag` and an executable named `rev2`.

When executed rev2 asks for a password, just like rev1, but this time the output of the `strings` command wasn't helpful, so I disassembled it using ghidra.

After some variable renaming and cleanup the disassembled code looked like this:
```
initialize_flag();
puts("Give me your password: ");
input_size = read(0,inputBuffer,31);
inputBuffer[(int)input_size + -1] = '\0';
i = 0;
while (i < (int)input_size + -1) {
  inputBuffer[i] = inputBuffer[i] + -119;
  i = i + 1;
}
passwordCorrect = strcmp(inputBuffer,&PASSWORD_OBFUSCATED);
if (passwordCorrect == 0) {
  puts("Thats the right password!");
  printf("Flag: %s",flagBuffer);
}
else {
  puts("Thats not the password!");
}
```
In this code snipped you can see, that it reads the password, modifies it in the while-loop and does a string-compare with a predefined string in the end. So to get the password we just have to reverse what happens in the while-loop and reconstruct the password out of the predefined string, lying at `&PASSWORD_OBFUSCATED`.

Looking at the while-loop, it just goes through the input and subtracts `119` from every character, so I just added `119` onto the string at `&PASSWORD_OBFUSCATED` and took mod 256 on it, to encorporate possible overflows, giving me the password `sta71c_tr4n5f0rm4710n_it_isw`.

## Security Threads
The problem, leading to the flag was, that it was possible to reverse the operations done to get the obfuscated password.

## Possible Fix
A possible fix, to prevent attackers to extract the password would be to use one-way operations or functions, which aren't as easy to reverse. A good example would be to use a hash algorithm.
