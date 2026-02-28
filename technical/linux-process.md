When it comes to processes, the most common interview question is the relationship between processes and threads. Here is the answer first: **in Linux, there is almost no difference between processes and threads**.

In Linux, a process is just a data structure. Once you understand it, you can understand how file descriptors, redirection, and pipes work under the hood. In the end, we will also see why we say threads and processes are almost the same from the operating system’s point of view.

### 1. What Is a Process

First, in an abstract way, our computer looks like this:

![](../pictures/linuxProcess/1.jpg)

The big rectangle is the computer’s **memory space**. The small rectangles inside are **processes**. The circle at the bottom left is the **disk**. The shape at the bottom right is some **I/O devices**, such as mouse, keyboard, and monitor. Note that the memory space is divided into two parts: the upper part is **user space**, the lower part is **kernel space**.

User space holds the resources used by user processes. For example, if you create an array in your code, that array is in user space. Kernel space holds system resources used by kernel processes. These are usually not accessible to user processes. But some kernel-space resources can be shared with user processes, such as some shared libraries.

We write a hello program in C, compile it into an executable file, then run it in the terminal. It prints “hello world” and exits. From the OS view, a new process is created. This process reads the executable file into memory, then runs it, then exits.

**The compiled executable is just a file**, not a process. To really run, the executable must be loaded into memory and wrapped as a process. The process is created by the OS. Each process has its own properties, such as process ID (PID), process state, open files, and so on. After the process is created, the OS loads your program into it, then your program runs.

So how does the OS create a process? **For the OS, a process is just a data structure**. Let’s look directly at Linux source code:

```cpp
struct task_struct {
	// process state
	long			  state;
	// virtual memory structure
	struct mm_struct  *mm;
	// process ID
	pid_t			  pid;
	// pointer to the parent process
	struct task_struct __rcu  *parent;
	// list of child processes
	struct list_head		children;
	// pointer to filesystem information
	struct fs_struct		*fs;
	// an array containing pointers to files opened by the process
	struct files_struct		*files;
};
```

`task_struct` is the kernel’s description of a process. You can also call it the “process descriptor”. The full code is complex; here we only show some common fields.

Two interesting fields are the `mm` pointer and the `files` pointer. `mm` points to the process’s virtual memory, which is where resources and executables are loaded. `files` points to an array that holds pointers to all files opened by this process.

### 2. What Is a File Descriptor

Let’s talk about `files`. It is an array of file pointers. In general, a process reads input from `files[0]`, writes output to `files[1]`, and writes errors to `files[2]`.

For example, in our view, C’s `printf` prints to the terminal. But from the process’s view, it writes data to `files[1]`. Similarly, `scanf` reads data from `files[0]`.

**When a process is created, the first three entries of `files` are filled with default values: standard input, standard output, and standard error. What we call “file descriptor” is just the index of this file pointer array.** So by default, file descriptor 0 is input, 1 is output, 2 is error.

We can draw a picture:

![](../pictures/linuxProcess/2.jpg)

On a normal computer, the input stream is the keyboard, the output stream is the monitor, and the error stream is also the monitor. So this process connects to the kernel with three “lines”. Because hardware is managed by the kernel, our process must use “system calls” to ask the kernel to access hardware.

::: note Note

Don’t forget, in Linux everything is abstracted as a file. Devices are also files, which can be read and written.

:::

If our program needs other resources, such as opening a file to read or write, this is also simple. We make a system call and let the kernel open the file. That file will then be placed in the 4th position of `files`:

![](../pictures/linuxProcess/3.jpg)

With this, **input redirection** is easy to understand. When a program wants to read data, it reads from `files[0]`. So if we point `files[0]` to a file, the program will read from that file instead of the keyboard:

```shell
$ command < file.txt
```

![](../pictures/linuxProcess/5.jpg)

Similarly, **output redirection** is just pointing `files[1]` to a file. Then the program’s output will be written to that file, not to the monitor:

```shell
$ command > file.txt
```

![](../pictures/linuxProcess/4.jpg)

Error redirection is the same idea, so we skip it.

The **pipe operator** works in a similar way. It connects the output stream of one process to the input stream of another process with a “pipe”. Data flows in this pipe. This design is very elegant:

```shell
$ cmd1 | cmd2 | cmd3
```

![](../pictures/linuxProcess/6.jpg)

Now you can see how clever “everything is a file” is in Linux. No matter if it is a device, another process, a socket, or a real file, everything can be read and written. The OS puts them all into one simple `files` array. The process only needs a simple file descriptor to access the right resource. The OS hides the complex details, which decouples things well and is both simple and efficient.

### 3. What Is a Thread

First we need to be clear: both multi-process and multi-thread are concurrency. Both can improve CPU usage. So the key question is: what is the difference between them?

Why do we say threads and processes are almost the same in Linux? Because from the Linux kernel’s view, it does not really treat them as different things.

We know that the system call `fork()` can create a new child process, and the function `pthread()` can create a new thread. **But both threads and processes are represented with the same `task_struct` structure. The only difference is which data areas are shared.**

In other words, a thread looks just like a process. The difference is: some data areas of a thread are shared with its parent process, while a child process gets its own copy, not a shared area. For example, in threads, the `mm` structure and `files` structure are shared. Look at the two diagrams:

![](../pictures/linuxProcess/7.jpg)

![](../pictures/linuxProcess/8.jpg)

So, in our multi-thread programs, we need locks to avoid multiple threads writing to the same area at the same time, or data may get corrupted.

You may ask: **since processes and threads are similar, and processes do not share data so there is no data race problem, why are threads used more widely than processes?**

Because in real life, concurrent tasks that share data are more common. For example, ten people withdraw 10 yuan each from one bank account. We want the shared account balance to decrease by 100 yuan in total, not ten separate copies each decreasing 10 yuan.

We should also note: only in Linux are threads treated as processes that share some data. The kernel does not give them a special structure. Many other OSes treat threads and processes differently; threads have their own special data structures. In my opinion, this is less simple than Linux’s design and increases system complexity.

In Linux, creating threads and processes is both very efficient. To handle the problem of copying memory when creating a new process, Linux uses a copy-on-write strategy. That is, it does not actually copy the parent’s memory at first. It copies only when a write happens. **So creating a new process or a new thread in Linux is very fast.**
