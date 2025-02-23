# RScode

Before the advent of ZK, RS codes were already widely used in the domain of **Channel Coding and Decoding**.

To transfer a message, the raw message undergoes a series of coding and decoding stages. The process can be organized as follows:

Encoding Side (Sender):

1.	Source Coding: Compresses the raw message to reduce redundancy.
2.	Cryptographic Coding: Encrypts the message to ensure its security.
3.	**Channel Coding**: Adds error-correcting codes to make the message resilient to errors during transmission.
4.	Line Coding: Converts the coded message into a signal suitable for transmission through the channel.

Channel:

The message travels through the communication channel, which may introduce noise, distortions, or errors.

Decoding Side (Reciver):

1.	Line Decoding: Converts the received signal back into its coded message form.
2.	**Channel Decoding**: Detects and corrects errors introduced by the channel.
3.	Cryptographic Decoding: Decrypts the message to restore its secure content.
4.	Source Decoding: Decompresses the message to reconstruct the original raw message.

This structured process ensures the message is accurately transmitted through the channel while maintaining security and reliability.

## Basic concept

### RS-code is a block code.

For example, given a message of length $x$ to be encoded, the encoding process is performed step by step, processing the message in blocks of size $k$ at each step.

In other words, the raw message is divided into multiple blocks, each containing $k$ elements of information. Each block is referred to as a message .

Each message  of size $k$ is mapped to a code word consisting of $n$ elements in $F_q$:

$$
F_{q}^{k} \rightarrow F_{q}^{n}
$$

$k$: The message  length,

$F_{q}^{k}$ The set of all possible messages of length  k .

$n$: The code word length

Code Space: A q-array code C is a subset of $F_{q}^{n}$ . Since $F_{q}^{n}$ contains many more elements than $F_{q}^{k}$ ($F_{q}^{n} \gg F_{q}^{k}$ ), not all elements of $F_{q}^{n}$ are used as codewords.

code word: The element in code space

We also need to mention another two terms:

1. Code Rate: $r := k/n$ The ratio of message  length to code word length.

2. Distance: $d := n - k + 1$ Minimum distance (measured as the number of symbol positions in which two codewords differ, we will explain it later).

Please keep the above information in mind, as different papers and textbooks may use varying notations at times.

### RS-code is a Liner Code

First, let's recap the Reed-Solomon (RS) coding's process step by step:
1.	Message Representation:

    The message is divided into blocks of size $k$, where each block is treated as a sequence of symbols from a finite field $F_q$. Each block is called a message .

2.	Polynomial Representation:

    Each message  is converted into a polynomial $P(x)$ of degree less than $k$. The coefficients of the polynomial come from the finite field $F_q$.

3.	Evaluation:

    The polynomial $P(x)$ is evaluated at $n$ distinct points in $F_q$, producing a sequence of $n$ symbols: $c = (P(a_1), P(a_2), \dots, P(a_n))$ .

    These evaluation points $\{a_1, a_2, \dots, a_n\}$ are predefined and fixed.

4.	Codeword Construction:

    The resulting sequence of $n$ symbols is called a codeword. Each codeword is a representation of the original message , with additional redundancy added.

5.	Error Correction: 

    *You don't need to fully understand when you just start your zk journey, this property is not untilzed in zk for now(2024, Dec, 31th)*

    In the decoding process, the receiver uses the properties of polynomials to correct errors. If fewer than $(n-k)/2$ symbols are corrupted, the original message  can be fully recovered.
#### Why do we call it linear code?

The RS encoding process can be expressed as a matrix multiplication.

- For a message  $m = (m_0, m_1, \dots, m_{k-1})$ in $F_q^k$, the codeword $c = (c_0, c_1, \dots, c_{n-1})$ in $F_q^n$ is obtained by multiplying $m$ with a generator matrix $G$.

$$
c = m \cdot G
$$

Where:
- $m$ is a $1 \times k$ row vector.
- $G$ is a $k \times n$ generator matrix whose entries are evaluations of a basis of polynomials at the $n$ predefined points in $F_q$.
- $c$ is the resulting $1 \times n$ row vector.

Matrix multiplication is a linear operation, (we also have [convolution code](https://en.wikipedia.org/wiki/Convolutional_code) but which is outof scope.)

This matrix is a Vandermonde matrix over $F_P$ ([read more](https://en.wikipedia.org/wiki/Reed%E2%80%93Solomon_error_correction#Constructions_(encoding):~:text=%5D-,This%20matrix%20is%20a%20Vandermonde%20matrix%20over,.,-Systematic%20encoding%20procedure)).

If the code $F$ is a linear function. That is, $F(a \cdot m_0 + b \cdot m_1) = a \cdot F(m_0) + b \cdot F(m_1)$ for any messages $m_0, m_1 \in F_{q}^{k}$ and scalars $a, b \in F_{q}$.

## What is an MDS Code?

An MDS Code (Maximum Distance Separable Code) is a type of error-correcting code that achieves the maximum possible minimum distance $d_{\text{min}}$ for a given codeword length $n$ and message  length $k$.

Key Properties of MDS Codes

1.	Maximum Minimum Distance: $d_{\text{min}} = n - k + 1$

    - $n$: The length of the codeword (number of symbols in the encoded block).
    - $k$: The length of the message  (number of information symbols in the block).
    - $d_{\text{min}}$: The minimum Hamming distance between any two codewords.

2.	Optimal Error-Correction:
    - MDS codes can detect $d_{\text{min}} - 1$ errors.
    - They can correct up to $\lfloor (d_{\text{min}} - 1) / 2 \rfloor$ errors.

3.	Singleton Bound:

    MDS codes achieve the theoretical maximum distance bound: $d_{\text{min}} \leq n - k + 1$

4.	Efficient Recovery:

    Even if up to n - k symbols are lost, the original message can still be fully reconstructed.

As the distance of RScode is also $d_{\text{min}} \leq n - k + 1$, which is a kind of MDS
