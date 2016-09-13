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
using System;
using System.ComponentModel;
using System.Diagnostics;
using System.Linq;
using System.Runtime.InteropServices;

namespace TlsExample
{
    unsafe class Program
    {
        [Flags]
        public enum ProcessAccessFlags : uint
        {
            All = 0x001F0FFF
        }

        [Flags]
        public enum ThreadAccessFlags : uint
        {
            All = 0x1F03FF
        }

        public enum ThreadInfoClass : int
        {
            ThreadBasicInformation = 0
        }

        [StructLayout(LayoutKind.Sequential, Pack = 1, Size = 48)]
        public class ThreadBasicInformation
        {
            public uint ExitStatus;
            public uint _padding;
            public ulong TebBaseAddress;
            public ulong ProcessID;
            public ulong ThreadId;
            public ulong AffinityMask;
            public uint Priority;
            public uint BasePriority;
        }

        [DllImport("kernel32.dll", EntryPoint = "OpenProcess", SetLastError = true)]
        private static extern IntPtr UnmanagedOpenProcess(
            ProcessAccessFlags dwDesiredAccess,
            bool bInheritHandle,
            uint dwProcessId
        );

        [DllImport("kernel32.dll", EntryPoint = "OpenThread", SetLastError = true)]
        private static extern IntPtr UnmanagedOpenThread(
            ThreadAccessFlags dwDesiredAccess,
            bool bInheritHandle,
            uint dwThreadId
        );

        [DllImport("kernel32.dll", EntryPoint = "CloseHandle", SetLastError = true)]
        [return: MarshalAs(UnmanagedType.Bool)]
        private static extern bool UnmanagedCloseHandle(IntPtr hObject);

        [DllImport("kernel32.dll", EntryPoint = "ReadProcessMemory", SetLastError = true)]
        private static extern bool UnmanagedReadProcessMemory(
            IntPtr hProcess,
            UIntPtr lpBaseAddress,
            [Out] byte[] lpBuffer,
            int dwSize,
            out int lpNumberOfBytesRead
        );

        [DllImport("ntdll.dll", EntryPoint = "NtQueryInformationThread", SetLastError = true)]
        private static extern int UnmanagedNtQueryInformationThread(
            IntPtr threadHandle,
            ThreadInfoClass threadInformationClass,
            IntPtr threadInformation,
            int threadInformationLength,
            IntPtr returnLengthPtr
        );

        public static IntPtr OpenProcess(ProcessAccessFlags access, bool inheritHandle, uint processId)
        {
            IntPtr handle = UnmanagedOpenProcess(access, inheritHandle, processId);
            if (handle == null)
            {
                throw new Win32Exception();
            }
            return handle;
        }

        public static IntPtr OpenThread(ThreadAccessFlags access, bool inheritHandle, uint threadId)
        {
            IntPtr handle = UnmanagedOpenThread(access, inheritHandle, threadId);
            if (handle == null)
            {
                throw new Win32Exception();
            }
            return handle;
        }

        public static void CloseHandle(IntPtr handle)
        {
            if (!UnmanagedCloseHandle(handle))
            {
                throw new Win32Exception();
            }
        }

        public static void ReadProcessMemory(IntPtr hProcess, UIntPtr lpBaseAddress, [Out] byte[] lpBuffer, int dwSize, out int lpNumberOfBytesRead)
        {
            if (!UnmanagedReadProcessMemory(hProcess, lpBaseAddress, lpBuffer, dwSize, out lpNumberOfBytesRead))
            {
                throw new Win32Exception();
            }
        }

        public static ThreadBasicInformation GetThreadInformation(IntPtr threadHandle)
        {
            ThreadBasicInformation info = new ThreadBasicInformation();
            int size = Marshal.SizeOf(typeof(ThreadBasicInformation));
            var buf = Marshal.AllocHGlobal(size);

            if (UnmanagedNtQueryInformationThread(threadHandle, ThreadInfoClass.ThreadBasicInformation, buf, size, IntPtr.Zero) != 0)
            {
                throw new Win32Exception();
            }

            Marshal.PtrToStructure(buf, info);
            Marshal.FreeHGlobal(buf);
            buf = IntPtr.Zero;

            return info;
        }

        static void Main(string[] args)
        {
            ThreadBasicInformation info = null;
            var process = Process.GetProcessesByName("halo5forge").First();

            // get the main thread's info (assuming it's the first one with a running status)
            foreach (ProcessThread thread in process.Threads)
            {
                if (thread.ThreadState != ThreadState.Running)
                    continue;

                var threadHandle = OpenThread(ThreadAccessFlags.All, false, (uint)thread.Id);
                info = GetThreadInformation(threadHandle);
                CloseHandle(threadHandle);

                break;
            }

            //using (ProcessMemoryStream ps = new ProcessMemoryStream(process))
            //{
            //    ulong tlsPtr = ps.ReadUInt64(GetMainHaloThreadInfo().TebBaseAddress + 0x58);
            //    ulong tls = ps.ReadUInt64(tlsPtr);
            //}
        }
    }
}
```