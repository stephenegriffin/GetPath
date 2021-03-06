# GetPath
Command line test harness for **GetFinalPathNameByHandleW**

Pass in a file or directory name to test running [**GetFinalPathNameByHandleW**](https://msdn.microsoft.com/en-us/library/windows/desktop/aa364962(v=vs.85).aspx).

In order to run GetFinalPathNameByHandleW, we first need to open the file. We do this using [**CreateFileW**](https://msdn.microsoft.com/en-us/library/windows/desktop/aa363858(v=vs.85).aspx), like so:
```C+
auto hnd = CreateFileW(dirname.c_str(), flag,
   FILE_SHARE_READ | FILE_SHARE_WRITE | FILE_SHARE_DELETE, nullptr,
   OPEN_EXISTING, FILE_FLAG_BACKUP_SEMANTICS, nullptr);
```

There is some debate about the proper flag to pass. Is [GENERIC_READ](https://msdn.microsoft.com/en-us/library/windows/desktop/aa446632(v=vs.85).aspx) (0x80000000) required, or can you get by with 0? This code will allow you to test this, since it will try it both ways:
```dos
C:\src\GetPath\Debug>pushd \\machine\c$\tools

Z:\Tools>C:\src\getpath\Debug\GetPath.exe Z:\Tools
GetPath testing GetFinalPathNameByHandleW calls against Z:\tools

First test: don't open file first, just try to get the path
        GetPath: Testing access flags 0x00000000 with Z:\Tools
        GetPath: GetFinalPathNameByHandleW returned: \\?\C:\Tools
        GetPath: Testing access flags 0x80000000 with z:\Tools
        GetPath: GetFinalPathNameByHandleW returned: \\?\C:\Tools
...
```
Clearly for this path, either 0 or GENERIC_READ worked. This may not be true for all network devices.

The initial test is followed by a series of tests where we first open the file with varying levels of access before calling the GetPath routine, to see how having another handle open on the file affects our CreateFileW and GetFinalPathNameByHandleW calls.
