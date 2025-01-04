# BabyBear

The field’s order is  $15 \cdot 2^{27} + 1$ (equivalently,  $2^{31} - 2^{27} + 1$ ).

This choice of field offers two key benefits:

1.	It allows for 32-bit addition without overflow.

2.	It maximizes the power of 2 as a factor of  P-1 , which is beneficial for certain cryptographic and computational applications.

## two-adicity

Two-adicity is a property of a finite field that describes the size of the largest multiplicative subgroup whose order is a power of 2. Specifically, if a field has a two-adicity of  n , it means there exists a subgroup of size  2^n  in its multiplicative group

```rust
// Plonky3/baby-bear/src/baby_bear.rs
impl TwoAdicData for BabyBearParameters {
    // The subgroup whose order is 2 power by 27
    const TWO_ADICITY: usize = 27;

    type ArrayLike = &'static [BabyBear];
    // ...
}
```

### Subgroups

For any prime field , the multiplicative group (non-zero elements under multiplication modulo $P$) always has an order of $P - 1$. This group is cyclic, meaning there exists a generator $g$ such that every element in the group can be expressed as a power of $g$￼.

Example: Subgroups in $F_{13}$

1.	Multiplicative Group:
    - Elements: ￼$\{1, 2, \cdots, 12\}$ (excluding 0 because it does not have a multiplicative inverse).
	- Order: 12.
	- Since 12 = 2 * 2 * 3￼, the multiplicative group ￼ has subgroups of orders 1, 2, 3, 4, 6, and 12.
2.	Subgroups:
	- Order 12 (Full Group):
    	- Generator g = 2￼, such that: $G=\{2^0,2^1,2^2,...,2^{11}\}\bmod13=\{1,2,4,8,3,6,12,11,9,5,10,7\}.$
￼
	- Order 4:
    	- Subgroup: $H=\{1,g^3,g^6,g^9\},$ where g = 2
    	- Elements: $H=\{1,8,12,5\}.$
    	- Generator: $h = g^3 = 8$

  	- Order 3:
    	- Subgroup: $H=\{1,g^4,g^8\},$ where g = 2
    	- Elements: $H=\{1, 3, 9\}.$
    	- Generator: $h = g^4 = 3$

In this case the two-adicity of $F_{13}$ is 2

### Importance in FFT/FRI

Two-adicity is really helpful in FRI because it allows us to efficiently reduce the size of the domain where we evaluate the polynomial. During each step of the FRI protocol, we compute the polynomial on a new domain that is half the size of the previous one. This process relies on having subgroups in the field whose sizes are powers of 2, thanks to its two-adicity.

The beauty of this is that two-adicity guarantees we have enough smaller subgroups (like  2^n, 2^{n-1}, \dots ) to keep halving the domain until it’s small enough to verify easily. This step-by-step reduction is a key part of FRI’s design and makes it efficient to create and verify proofs.

## Montgomery Form

Modular reduction is computationally expensive. [Montgomery multiplication]((https://en.wikipedia.org/wiki/Montgomery_modular_multiplication)) provides an efficient solution for performing modular arithmetic operations.

### Key Insight
While humans find division by powers of 10 (1, 10, 100, ..., 10^n) natural, computers perform more efficiently with powers of 2. Montgomery arithmetic leverages this by transforming calculations into a domain where divisions become simple bit shifts.

### The Algorithm
The Montgomery multiplication (Mon) of two numbers A and B is defined as:

$$
\text{Mon}(A, B, \text{N}) = \text{Mon}(A, B) = 
\begin{cases}
\frac{A \cdot B + (A \cdot B \cdot \text{N'} \bmod R) \cdot \text{N}}{R} - \text{N} & \text{if Result} \geq \text{N} \\
\frac{A \cdot B + (A \cdot B \cdot \text{N'} \bmod R) \cdot \text{N}}{R} & \text{otherwise}
\end{cases}
$$

where:
- R is 2^`MONTY_BITS`
- N' is N^(-1) mod R which is named as `MONTY_MU` in plonky3

### Example: Computing (a * b * c) mod Prime

Let's walk through an example using the BabyBear field:

1. Preprocess:
```rust
// Plonky3/baby-bear/src/baby_bear.rs
impl MontyParameters for BabyBearParameters {
    // Our field modulus
    const PRIME: u32 = 0x78000001;
    // Bit length of Prime
    // The power of Montgomery constant(The R in above formular)
    const MONTY_BITS: u32 = 32;
    // Montgomery multiplier = PRIME^(-1) mod 2^MONTY_BITS
    const MONTY_MU: u32 = 0x88000001;
}
```

2. Convert inputs to Montgomery domain

    A = (a * R) mod Prime

    B = (b * R) mod Prime

    C = (c * R) mod Prime


3. Modular Calculation in Montgomery domain

    intermediate_O = Mon(A, B)

    intermediate_1 = Mon(C, intermediate_O)

4. Convert result from Montgomery domain to Babybear field

    result = Mon(intermediate_1, 1)
