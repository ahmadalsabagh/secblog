---
title: Bypass Windows Defender's Signature Based Detection
date: "2021-01-02"
featuredImage: './defender.png'
---

A practical example of bypassing Windows Defender's signature detection by modifying source code<!-- end -->. I'll be using <a href="https://github.com/GhostPack/Rubeus">Rubeus</a> as my malicious binary and I suggest you clone it and follow along to better your learning experience.

# What is Signature Based Detection?

Before starting we need to understand what signature based detection is and how antivirus agents use it to catch malicious binaries. An antivirus signature is a continuous sequence of bytes within a malicious binary that uniquely identifies it. Files on your machine are scanned and compared against the a signature database and if there is a match then that file is quarantined and removed off the machine.

If you compile Rubeus and upload the binary to VirusTotal a hash of the file will be generated. This hash <i>can</i> be used as a signature to uniquely identify the file, although most antivirus agents will create custom signatures of the file by taking excerpts of the source code. The following image shows VirusTotal's output after uploading an unmodified Rubeus binary.

![Rubeus-vt](./vt-rubeus-hash.png)

What happens if we remove a single comment from the source code?

![Rubeus-vt-2](./vt-rubeus-hash-2.png)

You should notice 2 differences.

1.  The signature (hash) of the file changed
2.  The detection rate went down. This means some AV vendors only used the hash of the file for signature detection

# Reducing The Detection Rate

First thing's first, let's find and replace the term 'Rubeus' with a new word. I'm using Visual Studio so to do that use Ctrl+Shift+H:

![Find-and-replace](./find-and-replace-all.png)

You also want to do the following:
1.  Open up the .sln file with a text editor and replace all instances of Rubeus with your new word
2.  Modify the 'Rubeus' folder names of the project and 'Rubeus.csproj' to your new word
3.  Right click the project in Visual Studio and select 'Properties' then replace all instances of 'Rubeus' to your new word
4.  Compile and build as a x64 executable

![Rubeus-vt-3](./vt-rubeus-hash-3.png)
Detection rate dropped to 23

### Modifying Variables, Comments And Functions

We want to change the program as much as possible without changing the functionality. One thing you can do is find and replace all on variable names and function names. Look for malicious keywords like 'Bruteforcer' and replace that. Remove comments and modify text within print methods (e.g. Console.WriteLine()).

![Find-and-replace-2](./find-and-replace-all-2.png)

The problem with this method is we are arbitrarily replacing words without knowing whether it'll have an effect or not. In the next section we'll try to pinpoint the signature.

# Finding The Malicious Signature

So far it's been simple changes and Defender is still detecting our program. We need a better way to pinpoint the signature that's triggering Defender.

What we want to do next is split our binary by grabbing the first X bytes and slowly increasing the number of bytes until Defender detects the binary as a malicious file. My main binary file is 254,464 bytes, so I'll start off by getting the first 180,000 bytes and seeing if Defender triggers on it. Make sure you whitelist your working directory first.

    #Get the first 180,000 bytes from Mrdox.exe and save it to test.exe
    #Use either Powershell or Bash

    #Powershell
    Get-Content .\Mrdox.exe -Encoding Byte -TotalCount 180000 | Set-Content test.exe -Encoding byte

    #Bash
    head -c 180000 Mrdox.exe > test.exe

![No-detection](./no-detection.gif)

Defender doesn't trigger. Let's change that to 181,000 bytes instead:

    #Powershell
    Get-Content .\Mrdox.exe -Encoding Byte -TotalCount 181000 | Set-Content test.exe -Encoding byte

    #Bash
    head -c 181000 Mrdox.exe > test.exe


Still nothing... let's try 181,500 bytes:

    #Powershell
    Get-Content .\Mrdox.exe -Encoding Byte -TotalCount 181500 | Set-Content test.exe -Encoding byte

    #Bash
    head -c 181500 Mrdox.exe > test.exe


![Detection](./detection.gif)

Great! Now we know that Defender is triggering somewhere between 181,000 and 181,500 bytes. Let's open the last file with <a href="https://mh-nexus.de/en/hxd/">HxD</a> and check out the last 500 bytes.

![Hxd-Analysis](./hxd-analysis.png)

Defender's signature is somewhere in the highlighted part. The most common word we see is 'tickets' so let's do a find and replace on 'tickets' and replace it with a different word.

![Results](./find-and-replace-all-results.png)

Now when we scan the file Defender doesn't trigger! We successfully bypassed Defender and can now run Rubeus.

![Clean-Scan](./clean-scan.gif)

![Clean-File](./clean-file.png)

# References

https://www.youtube.com/watch?v=9pwMCHlNma4