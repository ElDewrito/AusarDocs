# Memory Storage Locations

## Untracked Allocations

## Thread Local Storage

The game engine uses [Thread Local Storage (TLS)](https://en.wikipedia.org/wiki/Thread-local_storage) for global thread-specific dynamically allocated objects.

A pointer to the TLS is located in the [Thread Environment Block (TEB)](http://shitwefoundout.com/wiki/Win32_Thread_Environment_Block) which can be obtained in a 64-bit environment via the `GS` segment selector at offset `0x58`. The game engine only utilizes TLS slot 0 currently, which contains an array of pointers to individual sub-allocations.

The TLS address can be obtained in a local execution context via `__readgsqword` or remotely via `NtQueryInformationThread`.

**Local Example (C++)**

```
auto tlsPtr = *(uint64_t**)__readgsqword(0x58);
```

**Remote Example (C#)**

```
// TODO: NtQueryInformationThread implementation
```