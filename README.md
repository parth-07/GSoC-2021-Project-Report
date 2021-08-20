## GSoC 2021 Project Report

# Add Support for differentiating functor objects in Clad

**Author:** Parth Arora

**Organisation:** CERN-HSF

**Mentors:** Vassil, David

## Clad overview

Clad is an automatic differentiation clang plugin for C++. It differentiates mathematical functions represented as C++ functions.
Differentiated functions can be executed directly and conveniently through clad as well as clad can also produce source code of
the differentiated function. The produced differentiated function source code contain no additional dependency and thus can be
used independent of clad.

Clad analyses the Abstract Syntax Tree (AST) produced by clang compiler to differentiate the functions using automatic differentiation.
Automatic Differentiation (AD) avoids the disadvantages 

## Project overview

Many computations are modelled using functor objects and lambda expressions. These constructs are becoming more and more popular and
relevant in the modern C++. There was a growing need to support differentiating these constructs in clad. Thus, this project add
support to directly differentiate functor objects and lambda expressions in clad.

This project also add support for increases the supported subset of C++ syntax and make clad more robust by adding an automated testing
framework and fixing various issues.


## Project Goals
- Add support for differentiating functor objects and lambdas using both forward and reverse mode in clad.
- Extend clad by adding support for differentiating more C++ syntax and constructs.
- Make clad more robust by adding an automatic testing framework and fixing various existing issues. 

## Project Results

### Add support for differentiating functor objects and lambda expressions.

Now, clad supports differentiating functor objects and lambda expressions in both forward and reverse mode.

```c++
class Experiment;

// both ways are equivalent
auto d_E = clad::differentiate(&E, "i");    // passing functor by pointer
auto d_ERef = clad::differentiate(E, "i");  // pasing functor by reference

auto lambda = [&a](double i, double j) {
    return i*j;
  };

// both ways are equivalent
auto d_lambda = clad::differentiate(&lambda, "i");    // passing lambda by pointer
auto d_lambdaRef = clad::differentiate(lambda, "i");  // passing lambda by reference

```
In brief, core idea used for the differentiation of functor objects and lambda expressions is as follows:
- Implicitly differentiate the overloaded call operator that is defined by the functor(or lambda expression) type.

- Store the address of the functor object in the resulting `CladFunction` object.

  (`CladFunction` class stores all the necessary information about the derived function to conveniently access, execute and debug them.)
  
- Now, `Cladfunction` automatically takes stored functor object address to execute the derived function. 
  Thus, there's no need to explicitly pass the functor object (or lambda expression) while executing the derived function through the 
  `CladFunction` object.


#### Some of the major challenges faced during its implementation:
- The Implementation depended on the differentiation of member functions using clad differentiation functions.

  Clad differentiation functions are as follows:
  
  - `clad::differentiate` - differentiates function with respect to a single parameter using forward mode automatic differentiation.
  - `clad::gradient` - differentiates function with respect to multiple parameters using reverse mode automatic differentiation.
  - `clad::hessian` - compute hessian matrix of a function with respect to multiple parameters.
  - `clad::jacobian` - compute jacobian matrix of a function with respect to multiple paramters.
  
  Differentiation of member functions using `clad::gradient`had a bug which used to cause program crash sometiems. The cause of the
  bug was inconsistencies in the clang Abstract Syntax Tree in the updated call expression site. It took us quite a while to solve
  this bug in an optimal way.
  
  `clad::hessian` did not support differentiating member functions before. Hence, the support for differentiating member functions in
  `clad::hessian` was added.
  
- Obtaining the correct CVR qualified functor type while allowing functor objects to be passed both by pointers and references was 
  interesting to implement, and had its fair share of complexities.

### Extend clad by adding support for differentiating more C++ syntax and constructs.

#### Added support for differentiating `while` and `do-while` statements in both the forward and the reverse mode automatic differentiation.

It was thought-provoking and exciting to handle all the corner cases associated with applying automatic differentiation to `while`
and `do-while` statements.

With this added support, clad can now differentiate, in both the forward and the reverse mode automatic differentiation, functions containing
`while` and `do-while` statements such as `function_containing_while` shown below.

```c++
double fn_containing_while(double i, double j) {
  int counter = 3;
  double res = i;
  while(counter--) {
    res += i*j;
  }
  return res;
}
```


#### Added support for differentiating `switch` statement in the forward mode automatic differentiation.

It was particularly interesting to handle switch statements because of the scope rules associated with the case lables and the
sudden change in the flow of control depending on the switch condition.

With this added support, clad can now differentiate functions containing `switch` statements such as `function_containing_switch` shown below:

```c++
double fn_containing_switch(double i, double j, int choice) {
  double res = i+j;
  switch(choice) {
    case 0:
      res += i*i;
      break;
    case 1:
      res += j*j;
      break;
    default:
      res += i*j;
  }
  return res;
}
```

### Make clad more robust by adding an automatic testing framework and fixing various existing issues.

#### Automatic testing of the reverse mode differentiation using the forward mode differentiation

To ensure clad is producing consistent results, now we can enable additional testing using `-fenable-reverse-mode-testing`
List fixed issues

### Work done

#### Pull requests

#### Issues opened

## Results
