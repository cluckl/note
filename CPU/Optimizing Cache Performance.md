[TOC]



# Overview

* **cache**：存储最近访问的数据的自动管理内存(automatically managed fast memory)。

* **Block (line)**: Unit of storage in the cache

## Locality

局部性(locality)：

* Temporal locality (时间局部性)：Recently accessed data will be again accessed soon.
* Spatial locality (空间局部性)：Data near to the current memory location that is being fetched, may be needed soon.

## Cache hierarchy

单级cache无法满足**容量**和**速度**的双重要求，因此引入**Cache hierarchy**：

![$image-20210515151937700$](https://raw.githubusercontent.com/cluckl/Pinnned-repo/master/img/20210515151944.png)

第i级cache(非最外层的cache)的访问时间(access time)：
$$
T_i = h_i t_i + m_i(t_i + T_{i + 1})
= t_i + m_iT_{i + 1}
$$

# Placement

## Direct-Mapped Cache



## Full associative cache

## Set-Associative Cache



# Reducing miss rate

Performance means **AWAT**(Average memory access time):
$$
AWAT=( rate_{hit} \times latency_{hit} ) + ( rate_{miss} \times latency_{miss} )
$$

## Enhancements to associativity

### Victim caches

Use a small **fully associative buffer** (victim cache) to store **evicted blocks**.

![image-20210424234144587](https://raw.githubusercontent.com/cluckl/Pinnned-repo/master/img/image-20210424234144587.png)

> Jouppi , "Improving Direct Mapped Cache Performance by the Addition of a Small Fully Associative Cache and Prefetch Buffers", ISCA 1990 . 

* Avoid ping ponging of cache blocks mapped to the same set (if two cache blocks continuously accessed in nearby time conflict with each other)
* **Increases miss latency** if accessed serially with L2
* Adds complexity

### Hashing

Use better “randomizing” index functions

* Reduce conflict misses
  * by distributing the accessed memory blocks more evenly to sets
  * Example of conflicting accesses: stride access pattern where stride value equals number of sets in cache
* More complex to implement: **lengthen critical path**

### pseudo associativity



### Skewed associativity

