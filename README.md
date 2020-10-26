[![Build Status](https://travis-ci.com/milankl/LogFixPoint16s.jl.svg?branch=master)](https://travis-ci.com/milankl/LogFixPoint16s.jl)
[![Build Status](https://ci.appveyor.com/api/projects/status/github/milankl/LogFixPoint16s.jl?svg=true)](https://ci.appveyor.com/project/milankl/LogFixPoint16s-jl)

# LogFixPoint16s.jl

Exports LogFixPoint16 - a 16bit [logarithmic fixed-point number](https://en.wikipedia.org/wiki/Logarithmic_number_system) format with 1 sign bit, 6 signed integer bits, and 9 fraction bits.

### Example use

```julia
julia> using LogFixPoint16s
julia> v = LogFixPoint16.(rand(Float32,5))
5-element Array{LogFixPoint16,1}:
 LogFixPoint16(0.8925083)
 LogFixPoint16(0.4919428)
 LogFixPoint16(0.69759846)
 LogFixPoint16(0.25616693)
 LogFixPoint16(0.57248604)

julia> sum(v)
LogFixPoint16(2.9139352)
```

### Features

Exports `LogFixPoint16, iszero, isnan, signbit, zero, nan, floatmin, floatmax, one, -, inv, *, / , +, -, sqrt, nextfloat, prevfloat, ==, <=, >, >=, show, bitstring` as well as conversions to and from `Float64, Float32, Float16, Int`.

Although `LogFixPoint16` is always a 16-bit format, the number of fraction bits (in exchange for integer bits) can be adjusted between 7 and 11. For 7 fraction bits, `LogFixPoint16` has a similar dynamic range-precision trade-off as `BFloat16`; 10 fraction bits are similar to `Float16`.

```
julia> LogFixPoint16s.set_nfrac(7)
┌ Warning: LogFixPoint16 was changed to 8 integer and 7 fraction bits.
└ @ Main.LogFixPoint16s ~/git/LogFixPoint16s.jl/src/LogFixPoint16s.jl:24
```

### Bias

Currently, `LogFixPoint16` is slightly biased away from zero, but is positive/negative bias-free.

### Theory

A number `X` is encoded as LogFixPoint16 as

```
X = (-1)^s * 2^x
```
with `s` being the sign bit and `x` the fixed-point number in the exponent. E.g. the number `3` is encoded as

```julia
julia> bitstring(LogFixPoint16(3),:split)
"0 1000001 10010110"
```
The sign bit is `0`, the sign bit of the signed integer is `1` (meaning +, [excess-8](https://en.wikipedia.org/wiki/Signed_number_representations#Comparison_table) representation) such that the integer bits equal to `1`. The fraction bits are 1/2 + 1/16 + 1/64 + 1/128. Together this is

```
0 1000001 10010110 = +2^(1 + 1/2 + 1/16 + 1/64 + 1/128) = 2^1.5859375 ≈ 3
```
The only exceptions are the bitpatterns `0x0000` (zero) and `0x8000` (Not-a-Real, NaR). The smallest/largest representable numbers are

```julia
julia> floatmin(LogFixPoint16)
LogFixPoint16(2.3314606e-10)

julia> floatmax(LogFixPoint16)
LogFixPoint16(4.2891566e9)
```
 
### Decimal precision

Logarithmic fixed-point numbers are placed equi-distantly on a log-scale. Consequently, their decimal precision is perfectly flat throughout the dynamic range of representable numbers. In contrast, floating-point numbers are only equi-distant in logarithmic space when the significand is held fixed; the significant bits, however, are placed equi-distant in linear space.

As a consequence there is no rounding error for logarithmic fixed-point numbers in multiplication, division or power/root - similarly as there is no rounding error for fixed-point numbers for addition and subtraction.

![decimal precision](figs/decimal_precision.png?raw=true "decimal precision")

LogFixPoint16 with 10 fraction bits have a similar decimal precision / dynamic range trade-off as Float16, and 7 fraction bits are similar to BFloat16. However, these decimal precision only apply to additions, as multiplications are rounding error-free. `LogFixPoint16s.jl` also allows additionally for 8,9 or 11 fraction bits, which are not shown.

### Benchmarks

`LogFixPoint16` is considerably faster than `Float16` for addition (x5) and multiplication (x33)

```julia
julia> using LogFixPoint16s, BenchmarkTools
julia> A = rand(Float32,1000,1000);
julia> B = rand(Float32,1000,1000);
julia> C,D = Float16.(A),Float16.(B);
julia> E,F = LogFixPoint16.(A),LogFixPoint16.(B);

julia> @btime +($A,$B);       # Float32
  442.167 μs (2 allocations: 3.81 MiB)

julia> @btime +($C,$D);       # Float16
  17.215 ms (2 allocations: 1.91 MiB)

julia> @btime +($E,$F);       # LogFixPoint16
  3.654 ms (2 allocations: 1.91 MiB)
  
julia> @btime .*($A,$B);      # Float32
  457.807 μs (2 allocations: 3.81 MiB)

julia> @btime .*($C,$D);      # Float16
  17.760 ms (2 allocations: 1.91 MiB)
  
julia> @btime .*($E,$F);      # LogFixPoint16
  533.345 μs (2 allocations: 1.91 MiB)
```

### Installation

`LogFixPoint16s.jl` is registered in the Julia Registry, so simply do

```julia
julia> ] add LogFixPoint16s
```
where `]` opens the package manager.
