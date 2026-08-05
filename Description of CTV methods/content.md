# Methods To Achieve Compile-Time Variables In C++

By Alexey Saldyrkine

Table of contents
- [What is a Compile-Time Variable](#What-is-a-Compile-Time-Variable)
- [Methods overview](#Methods-overview)
- [How to make a variable using a map](#How-to-make-a-variable-using-a-map)
- [C++26 and reflection](#C++26-and-reflection)
- [Detailed explanation](#Detailed-explanation)
	- [CTS](#CTS)
	- [FFI](#FFI)
	- [Map](#Map)
	- [List](#List)
	- [Variable](#Variable)
- [Usage examples](#Usage-examples)
- [Bibliography](#Bibliography)


The following is a detailed description and explanation of how to create and use maps, lists, and variables at compile time in C++. Familiarity with C++26 reflection is recommended, but a brief description of the reflection functions used will be provided.

## What is a Compile-Time Variable

A compile-time variable is a constexpr variable that acts like a normal run-time variable. Its value can be changed throughout the compilation phase. This differs from normal constexpr variables, whose value can’t be changed, and run-time variables, which aren’t constant expressions and can’t be used during compilation.

## Methods overview

To make a constant expression act like a variable, you need the ability to store a state and retrieve it during compile time. In this case, a state is the value of the variable. Two methods for storing state will be described: the template-class specialization method and the friend-function injection method. Both methods act like key-value maps, allowing you to store and retrieve values by key.

The class template specialization (CTS) method uses a template class specialization to store states. Where the template parameter of the class specialization is the key, and the value is within the definition of the specialization.

​The friend function injection (FFI) method uses friend-function overloading to store a state. The function parameter's data type is the key, and the value is stored in the function body as the return value.

Both of these methods can be used to set a key-value pair, retrieve a value by key, and test whether a given key is set. An important thing to note is that these methods can only add new key-value pairs. They can’t remove existing key-value pairs.

## How to make a variable using a map

Since states can only be added and not removed, the variable must have a way to find and refer to its latest set value. This can be done by using an append-only list. When the variable is reassigned, the new value is pushed to the end of the list. This way, the variable's current value will be the back element of the list.

The list can be made from a key-value map. The list can be thought of as an infinite-indexed array, where each index is a key in the map. So the first element will have an index of 0 and will be stored in the map at key 0, and the last element in a list of size N will have an index of N-1 and will be stored at key N-1. The size or index of the last element cannot be stored directly, as that would require a compile-time variable. Because the index of the last element cannot be stored, it must be found each time it’s needed. Since we can check whether any given key is set, we can perform a binary search to find the smallest not-set key in the interval [0,R], where R is a not-set integer key bigger than zero. The result will be the size of the list. The index of the last element is the size of the list minus 1.

The map can be implemented using the CTS and FFI methods to store and retrieve key-value pairs.

## C++26 and reflection

C++26 introduced static reflection to the language. Reflection allows a program to introspect itself and automatically generate code during compilation.

Prior to C++26, the CTS and FFI methods were available, but they were inconvenient. But with C++26, especially with the introduction of reflection, the previously explicit and verbose operations can be obfuscated behind functions, giving them a similar look and feel to normal run-time maps, lists, and variables.

The following is a list and brief description of the reflection functions and features used by the methods.

meta\:\:info. meta\:\:info is the type of a reflection value.

^^. The splice operator (^^) is used to get the reflection of a given language construct. It is important to note that if the splice operator is used on a variable Var with the value Val, the resulting reflection will be of Var, not Val. To obtain Val's reflection from Var, the meta\:\:reflect_constant function is used.

[\:\:]. The splice specifier ([\:expr\:]) is used to get a language construct from its reflection, effectively copying (splicing) it into the source code.

meta\:\:reflect_constant. The meta\:\:reflect_constant function produces a reflection of the result from evaluating a given expression. If a variable Var with the value Val is given, Var will evaluate to Val and be reflected.

meta\:\:substitute. The meta\:\:substitute function substitutes a given set of template arguments into a template. Given a reflection of a template ‘Templ’ and a range of reflections of template arguments ‘Args’ that are valid for the template, the meta\:\:substitute will return the reflection of the type ‘Templ\<Args…\>’.

met\:\:extract. The meta\:\:extract function returns a reflection of the value represented by a given reflection. If a reflection of a variable Var with the value Val is given, then meta\:\:extract returns a reflection of Val.

meta\:\:current_function. The meta\:\:current_function function returns the reflection of the function whose scope is the smallest enclosing function scope at its call site.

meta\:\:parameters_of. The meta\:\:parameters_of function is used to get the reflections of a given function’s parameters. Its result is a std\:\:vector\<meta\:\:info\> of function parameter reflections.

meta\:\:has_identifier. The meta\:\:has_identifier function checks whether a given reflected entity has an identifier. In this case, it is used to check whether a given reflection representing a function parameter has an identifier.

meta\:\:data_member_spec. The meta\:\:data_member_spec function returns a data member description used by the meta\:\:define_aggregate function. It accepts a reflection of a type that will be the data members' type, along with a set of data member options. Only the annotations option will be used. This option will provide a list of reflections to populate the data members' annotation values.

meta\:\:define_aggregate. The meta\:\:define_aggregate function is used to complete a given incomplete class. It accepts a reflection of an incomplete class and a range of data member descriptions. In this case, since only one descriptor will be provided, the completed type will have a single data member.

meta\:\:is_complete_type. The meta\:\:is_complete_type function is used to check whether a given reflected class is a complete class.

Annotations. Annotations are a new way to attach information to declarations that reflection can observe. They allow attaching user-defined metadata to code declarations and retrieving it at compile-time. Annotations can be thought of as attributes that can have any reflectable value and have that value be accessible during compilation.

## Detailed explanation
 
As the two methods are used to create key-value maps, they will be described using three free functions: set, get, and check. The ‘set’ function will store a key-value pair. The ‘get’ function will retrieve the value for a given key. The ‘check’ function checks whether a given key is set. The key and value parameter types will be ‘meta::info’. This allows using any reflectable value as the key or value without having to template the free functions. Additionally, the functions will accept the parameter ‘tag’ of type meta\:\:info. The tag is a unique reflection value that will identify different maps. The tag acts like a second key, allowing different maps to use the same main keys without overlap.

### CTS

The template class specialization method will create template specializations of a base template class to store key-value pairs. To save a key-value pair, a specialization with the tag and the key as template parameters will be created, and the value will be stored as an annotation on a specialization’s data member. To retrieve a value, the base template class is instantiated with the tag and key as template parameters. To check whether a tag and key were set, the instantiated class will be checked to see if it is a complete class.

The base template class (BTC) must be declared incomplete. The BTC template will need to accept 2 non-type template parameters of type ‘meta\:\:info’ or ‘auto’. The BTC acts as the storage medium, with its specializations storing the value, and the tag and key are the specializations' template parameters. If two maps use different TBCs, they will never overlap, even if they use the same tag and key. The unique tags prevent two maps from overlapping if they use the same TBC and keys.

#### Set

The ‘set’ function will create a template specialization of the BTC, using the tag and key as template parameters, and store the value as an annotation on a data member of the specialization. This function will accept the tag, key, value, and BTC reflections as the function parameters.

```cpp
consteval void CTS_set(meta::info tag, meta::info key, meta::info value, meta::info TBC){
   auto refl = meta::substitute(TBC,{meta::reflect_constant(tag),meta::reflect_constant(key)});
  meta::define_aggregate(refl,{data_member_spec(^^char,{.name="dm",.annotations={value}})});
}
```

The ‘meta\:\:substitute’ function creates a reflection of TBC with the key as its template parameter. Importantly, this function does not cause the template class to be instantiated. The tag and key are reflected an additional time, so that the specialization will have non-type template parameters of type meta\:\:info, rather than the type of the value that key and tag reflect.
The ‘meta\:\:define_aggregate’ function defines a class. Specifically, in this case, it defines a template class specialization of TBC with the tag and key as the template parameter. It is equivalent to writing:

```cpp
template<>
struct TBC<tag,key>{
	[[=value]]   
	char dm;
};

```

Here, the template class specialization is defined with one data member named ‘dm’. This data member is declared with an annotation of the value. This way, the value is stored as an annotation of the data member of the template class specialization.

#### Get
The ‘get’ function will return a reflection of a stored value. This function will accept the tag, key, and TBC reflections as the function parameters. To get a stored value, a helper variable template ‘CTS_get_helper’ will be used.

```cpp
template<meta::info r>
constexpr meta::info CTS_get_helper = ^^[:r:]::dm;


consteval meta::info CTS_get(meta::info tag, meta::info key, meta::info TBC){
   auto refl = meta::substitute(TBC,{meta::reflect_constant(tag),meta::reflect_constant(key)});
   meta::info dm_refl = meta::extract<meta::info>(meta::substitute(^^CTS_get_helper,{meta::reflect_constant(refl)}));
   meta::info ret = meta::constant_of(meta::annotations_of(dm_refl)[0]);
   return ret;
}
```
First, the function returns the specialization of TBC reflected, with the tag and key as template parameters, just like the ‘set’ function. It then creates a reflection of ‘CTS_get_helper’ with the specialization reflection as the template parameter, using the ‘meta::substitute’ function. As with template class specializations, this won't instantiate the variable template. The variable template will be equal to the reflection of the data member ‘dm’ of the given class reflection. The ‘meta::extract’ function is used to instantiate the variable template and retrieve its value. Finally, the function ‘meta::annotations_of’ is used to get a vector of annotations of ‘dm’. Since ‘dm’ is created with the value as its only annotation, the annotation of the value can be gotten at index 0. To get the value from the annotation, the function ‘meta::constant_of’ is used.

#### Check

The ‘check’ function will return whether a given tag and key were set. This function will accept the tag, key, and TBC reflections as the function parameters.

```cpp
consteval bool CTS_check(meta::info tag,meta::info key, meta::info TBC){
   auto refl = meta::substitute(TBC,{meta::reflect_constant(tag),meta::reflect_constant(key)});
   return meta::is_complete_type(refl);
}
```

First, the function retrieves the specialization of TBC with the tag and key as template parameters, as before. To tell if the tag and key were set, the ‘meta\:\:is_complete_type’ function is used. Here is the reason the TBC must be declared as an incomplete type. If the tag and key weren't set, instantiating the TBC with them will result in an incomplete class, as the TBC is incomplete and there is no specialization for the tag and key. If the tag and key are set,the specialization will be used during instantiation, and the resulting class will be complete, since the ‘set’ function introduced the specialization.

### FFI
The friend function injection method uses two classes, FFI_get_struct and FFI_set_struct, to store and retrieve key-value pairs. These classes will use the same two friend functions: get_func and check_func. When a friend function is declared in a class, it belongs to the class's innermost enclosing namespace. Since the classes will use the same friend functions, they must be declared within the same namespace.

​The FFI_get_struct will have two non-type template parameters of type meta\:\:info\: the tag and key.

​The get_func will be declared as a function that returns a value of type meta\:\:info and accepts a parameter of type FFI_get_struct, which is implicitly FFI_get_struct\<tag,key\> within the class body. Importantly, this function is not defined. It is only declared. This function stores and retrieves the value associated with a key.

​The check_func function will be defined to return a value of type meta\:\:info and to accept a parameter of type FFI_get_struct\<tag,key>. Importantly, the parameter must not have an identifier. This function will return a reflection of itself using the ‘meta\:\:current_function’ function.

```cpp
template<meta::info tag, meta::info key>
struct FFI_get_struct{
   friend consteval meta::info get_func(FFI_get_struct);
   friend consteval meta::info check_func(FFI_get_struct){
       return meta::current_function();
   }
};
```

When FFI_get_struct gets instantiated with a given tag and key, two things happen.

First, the function declaration get_func(FFI_get_struct\<tag, key\>) is injected into the enclosing namespace. It will have no definition, so it can’t be called yet.

Second, the function definition check_func(FFI_get_struct\<tag, key\>) is injected into the enclosing namespace. It has a body and can be called, but its parameter lacks an identifier.

For all pairs of tags and keys that ‘FFI_get_struct’ is instantiated with, there will be the get_func(FFI_get_struct\<tag,key\>) and check_func(FFI_get_struct\<tag,key\>) functions in the enclosing namespace. This means there is an overload for each tag-key pair. This way, the functions act like a key-value map, accessed via function overloading, with the function parameter type serving as the key.

The FFI_set_struct will have three non-type template parameters of type meta\:\:info\: the tag, key, and value.

The get_func function will be defined to return a meta\:\:info value and accept a parameter of type FFI_get_struct\<tag, key\>. This function returns a reflection of the ‘value’ non-type template parameter, using the ‘meta\:\:reflect_constant’ function.

The check_func function will be declared to return a value of type meta\:\:info and to accept a parameter of type FFI_get_struct\<tag,key\>. Importantly, the parameter will have an identifier.

```cpp
template<meta::info tag, meta::info key, meta::info value>
struct FFI_set_struct{
   friend consteval meta::info get_func(FFI_get_struct<tag,key>){
       return meta::reflect_constant(value);
   }
   friend consteval meta::info check_func(FFI_get_struct<tag,key> name);
};
```

When FFI_set_struct gets instantiated with a given tag, key, and value, two things will happen.

First, the function definition get_func(FFI_get_struct\<tag, key\>) is injected into the enclosing namespace. It can be called.

Second, the function declaration check_func(FFI_get_struct\<tag, key\> name) is injected into the enclosing namespace. This function will have an identifier.

Function injection is commutative, meaning that the result of instantiating FFI_get_struct and FFI_set_struct is the same no matter the order. It is also additive: if there is already a function declaration with no body in a namespace, and the same function with a definition is injected, the function becomes defined and can be called. Also, if there is already a function with a parameter that has no identifier in a namespace, and the same function with an identifier for the same parameter is injected, then the function’s parameter now has an identifier. Importantly, function injection cannot remove or alter any property of a function in a namespace, so if a function’s parameter already has an identifier or a function has a definition, then there is no way to remove or overwrite them.

As friend functions are anonymous, they can only be called using argument-dependent lookup. So if you want to call get_func for a given tag and key, you would do it as so:
```cpp
get_func(FFI_get_struct<tag,key>{});
```
This will implicitly instantiate FFI_get_struct. This means that there are two possible cases for what get_func will do. If get_func is called before FFI_set_struct is instantiated with the same tag and key, then it will have no definition, and a compilation error will occur. If it gets called after FFI_set_struct is instantiated, it will have a definition that returns the reflection of the ‘value’ non-type template parameter. This way, the get_func function can be referred to regardless of whether FFI_set_struct was instantiated, and the return value of get_func can be set using FFI_set_struct.

If FFI_set_struct is instantiated twice with the same tag and key but different value parameters, a compilation error will occur. The first instantiation will give the get_func function a definition. The second instantiation will try to define it again, but because the function already has one, a function redefinition error will occur.


#### Set

The ‘set’ function will instantiate FFI_set_struct with the given parameters. This function will accept the tag, key, and value reflections as the function parameters.

```cpp
template<typename T>
constexpr std::size_t size_of_class = sizeof(T);


consteval void FFI_set(meta::info tag, meta::info key, meta::info value){
   auto refl = meta::substitute(^^FFI_set_struct,{meta::reflect_constant(tag),
               meta::reflect_constant(key),meta::reflect_constant(value)});
   (void)meta::extract<size_t>(meta::substitute(^^size_of_class,{refl}));
}
```

The function first obtains a reflection of FFI_set_struct with the given tag, key, and value as its template parameters. It will be the same as the reflection ^^FFI_set_struct\<tag,key,value>\. The substitute function will not perform instantiation when substituting class templates. To instantiate FFI_set_struct\<tag,key,value\>, a substitution with the variable template ‘size_of_class’ will be used. When a variable template is instantiated, its value will be evaluated. If the value is the size of a class, it will instantiate the class if it hasn’t been instantiated already. To instantiate the variable template, the ‘meta\:\:extract’ function is used, and the result is voided.

#### Get

The ‘get’ function will return the stored value for a given key and tag.

```cpp
template<meta::info tag, meta::info key>
constexpr auto FFI_get_helper = [:get_func(FFI_get_struct<tag,key>{}):];


consteval meta::info FFI_get(meta::info tag, meta::info key){
   auto refl = meta::substitute(^^FFI_get_helper,{meta::reflect_constant(tag),meta::reflect_constant(key)});
   return meta::extract<meta::info>(refl);
}
```

The template variable ‘FFI_get_helper’ will be used to call the ‘get_func’ function. This template has two non-type template parameters: the tag and key. When this template variable is instantiated, its value is evaluated. During evaluation, ‘get_func’ is called with a parameter of type FFI_get_struct\<tag,key\>. This returns a reflection of the stored value for the provided tag and key. The splice operator ‘[\:\:]’ will extract the value from its reflection.

The ‘get’ function will first create a reflection of ‘FFI_get_helper’ with the tag and key as its template parameters. To instantiate the variable template and retrieve its value, the ’meta\:\:extract’ function is used. This value is returned.

#### Check

The ‘check’ function will return whether a given tag and key were set. This function will accept the tag and key as the function parameters.

```cpp
template<meta::info tag, meta::info key>
constexpr auto FFI_check_helper = check_func(FFI_get_struct<tag,key>{});


consteval bool FFI_check(meta::info tag, meta::info key){
   meta::info func_refl = meta::extract<meta::info>(meta::substitute(^^FFI_check_helper,{meta::reflect_constant(tag),meta::reflect_constant(key)}));
   return meta::has_identifier(meta::parameters_of(func_refl)[0]);
}
```

The ‘check’ function uses the template variable ‘FFI_check_helper’ in a similar way to the ‘get’ function, except it returns a reflection of the ‘check_func’ function. The ‘meta\:\:parameters_of’ function will return a list of reflections of the function’s parameters. By indexing the list at 0, you get the reflection of the first parameter of ‘check_func’. This parameter will have an identifier only if the tag and key were set. The ‘meta\:\:has_identifier’ function checks whether the given parameter has an identifier. The result is returned.

### Map

The compile-time map will act like std\:\:map. For simplicity, this map will only have three member functions: insert, operator[], and contains. They will be wrapper functions over the set, get, and check functions. The map will have two template parameters: the key and value types. Key_t and Value_t, respectively. The only difference between the interfaces of the CTS and FFI maps is that the CTS constructor will require a reflection of the TBC. The tag value will be automatically generated by reflecting the ‘this’ value, meaning the unique tag for each map object will be the reflection of its own address. Requiring a reflection of its own address forces the object to be a constexpr variable.

The member functions will have the consteval specifier and will be const-qualified so they can be used in a consteval context.
The ‘insert’ function will accept a key and a value parameter of type Key_t and Value_t. It will call the appropriate ‘set’ function to insert the key-value pair into the map with the tag. It will return void. If the same key is inserted twice, a compilation error will occur.

The ‘operator[]’ function will accept a key parameter of type Key_t. It will call the appropriate ‘get’ function to get the saved value at the given key and tag. If a given is not set, a compilation error will occur.

The ‘contains’ function will accept a key parameter of type Key_t. It will call the appropriate ‘check’ function to verify whether a value was saved under the given key and tag. It will return a bool value.

#### CTS
```cpp
template<typename Key_t,typename Value_t>
struct CTS_map{
   meta::info TBC;
   meta::info tag = meta::reflect_constant(this);
   consteval CTS_map(meta::info TBC_):TBC(TBC_){}


   consteval void insert(Key_t key, Value_t value) const{
       CTS_set(tag,meta::reflect_constant(key),meta::reflect_constant(value),TBC);
   }
   consteval Value_t operator[](Key_t key) const{
       return meta::extract<Value_t>(CTS_get(tag,meta::reflect_constant(key),TBC));
   }
   consteval bool contains(Key_t key) const{
       return CTS_check(tag,meta::reflect_constant(key),TBC);
   }
};
```
#### FFI
```cpp
template<typename Key_t,typename Value_t>
struct FFI_map{
  meta::info tag = meta::reflect_constant(this);


  consteval void insert(Key_t key, Value_t value) const{
      FFI_set(tag,meta::reflect_constant(key),meta::reflect_constant(value));
  }
  consteval Value_t operator[](Key_t key) const{
      return meta::extract<Value_t>(FFI_get(tag,meta::reflect_constant(key)));
  }
  consteval bool contains(Key_t key) const{
      return FFI_check(tag,meta::reflect_constant(key));
  }
};
```

### List

The compile-time list acts like an append-only list with zero-indexed random access. The list will have one template parameter: the list's value type. The list will include 4 consteval, const-qualified member functions: size, operator[], back, and push.

The list will have one template parameter: the value type. The list is built on top of the map, so it will have the compile-time map as a data member. The map's key type will be ‘std::size_t’, and the value type will be the list's value type. As the map is a member variable, this forces the list object to be a constexpr variable as well.

All elements of the list will be 0-indexed in the order they are pushed. They will be stored in the map, with their indices as keys and the elements as values.

The ‘size’ function returns the number of elements in the list. It has no function parameters and returns a std::size_t value. As a list object is a constexpr variable, the size of the list can't be stored. Instead, the size will be found each time this function is called. A binary search will be used to determine the list's size. The function will search for the smallest unused key in the map. This key will be equal to the number of elements in the list. The beginning search interval will be [0,R], where R is the smallest power of some integer ‘Hint’ that is not used as a key.

```cpp
consteval std::size_t size() const{
       std::size_t hint = 10;
       std::size_t l=0;
       std::size_t r=hint;
       while(mp.contains(r)) r*=hint;
       while(l<r){
           std::size_t mid = (l+r)/2;
           if(mp.contains(mid)){
               l=mid+1;
           }else{
               r=mid;
           }
       }
       return l;
   }
```

The ‘operator[]’ function returns the value of the element at the given index. It has one function parameter of type std::size_t, which is the index of the element to return, and it returns a value of the list’s value type. This function simply calls the ‘operator[]’ of the map with the same index. Calling this function with an index larger than or equal to the list's size will cause a compilation error, as the index would be out of range.

```cpp
consteval Value_t operator[](std::size_t index){
       return mp[index];
   }
```

The ‘back’ function returns the value of the last element in the list. It has no function parameters and returns a value of the list’s value type. Since all the elements are 0-indexed in the map, the index of the back element is equal to the size of the list minus one. If this function is called on an empty list, a compilation error will occur.

```cpp
consteval Value_t back() const{
       return mp[size()-1];
   }
```

The ‘push’ function will append a given value to the end of the list. It has one function parameter of the list’s value type and no return value. The index of the new element will be equal to the list's size before it is appended. The function inserts the given value into the map, using the new index as the key.

```cpp
consteval void push(Value_t value) const{
       mp.insert(size(),value);
   }
```

#### CTS
```cpp
template<typename Value_t>
struct list{
   CTS_map<std::size_t,Value_t> mp;
   consteval CTS_list(meta::info TBC):mp(TBC){}
   consteval std::size_t size() const{
       std::size_t hint = 10;
       std::size_t l=0;
       std::size_t r=hint;
       while(mp.contains(r)) r*=hint;
       while(l<r){
           std::size_t mid = (l+r)/2;
           if(mp.contains(mid)){
               l=mid+1;
           }else{
               r=mid;
           }
       }
       return l;
   }
   consteval Value_t back() const{
       return mp[size()-1];
   }
   consteval Value_t operator[](std::size_t index){
       return mp[index];
   }
   consteval void push(Value_t value) const{
       mp.insert(size(),value);
   }
};
```
#### FFI
```cpp
template<typename Value_t>
struct FFI_list{
   FFI_map<std::size_t,Value_t> mp;
  
   consteval std::size_t size() const{
       std::size_t hint = 10;
       std::size_t l=0;
       std::size_t r=hint;
       while(mp.contains(r)) r*=hint;
       while(l<r){
           std::size_t mid = (l+r)/2;
           if(mp.contains(mid)){
               l=mid+1;
           }else{
               r=mid;
           }
       }
       return l;
   }


   consteval Value_t back() const{
       return mp[size()-1];
   }


   consteval Value_t operator[](std::size_t index) const{
       return mp[index];
   }


   consteval void push(Value_t value) const{
       mp.insert(size(),value);
   }
};
```

### Variable

The compile-time variable is a thin semantic wrapper over the list. The variable will have one template parameter: the variable’s value type. The variable will have a compile-time list as a data member, with the list’s template parameter being the variable’s value type. The current value of the variable will be the back element of the list. When a new value is assigned to the variable, it is pushed to the end of the list. The variable will have two member functions: the assignment operator and an implicit conversion function. 
 
The assignment operator takes a value parameter and returns a reference to the calling variable. The function will call the list’s push function with the given value.

```cpp
consteval const variable& operator=(V value) const{
       lst.push(value);
       return *this;
   }
```

The implicit conversion function will return a value of the variable’s value type. The function will call the list’s back function and return the list's last element.

```cpp
consteval operator V() const{
       return lst.back();
   }
```

#### CTS

```cpp
template<typename Value_t>
struct CTS_variable{
   CTS_list<Value_t> lst;
   consteval CTS_variable(meta::info TBC):lst(TBC){}
   consteval const variable& operator=(Value_t value) const{
       lst.push(value);
       return *this;
   }
   consteval operator Value_t() const{
       return lst.back();
   }
};
```
#### FFI
```cpp
template<typename Value_t>
struct FFI_variable{
   FFI_list<Value_t> lst;
  
   consteval const variable& operator=(Value_t value) const{
       lst.push(value);
       return *this;
   }


   consteval operator Value_t() const{
       return lst.back();
   }
};
```

## Usage examples
### CTS
```cpp
template<auto,auto>
struct storage_class;

constexpr CTS_map<int,int> cts_map(^^storage_class);


consteval{
   cts_map.insert(1,2);
   cts_map.insert(32,55);
}


static_assert(cts_map[1]==2);
static_assert(cts_map[32]==55);


constexpr CTS_list<int> cts_list(^^storage_class);


consteval{
   cts_list.push(53);
}


static_assert(cts_list.back()==53);


consteval{
   cts_list.push(33);
}


static_assert(cts_list.back()==33);




constexpr CTS_variable<double> cts_var(^^storage_class);


consteval{
   cts_var = 6.5;
}


static_assert(cts_var == 6.5);


consteval{
   cts_var = 72.34;
}


static_assert(cts_var == 72.34);
```
### FFI
```cpp
constexpr FFI_map<int,int> ffi_map;


consteval{
   ffi_map.insert(1,2);
   ffi_map.insert(32,55);
}


static_assert(ffi_map[1]==2);
static_assert(ffi_map[32]==55);


constexpr FFI_list<int> ffi_list;


consteval{
   ffi_list.push(53);
}


static_assert(ffi_list.back()==53);


consteval{
   ffi_list.push(33);
}


static_assert(ffi_list[0]==53);
static_assert(ffi_list.back()==33);




constexpr FFI_variable<char> ffi_var;


consteval{
   ffi_var = 'f';
}


static_assert(ffi_var == 'f');


consteval{
   ffi_var = 'g';
}


static_assert(ffi_var == 'g');
```
## Bibliography
* Reflection for C++26 \(https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2025/p2996r13.html\)
* Annotations for Reflection \(https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2025/p3394r4.html\)
* Miscellaneous Reflection Cleanup \(https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2026/p3795r2.html\)
* Special thanks to u/friedkeenan for sharing his implementation of the FFI method on the cpp reddit \(https://www.reddit.com/r/cpp/comments/1t75cn1/cvl_a_c26_library_for_mutating_consteval_state/\)

