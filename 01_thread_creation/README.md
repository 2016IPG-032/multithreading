
# C++ Thread Creation

This repository contains examples of how to create and manage threads in C++, from the older C++11 standard up to modern C++20 features.

## 1. The Absolute Basics: Function Pointers
The simplest way to create a thread in C++ (since C++11) is using the `std::thread` class from the `<thread>` header and passing a standard function pointer to it.

### Code Example: `01_basic_function.cpp`
```cpp
#include <iostream>
#include <thread>

void printMessage() {
    std::cout << "Hello from a basic thread! Thread ID: " 
              << std::this_thread::get_id() << "\n";
}

int main() {
    std::cout << "Main thread ID: " << std::this_thread::get_id() << "\n";

    // Create and start the thread
    std::thread t1(printMessage);

    // Wait for the thread to finish before exiting main
    t1.join(); 

    std::cout << "Thread finished. Back to main.\n";
    return 0;
}
```
Key Concept: join() vs detach()
join(): The main thread halts and waits until t1 completely finishes its execution. If you don't call join() or detach() before the std::thread object is destroyed, your program will crash (std::terminate).

detach(): Separates the thread from the std::thread object, allowing it to execute completely independently in the background.

## 2.Intermediate: Passing Arguments & Lambda Functions

In real-world applications, you usually need to pass data to a thread. C++ threads accept arguments easily, and modern C++ frequently uses Lambdas (anonymous functions) inline.

Code Example: 02_lambdas_and_args.cpp
```cpp
#include <iostream>
#include <thread>
#include <string>

void greetUser(std::string name, int age) {
    std::cout << "Hello " << name << ", you are " << age << " years old.\n";
}

int main() {
    std::string user = "Adeep";
    
    // 1. Passing arguments to a standard function
    std::thread t1(greetUser, user, 25);

    // 2. Creating a thread using an inline Lambda function
    std::thread t2([](const std::string& task, int duration) {
        std::cout << "Starting task: " << task << " for " << duration << "s\n";
    }, "Data Processing", 5);

    t1.join();
    t2.join();
    return 0;
}
```

Crucial Rule: Arguments passed to std::thread are copied by default into internal storage accessible by the new thread. If you want to pass a value by reference, you must wrap it in std::ref(variable).

##3. Advanced: Member Functions & Functors
Sometimes your thread needs to execute a method inside a specific class instance, or an object acting like a function (a functor).

Code Example: 03_member_functions.cpp

```cpp
#include <iostream>
#include <thread>

class BackgroundWorker {
public:
    void doWork(int id) {
        std::cout << "Worker " << id << " is processing data on a class instance method.\n";
    }
};

int main() {
    BackgroundWorker worker;
    
    // To execute a member function, pass:
    // 1. The address of the member function (&ClassName::MethodName)
    // 2. The pointer/address of the object instance (&worker)
    // 3. Any arguments the function requires
    std::thread t1(&BackgroundWorker::doWork, &worker, 42);

    t1.join();
    return 0;
}
```
4. Modern C++ (C++20): std::jthread
Manually calling .join() everywhere can lead to bugs. If an exception occurs before .join() is called, your program crashes. C++20 introduced std::jthread (Joining Thread).

It automatically calls .join() in its destructor when it goes out of scope.

It introduces built-in cooperative cancellation via stop tokens.

##Code Example: 04_modern_jthread.cpp

```cpp
#include <iostream>
#include <thread>
#include <chrono>

int main() {
    // std::jthread automatically joins when main finishes! No .join() required.
    std::jthread jt1([](std::stop_token token) {
        while (!token.stop_requested()) {
            std::cout << "Working safely...\n";
            std::this_thread::sleep_for(std::chrono::milliseconds(500));
        }
        std::cout << "Stop requested, exiting thread gracefully.\n";
    });

    // Let it run for 1.5 seconds
    std::this_thread::sleep_for(std::chrono::seconds(2));
    
    // jt1 destructor automatically requests stop and joins here
    return 0;
}
```