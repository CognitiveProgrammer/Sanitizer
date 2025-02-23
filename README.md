### Memory Safe C & C++ Using Address Sanitizer & Leak Sanitizer

Memory safety is a big topic in software development, especially in the areas of real time and mission critical softwares and this is also the primary reason why people criticise C & C++ and promote memory safe languages.

Nonetheless, C and C++ continue to be prevalent and will remain so for the foreseeable future. A significant portion of the web, databases, and kernels incorporate C and C++. The simplicity, familiarity, availability, and ecosystem surrounding C and C++ is huge.

Therefore, we’ll have to write and maintain C and C++, and fortunately, there are tools available to assist us in identifying and correcting many memory-related errors, with ***sanitizers*** being one such tool.


#### Introduction to Sanitizers

Sanitizers are the tools created by Google to detect address, memory, and other errors. However, the actual code resides in the LLVM repository, and in most cases, you can either use it directly or by enabling configurations in IDEs like Visual Studio and CLion.

In this blog, we’ll understand the usage of sanitizers using the command line g++ in the debian linux environment.

There are lots of sanitizers like

*   Address Sanitizers
*   Leak Sanitizers
*   Threads Sanitizers
*   Memory Sanitizers

This blog is all about using the `Address Sanitizer` which includs the `Leak Sanitizer`

### Address Sanitizers

`Address Sanitizers` are used for memory error detector in C & C++ programming language and logs various categories of address related errors. 

#### Dangling Pointer Error (Use after free)

Let’s see a simple C++ code below

```cpp
      1 #include <iostream>
      2 using namespace std;
      3 
      4 int main() {
      5 
      6         int *temp = new int[10];
      7         delete[] temp;
      8 
      9         temp[1] = 100; // use after free
     10 
     11         return 0;
     12 }

```
In the code above, we can see that we’re using `temp[1] = 100` after `delete[] temp` 

Let’s compile the code with `Address Sanitizer` using g++

`g++ <filename>.cpp -fsanitize=address`

and when we run the `a.out` we get the following output

```css
    ./a.out 
    =================================================================
    ==25357==ERROR: AddressSanitizer: heap-use-after-free on address 0xffffa5800f54 at pc 0xaaaab4c50cd4 bp 0xffffffddd870 sp 0xffffffddd880
    WRITE of size 4 at 0xffffa5800f54 thread T0
        #0 0xaaaab4c50cd0 in main (/home/parallels/cpp/a.out+0xcd0)
        #1 0xffffa9a473f8 in __libc_start_call_main ../sysdeps/nptl/libc_start_call_main.h:58
        #2 0xffffa9a474c8 in __libc_start_main_impl ../csu/libc-start.c:392
        #3 0xaaaab4c50b6c in _start (/home/parallels/cpp/a.out+0xb6c)

    0xffffa5800f54 is located 4 bytes inside of 40-byte region [0xffffa5800f50,0xffffa5800f78)
    freed by thread T0 here:
        #0 0xffffa9f7c734 in operator delete[](void*) ../../../../src/libsanitizer/asan/asan_new_delete.cpp:163
        #1 0xaaaab4c50c78 in main (/home/parallels/cpp/a.out+0xc78)
        #2 0xffffa9a473f8 in __libc_start_call_main ../sysdeps/nptl/libc_start_call_main.h:58
        #3 0xffffa9a474c8 in __libc_start_main_impl ../csu/libc-start.c:392
        #4 0xaaaab4c50b6c in _start (/home/parallels/cpp/a.out+0xb6c)

    previously allocated by thread T0 here:
        #0 0xffffa9f7bcec in operator new[](unsigned long) ../../../../src/libsanitizer/asan/asan_new_delete.cpp:102
        #1 0xaaaab4c50c60 in main (/home/parallels/cpp/a.out+0xc60)
        #2 0xffffa9a473f8 in __libc_start_call_main ../sysdeps/nptl/libc_start_call_main.h:58
        #3 0xffffa9a474c8 in __libc_start_main_impl ../csu/libc-start.c:392
        #4 0xaaaab4c50b6c in _start (/home/parallels/cpp/a.out+0xb6c)

    SUMMARY: AddressSanitizer: heap-use-after-free (/home/parallels/cpp/a.out+0xcd0) in main
    Shadow bytes around the buggy address:
      0x200ff4b00190: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
      0x200ff4b001a0: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
      0x200ff4b001b0: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
      0x200ff4b001c0: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
      0x200ff4b001d0: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
    =>0x200ff4b001e0: fa fa fa fa fa fa fa fa fa fa[fd]fd fd fd fd fa
      0x200ff4b001f0: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
      0x200ff4b00200: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
      0x200ff4b00210: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
      0x200ff4b00220: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
      0x200ff4b00230: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
    Shadow byte legend (one shadow byte represents 8 application bytes):
      Addressable:           00
      Partially addressable: 01 02 03 04 05 06 07
      Heap left redzone:       fa
      Freed heap region:       fd
      Stack left redzone:      f1
      Stack mid redzone:       f2
      Stack right redzone:     f3
      Stack after return:      f5
      Stack use after scope:   f8
      Global redzone:          f9
      Global init order:       f6
      Poisoned by user:        f7
      Container overflow:      fc
      Array cookie:            ac
      Intra object redzone:    bb
      ASan internal:           fe
      Left alloca redzone:     ca
      Right alloca redzone:    cb
      Shadow gap:              cc
    ==25357==ABORTING

```
You have to just look into the first few lines to be able to get what this error is all about

```css
    ==25357==ERROR: AddressSanitizer: heap-use-after-free on address 0xffffa5800f54 at pc 0xaaaab4c50cd4 bp 0xffffffddd870 sp 0xffffffddd880
    WRITE of size 4 at 0xffffa5800f54 thread T0
        #0 0xaaaab4c50cd0 in main (/home/parallels/cpp/a.out+0xcd0)

```
you can see ***heap-use-after-free ***at a particular address.

However, in the whole error message you can’t see the line number at which the error has occurred and this is fine for a few lines of code as shown above but will not work for real world work items. 

So to make sure that we get the line number associated with the error must be reported we have to compile the code with `-g` options 

`g++ <filename>.cpp -fsanitize=address -g`

and here is the output we’ll get after this

```css
    ./a.out 
    =================================================================
    ==27168==ERROR: AddressSanitizer: heap-use-after-free on address 0xffff8f900f54 at pc 0xaaaad7a90cd4 bp 0xffffe77735c0 sp 0xffffe77735d0
    WRITE of size 4 at 0xffff8f900f54 thread T0
        #0 0xaaaad7a90cd0 in main /home/parallels/cpp/1.cpp:9
        #1 0xffff93b573f8 in __libc_start_call_main ../sysdeps/nptl/libc_start_call_main.h:58
        #2 0xffff93b574c8 in __libc_start_main_impl ../csu/libc-start.c:392
        #3 0xaaaad7a90b6c in _start (/home/parallels/cpp/a.out+0xb6c)
    ....................truncated

```

***

you can see the source line number which is causing this error to occur. 

***

#### Stack & Heap Buffer Overflow

One of the biggest issue of C and C++ programs are bounds checking of the static and dynamic buffers. To prevent this we need to do costly bounds check each and every time we’re accessing the buffer. 

With `Address Sanitizer` , we can see the stack and help buffer overflow. 

Let’s see the code snippet below

```cpp
    int main() {

        int temp[10];
        temp[10] = 100;

        return 0;
    }

```
And the address sanitizer correctly provides the `stack buffer overflow` issue as depicted the result snippet below

```css
    ./a.out 
    =================================================================
    ==35178==ERROR: AddressSanitizer: stack-buffer-overflow on address 0xffffd6e505b8 at pc 0xaaaadb4d0e48 bp 0xffffd6e50520 sp 0xffffd6e50530
    WRITE of size 4 at 0xffffd6e505b8 thread T0
        #0 0xaaaadb4d0e44 in main /home/parallels/cpp/3.cpp:7
        #1 0xffffa94673f8 in __libc_start_call_main ../sysdeps/nptl/libc_start_call_main.h:58
        #2 0xffffa94674c8 in __libc_start_main_impl ../csu/libc-start.c:392
        #3 0xaaaadb4d0c6c in _start (/home/parallels/cpp/a.out+0xc6c)
    ....................truncated

```

Similarly, if we run the code snippet below

```cpp
    int main() {

        int *temp = new int[10];
        temp[10] = 100;

        return 0;
    }

```
We’ll correctly get the `heap buffer overflow` from the address sanitizer

```css
    ==35672==ERROR: AddressSanitizer: heap-buffer-overflow on address 0xffffb0200f78 at pc 0xaaaac6340c80 bp 0xffffd9510660 sp 0xffffd9510670
    WRITE of size 4 at 0xffffb0200f78 thread T0
        #0 0xaaaac6340c7c in main /home/parallels/cpp/2.cpp:7
        #1 0xffffb44073f8 in __libc_start_call_main ../sysdeps/nptl/libc_start_call_main.h:58
        #2 0xffffb44074c8 in __libc_start_main_impl ../csu/libc-start.c:392
        #3 0xaaaac6340b2c in _start (/home/parallels/cpp/a.out+0xb2c)
    ....................truncated

```

#### What about C++ STL code ?

Modern C++ is incomplete without STL and if sanitizers are not working for STL, then its likely be unusable for more of the modern C++ code. 

Fortunately we’ll get both stack and heap buffer overflows for STL also. 

Let’s see a simple STL code of `std::array<>` as written below where we have the issue of stack buffer overflow

```cpp
    #include<vector>
    #include<array>
    using namespace std;

    int main() {
            array<int, 3> v1 = { 1, 2, 3};

            v1[3] = 200;  // problem
            return 0;
    }

```
When we compile and run the same, we got the same `stack buffer issue` by using address sanitizer as reported below

```css
    =================================================================
    ==36017==ERROR: AddressSanitizer: stack-buffer-overflow on address 0xffffc20fe63c at pc 0xaaaab05a0c08 bp 0xffffc20fe5c0 sp 0xffffc20fe5d0
    WRITE of size 4 at 0xffffc20fe63c thread T0
        #0 0xaaaab05a0c04 in main /home/parallels/cpp/astl.cpp:7
        #1 0xffff90d573f8 in __libc_start_call_main ../sysdeps/nptl/libc_start_call_main.h:58
        #2 0xffff90d574c8 in __libc_start_main_impl ../csu/libc-start.c:392
        #3 0xaaaab05a096c in _start (/home/parallels/cpp/a.out+0x96c)
    ....................truncated

```

Let’s see another set of code with `std::vector<>` where we’re trying to do buffer overrun.

```cpp
    #include<vector>
    using namespace std;

    int main() {
        vector<int> v1 = { 1, 2, 3}; 

        v1[3] = 200; // problem
        return 0;
    }

```

and this is reported as `heap buffer overflow` by the address sanitizer as 

```css
    ==37544==ERROR: AddressSanitizer: heap-buffer-overflow on address 0xffff8a0007bc at pc 0xaaaada7412d4 bp 0xffffe2e79e10 sp 0xffffe2e79e20
    WRITE of size 4 at 0xffff8a0007bc thread T0
        #0 0xaaaada7412d0 in main /home/parallels/cpp/stl.cpp:7
        #1 0xffff8e1e73f8 in __libc_start_call_main ../sysdeps/nptl/libc_start_call_main.h:58
        #2 0xffff8e1e74c8 in __libc_start_main_impl ../csu/libc-start.c:392
        #3 0xaaaada740fac in _start (/home/parallels/cpp/a.out+0xfac)
    ....................truncated

```

#### It Works with STL, but do consider this ..

The heap memory allocated by the STL containers like `vectors` are not linear based on the usage in the code. To achieve what we call it as amortized constant time, vector usage mathematical factor to increase its size

> A vector when reached its limit increases the size of vector by 1.5x — 2x depending upon the compiler

So whether the sanitizer can catch heap buffer overflow or not is based on the `vector capacity` and not on the current`vector size` 

As an example, let’s have a look at the code below

```cpp
    #include <iostream>
    #include <vector>
    using namespace std;

    int main() {
        vector<int> v1; 
        v1.push_back(10);
        v1.push_back(20);
        v1.push_back(30);

        cout<<"Vector Capacity = "<<v1.capacity()<<endl; // capcaity = 4
        cout<<"Vector size = "<<v1.size()<<endl;  // size = 3

        v1[3] = 200; // no error, because of the capacity

        return 0;
    }

```

Even though we’re not using `v1[3]`, it will not raise any issue because the capacity of the `vector` is still `4` . 

However, if we change the code and try to access the `v1[4]` , we’ll get the `heap buffer overflow` error.

#### Why this is an important factor?

> STL containers memory is not allocated linearly and is based on the usage of the container. Hence, to make it reliable you need to unit test with all possible ranges so that any potential issue can be caught by the sanitizer

#### Global Buffer Overflow

Similar to stack and heap buffer overflow, there is something called `global buffer overflow` and as the name suggest, it reports any global buffer overflow allocated statically.

if the global variables is allocated dynamically, then even for the global variable the reported error will be `heap buffer overflow` 

Let’s see a simple C++ example below

```cpp
    int global_array[10] = {-1}; 

    int main() {
        global_array[10] = 100;

        return 0;
    }

```

And the reported output will be 

```css
    ==74900==ERROR: AddressSanitizer: global-buffer-overflow on address 0xaaaad75a21b0 at pc 0xaaaad7590ca8 bp 0xffffe1139cc0 sp 0xffffe1139cd0
    WRITE of size 4 at 0xaaaad75a21b0 thread T0
        #0 0xaaaad7590ca4 in main /home/parallels/cpp/glob.cpp:7
        #1 0xffffb95673f8 in __libc_start_call_main ../sysdeps/nptl/libc_start_call_main.h:58
        #2 0xffffb95674c8 in __libc_start_main_impl ../csu/libc-start.c:392
        #3 0xaaaad7590b6c in _start (/home/parallels/cpp/a.out+0xb6c)
    ....................truncated

```

#### The Leak Sanitizer

Memory leakage in C & C++ programs were the primary trigger for the creation of things like garbage collection. This is where the leak sanitizer comes into picture

> The leak sanitize identifies the memory leaks within the code.

Let’s see a simple example of a code which has `new[]` but has no corresponding `delete[]` 

```cpp
    int main() {
        int *numPtr = new int[100];
        numPtr = nullptr;
        return 0;
    }

```

When we compile the code as 

`g++ leak.cpp -fsanitize=address -g`

and run the code, here is the output we get

```css
    =================================================================
    ==3400==ERROR: LeakSanitizer: detected memory leaks

    Direct leak of 400 byte(s) in 1 object(s) allocated from:
        #0 0xffffa4b5bcec in operator new[](unsigned long) ../../../../src/libsanitizer/asan/asan_new_delete.cpp:102
        #1 0xaaaac3b40be0 in main /home/parallels/cpp/leak.cpp:5
        #2 0xffffa46273f8 in __libc_start_call_main ../sysdeps/nptl/libc_start_call_main.h:58
        #3 0xffffa46274c8 in __libc_start_main_impl ../csu/libc-start.c:392
        #4 0xaaaac3b40aec in _start (/home/parallels/cpp/a.out+0xaec)

    SUMMARY: AddressSanitizer: 400 byte(s) leaked in 1 allocation(s).

```
it clearly identifies the leaks happened for a particular variable. 


I do hope and believe that this will help you to write address and memory issues free C & C++ code.

Thanks for visting my github page and reading about the sanitizers.

Daksh
