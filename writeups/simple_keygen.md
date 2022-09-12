# Simple Keygen

Link: https://crackmes.one/crackme/5c2acb8933c5d46a3882b8d4

|           |                   |
|-----------|-------------------|
|Author     |               Yuri|
|Language   |              C/C++|
|Upload     | 2:08 AM 01/01/2019|
|Platform   |    Unix/linux etc.|
|Difficulty |                1.9|
|Quality    |                5.0|
|Arch       |             x86-64|

## Recon
After extracting the **Keygen.tar.gz** with the `tar -xvf Keygen.tar.gz` connad we got a binary called `SimpleKeyGen`.  
With the `fime SimpleKeyGen` command we see that thisis a `ELF 64-bit` binary.  

Let's open it with [rz-cutter](https://archlinux.org/packages/community/x86_64/rz-cutter/) (I'm using the ghidra standalone dubugger).
In the `main()` function we see that there's a call for `checkSerial()` function what checks the value of `argv[1]`.  
```c
iVar1 = checkSerial(argv[1]);
if (iVar1 == 0) {
    puts("Good Serial");
    uVar2 = 0;
} else {
    puts("Bad Serial");
    uVar2 = 0xffffffff;
}
```

### Reversing checkSerial()
For the next time it's very confusing, so the next step is the deobfuscating.  
```c
undefined8 checkSerial(char *arg1)
{
    int64_t iVar1;
    undefined8 uVar2;
    uint64_t uVar3;
    char *s;
    int64_t var_14h;
    
    iVar1 = strlen(arg1);
    if (iVar1 == 0x10) {
        for (var_14h._0_4_ = 0; uVar3 = strlen(arg1), (uint64_t)(int64_t)(int32_t)var_14h < uVar3;
            var_14h._0_4_ = (int32_t)var_14h + 2) {
            if ((int32_t)arg1[(int32_t)var_14h] - (int32_t)arg1[(int64_t)(int32_t)var_14h + 1] != -1) {
                return 0xffffffff;
            }
        }
        uVar2 = 0;
    } else {
        uVar2 = 0xffffffff;
    }
    return uVar2;
}
```
After the manual deobfuscating it will look like this:  
```c
int checkSerial(char *input) {
    int input_len;
    int ret;
    int i;

    input_len = strlen(input);
    // 16 0x10
    if (input_len == 16) {
        for (i = 0; i < input_len; i = i + 2) {
            if ( input[i] - input[i + 1] != -1) {
                // fail
                return 0xffffffff;
            }
        }
        // win
        ret = 0;
    } else {
        // fail
        ret = 0xffffffff;
    }
    return ret;
}
```
We got many information now.  
The `input_len` variable must be 16, the variable **i** in increased by 2 every itteration and the most important codeblock:
```c
 if ( input[i] - input[i + 1] != -1) {
    // fail
    return 0xffffffff;
}
```

## Solution
We know that `input[i] - input[i + 1]` must be **-1** so every 2nd int must be greater by 1 than the previous one.  
Here's a working example:
```
index 0 1
value 2 3
```
We could do it 8 times like
`2323232323232323`

### Personal solve time
2022/09/11  
0930 - 0950