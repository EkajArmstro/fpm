# fpm
A C++ header-only fixed-point math library. "fpm" stands for "fixed-point math".

## Usage
`fpm` defines the `fpm::fixed` class, which is templated on the underlying integer type and the number of bits in the fraction:
```c++
namespace fpm {
    template <typename BaseType, typename IntermediateType, unsigned int FractionBits>
    class fixed;
}
```
**Note:** It's recommended to use a *signed* integer type for `BaseType` (and `IntermediateType`) to emulate floating-point numbers 
and to allow the compiler to optimize the computations, since overflow and underflow are undefined
for signed integer types.

To use this class, simply include its header:
```c++
#include <fpm/fixed.h>
```
You may wish to typedef a particular choice of underlying type, intermediate type and fraction bitcount, e.g.:
```c++
using position = fpm::fixed<std::int32_t, std::int64_t, 16>;
```
This defines a signed 16.16 fixed-point number with a range of -32768 to 65535.999985... and a resolution of 0.0000153... It uses 64-bit integers as intermediate type during calculations to avoid loss of information.

For your convenience, several popular fixed-point formats have been defined in the `fpm` namespace:
```c++
namespace fpm {
    using fixed_16_16 = fixed<std::int32_t, std::int64_t, 16>;  // Q16.16 format
    using fixed_24_8  = fixed<std::int32_t, std::int64_t, 8>;   // Q24.8 format
    using fixed_8_24  = fixed<std::int32_t, std::int64_t, 24>;  // Q8.24 format
}
```


## Conversion to and from float
The intent behind `fpm` is to replace floats for purposes of performance or portability. Thus, it guards against accidental usage of floats by requiring explicit conversion:
```c++
fpm::fixed_16_16 a = 0.5;        // Error: implicit construction from float
fpm::fixed_16_16 b { 0.5 };      // OK: explicit construction from float
fpm::fixed_16_16 c = b * 0.5;    // Error: implicit conversion from float
float d = b;                     // Error: implicit conversion to float
float e = static_cast<float>(b); // OK: explicit conversion to float
```

For integers, this still applies to initialization, but arithmetic operations *can* use integers: 
```c++
fpm::fixed_16_16 a = 2;        // Error: implicit construction from int
fpm::fixed_16_16 b { 2 };      // OK: explicit construction from int
fpm::fixed_16_16 c = b / 2;    // OK
int d = b;                     // Error: requires explicit conversion
int e = static_cast<int>(b);   // OK: explicit conversion to int
```
You must still guard against underflow and overflow, though.

## Accuracy and performance
Please refer to the pages for [accuracy](accuracy.md) and [performance](performance.md) results.

## Limitations
Unlike floating-point numbers, `fpm::fixed`:
* can not represent Not-a-Number, infinity or negative zero.
* does not have a notion of epsilon or subnormal numbers.
* does have a risk of overflow and underflow.

Notably the last point requires careful use of fixed-point numbers: like integers, you must ensure that they do not overflow or underflow.

## Alternatives
* [libfixmath](https://github.com/PetteriAimonen/libfixmath): C99 library, only supports Q16.16 format backed by 32-bit integers.
* [fixed_point](https://github.com/johnmcfarlane/fixed_point): C++11 header-only library, also supports generic precision.