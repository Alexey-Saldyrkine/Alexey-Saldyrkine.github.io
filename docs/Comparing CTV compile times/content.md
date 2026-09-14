# Comparing compilation times for CTS and FFI methods for storing states.

The purpose of this text is to provide the reader with an understanding of how the compilation times for the CTS and FFI methods compare relative to one another. This text will examine and compare the theoretical complexity and experimentally measured compilation times. The insertion and lookup times for the map, list, and variable for each method will be measured.

You can find the full description and explanation of the CTS and FFI methods [here]. The short version is that both methods save and retrieve key-value data pairs during compilation time. 

The Class Template Specialization (CTS) method uses class template specialization as the storage medium. The template parameters of the specialization are the key, and the value is stored as an annotation of a data member in the specialization. To retrieve a value from a key, the base template is instantiated with the key, then the annotation value is gotten from the resulting type. To check whether a given key was inserted, the base case template must be declared incomplete. Then the template, instantiated with the key, will be checked for being complete. It will be complete if and only if the key was inserted previously.

The Friend Function Injection (FFI) method uses function overloads as the storage medium. The function parameter types are the key, and the value is returned when the function is called. To store a key-value pair, a function overload is created with the key types as the function parameter types and the value as the return value. To retrieve a value, the overloaded function is called with values of types matching the key types, and the result of the call is the value. To check whether a given key was inserted, the function parameter is checked for an identifier. It will have an identifier if and only if the key was inserted previously. Details can be found [here].

The compilation times measured were from GCC. results may vary in other implementations or across compiler versions.


## CTS

The CTS method relies on template specialization to store states. This means that for the CTS map, the complexity of insertion and lookup will be equal to that of creating and instantiating a template specialization.

In GCC, all explicit template specializations are stored in a hash table, where the hash is computed from the base template and the specialization's template arguments. The hash table is global across the entire translation unit. Partial specializations are handled differently, but that is not relevant here, as the CTS method uses only explicit template specializations, not partial ones. This means that using the CTS method effectively amounts to using an inbuilt compile-time hash table. From this we can expect the complexity of inserting an element into a CTS map to be $$O\left(1\right)$$ on average and $$O\left(n\right)$$ in the worst case scenario. We should expect the same for lookup complexity. For the purposes of this paper, only the average case will be considered.

### Map

The complexity of inserting or looking up n elements in a CTS map should be $$O\left(n\right)$$, since each insertion or lookup is $$O\left(1\right)$$ and is repeated n times.

### List

When an element is pushed to the back of a CTS list, the list's size must be determined first. After which, an insertion into a CTS map is performed. A similar process is used when retrieving the last element of the list. As such, the complexity of the ‘push’ and ‘back’ functions is equal to the complexity of finding the size of the list plus the complexity of an insertion or a lookup in a CTS map. As the complexity of an insertion or a lookup for a CTS map is $$O\left(1\right)$$, then the complexity of an insertion or a lookup of the back element of a CTS list is the same as finding the list's size.

Since the list is implemented as an array, each element is stored at an index in a CTS map starting at 0, and the last element is stored at the index equal to the list's size minus one. To find the size of the list, a binary search for the smallest unused index will be performed on the interval $$\left[0,R\right]$$, where $$R$$ is the smallest power of some hint constant $$C$$ that exceeds the size of the list. At each step of the binary search, a lookup in a CTS map occurs, which has $$O\left(n\right)$$ complexity. Thus, the complexity of a binary search on the interval is $$O\left(\log_2\left(R\right)\right)*O\left(1\right)$$, equivalent to $$O\left(\log_2\left(R\right)\right)$$. 

$$O\left(\log_2\left(R\right)\right)$$ is equivalent to $$O\left(\log_2\left(m\right)\right)$$, where $$m$$ is the size of the list, as:

$$R = C^a,C^{a - 1} < m < C^a\Rightarrow$$
$$ \frac{R}{C} < m < R\Rightarrow$$
$$\log_2\left(\frac{R}{C}\right) < \log_2\left(m\right) < \log_2\left(R\right)\Rightarrow$$
$$\log_2\left(R\right) - \log_2\left(C\right) < \log_2\left(m\right) < \log_2\left(R\right)\Rightarrow$$
$$O\left(\log_2\left(R\right) - \log_2\left(C\right)\right) < O\left(\log_2\left(m\right)\right) < O\left(\log_2\left(R\right)\right),O\left(\log_2\left(C\right)\right) = 0\Rightarrow$$
$$ O\left(\log_2\left(R\right)\right) = O\left(\log_2\left(m\right)\right)$$

With this, we can say that the complexity of determining the size of the list is $$O\left(\log_2\left(m\right)\right)$$, and that the complexity of of looking up the back element $$n$$ times is $$O\left(n\log_2\left(m\right)\right)$$, equivalent to $$O\left(n\log_2\left(m\right)\right) = O\left(n\right)$$, where $$m$$ is the number of elements in the list.

WIP.

