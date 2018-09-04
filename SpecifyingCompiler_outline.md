# Goal: a practical, high performance OS and ecosystem built around strong formal proofs of safety. 

## Strategies:
    1. Make formal safety proofs easy.
    2. Make formal safety proofs strong. 
    3. Make formally proven items high performance and high capability.

## Execution: 
Prototype a 'specifying compiler' C, a 'standard proof library' SPL, and a runtime R.
- Compiler inputs: 
    - A program P in language LP.
    - Annotations about the program in formal language LF.
- Compiler outputs:
  - A binary B for architecture A and runtime R.
  - A full formal specification S of behavior properties of B as run on A and R.
  - A machine checkable proof P of specific safety properties of Bs behavior as run on A and R.
  - Note: I am not attempting to verify that B running on A and R corresponds to P.
      - The correctness of the compilation is not being proven, just safety properties of the result. I speculate that this will be easier.
        
## Constraints:
   - To serve strategy 1 LP should have first class affordances for safety.
   - To serve strategy 3 LP should be a low level language.
       - I think Rust would be a good candidate for LP.
       - I think C/C++/Fortran could work, but would need a lot of annotations in LF to ever be performant.
       - I think Python/Java/Ruby/ect. could work, but the large runtimes would require a lot of specification and verification. 
   - To serve strategy 2 A should be well specified and have verifiable implementations, firmware and drivers.
   - To serve strategy 3 A should be high performance.
       - I think RISC-V is a good candidate for A.
   - To serve strategy 2 R should be well specified, verifiable, and should have first class affordances for safety.
       - I think that R should be written in LP with annotations in LF and compiled with C.
       - I think that R should be written with an api based on Capabilities. 
   - To serve strategy 3 R should have first class affordances for performance.
       - R should check and take advantage of formally proven properties of the binaries it loads to provide high performance. 
       - Potential directions:
           - Single address space.
           - All code runs in supervisor mode.
           - Semi-cooperative preemptive multi tasking to minimize cost of execution context switching.
           - Very high performance unidirectional data channels.
   - To serve strategy 3 C should be an optimizing compiler.
    
