# Introduction
This project extends the xv6 operating system to correctly handle null pointer dereferences.

In the original xv6, the first page of memory is mapped and accessible, which means address 0x0 does not trap as expected. Modern operating systems always mark the first page invalid to prevent this behavior. 

The objective of this project is to modify xv6 so:
- The null page (0x0000–0x0FFF) is always unmapped.
- Any attempt to dereference a null pointer results in a trap and process termination.
- System calls properly validate user pointers before accessing them.

These changes are verified using the provided test program test1-test2-test3.c


# Setup
```bash
cd xv6_patched
    # Enter the correct directory
```

```bash
vim +115 kernel/makefile.mk
    # Fix for "make: ./kernel/sign.pl: Command not found" error
```

```bash
vim +19 user/makefile.mk
    # Added test1-test2-test3 to user
```

# Preview

```bash
make qemu-nox
ls
test1-test2-test3
```

# Test cases 1 and 2

### kernel/exec.c: Change the value of PGSIZE to leave the null page inaccessible

```bash
vim +35 kernel/exec.c
    # changed `sz=0;` to `sz=PGSIZE;`
    # to start loading program at PGSIZE (0x1000) and prevent memory access in the first page
```

By starting at PGSIZE (0x1000) instead of 0x0, the first page is left unmapped. This makes a protected zone, where any access to the address 0-4095 will cause a page fault and kill the process, which catches null pointer bugs immediately.

### makefile.mk: Change the place where the program will start to load

```bash
vim +77 user/makefile.mk
vim +126 kernel/makefile.mk
    # Changed memory location to 0x1000 (4096)
    # to link user programs to start at 0x1000
```

The linker must know where the code will run so it can generate the correct addresses for jumps and function calls. Since exec.c loads user programs at 0x1000, the linker must compile them for 0x1000. Otherwise, wrong addresses result in a crash.

```bash
vim +87 kernel/proc.c
```

Changed `p->sz = PGSIZE;` to `p-sz = 2*PGSIZE;` because the process's address space now spans 0-0x2000 (even though the first page is unmapped).

```bash
vim +94 kernel/proc.c
```

Changed `p->tf->esp = PGSIZE;` to `p->tf->esp = 2*PGSIZE;` because the stack needs room to grow downwards from the top of memory.

```bash
vim +95 kernel/proc.c
```

Changed `p->tf->eip = 0;` to `p->tf->eip = PGSIZE;` because the code now starts here, not in the null space.

```bash
vim +198 kernel/vm.c
```

Changed `mappages(pgdir, 0...` to `mappages(pgdir, (void*)PGSIZE...` to map second page (rather than the null page) to physical memory.

```bash
vim +309 kernel/vm.c
```

Changed 'i = 0' to 'i = PGSIZE' to ensure child processes only copy the parent's memory excluding the null page.



# Test Case 3

### In Fetchint method, do not extend outside of the stack into the code of the program
```bash
vim +22 kernel/syscall.c
vim +39 kernel/syscall.c
    # Added `if(p->pid > 1 && addr < 4096)` 
    # and `return -1`
```

On lines 22 and 39 in fetchint & fetchstr, the problem was that the addresses inside the first 4 KB (the null page) are never valid user data. Real programs don’t keep data there; the OS intentionally marks it invalid. The program fixes this by stopping bad pointers (cases a & b) before xv6 ever dereferences them.

### argptr: check that the pointers remain in the stacdk and do not point to the code of the program

```bash
vim +70 kernel/syscall.c
    # Added if((unit)i < 4096 || (uint)i=USERTOP)
    # and `return -1`
        # P.S. USERTOP is defined in include/param.h:15 as 0xA0000 = 655,360
```

On line 70 in argptr USERTOP marks the boundary between legal user memory and the code/kernel region. If a pointer touches that boundary, it means: it points into the code segment, or its size might spill into it. The third bad pointer starts inside valid stack memory, but its length goes past the legal range. The program fixes this by checking blocks the entire scenario. 

# Testing cases

```bash
vim +16 user/test1-test2-test3.c
    # Un-commented test3 "bounds" so the next part of the project can be tested
```

# Conclusion
This project required modifying xv6 to invalidate the null page, update memory layout loading logic, and strengthen pointer validation inside system calls.

With these changes, xv6 works like a modern OS when there’s null pointer dereferences and incorrect syscall pointers.

All three test cases from the provided test program were successfully passed. 