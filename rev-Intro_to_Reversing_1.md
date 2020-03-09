# Writeup
The given zip contained two files. A `flag` file and a `rev1` file.
When looking at theses using the `file` command you can find out, that `flag` is plain ASCII text and `rev1` is an executable.

Looking into the `flag` File we can see, that it is just a dummy flag, which is probably used by the `rev1` executable.

So I executed `rev1` to see what would happen. It asks for a password. As stated in the linked guide, I tried to get more information about `rev1` by using the `strings` command on it. It printed out like this:

```
[...]
Give me your password: 
y0u_5h3ll_p455
Thats the right password!
Flag: %s
Thats not the password!
./flag
[...]
```
This strengthened my suspicion, that the program would just print `./flag`, when given the right password. Also `y0u_5h3ll_p455` looked like it could be a password, so I executed `rev1` again, typed it in and it worked.

To get the flag I then just had to use the given command and type in the password `y0u_5h3ll_p455`, connecting me to the remote server, which was probably running our executable:
```
hax1.allesctf.net 9600
```
## Security Threads
The problem, which led to the flag was, that the password was saved as a plain String, so it could be easily readout of the executable.
## Possible fix
To fix this issue, you could try to obscure the password by encrypting or hashing it. This still wouldn`t provide full protection, since it could probably still be reverse engineered, but it would prevent "simply reading it out" using `strings`.
