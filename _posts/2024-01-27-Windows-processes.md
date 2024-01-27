---
layout: post
title:  "Windows Processes and their components"
---
# Exploring Windows Processes


## Defining a Windows Process

A Windows process is essentially a live representation of a program or application on a Windows system. These processes can be initiated by users or automatically by the system. They utilize system resources like memory, disk space, and processor time to accomplish their tasks.

### Threads in Windows Processes

In the realm of Windows processes, threads play a vital role. Each process comprises one or more threads, running concurrently. A thread is an independent sequence of executable instructions within its parent process. These threads have the capability to communicate and share data with each other. The operating system is responsible for their scheduling and management, ensuring they operate within the process's context.

### Memory Management in Processes

Processes in Windows are memory-dependent for storing data and instructions. This memory is allocated at the creation of the process, with the process itself dictating the allocation size. Windows manages this memory using a combination of virtual and physical spaces. The concept of virtual memory allows the OS to simulate more memory than physically available, creating a virtual address space for applications. This virtual space is segmented into "pages," which are assigned to processes as needed.

#### Classifications of Process Memory

1. **Private Memory**: This memory is exclusive to a process, storing data specific to it, and is not accessible by other processes.
2. **Mapped Memory**: This type facilitates data sharing between processes, useful for shared libraries and files. While visible to other processes, it remains protected against unauthorized modifications.
3. **Image Memory**: It encompasses the executable file's code and data, often linked to DLL files within the process's address space.

## The Process Environment Block (PEB)

The Process Environment Block (PEB) in Windows is a critical data structure. It encompasses diverse information about a process, including parameters, startup details, heap allocations, and loaded DLLs. Unique to each process, the PEB plays a key role in how the operating system and Windows loader handle process execution and management.

### PEB Structure

Here's a look at the PEB structure in C:

```c
typedef struct _PEB {
  BYTE Reserved1[2];
  BYTE BeingDebugged;
  BYTE Reserved2[1];
  PVOID Reserved3[2];
  PPEB_LDR_DATA Ldr;
  PRTL_USER_PROCESS_PARAMETERS ProcessParameters;
  PVOID Reserved4[3];
  PVOID AtlThunkSListPtr;
  PVOID Reserved5;
  ULONG Reserved6;
  PVOID Reserved7;
  ULONG Reserved8;
  ULONG AtlThunkSListPtr32;
  PVOID Reserved9[45];
  BYTE Reserved10[96];
  PPS_POST_PROCESS_INIT_ROUTINE PostProcessInitRoutine;
  BYTE Reserved11[128];
  PVOID Reserved12[1];
  ULONG SessionId;
} PEB, *PPEB;

```

### Key Components of the PEB

    BeingDebugged: A flag indicating if the process is under debugging.
    Ldr: A pointer to PEB_LDR_DATA, detailing loaded DLL modules in the process.
    ProcessParameters: Contains command line parameters passed during process creation.
    AtlThunkSListPtr & AtlThunkSListPtr32: Used by the ATL module for managing thunking functions, crucial for calling functions in a different address space.
    PostProcessInitRoutine: A pointer for a function called post-TLS initialization for all threads in the process.
    SessionId: A unique identifier for a user session.

## The Thread Environment Block (TEB)

Another critical structure, the Thread Environment Block (TEB), houses information about a thread in Windows, including environment, security context, and more.

### The TEB Structure

```c
typedef struct _TEB {
  PVOID Reserved1[12];
  PPEB ProcessEnvironmentBlock;
  PVOID Reserved2[399];
  BYTE Reserved3[1952];
  PVOID TlsSlots[64];
  BYTE Reserved4[8];
  PVOID Reserved5[26];
  PVOID ReservedForOle;
  PVOID Reserved6[4];
  PVOID TlsExpansionSlots;
} TEB, *PTEB;
```

### Notable TEB Elements

    ProcessEnvironmentBlock (PEB): A pointer to the process's PEB.
    TlsSlots: Designated for storing thread-specific data.
    TlsExpansionSlots: Additional slots for thread-local storage, primarily for system DLLs.



## Process and Thread Handles in Windows

Windows assigns unique identifiers to each process (PID) and thread. These identifiers are critical for differentiating and managing various processes and threads.
Windows API for Process and Thread Management

    OpenProcess: Retrieves a handle to a process using its PID.
    OpenThread: Retrieves a handle to a thread using its thread ID.

These APIs play a vital role in tasks like process suspension. Remember, it's essential to close handles with CloseHandle to prevent resource leaks.



## Undocumented Structures

When referencing the Windows documentation for a structure, one may encounter several reserved members within the structure. These reserved members are often presented as arrays of BYTE or PVOID data types. This practice is implemented by Microsoft to maintain confidentiality and prevent users from understanding the structure to avoid modifications to these reserved members.

However, we have to be able to work with these undocumented members. Therefore, you might be better of staying away from Microsoft's documentation and instead use other websites that have the full undocumented structure, which was likely derived through reverse engineering.

[ProcessHackers header files](https://github.com/winsiderss/systeminformer/tree/master/phnt/include)
[Vergilius Project is a fantastic source for kernel structures](https://www.vergiliusproject.com/)


### Finding Reserved Members

One way to determine what the PEB's reserved members hold is through the `!peb` command in WinDbg.