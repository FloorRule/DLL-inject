# MyDLL – Example Windows Dynamic-Link Library

This project demonstrates how to build a basic **Windows DLL in C/C++**, including:

* Exported and non-exported functions
* Use of `__declspec(dllexport)` and `__declspec(dllimport)`
* Handling DLL load/unload events using `DllMain`
* Providing a header file for consumers of the DLL

This repository contains two main files:

* **mydll.cpp** – Implementation of the DLL
* **mydll.h** – Public header for import/export macros

---

## Features

### ✔ Exported function (`Share`)

This function is marked with `__declspec(dllexport)` and can be called from any external program that loads the DLL:

```cpp
DECLDIR void Share();
```

When called, it prints:

```
I am an exported function, can be called outside the DLL
```

### ✔ Internal function (`Keep`)

This function is **not exported**, meaning it can only be used inside the DLL:

```cpp
void Keep();
```

It prints:

```
I am not exported, can be called only within the DLL
```

### ✔ DLL Entry Point (`DllMain`)

The DLL defines `DllMain`, which is executed automatically by Windows whenever:

* The DLL is loaded
* A process or thread attaches/detaches
* The DLL is unloaded

In `DLL_PROCESS_ATTACH`, both `Share()` and `Keep()` are called automatically.

---

## File Breakdown

### `mydll.h`

```cpp
#ifdef DLL_EXPORT
#define DECLDIR __declspec(dllexport)
#else
#define DECLDIR __declspec(dllimport)
#endif

extern "C"
{
    DECLDIR void Share();
    void Keep();
}
```

This header:

* Exports `Share()` when building the DLL
* Imports `Share()` when used by external programs
* Uses `extern "C"` to prevent C++ name mangling

### `mydll.cpp`

```cpp
DECLDIR void Share() { ... }
void Keep() { ... }

BOOL APIENTRY DllMain(HANDLE hModule, DWORD reason, LPVOID reserved)
{
    switch(reason)
    {
        case DLL_PROCESS_ATTACH:
            Share();
            Keep();
            break;
        ...
    }
}
```

This file includes:

* The implementation of `Share()` and `Keep()`
* The `DllMain` function that runs on DLL load/unload

---

## How to Build the DLL

### Using Visual Studio

1. Create a new **Win32 Project**
2. Choose **DLL**
3. Add `mydll.cpp` and `mydll.h`
4. Ensure **Preprocessor Definitions** contains:

   ```
   DLL_EXPORT
   ```
5. Build → **Build Solution**
6. Visual Studio will generate:

   * `mydll.dll`
   * `mydll.lib`

---

## How to Use the DLL from C++

### Example Usage Code

```cpp
#include <Windows.h>
#include <iostream>

typedef void (*ShareFunc)();

int main()
{
    HMODULE dll = LoadLibrary("mydll.dll");
    if (!dll)
    {
        std::cout << "Failed to load DLL\n";
        return 1;
    }

    ShareFunc Share = (ShareFunc)GetProcAddress(dll, "Share");
    if (Share)
        Share();

    FreeLibrary(dll);
    return 0;
}
```

---

## How to Use the DLL from C#

```csharp
using System;
using System.Runtime.InteropServices;

class Program
{
    [DllImport("mydll.dll")]
    public static extern void Share();

    static void Main()
    {
        Share();
    }
}
```

---

## Notes

* `Share()` is the only exported function
* `Keep()` cannot be used externally
* `DllMain` will execute automatically upon DLL load
* `extern "C"` ensures function names remain unmangled

---
