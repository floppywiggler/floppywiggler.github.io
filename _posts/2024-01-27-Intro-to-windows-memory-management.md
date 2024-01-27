---
published: false
---
# Windows Memory Management (MalDev2)

## Intro to Windows Memory Management

This post dives a little bit into the essentials of Windows memory management, a key aspect for anyone aiming to develop malware or wants to understand how windows works under the hood.

### Virtual Memory & Paging in Windows

In operating systems like Windows, the memory utilized by processes is not a direct reflection of the physical memory (such as RAM). Instead, virtual memory addresses are employed by processes, which are then mapped onto physical memory addresses. This system primarily aims to optimize physical memory usage. Virtual memory might correspond to physical memory, but it can also reside on disk storage. A notable advantage of virtual memory is that it enables multiple processes to share the same physical memory space while maintaining distinct virtual memory addresses. This is facilitated through memory paging, which segments memory into 4KB units known as “pages.”

Refer to the image below taken from “Windows Internals 7th edition – Part 1“. These books are a must if you are interested in this topic, there is no better resource.
![virtual-memory.png]({{site.baseurl}}/_posts/virtual-memory.png)

### Page States in Virtual Memory

Pages in a process’s virtual address space can exist in one of three states:

    Free: These pages are not in use (neither committed nor reserved) and are inaccessible to the process. They stand ready for reservation, commitment, or both. Accessing a free page may trigger an access violation exception.
    Reserved: These pages are earmarked for future use. Their address range is off-limits to other allocation functions, and they lack physical storage, making them inaccessible until they are committed.
    Committed: These pages have been allocated memory from the total available RAM and paging files. Accessible and governed by specific memory protection constants, these pages are only loaded into physical memory when first read or written to. Upon process termination, the storage for these pages is released.

### Setting Page Protection Options

Once pages are committed, they must be assigned protection settings. The available memory protection constants are detailed in the provided reference, with examples including:

    PAGE_NOACCESS: Blocks all access to the committed region. Any read, write, or execute attempt will cause an access violation.
    PAGE_EXECUTE_READWRITE: Permits reading, writing, and executing, but this setting is generally a red flag (Indicator of Compromise) due to the rarity of a memory region being both writable and executable.
    PAGE_READONLY: Allows only read access. Writing to this region will lead to an access violation.

### Built-in Memory Protections in Modern Operating Systems

Modern operating systems, including Windows, incorporate various memory protection mechanisms to counter exploits and attacks, essential for malware development and debugging:

    Data Execution Prevention (DEP): From Windows XP and Server 2003 onwards, DEP prevents code execution in non-executable memory regions, like those set to PAGE_READONLY.
    Address Space Layout Randomization (ASLR): ASLR hinders the exploitation of memory corruption vulnerabilities by randomizing the address space locations of critical process components, such as the executable’s base and the positions of the stack, heap, and libraries.

### Differences in x86 and x64 Memory Spaces

For Windows processes, it’s crucial to distinguish between x86 and x64 architectures. x86 processes are limited to a 4GB memory space, while x64 processes can access a substantially larger space of 128TB.
### Example of Memory Allocation in Windows

The following code snippets illustrate various approaches to memory allocation in Windows using C functions and APIs. The primary objective in memory interaction is allocation, as demonstrated below:

// Allocating a 100-byte memory buffer

// Method 1 - Using malloc()

PVOID pAddress = malloc(100);

// Method 2 - Using HeapAlloc()

PVOID pAddress = HeapAlloc(GetProcessHeap(), 0, 100);

// Method 3 - Using LocalAlloc()

PVOID pAddress = LocalAlloc(LPTR, 100);

Each memory allocation function returns the base address, a pointer to the start of the allocated memory block. Depending on the assigned protection, various operations can be performed using this pointer.

Refer to the provided image to see pAddress under a debugger.
![image.png]({{site.baseurl}}/_posts/image.png)


Note: Allocated memory may initially be empty or contain random data. Some allocation functions include an option to initialize the memory to zero during allocation.
![image-1.png]({{site.baseurl}}/_posts/image-1.png)


### Example of Writing to Allocated Memory

Following allocation, writing to the memory buffer is a common next step. For instance:

PVOID pAddress = HeapAlloc(GetProcessHeap(), HEAP_ZERO_MEMORY, 100);

CHAR* cString = "Malware is fun";

memcpy(pAddress, cString, strlen(cString));

Here, HEAP_ZERO_MEMORY in HeapAlloc ensures the memory is zero-initialized before the string is copied using memcpy. The final parameter in memcpy specifies the copy length. Re-examine the buffer to confirm successful data writing.
![image-3.png]({{site.baseurl}}/_posts/image-3.png)


### Freeing Allocated Memory

To prevent memory leaks, it’s advisable to free any allocated buffers when they’re no longer needed. The deallocation function used should correspond to the allocation method:

    malloc() requires free().
    HeapAlloc() requires HeapFree().
    LocalAlloc() requires LocalFree().

The accompanying images illustrate HeapFree in action, releasing memory at address 0x0000017129913580. Post-release, this address still exists but is overwritten with random data, likely from a new allocation by the OS within the process.

That’s the basics of it. You now know how to allocate, write to and free memory within your process. That allows us to take on the next part and talk a little bit more about the specifics and details of the Windows Process.
![image-4.png]({{site.baseurl}}/_posts/image-4.png)


###  Code used in this post:

#include <Windows.h>
#include <stdio.h>


int main() {
	PVOID pAddress = HeapAlloc(GetProcessHeap(), HEAP_ZERO_MEMORY, 100);
	printf("[+] Base address of allocated memory: 0x%p \n", pAddress);

	CHAR* cString = " Malware is fun";
	printf("Inserting data in memory...\n");
	memcpy(pAddress, cString, strlen(cString));
	printf("[+] Memory of pAddress holds: %s\n", (char*)pAddress);

	printf("[+] Let's free up the memory!\n");
	HeapFree(GetProcessHeap(), 0, pAddress);

	if (pAddress != NULL) {
		printf("[+] Memory should be freed. New content: %s\n", (char*)pAddress);
	}
	else {
		printf("[+] Memory of pAddress is empty\n");
	}
	getchar();
	return 0;
 }