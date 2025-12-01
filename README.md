# Upgrading Xv6 OS to throw an exception when a null pointer is dereferenced

<img src="./AllTestsPass.png" alt="test1-test2-test3 is a command provided by the instructor of this course. A terminal window of Qemu running the three provided tests to ensure that null pointers cannot be dereferenced. All three tests are passed">

# Video presentation
"Upgrading Xv6 OS to throw an exception when a null pointer is dereferenced." youtu.be/p9kv0_HHg1Q?si=RtAnNFD7SLGdU5o1

# xv6-Mini-Project-3
The goal of this project is to implement null page protection in xv6. This will ultimately ensure null pointer references are caught, making debugging simpler.

Instructor: Associate Professor Dr. Luis G. Jaimes. Directory entry: https://www.floridapoly.edu/faculty/bios/luis-jaimes/

Students: Nathan Cader (report), [Benjamin Voor](https://www.github.com/Benjamin-Voor) (video presentation), [Aidan Guenther](https://github.com/guentheraidan) (parts 1 and 2 and GitHub repository), [Khalee Johnson](https://www.github.com/Khalee242) (part 3)

This is a class assignment called "Mini-Project 3" for the Operating Systems Concepts course (course code COP4610)

This project extends the xv6 operating system to correctly handle null pointer dereferences.

In the original xv6, the first page of memory is mapped and accessible, which means address 0x0 does not trap as expected. Modern operating systems always mark the first page invalid to prevent this behavior.

The objective of this project is to modify xv6 so:

- The null page (0x0000–0x0FFF) is always unmapped.
- Any attempt to dereference a null pointer results in a trap and process termination.
- System calls properly validate user pointers before accessing them.


These changes are verified using the provided test program test1-test2-test3.c

This project required modifying xv6 to invalidate the null page, update memory layout loading logic, and strengthen pointer validation inside system calls.

With these changes, xv6 works like a modern OS when there’s null pointer dereferences and incorrect syscall pointers.

All three test cases from the provided test program were successfully passed.

The textbook for this course is freely available from the following link, credits to Dr. Remzi H. Arpaci-Dusseau: https://pages.cs.wisc.edu/~remzi/OSTEP/

VIDEO Project descriptions by Dr. Remzi H. Arpaci-Dusseau (Book author) : "Discussion of P3 (Part 1)": https://youtu.be/M2CPjjVTmpg?si=LNnBqVAAb0W0s-Q5

The Xv6 OS is also freely available online, credits to MIT. Here is a link to Arpaci-Dusseau's installation instructions: https://www.github.com/remzi-arpacidusseau/ostep-projects/blob/master/INSTALL-xv6.md/

No license is provided. We fall under fair use in the USA, as this is for educational purposes at Florida Polytechnic University.