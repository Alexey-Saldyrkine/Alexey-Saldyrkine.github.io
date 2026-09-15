# Comparing compilation times for CTS and FFI methods for storing states.

The purpose of this text is to provide the reader with an understanding of how the compilation times for the CTS and FFI methods compare relative to one another. This text will examine and compare the theoretical complexity and experimentally measured compilation times. The insertion and lookup times for the map, list, and variable for each method will be measured.

You can find the full description and explanation of the CTS and FFI methods [here]. The short version is that both methods save and retrieve key-value data pairs during compilation time. 

The Class Template Specialization (CTS) method uses class template specialization as the storage medium. The template parameters of the specialization are the key, and the value is stored as an annotation of a data member in the specialization. To retrieve a value from a key, the base template is instantiated with the key, then the annotation value is gotten from the resulting type. To check whether a given key was inserted, the base case template must be declared incomplete. Then the template, instantiated with the key, will be checked for being complete. It will be complete if and only if the key was inserted previously.

The Friend Function Injection (FFI) method uses function overloads as the storage medium. The function parameter types are the key, and the value is returned when the function is called. To store a key-value pair, a function overload is created with the key types as the function parameter types and the value as the return value. To retrieve a value, the overloaded function is called with values of types matching the key types, and the result of the call is the value. To check whether a given key was inserted, the function parameter is checked for an identifier. It will have an identifier if and only if the key was inserted previously. Details can be found [here].

The compilation times measured were from GCC 16. results may vary in other implementations or across compiler versions.

## Theoretical expectations

### CTS

The CTS method relies on template specialization to store states. This means that for the CTS map, the complexity of insertion and lookup will be equal to that of creating and instantiating a template specialization.

In GCC, all explicit template specializations are stored in a hash table, where the hash is computed from the base template and the specialization's template arguments. The hash table is global across the entire translation unit. Partial specializations are handled differently, but that is not relevant here, as the CTS method uses only explicit template specializations, not partial ones. This means that using the CTS method effectively amounts to using an inbuilt compile-time hash table. From this we can expect the complexity of inserting an element into a CTS map to be $$O\left(1\right)$$ on average and $$O\left(n\right)$$ in the worst case scenario. We should expect the same for lookup complexity. For the purposes of this paper, only the average case will be considered.

#### Map

The complexity of inserting or looking up n elements in a CTS map should be $$O\left(n\right)$$, since each insertion or lookup is $$O\left(1\right)$$ and is repeated n times.

#### List

When an element is pushed to the back of a CTS list, the list's size must be determined first. After which, an insertion into a CTS map is performed. A similar process is used when retrieving the last element of the list. As such, the complexity of the ‘push’ and ‘back’ functions is equal to the complexity of finding the size of the list plus the complexity of an insertion or a lookup in a CTS map. As the complexity of an insertion or a lookup for a CTS map is $$O\left(1\right)$$, then the complexity of an insertion or a lookup of the back element of a CTS list is the same as finding the list's size.

Since the list is implemented as an array, each element is stored at an index in a CTS map starting at 0, and the last element is stored at the index equal to the list's size minus one. To find the size of the list, a binary search for the smallest unused index will be performed on the interval $$\left[0,R\right]$$, where $$R$$ is the smallest power of some hint constant $$C$$ that exceeds the size of the list. At each step of the binary search, a lookup in a CTS map occurs, which has $$O\left(n\right)$$ complexity. Thus, the complexity of a binary search on the interval is $$O\left(\log_2\left(R\right)\right)*O\left(1\right)$$, equivalent to $$O\left(\log_2\left(R\right)\right)$$. 

$$O\left(\log_2\left(R\right)\right)$$ is equivalent to $$O\left(\log_2\left(m\right)\right)$$, where $$m$$ is the size of the list, as:

$$R = C^a,C^{a - 1} < m < C^a\Rightarrow$$
$$ \frac{R}{C} < m < R\Rightarrow$$
$$\log_2\left(\frac{R}{C}\right) < \log_2\left(m\right) < \log_2\left(R\right)\Rightarrow$$
$$\log_2\left(R\right) - \log_2\left(C\right) < \log_2\left(m\right) < \log_2\left(R\right)\Rightarrow$$
$$O\left(\log_2\left(R\right) - \log_2\left(C\right)\right) < O\left(\log_2\left(m\right)\right) < O\left(\log_2\left(R\right)\right),O\left(\log_2\left(C\right)\right) = 0\Rightarrow$$
$$ O\left(\log_2\left(R\right)\right) = O\left(\log_2\left(m\right)\right)$$

With this, we can say that the complexity of determining the size of the list is $$O\left(\log_2\left(m\right)\right)$$, and that the complexity of of looking up the back element $$n$$ times is $$O\left(n\log_2\left(m\right)\right) = O\left(n\right)$$, where $$m$$ is the number of elements in the list.

Each time an element is pushed to the back of the list, the size of the list before insertion must be found. The complexity of finding size is $$O\left(\log_2\left(m\right)\right)$$. If $$n$$ elements are inserted into an empty list, each insertion will increase $$m$$ by one. This means that the complexity of pushing back n elements will be equal to $$O\left(\sum_{i = 1}^n\log_2\left(i\right)\right) = O\left(\log_2\left(n!\right)\right)$$. This can be approximated using Stirling’s approximation to $$O\left(n\log_2\left(n\right) - n\log_2\left(e\right) + \frac{1}{2}\log_2\left(2\pi n\right)\right)$$, which is equivalent to $$O\left(n\log_2\left(n\right)\right)$$.

When getting an element from a TCS list by index, you are performing a lookup in a TCS map with a complexity of $$O\left(1\right)$$. So, the complexity of getting $$n$$ elements from a CTS list is $$O\left(n\right)$$.​

To summarize, the complexity of pushing $$n$$ elements into an empty CTS list $$O\left(n\log_2\left(n\right)\right)$$. The complexity of looking up the back element $$n$$ times is $$O\left(n\right)$$. And the complexity of retrieving $$n$$ elements by index from a list is $$O\left(n\right)$$.

#### Variable 

Since the TCS variable is a wrapper around a TCS list, its complexity is the same. Assigning a value to a TCS variable is the same as pushing a value onto the end of a TCS list and has complexity $$O\left(\log_2\left(m\right)\right)$$. Setting the value $$n$$ times has complexity $$O\left(n\log_2\left(n\right)\right)$$. Getting the value of the variable has complexity equal to that of retrieving the last element of the list, i.e., $$O\left(\log_2\left(m\right)\right)$$. Getting the value n times will have complexity $$O\left(n\log_2\left(m\right)\right) = O\left(n\right)$$.

### FFI
The FFI method relies on creating function overloads using friend functions. This means that, for the FFI map, the complexity of insertion and lookup will be equal to that of adding and resolving a function overload.

In GCC, a scope contains a number of code elements. Declarations with the same name, such as function overloads, are stored in a list, and a scope can have multiple of these lists. When a new hidden friend overload is declared, which the FFI method does, the scope’s declaration lists are searched linearly for a list with a matching identifier. Since the number of lists does not increase with the amount of overloads for a single function, this process has $$O\left(1\right)$$ complexity effectively. Once the list is found, the compiler must ensure that the one-definition rule won't be violated if the new overload is added. To do this, the compiler must verify that no existing overload is redefined by the new overload. This process takes $$O\left(m\right)$$ time, where $$m$$ is the number of existing overloads. This means that the FFI insertion complexity is $$O\left(1\right) + O\left(m\right) = O\left(m\right)$$.

Overload resolution is a complex topic, but because the FFI method only adds non-template functions to a single scope, the process is a bit simpler. Overload resolution has three phases: creating the set of candidate functions, trimming the set to only viable functions, and finally, ranking the functions. 

The complexity of gathering candidate functions is $$O\left(1\right)$$ relative to the number of added overload functions, since the FFI method creates no additional scopes when an overload is added. 

The trimming phase is simple, as all the overloads have a single parameter. This phase has complexity $$O\left(m\right)$$, since each added overload must be checked.

The final phase is the most complex one. But in this case, it will be a simple linear search for an overload that matches the parameter type. It will have a complexity of $$O\left(m\right)$$.

As such, overload resolution and FFI lookup have $$O\left(m\right)$$ complexity, where $$m$$ is the number of overloads previously added.

​To check whether a key was inserted into an FFI map, a function overload resolution will occur. As such, it will have complexity $$O\left(m\right)$$. Checking $$n$$ times will have the complexity of $$O\left(n*m\right)=O\left(n\right)$$.

To summarize, FFI insertion has complexity $$O\left(m\right)$$, FFI lookup has complexity $$O\left(m\right)$$, and FFI check has complexity $$O\left(m\right)$$.

#### Map

We should expect the complexity of inserting n elements into an empty FFI map to be $$O\left(\sum_{i = 1}^ni\right) = O\left(\frac{n\left(n + 1\right)}{2}\right) = O\left(n^2\right)$$.

The complexity of looking up n elements by index will be $$O\left(n*m\right)=O\left(n\right)$$, where $$m$$ is the amount of elements previously inserted into all FFI maps.

The complexity of cheking if a FFI map contains an element $$n$$ times is $$O\left(n*m\right)=O\left(n\right)$$.

#### List

The FFI list works the same way as the CTS list, but utilizes an FFI map instead of a CTS map.

The complexity of getting the size of the list will be $$O\left(m\log_2\left(m\right)\right)$$ compared to CTS list’s $$O\left(\log_2\left(m\right)\right)$$, as FFI map lookup is $$O\left(m\right)$$ instead of $$O\left(m\right)$$.

Each time an element is pushed back, the list size is determined, and an FFI map insertion occurs. The complexity of pushing a single element is $$O\left(m + m\log_2\left(m\right)\right)$$, where $$m$$ is the size of the list. From this, we can say that the complexity of inserting $$n$$ elements into an empty FFI list is $$O\left(\sum_{i = 1}^n\left(i + i\log_2\left(i\right)\right)\right)$$, which can be approximated[note] to $$O\left(\frac{n^2}{2}\log_2\left(n\right) - \frac{n^2}{4} + \frac{n}{2}\log_2\left(n\right) + \frac{n(n + 1)}{2}\right) = O\left(n^2\log_2\left(n\right)\right)$$

The complexity of looking up the back element $$n$$ times is $$O\left(n*m\log_2\left(m\right)\right) = O\left(n\right)$$, where $$m$$ is the amount of elements inserted in all FFI maps.

When getting an element from a FFI list by index, you are performing a lookup in a FFI map with a complexity of $$O\left(m\right)$$. So, the complexity of getting $$n$$ elements from a CTS list is $$O\left(n*m\right)=O\left(n\right)$$.​

​In summary, the complexity of inserting $$n$$ elements into an empty FFI list is $$O\left(n^2\log_2\left(n\right)\right)$$. The complexity of looking up the back element $$n$$ times is $$O\left(n\right)$$. The complexity of looking up n elements by index is $$O\left(n\right)$$.

#### Variant

As the FFI variable is a wrapper around an FFI list, its complexity is the same.

Assigning a value to a FFI variable is the same as pushing a value to the back of a FFI list and has a complexity of $$O\left(m\log_2\left(m\right)\right)$$. Setting an empty variable’s value $$n$$ times will have a complexity of $$O\left(n^2\log_2\left(n\right)\right)$$.

Getting the value of the variable has complexity equal to that of retrieving the last element of the list, i.e., $$O\left(m\log_2\left(m\right)\right)$$. Getting the value $$n$$ times will have a complexity of $$O\left(n*m\log_2\left(m\right)\right) = O\left(n\right)$$.



### Summary table
#### Expected complexities for performing the operation n times 
|Operation|TCS|FFI|
|:---:|:---:|:---:|
|Map Insert|$$O\left(n\right)$$|$$O\left(n^2\right)$$|
|Map Lookup|$$O\left(n\right)$$|$$O\left(n*m\right)=O\left(n\right)$$|
|Map Contains|$$O\left(n\right)$$|$$O\left(n*m\right)=O\left(n\right)$$|
|List Push|$$O\left(n\log_2\left(n\right)\right)$$|$$O\left(n^2\log_2\left(n\right)\right)$$|
|List Back|$$O\left(n*\log_2\left(m\right)\right)=O\left(n\right)$$|$$O\left(n*m\log_2\left(m\right)\right)=O\left(n\right)$$|
|List Get by Index|$$O\left(n\right)$$|$$O\left(n*m\right)=O\left(n\right)$$|
|Variable Set|$$O\left(n\log_2\left(n\right)\right)$$|$$O\left(n^2\log_2\left(n\right)\right)$$|
|Variable Get|$$O\left(n*\log_2\left(m\right)\right)=O\left(n\right)$$|$$O\left(n*m\log_2\left(m\right)\right) = O\left(n\right)$$|

## Experimental data 

The goal of measuring compilation times is to test whether theoretical expectations align with experimental data. There are 5 operations for each method: map insert, map lookup, list push, list back, and list lookup, which will be tested. The functions for the variable won't be tested, as they are essentially identical to the list's functions, so all results for the list apply to the variable. A visual method will be used to decide whether the expected complexity is accurate for an operation. The resulting compilation times will be mapped on a graph. On the graph, the x-axis will be the number of times an operation was performed. The y-axis will be the compilation time to perform x operations. A function of the expected complexity will be fitted to the data. The expected complexity will be considered accurate if the resulting data points fit the resulting function. This way, if we expect an operation to have O(n) complexity, the data points should lie on a straight line. 

### Methodology

All data was gathered from measuring the compilation time of the GCC C++ compiler version 16.1 on a Linux machine.

As compilation is a deterministic process, there should be a minimum perfect time that a compilation takes. Every measured time will be the sum of this perfect time and some time added due to interference. This interference comes from non-deterministic sources, such as being scheduled on a core with other processes, being interrupted, page faults, and other hardware and system delays. Since I wanted to get as close to this perfect time as possible, I tried to minimize outside interference by isolating the cores that would run the compiler, so that no other processes would be scheduled on those cores. I also set each compilation to run on its own core so that they would not interfere with each other directly. I also measured only the time the compilation spent in user mode to minimize system interference. 

Even with this, compiling a program with the same parameters multiple times will result in different compilation times. So to get as close to the perfect value as possible, the minimal time will be counted as the compilation time for the set of parameters. I arbitrarily decided that the minimum of K compilations is good enough. This means that each data point represents the minimum time obtained by independently compiling the same program with the same parameters K times.

For gathering the experimental data, two types of programs were used: the master and test programs. The master program will launch multiple compilations of test programs in parallel and record their compilation times. The compilation will be assigned to a core using the ‘taskset’ program, and compilation times will be measured using the ‘/bin/time’ command with millisecond precision and measuring only time in user space. Each test program will have several parameters, specified as predefined macros via the -D flag. The most important of these parameters will be x, the number of times the tested operation will be performed. The x parameter will range from 0 to n with a given step between compilations of the same test. For the lookup and contains tests, a second parameter, m, will be provided. This parameter will be the number of insertion or push operations performed before the tested operation begins. The parameter m will not change between compilations of the same test. Since only the x parameter will change between compilations of the same test, the resulting data for the test will show only differences in compilation times caused by the tested operation. 









WIP.

