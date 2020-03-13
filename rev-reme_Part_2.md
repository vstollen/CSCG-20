# Writeup
The program given in `reme Part 1` contained two flags, one for every part. So we again had our three files: a `ReMe.dll`, `ReMe.deps.json` and `ReMe.runtimeconfig.json`.

From the first challenge, we already know the usage of the program `Reme.dll`, which took two parameters: a password and the flag, and the password itself: `CanIHazFlag?`.
So next I had to find out, what the rest of the program was doing and find the flag. So again, I decompiled the dll using dnSpy and looked at the Main function, which looked like this:

```C#
private static void Main(string[] args)
{
    Program.InitialCheck(args);
    byte[] ilasByteArray = typeof(Program).GetMethod("InitialCheck", BindingFlags.Static | BindingFlags.NonPublic).GetMethodBody().GetILAsByteArray();
    byte[] array = File.ReadAllBytes(Assembly.GetExecutingAssembly().Location);
    int[] array2 = array.Locate(Encoding.ASCII.GetBytes("THIS_IS_CSCG_NOT_A_MALWARE!"));
    MemoryStream memoryStream = new MemoryStream(array);
    memoryStream.Seek((long)(array2[0] + Encoding.ASCII.GetBytes("THIS_IS_CSCG_NOT_A_MALWARE!").Length), SeekOrigin.Begin);
    byte[] array3 = new byte[memoryStream.Length - memoryStream.Position];
    memoryStream.Read(array3, 0, array3.Length);
    byte[] rawAssembly = Program.AES_Decrypt(array3, ilasByteArray);
    object obj = Assembly.Load(rawAssembly).GetTypes()[0].GetMethod("Check", BindingFlags.Static | BindingFlags.Public).Invoke(null, new object[]
    {
        args
    });
}
```

So lets break it down, line by line. After the inital debugger and password checks from part 1, it reads in the method `InitialCheck` and saves the MSIL representation of the method in a `byte[]` array:

```C#
byte[] ilasByteArray = typeof(Program).GetMethod("InitialCheck", BindingFlags.Static | BindingFlags.NonPublic).GetMethodBody().GetILAsByteArray();
```

Then it reads in its dll and saves it in another `byte[]` array.

```C#
byte[] array = File.ReadAllBytes(Assembly.GetExecutingAssembly().Location);
```

In the next few lines, it locates the first occurrence of the string `"THIS_IS_CSCG_NOT_A_MALWARE!"` and saves the part of the file, following this string into an array called `array3`.

```C#
int[] array2 = array.Locate(Encoding.ASCII.GetBytes("THIS_IS_CSCG_NOT_A_MALWARE!"));
MemoryStream memoryStream = new MemoryStream(array);
memoryStream.Seek((long)(array2[0] + Encoding.ASCII.GetBytes("THIS_IS_CSCG_NOT_A_MALWARE!").Length), SeekOrigin.Begin);
byte[] array3 = new byte[memoryStream.Length - memoryStream.Position];
memoryStream.Read(array3, 0, array3.Length);
```

In the last step, it decrypts array3 using the saved MSIL representation of the `InitialCheck` method as password and then executes the `Check` method of the now decrypted executable, giving it the command line arguments as parameters.

```C#
byte[] rawAssembly = Program.AES_Decrypt(array3, ilasByteArray);
object obj = Assembly.Load(rawAssembly).GetTypes()[0].GetMethod("Check", BindingFlags.Static | BindingFlags.Public).Invoke(null, new object[]
{
    args
});
```

So to get the password I somehow had to get the decrypted executable and reverse that. Using a debugger was not an option, because of the debugger checks, which also couldn't be removed by patching the dll, because the `InitialCheck` function was used as the password for decryption. So I tried to patch the dll, that it saves the decrypted part into a file, by adding this line after the decryption:

```C#
byte[] rawAssembly = Program.AES_Decrypt(array3, ilasByteArray);
File.WriteAllBytes("checker.dll", rawAssembly);
```

This didn't work, because after recompilation dnSpy cut out the encrypted part of the dll, which could be seen by looking at it with a hex editor.
So I made a copy of the original untouched dll and loaded that, instead of the patched dll, which I also used to get the MSIL representation of the `InitialCheck` method.
I did this by manually typing in the Path to the dll and getting the `InitialCheck` method out of it.

```C#
byte[] array4 = File.ReadAllBytes("PATH_TO_ORIGINAL_DLL");
Console.WriteLine(typeof(Program).FullName);
byte[] ilasByteArray = Assembly.Load(array4).GetType("ReMe.Program", true, true).GetMethod("InitialCheck", BindingFlags.Static | BindingFlags.NonPublic).GetMethodBody().GetILAsByteArray();
```

When executing this, it finally worked and I got myself the decrypted program, saved in the `checker.dll` file, which I could then load into dnSpy to decompile it.
Here is my decompiled version of the `Check` method:

```C#
public static void Check(string[] args)
{
    bool flag = args.Length <= 1;
    if (flag)
    {
        Console.WriteLine("Nope.");
    }
    else
    {
        string[] array = args[1].Split(new string[]
        {
            "_"
        }, StringSplitOptions.RemoveEmptyEntries);
        bool flag2 = array.Length != 8;
        if (flag2)
        {
            Console.WriteLine("Nope.");
        }
        else
        {
            bool flag3 = "CSCG{" + array[0] == "CSCG{n0w" && array[1] == "u" && array[2] == "know" && array[3] == "st4t1c" && array[4] == "and" && Inner.CalculateMD5Hash(array[5]).ToLower() == "b72f3bd391ba731a35708bfd8cd8a68f" && array[6] == "dotNet" && array[7] + "}" == "R3333}";
            if (flag3)
            {
                Console.WriteLine("Good job :)");
            }
        }
    }
}
```

And there we had it. The `Check` method splits the second argument between the `"_"` characters and then checks the resulting parts. Everything except one word is hardcoded, giving us `n0w_u_know_st4t1c_and_???_dotNet_R3333` as the flag, with only the `???` missing. The missing part is checked by its MD5 Hash, so I took the hash and loaded it into an online MD5 database (https://www.md5online.org/md5-decrypt.html), which quickly fount `dynamic` as the missing word. So there we had our flag: `CSCG{n0w_u_know_st4t1c_and_dynamic_dotNet_R3333}`.