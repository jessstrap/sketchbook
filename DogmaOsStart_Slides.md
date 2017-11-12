A practical, useable, single address space, os.
Name is a play on TempleOS and comes from the dogmatic nature of the loader. 

Uses a single virtual address space to enable paging, copy on write processes, and read only views on physical pages. 
The only code that is not allwed to be paged out is the top level page table and the binary of the paging handling code. Other regions can be marked with varying levels of 'shouldn't page out' but the policy for observing the markings is up to the init process.
Accidental attempts to write to read only views result in a call to an optional, per process, handler routine. If there is no handler routine defined by the process the process is torn down. The handler routine can also ask the process to be torn down. The syscall to set the handler for a process takes a closure, and a size for the ammount of stack that the closure will need. The parameters for the closure are not currently defined, but they will probably include process information and basic information to enable stack inspection. 
All stack manipulation is done through syscalls. (This is to allow eficient segmented stacks.) This includes function calls, function returns, and allocas. All functions must provide information about the maximum ammount of stack space they will consume. All allocas must provide a pointer to the stack frame that they will be deallocated with. 
Segmented stacks are required in order to keep the various stacks from fragmenteing the memory space and avoid the stack overflowing into other memory objects. The only form of stack overflow is out of memory/out of address space. How to correctly handle that is a policy decision, and out of scope for the kernal, dealing with it is delegated to the init process. 

There are no threads. Only processes. 

process model
Processes start off with very little. 
    an execution context.
        includes a stack pointer to a stacklet with at least enough space for the starting function.
        includes a pointer to a read only view of the loaded binary.
    a pointer to a read only view of the process visible section of the process descriptor.
        includes a pointer to the syscall closure for this process. 
        includes the parent processes id.
Shared libraries
    If a process wants to load a shared library it does a syscall. This syscall returns a pointer to a readonly view of the loaded binary of that library
    The process can later surrender access to this view to the os (using a syscall). After using this syscall the process cannot read this memory any more. (This is enforced by the loader.)
Dynamic loading syscall.
    this syscall is fallible.
    the process surrenders a chunk of memory to the kernal containing a binary to be loaded into an executable.
    the os then has the option of handing back a pointer to a read only view of the loaded binary that corresponds to the binary handed in. 
quasi cooperative premption.
    scheduler sets a 'preemption pending' bit in the processes block. 
    process can check this bit and and (optionally) save its context and yield to the os (jump into the scheduler). 
    when volentariliy yielded to, the os doesn't save or restore execution context other than stack pointer, process id, and instruction pointer. 
    allows the process to yield at times when it knows it has very little state in its execution context. (similar to a caller saved function or an inline assembly section that clobbers everything)
    compilers/runtimes are encouraged to insert this check/save/yield code wherever sensible. 
    
        
IPC
provides cheap cross process locking, channels, and sockets on top of these mechanisms
can yield to the scheduler.
can suspend self untill wake conditions.
can yield to another specific process. 
    time till preemption doesn't change.
    other processes stats in the scheduler don't change. 
can get os to mark/unmark exclusively owned areas of memory as read only.
can get os to mark/unmark exclusively owned areas of memory as non readable
capabilities:
    can set/unset status as listening for capabilities.
    capabilities send to processes that aren't listening are automatically disposed.
    capabilities owned by processes that are torn down are automatically disposed
        (disposal could mean deallocating memory or sending the capability to process 0)
    can recieve capabilities
    can use recieved capabilities.
    can send capabilities. 
    can send capabilities back to os (disposal)
    capability types
        exclusive ownership of memory region (page resolution)
            non use after transfer is enforced by the loader
        shared ownership of read only view on memory region (page resolution)
            some base level of atomic gaurentees on read/write resolution. 
                (usize?) 
                ?concurrency models? 
                ?provided by underlying hardware?
            if source process is torn down or deallocates the memory the dest processes have a guarentee that either
                reads will see last contents of the region.
                reads will see zeroes.
                    the page at virtual memory address 0 is guarenteed to be non readable and non writeable.            
        wake me
        tear me down
        suspend me
        debugging specific stuff
            read my memory while i am suspended. 
            set breakpoints in me 
            manipulate my memory while i am suspended
                how to keep this memory safe?
                    replace binary with a one with memory checks all over if invoked?
            modify my binary while i am suspended
                how to keep this memory safe?
                    replace binary with a one with memory checks all over if invoked?
        read/execute my binary
        monitoring capabilities
            if i am torn down
            if i hit a breakpoint
            if i am suspended
            ect.
            includes the ability to set a wake on condition. eg: wake me if x is torn down.
        process defined capability.
            could be used for file acls and such. 
            could be used for async io. 
            processes can ask the os to verify that a specific process has a capability
 
The loader (the dogmatic part.)
There is no linker, only a loader.
    All loaded binaries are read only. 
    if a process wants to uses copy on write processes, all references to memory/stack must be annotated so they can be updated on copy. 
    process must come with a machine checkable proof (checked by loader) that it 
        doesn't reference memory that hasnt been handed to it by the os.
        doesn't reference memory that it has handed back to the os.
        only calls syscalls in ways that are safe
            the definition of 'ways that are safe' for syscalls come (per syscall) from the proofs on the kernal. 
            Those proofs are checked at kernal compile time and shipped with the kernal binary.
            Those proofs are probably going to just revolve around exclusive access during syscall or non mutation during syscall
        ?typed assembly language?
        proof by construction?
        rustic type and region inference?
        syscalls for safe indexing?
        syscalls with rich types provided by the os?
        all memory references have to be on a sized type?
    processes can opt into recieving non zeroed memory if they can prove that they don't read from it before writing to it.
    processes can opt into being awoken with unclean context if they can prove that they don't read from it before writing to it. 
    if the proof doesn't check out the loader can reject the load.
    if the proof is absent the loader can replace all hazerdous instructions with less performant safe instructions. (checked indexing, checked pointer dereferencing, ect.)
    incoming binaries must be position independent. 
    time and space for translating from the binary to machine code should be order of size of binary.
The hardest part of making the os practical is going to be creating a binary format, proof format, and standard library of proofs that make it possible for compilers/runtimes to target/run on this platform without having performance penalties tha are worse than the costs of address space and protection ring based security. 
    
The bootloader. (sets up binaries and initial stack for kernal.)
    The outer bootloader is responsible for picking out a safe region of physical memory for the kernal binary and loading it there. stores address in registers
    The outer bootloader is responsible for picking out a safe region of physical memory for the kernals first stacklet. stores address and size in registers.
    jumps to the kernal.
    
The kernal: Driver for the processor and memory. 
Must know what parts of physical memory are 'sensitive'.
Sets up and administrates virtual memory and paging.
Sets up fault and interrupt handling. 
    can set bit for 'interrupt x has triggered.'
    can set waiting processes to runnable.
    can set 'preemption imminent' bit on a running process
    can preempt a process
        stores all registers, context, ect.
            on process resume all context is either restored or cleared. 
        jumps to scheduler.
    this is the only kernal code executed in a processes context without cooperation from a process
        this is the only kernal code that can't be interupted.
        this is the only kernal code that must be careful about registers/floats/ect.
        this is the only kernal code that has to be compiled with a restricted instruction set.
Sets up scheduling interrupt timer.
Handles execution of scheduler policy. 
    the scheduler data structures are probably the only place in the kernal where we need mutexes instead of just read only views and message passing. 
Provides processes and ipc. 
Provides the loader.
loads and starts process 1.
resets the os if processs 1 is torn down.
allows process 1 to configure kernal.
allows process 1 to allocate views on 'sensitive' regions of physical memory.
startup
    sets up memory/processes.
    loads process 1 binary.
    does other setup
    disables bios services.
    marks process 1 as executable.
    minimizes own stack space
    calls into scheduler.
 
process 1 (the driver driver)
The binary includes enough drivers and config to get access to binaries and config for the rest of the drivers.
Starts other drivers/services
    starting process:
        gets binary of driver.
        gets kernal to load driver into a process.
        gets kernal to start the drivers process
        upon request from the process gets kernal to give the process a view on sensitive memory.
        when driver processes are torn down, recieves ownership of these views from the kernal. 
        monitors driver proccesses. 
        can tell kernal to kill processes
        optionally restarts driver processes.
    Other drivers
        a service id to process id mapping service.
        async io servce (eventd?)
        hard drives
        network cards
        networking stack
        filesystems
        access controls for sensitive files. 
            config
            certificates
            binaries
            ect.
        page storage/retrieval mechanism for kernal.
        paging policy for kernal
        scheduling policy for kernal
        out of memory/address space handler for kernal
        logging system for kernal
        other hardware drivers
        authentication
        user setting storage
        hids
        text display
        framebuf display
        gpu driver
        generic gpu apis
        root display driver (wayland)
        sound devices
        sound services
        runlevels/acpi
        sessions
        remote services
            ssh
            remote login
        local login/screen management
            in turn can use sessions service to create a session and lauch a desktop environment in it.
        sudo/sudo ui/sudo protocals. (how to make this non phishable?)
        other services for shared runtimes
            posix?
            win32?
            android?
        other services for fancier ipc.
        driver multiple drivers to share a process.

questions to be addressed:
    can we have a driver that could allow a process to provide its own 'hardware assisted private address space emulation' and use that for its safety proof? Can we do it in a way that is performant and doesn't hurt the performance of other processes?
    
As a project would yield the following
a new ABI and executable format 
    inspired by
        intel assembly
        sysV ABI
        ELF
        (maybe) webAssembly
    Amenable to proofs of memory safety.
        typed. (annotations seperate, like dwarf)
        limited. (no syscalls, interupts, out of function jumps. allowed to link to runtime functions.)
        function/data/library pointers are modifiable by runtime at load time.
        position independent
    Fast linking/loading.
    compact.
A loader/linker/transpiler for the abi/format to x86_64 binary.
A single process runtime for the abi/format
An optimized storage/transmission format for (linkable) machine checkable proofs
    optimized for check time
    optimized for small size
A 'standard library' of theorems that can be used in the proofs. 
A 'standard library' of theorems about the runtime and its apis
A machine checkable definition of 'memory safety' in the context of the runtime. 
A checker for the proofs.
A set of aditional apis, theorems, and proof requirements for the runtime.
    Access to wayland
        DRM/DRI backed buffers
    Access to sound output
    Access to sensors
    Access to gpu compute. (to support the proofs: no binary blobs.)
        ?mesa/swrast-llvmpipe?
        ?mesa/DRM-DRI?
        ?openAmd/DRM-DRI?
    access to network
    access to filesystem
A llvm backend that targets the runtime/abi and outputs proofs/theorems of correctness in 
A port of chromium, firefox, or servo to the runtime/abi/proofs/theorems.
    no jit. only interpreter.
A port of V8 or spider monkey jit backend and runtime targeting the runtime/abi/proofs/theorems
A port of the runtime to kernal space.
    built on redoxOs or linux.
    loads the binaries to virtual kernal space.
    runs the binaries in a kernal space process.
Optimizations for the kernal space runtime
    remove need to communicate with userspace to pass data to sound driver/window manager/gpu driver/file system/network/ect.
    don't implement a second scheduler on top of the already existing one. 
Better drivers/apis built on this runtime and Capabilities. 
        
