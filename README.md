### **Fuzzing Technique Overview Documentation**

#### **Introduction to Fuzzing**

Fuzzing is a powerful technique used to test the security and robustness of software systems by injecting unexpected, malformed, or invalid inputs into the system. It helps uncover defects such as crashes, memory leaks, and security vulnerabilities.

#### **Types of Fuzzing:**

- **Mutation-based Fuzzing**: Takes valid inputs and mutates them by flipping bits, changing sequences, or adding random characters.
- **Generation-based Fuzzing**: Generates inputs based on rules or specifications, often targeting specific protocols or file formats.

#### **Fuzzing Tools:**

##### **1\. AFL (American Fuzzy Lop)**

- **Type**: Coverage-guided mutation fuzzer for C/C++ applications.
- **Overview**: AFL intelligently mutates inputs and tracks execution paths to maximize code coverage, making it ideal for fuzzing low-level applications.
- **Use Cases**: Popular in fuzzing Linux binaries and detecting issues like buffer overflows.

##### **2\. Jazzer**

- **Type**: Mutation-based fuzzer for Java applications.
- **Overview**: Jazzer targets JVM applications and is based on libFuzzer. It monitors code coverage and tracks runtime behavior to expose vulnerabilities like null pointer dereferences and assertion violations.
- **Use Cases**: Typically used for testing Java-based systems, it supports fuzzing of unit tests.

##### **3\. Peach Fuzzer**

- **Type**: Generation-based fuzzer.
- **Overview**: Peach is a fuzzing platform that allows highly customized input generation, focusing on protocols, file formats, and web services.
- **Use Cases**: Effective for testing structured data inputs such as XML or network protocols.

#### **Advantages of Fuzzing:**

- **Automation**: Once set up, fuzzers can run continuously, testing thousands or even millions of inputs.
- **Code Coverage**: By maximizing code paths explored, fuzzing uncovers even obscure bugs.
- **Vulnerability Discovery**: Fuzzing is one of the best ways to discover critical security flaws, such as buffer overflows or null pointer dereferences.

#### **Common Vulnerabilities Discovered Using Fuzzing:**

- **Buffer overflows**: Data overflow that overwrites adjacent memory.
- **Null pointer dereferences**: Crashes when accessing null memory.
- **Memory leaks**: Failures in deallocating memory, causing memory exhaustion.

#### **Environment Setup:**

### **Installing and Setting Up Ubuntu on a Virtual Machine (VM) using VirtualBox**

1. **Download Ubuntu ISO**:
    - Visit [Ubuntu's website](https://ubuntu.com/download) and download the latest version (LTS recommended).
2. **Install VirtualBox**:
    - Download and install VirtualBox from the [VirtualBox website](https://www.virtualbox.org/).
3. **Create a Virtual Machine**:
    - Open VirtualBox and click "New."
    - Name your VM , select "Linux" as the type, and "Ubuntu (64-bit)" as the version.
    - Allocate 2-4GB of RAM (4GB is better).
    - Create a virtual hard disk of at least 20GB, using the default settings.
4. **Start the VM and Install Ubuntu**:
    - Select your VM and click "Start."
    - Choose the Ubuntu ISO you downloaded.
    - In the Ubuntu installer, follow the installation steps set your language, keyboard, and user info. Choose the option to erase disk and install Ubuntu.
5. **Complete Installation**:
    - Once the installation is done, restart the VM.
    - Log in using the username and password you created.
6. **Update Ubuntu**:
    - Open a terminal and run the following commands to update your system:
        - Sudo apt-get update
        - Sudo apt-get upgrade

#### **Phase One AFL Experiment Steps:**

**Install GCC and Clang:**

- **Command:**
  - **Sudo apt install gcc**
  - **Sudo apt install clang**
- **Explanation:** These commands install the **GCC** (GNU Compiler Collection) and **Clang** compilers. Both are essential for compiling applications in C/C++ and other languages. AFL (American Fuzzy Lop) instruments binaries during compilation to add fuzzing capabilities, so having these compilers installed ensures compatibility with AFL’s fuzzing process.

**Download and install AFL:**

- **Command:**
  - wget <http://lcamtuf.coredump.cx/afl/releases/afl-latest.tgz> tar -xzvf afl-latest.tgz
  - cd afl-2.52b/
  - Make
  - Sudo make install
- **Explanation:**
  - wget downloads the latest version of AFL from its official website.
  - tar -xzvf extracts the downloaded file, which is compressed in a .tgz archive format.
  - cd afl-2.52b/ changes the directory to the AFL source directory.
  - make compiles the AFL source code into binaries, preparing it for installation.
  - sudo make install installs AFL system-wide so that it can be used from anywhere on the system.

**Enable LLVM mode for faster fuzzing:**

- **Command:**
  - cd afl-2.51b/llvm_mode/
  - sudo apt-get install llvm-dev llvm
  - Make
  - cd ..
  - Make
  - sudo make install
- **Explanation:**
  - **LLVM Mode**: AFL can use LLVM to instrument binaries. LLVM mode allows for better performance and faster fuzzing due to its optimization features.
  - sudo apt-get install llvm-dev llvm: Installs LLVM and development tools required for compilation.
  - make: Builds AFL in LLVM mode.
  - After returning to the root AFL directory (cd ..), the make and sudo make install commands are run again to ensure the AFL tool is updated to support LLVM-based fuzzing.

**Clone and compile the intentionally vulnerable application (Fuzzgoat):**

- **Command:**
  - git clone <https://github.com/fuzzstati0n/fuzzgoat>
  - cd fuzzgoat/
  - export CC=afl-clang-fast
  - Make
- **Explanation:**
  - git clone: Downloads the Fuzzgoat source code from its GitHub repository. Fuzzgoat is an intentionally vulnerable application designed for fuzzing experiments.
  - export CC=afl-clang-fast: Sets the environment variable CC to AFL’s afl-clang-fast compiler, which instruments the binary during compilation. Instrumentation adds code to the binary that allows AFL to interact with it while fuzzing.
  - make: Compiles the Fuzzgoat application using the afl-clang-fast compiler, generating an instrumented binary ready for fuzz testing.

**Create a directory for test inputs (corpus):**

- **Command:**
  - mkdir afl_in
  - cp /bin/ps afl_in/
- **Explanation:**
  - mkdir afl_in: Creates a new directory named afl_in to store seed input files, also known as the test corpus. These files serve as the starting point for AFL to begin fuzzing.
  - cp /bin/ps afl_in/: Copies the ps binary from the system to the afl_in directory as an initial seed input file. In real-world scenarios, you would provide more representative input files related to the target application, but for demonstration, any file can be used.

**Create a directory for fuzzing results:**

- **Command:**
  - mkdir afl_out
- **Explanation:**
  - mkdir afl_out: Creates a directory where AFL will store the results of its fuzzing process. AFL categorizes the results into three subdirectories:
    - crashes: Stores inputs that caused the application to crash.
    - hangs: Stores inputs that caused the application to hang or freeze.
    - queue: Contains inputs that AFL has generated but not yet tested.

**Run AFL:**

- **Command:**
  - afl-fuzz -i afl_in -o afl_out -- ./fuzzgoat @@
- **Explanation:**
  - afl-fuzz: The AFL fuzzing tool.
  - \-i afl_in: Specifies the directory (afl_in) where the seed input files (test corpus) are stored.
  - \-o afl_out: Specifies the directory (afl_out) where AFL will store the output results, including crashes, hangs, and test cases.
  - \-- ./fuzzgoat @@: Specifies the target binary to fuzz, ./fuzzgoat, and @@ represents the placeholder where AFL will insert its generated test cases into the command line for the target. In this case, AFL will pass the test case as an argument to fuzzgoat.

#### **References:**

<https://medium.com/ouspg/fuzz-testing-beginners-guide-da2c9179caa7#:~:text=Fuzzing,%20in%20short,%20is%20about%20inserting%20malformed,%20unexpected,%20or%20even>

<https://www.synopsys.com/glossary/what-is-fuzz-testing.html#:~:text=Fuzz%20testing%20or%20fuzzing%20is,as%20crashes%20or%20information%20leakage>

<https://www.thetesttribe.com/blog/fuzz-testing/>

<https://github.com/CodeIntelligenceTesting/jazzer>

<https://github.com/google/AFL>

<https://medium.com/@ayushpriya10/fuzzing-applications-with-american-fuzzy-lop-afl-54facc65d102>

<https://github.com/fuzzstati0n/fuzzgoat>

<https://github.com/google/fuzzing/blob/master/docs/afl-based-fuzzers-overview.md>

<https://peachtech.gitlab.io/peach-fuzzer-community/v3/Installation.html>
