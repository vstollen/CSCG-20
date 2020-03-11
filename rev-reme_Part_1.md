# Writeup
In this challenge we were given three files: a `ReMe.dll`, `ReMe.deps.json` and `ReMe.runtimeconfig.json`.

We were also told, that the dll should be executed using `.NET Core Runtime 2.2 with windows`. So that was the first thing I tried. It responded with an usage hint, asking for a password. So I needed to find out the password.

So I loaded the dll into dnSpy, a .NET decompiler and looked for the Main function, where the first call was:
```C#
Program.InitialCheck(args);
```
So I looked into the `InitialCheck()` method, which basically consisted out of a bunch of debugger checks (ending the program, if one was detected) and one check for the first argument, which turns out, would also be the flag:
```C#
bool flag5 = args[0] != StringEncryption.Decrypt("D/T9XRgUcKDjgXEldEzeEsVjIcqUTl7047pPaw7DZ9I=");
if (flag5)
{
	Console.WriteLine("Nope");
	Environment.Exit(-1);
}
else
{
	Console.WriteLine("There you go. Thats the first of the two flags! CSCG{{{0}}}", args[0]);
}
```
So I needed to find out what `StringEncryption.Decrypt("D/T9XRgUcKDjgXEldEzeEsVjIcqUTl7047pPaw7DZ9I=");` evaluates to and looked into it. The decompiled function looks like this:
```C#
public static string Decrypt(string cipherText)
{
    string password = "A_Wise_Man_Once_Told_Me_Obfuscation_Is_Useless_Anyway";
    cipherText = cipherText.Replace(" ", "+");
    byte[] array = Convert.FromBase64String(cipherText);
    using (Aes aes = Aes.Create())
    {
        Rfc2898DeriveBytes rfc2898DeriveBytes = new Rfc2898DeriveBytes(password, new byte[]
        {
            73,
            118,
            97,
            110,
            32,
            77,
            101,
            100,
            118,
            101,
            100,
            101,
            118
        });
        aes.Key = rfc2898DeriveBytes.GetBytes(32);
        aes.IV = rfc2898DeriveBytes.GetBytes(16);
        using (MemoryStream memoryStream = new MemoryStream())
        {
            using (CryptoStream cryptoStream = new CryptoStream(memoryStream, aes.CreateDecryptor(), CryptoStreamMode.Write))
            {
                cryptoStream.Write(array, 0, array.Length);
                cryptoStream.Close();
            }
            cipherText = Encoding.Unicode.GetString(memoryStream.ToArray());
        }
    }
    return cipherText;
}
```
Luckily it only uses Functions from the standard library, so I just copied it into a C# program and executed it with with the encrypted flag, resulting in the output: `CanIHazFlag?`. This was the password and also the flag, so I validated it, by calling the original program again, which then outputed the "official" flag:
```
There you go. Thats the first of the two flags! CSCG{CanIHazFlag?}
```