# Automatically Extracting PE File Information & Disabling ASLR

![Programmer](https://raw.githubusercontent.com/BarracudaByte/blog/refs/heads/main/images/extracting-pe-info.webp?raw=true)

When analysing malware or reverse engineering PE files, it is useful to get some general information about the file before diving further into it. 

There are some fantastic tools like *PE Info* or *PE-Bear* that do exactly that. However, when automating flows, it is useful getting that information from a script so it can then easier be passed on to further steps in the automation process. 

In this blog post we will take a look how to write a custom script to extract general information from a PE file. 

### Prerequesites

To follow along, you will need the following:
- Python 3.9+ 
- `pip` 
- VSCode or other IDE

Once you have confirmed you have `Python` and `pip` installed, run the following commands to create a virtual environment and install the pip package to parse PE files `pefile`. 

```bash
python -m venv .venv
# activate your virutal environment, on bash/zsh:
source .venv/bin/activate
pip install pefile
```

> If you are not sure how to activate your virtual environment, check out the [official python documentation](https://docs.python.org/3/library/venv.html#how-venvs-work) for venv.

### End Result

Goal of this blog is, to have in the end a script, that can parse an executable, and return the following information:

- image base address
- address of entry point
- ASLR (address space layout randomization) enabled

Additionally, we want to be able to disable ASLR to make any further analysis easier.

## Parsing PE Files Automatically

Before we begin writing any code, we need a PE file we can test our script on. As an example for the post, we will be using the `func_pointer.exe` from *HackTheBox* sherlock challenge [Payload](https://app.hackthebox.com/sherlocks/Payload).
To follow along you can use the same file, or any other PE file you have.

### Loading the executable

Loading the executable with pefile straightforward, we just have to pass the file path to the `pefile.PE()` function. In this case we are assuming the executable is in the same directory as the python script.
```python
import pefile

pe = pefile.PE('func_pointer.exe')
```

### Understand the PE File Structure

To automatically extract information from PE files, we need to understand the general structure of it.

PE files have four file headers:

1. MS-DOS Stub
2. Signature
3. COFF File Header
4. Optional Header

![PE File Headers](https://raw.githubusercontent.com/BarracudaByte/blog/refs/heads/main/images/pe_file_headers.webp?raw=true)

You can read more about the file headers in the [official Microsoft documentation](https://learn.microsoft.com/en-us/windows/win32/debug/pe-format#file-headers).

All the information we are looking for is in the optional header.

### Extracting Entry Point & Image Base Address

Luckily for us, the entry point and image base address are simply fields in the optional header. We can access the optional header with `pe.OPTIONAL_HEADER` and then we can access the `AddressOfEntryPoint` and the `ImageBase`. The only thing we need to watch out for is, that these fields return integers, in most cases we likely want hex instead though. We can use the `hex()` function to convert the integrers to hexadecimal. 

```python
print(f'Address of Entry Point: {hex(pe.OPTIONAL_HEADER.AddressOfEntryPoint)}')
print(f'Address of Image Base: {hex(pe.OPTIONAL_HEADER.ImageBase)}')
```

### Check if ASLR is enabled

The information is ASLR is enabled is also in the `OPTIONAL_HEADER` and it is stored as part of the [DLL Characteristics (Microsoft Documentation)](https://learn.microsoft.com/en-us/windows/win32/debug/pe-format#dll-characteristics). The constant `IMAGE_DLLCHARACTERISTICS_DYNAMIC_BASE` defines if the file can be relcated at load time, if yes then ASLR is enabled.

To see all the available fields in the `pe.OPTIONAL_HEADER` we can use the `dir()` function.

```python
>>> dir(pe.OPTIONAL_HEADER)
# Part of the output:
['AddressOfEntryPoint', 'BaseOfCode', 'CheckSum', 'DATA_DIRECTORY', 'DllCharacteristics', 'FileAlignment', 'IMAGE_DLLCHARACTERISTICS_APPCONTAINER', 'IMAGE_DLLCHARACTERISTICS_DYNAMIC_BASE', 'IMAGE_DLLCHARACTERISTICS_FORCE_INTEGRITY', 'IMAGE_DLLCHARACTERISTICS_GUARD_CF', 'IMAGE_DLLCHARACTERISTICS_HIGH_ENTROPY_VA', 'IMAGE_DLLCHARACTERISTICS_NO_BIND', 'IMAGE_DLLCHARACTERISTICS_NO_ISOLATION', 'IMAGE_DLLCHARACTERISTICS_NO_SEH', 'IMAGE_DLLCHARACTERISTICS_NX_COMPAT', 'IMAGE_DLLCHARACTERISTICS_TERMINAL_SERVER_AWARE', 'IMAGE_DLLCHARACTERISTICS_WDM_DRIVER', 'ImageBase', ...]
```

From the command output be we can see that we are lucky and there is a field called `IMAGE_DLLCHARACTERISTICS_DYNAMIC_BASE`.

```python
print(f'ASLR Enabled? {pe.OPTIONAL_HEADER.IMAGE_DLLCHARACTERISTICS_DYNAMIC_BASE}')
```

### Disable ASLR

Now that we know how we can check if ASLR is enabled, we can also disable it! Before we do that we need to understand the DLL Characterstics field a little bit better. 

![DLL Characteristics](https://raw.githubusercontent.com/BarracudaByte/blog/refs/heads/main/images/dll_characteristics.webp?raw=true)

The DLL characteristics is basically just a number, where each bit is set to 0 or 1 indicating if that feature is on or off. 

The first 4 bits are reserved, the one after that sets if the image can handle a high entropy 64-bit virtual address space and the bit after that defines if the DLL can relocate. So when ASLR is enabled that bit is set and to disable it we just need to unset it. 

To set or unset a bit we can use simple bit-wise `&` (AND) or `|` (OR) operators combined with the value of the `IIMAGE_DLLCHARACTERISTICS_DYNAMIC_BASE` which corresponds to `0x40`.

```python
# disable ASLR
pe.OPTIONAL_HEADER.DllCharacteristics &= ~pefile.DLL_CHARACTERISTICS['IMAGE_DLLCHARACTERISTICS_DYNAMIC_BASE']

# enable ASLR
pe.OPTIONAL_HEADER.DllCharacteristics |= pefile.DLL_CHARACTERISTICS['IMAGE_DLLCHARACTERISTICS_DYNAMIC_BASE']
```

If you query the status of the ASLR again right after making the change it will seem like it didn't work. To apply the changes we have to write the changes back to disk.

```python
# writes a new file to disk, with the changes that were made
pe.write('new_file_with_ASLR_disabled.exe')
```

## Conclusion

As we have seen, with Python it is easy and fast to quickly parse a PE file and get some general information out of it. We even were able to disable ASLR making further reversing of the file a lot easier.  

You can easily extend this script and add further functionality like using command-line arguments to pass the filename, so you can quickly run it before analysing files.