---
layout: post
title:  "Exploring HyperLogLog with Python"
date:   2023-08-08
categories: Biotechnology Longevity Bionics CRISPR Vaccine
---

![Line of Servers](/assets/servers.jpg)

In an era characterized by the explosive growth of digital information, managing and analyzing vast amounts of data has become a paramount challenge. The traditional methods of data processing and analysis, while effective for smaller datasets, often struggle to scale. Among the critical tasks in data analysis is the determination of cardinality — the count of distinct elements in a dataset. However, as datasets expand to unprecedented sizes, conventional cardinality estimation techniques reveal their limitations in terms of memory consumption and computational efficiency.

This is where the HyperLogLog algorithm (HLL) provides a unique solution. HLL is a probabilistic algorithm that tackles the cardinality estimation problem by allowing accurate approximations while significantly reducing memory requirements.

Consider a dataset with billions of distinct elements. Consider scenarios where you need to tally the quantity of individual customers accessing an online platform, unique IP addresses contributing to network traffic, or distinct keywords within a body of text. Traditional approaches would likely involve hash sets or arrays to store each unique element. However, these methods quickly consume memory, making them infeasible for large datasets.

The essence of HLL lies in probabilistic counting. Instead of storing each unique element, the algorithm employs hash functions to map elements to a fixed number of bits. These hashed values are sorted into different buckets and are then analyzed to estimate cardinality. HLL leverages the observation that, as the number of trailing zeros in the binary representation of a hash value increases, the likelihood of encountering that hash decreases exponentially.

Imagine each hash value for an element as a binary number, and the count of trailing zeros as a measure of its uniqueness. When many trailing zeros are found, it suggests that the element is likely to be rare and contributes to the overall diversity of the dataset. By looking at the lengths of these runs of trailing zeros in different buckets, HLL provides an estimation of the diversity of elements present, offering an intelligent method of counting without the necessity to consider each individual element separately.


&nbsp;

| Trailing Zeros (k) | Register Size (Bits) | Approximated Cardinality |
|---------------------|----------------------|-------------------------|
| 0 (2^0)             | 1                    | 1                       |
| 1 (2^1)             | 2                    | 2                       |
| 2 (2^2)             | 3                    | 4                       |
| 3 (2^3)             | 4                    | 8                       |
| 4 (2^4)             | 5                    | 16                      |

While this means of approximating cardinality is fairly accurate, outlier data points can still disrupt the accuracy of our estimates. Imagine one bizarre data point inflating the trailing zeros count. This could skew our overall cardinality estimation. To counteract this, HLL uses the harmonic mean of the trailing zero counts in each register. By using the harmonic mean instead of a regular average, more weight is granted to lower counts, which helps balance out the influence of outliers. This way, the impact of those rare elements on our cardinality estimation is minimized, resulting in a more reliable approximation.

Here’s a summary of how HLL works internally:

#### 1. Initialization.
Determine the desired precision by choosing the number of registers and the number of bits to represent the maximum number of trailing zeros.

#### 2. Assign Buckets.
Calculate a hash value for each unique element and allocate it to a specific register based on a portion of the hash value.

#### 3. Count Trailing Zeros.
Within each bucket, identify the longest sequence of trailing zeros in the binary representation of the hash value.

#### 4. Store Max Counts.
Keep track of the highest trailing zeros count found in each register. This count reflects the uniqueness of elements within that bucket.

#### 5. Estimate Cardinality.
Compute the harmonic mean of the maximum counts across all registers. This serves as an estimate of the total count of distinct elements in the dataset.

### Python Implementation
Here’s how to implement a simple Python version of the HyperLogLog algorithm:

Import the necessary modules:

```
import hashlib
import math
```

Define and initialize the `Hyperloglog` class:

```
class Hyperloglog:
    def __init__(self, num_registers):
        self.num_registers = num_registers
        self.registers = [0] * num_registers
```

This class represents the HLL data structure. It’s initialized with a given number of registers, each initially set to 0.

Define the `hash_to_register` method:

```
    def hash_to_register(self, hash_value):
        binary = bin(hash_value)[2:]
        return len(binary) - binary.rindex('1') - 1
```
This method takes a hash value, converts it to its binary representation, and determines the register index for that value.

Define the add_element method:

```
    def add_element(self, element):
        hash_value = hashlib.sha256(element.encode()).hexdigest()
        register_index = self.hash_to_register(int(hash_value, 16))
        self.registers[register_index] = max(self.registers[register_index], self.count_trailing_zeros(hash_value))
```

This method adds an element to the HLL data structure. It hashes the element using SHA-256, determines the corresponding register index, and updates the register by setting it to the maximum between its current value and the count of trailing zeros in the hash value’s binary representation.

Define the count_trailing_zeros method:

```
    def count_trailing_zeros(self, hash_value):
        return len(hash_value) - len(hash_value.rstrip('0'))
```

This method calculates the count of trailing zeros in the binary representation of a hash value.

Define the estimate_cardinality method:

```
    def estimate_cardinality(self):
        harmonic_mean = sum([2 ** -register for register in self.registers])
        raw_estimate = (self.num_registers ** 2) / harmonic_mean
        return self.apply_bias_correction(raw_estimate)
```

This method does the heavy lifting of calculating the estimated cardinality of the data stored in each register. It computes the harmonic mean of the values in the registers, applies a formula to get a raw estimate, and then applies bias correction. The formula (self.num_registers ** 2) / harmonic_mean is the crux of this raw estimation. It's derived from mathematical analysis that considers the total number of registers in the HyperLogLog data structure (self.num_registers). Squaring this value reflects the potential distribution of hash values across the registers, assuming an even spread.

Define the apply_bias_correction method:

```
  def apply_bias_correction(self, estimate):
        if estimate <= 5/2 * self.num_registers:
            zeros = self.registers.count(0)
            return self.num_registers * math.log(self.num_registers / zeros)
        elif estimate <= (1/30) * (2 ** 32):
            return estimate
        else:
            return -2 ** 32 * math.log(1 - (estimate / 2 ** 32))
```

This method applies bias correction to the estimated cardinality. It handles three different cases based on the estimate’s value. If the estimate is low, it logarithmically corrects for non-empty registers, enhancing precision. If the estimate is moderate, it returns the estimate as is. For large estimates, it employs a specific correction formula to mitigate overestimation. This process ensures accurate cardinality estimation across varying scenarios.

Use the Hyperloglog class:

```
if __name__ == "__main__":
    num_elements = 10_000
    num_registers = 1024
    hyperloglog = Hyperloglog(num_registers)
    for i in range(num_elements):
        hyperloglog.add_element(str(i))

    estimated_cardinality = hyperloglog.estimate_cardinality()
    print("Estimated Cardinality:", estimated_cardinality)
```
This section demonstrates use of the Hyperloglog class. It creates an instance of the class, adds elements to it, and then prints the estimated cardinality.

In terms of time complexity, the algorithm’s primary operations — hashing elements, updating registers, and calculating the harmonic mean — are generally performed in constant time (O(1)) per element. The estimation of cardinality through bias correction also involves straightforward calculations, contributing insignificantly to the overall time complexity. As a result, the algorithm’s time complexity is mainly influenced by the number of elements added, typically leading to a linear relationship (O(n)) between input size and computation time. In terms of space complexity, the algorithm’s memory consumption primarily depends on the number of registers used, translating to O(num_registers) space complexity.

### Conclusion
The HyperLogLog algorithm presents an elegant solution to the cardinality estimation problem by employing probabilistic counting techniques. It provides a memory-efficient means to estimate cardinality with a controlled trade-off between accuracy and memory usage. With its applications spanning across diverse fields, Hyperloglog serves as a prime example of how probabilistic algorithms can address complex challenges in data analysis and computation.

As data continues to grow in size and complexity, probabilistic data structures like HLL play a pivotal role in scalable and efficient data analysis. Whether it’s for optimizing database queries or gaining insights from massive data streams, HLL proves to be an invaluable tool for managing the increasing demands of today’s data intensive technical environment.

I’ve linked to some resources that I found useful in studying the HyperLogLog algorithm. I’m always happy to talk code so if you have any questions, comments, or catch any errors, contact me on [X](https://twitter.com/justgnnr).

### Resources


<iframe width="560" height="315" src="https://www.youtube.com/embed/eV1haPUt0NU?si=fE45xykNgfFxLKkd" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

&nbsp;

Flajolet, Philippe; Fusy, Éric; Gandouet, Olivier; Meunier, Frédéric (2007). [“Hyperloglog: The analysis of a near-optimal cardinality estimation algorithm”](http://algo.inria.fr/flajolet/Publications/FlFuGaMe07.pdf) (PDF). Discrete Mathematics and Theoretical Computer Science.

