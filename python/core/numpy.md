# NumPy Library Cheatsheet

## Installation and Setup

```bash
# Install NumPy
pip install numpy

# Install with other scientific packages
pip install numpy scipy matplotlib pandas

# Import NumPy
import numpy as np
```

## Array Creation

### Basic Array Creation
```python
import numpy as np

# From Python lists
arr1d = np.array([1, 2, 3, 4, 5])
arr2d = np.array([[1, 2, 3], [4, 5, 6]])
arr3d = np.array([[[1, 2], [3, 4]], [[5, 6], [7, 8]]])

# Specify data type
arr_int = np.array([1, 2, 3], dtype=np.int32)
arr_float = np.array([1, 2, 3], dtype=np.float64)
arr_complex = np.array([1+2j, 3+4j], dtype=np.complex128)

print(arr1d.shape)      # (5,)
print(arr2d.shape)      # (2, 3)
print(arr3d.shape)      # (2, 2, 2)
```

### Built-in Array Creation Functions
```python
# Zeros and ones
zeros_arr = np.zeros((3, 4))           # 3x4 array of zeros
ones_arr = np.ones((2, 3, 4))          # 2x3x4 array of ones
full_arr = np.full((3, 3), 7)          # 3x3 array filled with 7

# Identity and eye matrices
identity = np.eye(4)                   # 4x4 identity matrix
diagonal = np.diag([1, 2, 3, 4])       # Diagonal matrix

# Range arrays
range_arr = np.arange(10)              # [0, 1, 2, ..., 9]
range_step = np.arange(2, 10, 2)       # [2, 4, 6, 8]
linspace_arr = np.linspace(0, 1, 5)    # [0, 0.25, 0.5, 0.75, 1.0]

# Random arrays
random_arr = np.random.random((3, 3))          # Random floats [0, 1)
random_int = np.random.randint(0, 10, (3, 3))  # Random integers [0, 10)
normal_arr = np.random.normal(0, 1, (3, 3))    # Normal distribution
```

## Array Properties and Information

```python
arr = np.array([[1, 2, 3], [4, 5, 6]])

# Basic properties
print(arr.shape)        # (2, 3) - dimensions
print(arr.size)         # 6 - total number of elements
print(arr.ndim)         # 2 - number of dimensions
print(arr.dtype)        # int64 - data type
print(arr.itemsize)     # 8 - size of each element in bytes
print(arr.nbytes)       # 48 - total bytes consumed

# Array info
print(np.info(arr))     # Detailed information
```

## Array Indexing and Slicing

### Basic Indexing
```python
arr1d = np.array([10, 20, 30, 40, 50])
arr2d = np.array([[1, 2, 3], [4, 5, 6], [7, 8, 9]])

# 1D array indexing
print(arr1d[0])         # 10 (first element)
print(arr1d[-1])        # 50 (last element)
print(arr1d[1:4])       # [20, 30, 40] (slice)

# 2D array indexing
print(arr2d[0, 0])      # 1 (row 0, column 0)
print(arr2d[1, 2])      # 6 (row 1, column 2)
print(arr2d[0])         # [1, 2, 3] (entire first row)
print(arr2d[:, 1])      # [2, 5, 8] (entire second column)

# Advanced slicing
print(arr2d[0:2, 1:3])  # [[2, 3], [5, 6]] (subarray)
print(arr2d[::2, ::2])  # [[1, 3], [7, 9]] (every other element)
```

### Boolean Indexing
```python
arr = np.array([1, 2, 3, 4, 5, 6])

# Boolean conditions
mask = arr > 3
print(mask)             # [False, False, False, True, True, True]
print(arr[mask])        # [4, 5, 6]

# Direct boolean indexing
print(arr[arr > 3])     # [4, 5, 6]
print(arr[arr % 2 == 0])  # [2, 4, 6] (even numbers)

# Multiple conditions
print(arr[(arr > 2) & (arr < 5)])  # [3, 4]
print(arr[(arr < 2) | (arr > 5)])  # [1, 6]
```

### Fancy Indexing
```python
arr = np.array([10, 20, 30, 40, 50])

# Index with arrays
indices = [0, 2, 4]
print(arr[indices])     # [10, 30, 50]

# 2D fancy indexing
arr2d = np.array([[1, 2, 3], [4, 5, 6], [7, 8, 9]])
rows = [0, 2]
cols = [1, 2]
print(arr2d[rows, cols])  # [2, 9] (elements at (0,1) and (2,2))
```

## Array Operations

### Arithmetic Operations
```python
arr1 = np.array([1, 2, 3, 4])
arr2 = np.array([5, 6, 7, 8])

# Element-wise operations
print(arr1 + arr2)      # [6, 8, 10, 12]
print(arr1 - arr2)      # [-4, -4, -4, -4]
print(arr1 * arr2)      # [5, 12, 21, 32]
print(arr1 / arr2)      # [0.2, 0.33, 0.43, 0.5]
print(arr1 ** 2)        # [1, 4, 9, 16]

# Operations with scalars
print(arr1 + 10)        # [11, 12, 13, 14]
print(arr1 * 3)         # [3, 6, 9, 12]

# Comparison operations
print(arr1 > 2)         # [False, False, True, True]
print(arr1 == arr2)     # [False, False, False, False]
```

### Mathematical Functions
```python
arr = np.array([1, 4, 9, 16])

# Basic math functions
print(np.sqrt(arr))     # [1, 2, 3, 4]
print(np.square(arr))   # [1, 16, 81, 256]
print(np.exp(arr))      # Exponential function
print(np.log(arr))      # Natural logarithm
print(np.log10(arr))    # Base-10 logarithm

# Trigonometric functions
angles = np.array([0, np.pi/2, np.pi])
print(np.sin(angles))   # [0, 1, 0] (approximately)
print(np.cos(angles))   # [1, 0, -1] (approximately)
print(np.tan(angles))   # [0, inf, 0] (approximately)

# Rounding functions
arr_float = np.array([1.2, 2.7, 3.1, 4.9])
print(np.round(arr_float))    # [1, 3, 3, 5]
print(np.floor(arr_float))    # [1, 2, 3, 4]
print(np.ceil(arr_float))     # [2, 3, 4, 5]
```

## Array Manipulation

### Reshaping Arrays
```python
arr = np.arange(12)     # [0, 1, 2, ..., 11]

# Reshape
reshaped = arr.reshape(3, 4)    # 3x4 array
print(reshaped.shape)           # (3, 4)

# Flatten
flattened = reshaped.flatten()  # 1D array
print(flattened.shape)          # (12,)

# Ravel (flatten view)
raveled = reshaped.ravel()      # 1D view

# Transpose
transposed = reshaped.T         # 4x3 array
print(transposed.shape)         # (4, 3)

# Add/remove dimensions
expanded = np.expand_dims(arr, axis=0)    # Add dimension
squeezed = np.squeeze(expanded)           # Remove dimensions of size 1
```

### Joining and Splitting Arrays
```python
arr1 = np.array([[1, 2], [3, 4]])
arr2 = np.array([[5, 6], [7, 8]])

# Concatenation
concat_vertical = np.concatenate([arr1, arr2], axis=0)    # Vertical
concat_horizontal = np.concatenate([arr1, arr2], axis=1)  # Horizontal

# Stack operations
vstacked = np.vstack([arr1, arr2])     # Vertical stack
hstacked = np.hstack([arr1, arr2])     # Horizontal stack
dstacked = np.dstack([arr1, arr2])     # Depth stack

# Splitting
arr = np.arange(12).reshape(3, 4)
split_vertical = np.vsplit(arr, 3)     # Split into 3 parts vertically
split_horizontal = np.hsplit(arr, 2)   # Split into 2 parts horizontally
```

## Statistical Operations

### Basic Statistics
```python
arr = np.array([[1, 2, 3], [4, 5, 6], [7, 8, 9]])

# Basic statistics
print(np.sum(arr))          # 45 (sum of all elements)
print(np.mean(arr))         # 5.0 (average)
print(np.median(arr))       # 5.0 (median)
print(np.std(arr))          # 2.58 (standard deviation)
print(np.var(arr))          # 6.67 (variance)
print(np.min(arr))          # 1 (minimum)
print(np.max(arr))          # 9 (maximum)

# Along specific axes
print(np.sum(arr, axis=0))  # [12, 15, 18] (sum along rows)
print(np.sum(arr, axis=1))  # [6, 15, 24] (sum along columns)
print(np.mean(arr, axis=0)) # [4, 5, 6] (mean along rows)

# Other statistics
print(np.argmin(arr))       # 0 (index of minimum)
print(np.argmax(arr))       # 8 (index of maximum)
print(np.percentile(arr, 50))  # 5.0 (50th percentile)
print(np.corrcoef(arr))     # Correlation coefficient matrix
```

### Aggregation Functions
```python
arr = np.array([1, 2, 3, 4, 5])

# Cumulative operations
print(np.cumsum(arr))       # [1, 3, 6, 10, 15] (cumulative sum)
print(np.cumprod(arr))      # [1, 2, 6, 24, 120] (cumulative product)

# All/any
bool_arr = np.array([True, False, True, True])
print(np.all(bool_arr))     # False (all True?)
print(np.any(bool_arr))     # True (any True?)

# Unique values
arr_with_duplicates = np.array([1, 2, 2, 3, 3, 3, 4])
print(np.unique(arr_with_duplicates))  # [1, 2, 3, 4]
unique, counts = np.unique(arr_with_duplicates, return_counts=True)
print(unique)               # [1, 2, 3, 4]
print(counts)               # [1, 2, 3, 1]
```

## Linear Algebra

### Matrix Operations
```python
# Matrix multiplication
A = np.array([[1, 2], [3, 4]])
B = np.array([[5, 6], [7, 8]])

# Dot product (matrix multiplication)
C = np.dot(A, B)            # or A @ B
print(C)                    # [[19, 22], [43, 50]]

# Element-wise multiplication
element_wise = A * B
print(element_wise)         # [[5, 12], [21, 32]]

# Matrix properties
print(np.trace(A))          # 5 (trace - sum of diagonal)
print(np.linalg.det(A))     # -2.0 (determinant)
print(np.linalg.matrix_rank(A))  # 2 (rank)
```

### Advanced Linear Algebra
```python
# Matrix decompositions
A = np.array([[4, 2], [1, 3]])

# Eigenvalues and eigenvectors
eigenvalues, eigenvectors = np.linalg.eig(A)
print(eigenvalues)          # [5, 2]
print(eigenvectors)         # Corresponding eigenvectors

# Singular Value Decomposition (SVD)
U, s, Vt = np.linalg.svd(A)
print(s)                    # Singular values

# Matrix inverse
A_inv = np.linalg.inv(A)
print(A_inv)
print(np.dot(A, A_inv))     # Should be identity matrix

# Solving linear systems (Ax = b)
b = np.array([1, 2])
x = np.linalg.solve(A, b)
print(x)                    # Solution to Ax = b
```

## Broadcasting

### Broadcasting Rules
```python
# Broadcasting allows operations between arrays of different shapes
arr1 = np.array([[1, 2, 3],
                 [4, 5, 6]])  # Shape: (2, 3)

arr2 = np.array([10, 20, 30])  # Shape: (3,)

# Broadcasting in action
result = arr1 + arr2  # arr2 is broadcast to (2, 3)
print(result)
# [[11, 22, 33],
#  [14, 25, 36]]

# Scalar broadcasting
result_scalar = arr1 * 2
print(result_scalar)

# Broadcasting with different dimensions
arr3 = np.array([[10],
                 [20]])  # Shape: (2, 1)
result_2d = arr1 + arr3
print(result_2d)
# [[11, 12, 13],
#  [24, 25, 26]]
```

## Array Input/Output

### Saving and Loading Arrays
```python
arr = np.array([1, 2, 3, 4, 5])

# Save to binary format
np.save('array.npy', arr)
loaded_arr = np.load('array.npy')

# Save multiple arrays
arr1 = np.array([1, 2, 3])
arr2 = np.array([4, 5, 6])
np.savez('arrays.npz', arr1=arr1, arr2=arr2)
loaded = np.load('arrays.npz')
print(loaded['arr1'])
print(loaded['arr2'])

# Save to text format
np.savetxt('array.txt', arr, delimiter=',')
loaded_text = np.loadtxt('array.txt', delimiter=',')

# CSV files
data = np.array([[1, 2, 3], [4, 5, 6]])
np.savetxt('data.csv', data, delimiter=',', header='col1,col2,col3')
loaded_csv = np.loadtxt('data.csv', delimiter=',', skiprows=1)
```

## Performance Tips and Best Practices

### Vectorization
```python
# Avoid loops - use vectorized operations
import time

# Slow way (with loops)
def slow_calculation(arr):
    result = np.zeros_like(arr)
    for i in range(len(arr)):
        result[i] = arr[i] ** 2 + 2 * arr[i] + 1
    return result

# Fast way (vectorized)
def fast_calculation(arr):
    return arr ** 2 + 2 * arr + 1

arr = np.random.random(1000000)

# Time comparison
start = time.time()
slow_result = slow_calculation(arr)
slow_time = time.time() - start

start = time.time()
fast_result = fast_calculation(arr)
fast_time = time.time() - start

print(f"Slow time: {slow_time:.4f} seconds")
print(f"Fast time: {fast_time:.4f} seconds")
print(f"Speedup: {slow_time/fast_time:.1f}x")
```

### Memory Efficiency
```python
# Use views instead of copies when possible
arr = np.arange(1000000)

# Creates a copy (uses more memory)
arr_copy = arr.copy()

# Creates a view (shares memory)
arr_view = arr[::2]  # Every other element

# Check if it's a view or copy
print(arr_view.base is arr)  # True if it's a view

# In-place operations save memory
arr += 1  # In-place addition
arr *= 2  # In-place multiplication

# Use appropriate data types
# int8 uses less memory than int64
small_arr = np.array([1, 2, 3], dtype=np.int8)
large_arr = np.array([1, 2, 3], dtype=np.int64)
print(f"int8 size: {small_arr.nbytes} bytes")
print(f"int64 size: {large_arr.nbytes} bytes")
```

## Common Patterns and Recipes

### Data Processing Patterns
```python
# Replace values
arr = np.array([1, 2, -1, 4, -5, 6])
arr[arr < 0] = 0  # Replace negative values with 0
print(arr)  # [1, 2, 0, 4, 0, 6]

# Clipping values
arr = np.array([1, 5, 10, 15, 20])
clipped = np.clip(arr, 5, 15)  # Clip to range [5, 15]
print(clipped)  # [5, 5, 10, 15, 15]

# Binning data
data = np.array([1.2, 2.7, 3.1, 4.9, 5.5])
bins = np.array([0, 2, 4, 6])
binned = np.digitize(data, bins)
print(binned)  # [1, 2, 2, 3, 3]

# Finding elements
arr = np.array([1, 2, 3, 2, 4, 2, 5])
indices = np.where(arr == 2)
print(indices[0])  # [1, 3, 5] (indices where value is 2)

# Conditional selection
result = np.where(arr > 3, arr, 0)  # Keep values > 3, else 0
print(result)  # [0, 0, 0, 0, 4, 0, 5]
```

### Advanced Indexing Patterns
```python
# Multi-dimensional boolean indexing
arr2d = np.array([[1, 2, 3], [4, 5, 6], [7, 8, 9]])
mask = arr2d > 5
print(arr2d[mask])  # [6, 7, 8, 9]

# Advanced fancy indexing
# Select specific rows and columns
rows = [0, 2]
cols = [1, 2]
selected = arr2d[np.ix_(rows, cols)]  # Select intersection
print(selected)  # [[2, 3], [8, 9]]

# Random sampling
arr = np.arange(100)
random_indices = np.random.choice(len(arr), 10, replace=False)
random_sample = arr[random_indices]
print(random_sample)  # 10 random elements without replacement
```

## Integration with Other Libraries

### NumPy and Pandas
```python
import pandas as pd

# Convert NumPy array to Pandas DataFrame
arr = np.array([[1, 2, 3], [4, 5, 6]])
df = pd.DataFrame(arr, columns=['A', 'B', 'C'])
print(df)

# Convert Pandas DataFrame to NumPy array
df_values = df.values  # or df.to_numpy()
print(df_values)
print(type(df_values))  # <class 'numpy.ndarray'>
```

### NumPy and Matplotlib
```python
import matplotlib.pyplot as plt

# Generate data for plotting
x = np.linspace(0, 2*np.pi, 100)
y1 = np.sin(x)
y2 = np.cos(x)

# Create plot
plt.figure(figsize=(10, 6))
plt.plot(x, y1, label='sin(x)')
plt.plot(x, y2, label='cos(x)')
plt.legend()
plt.title('Trigonometric Functions')
plt.xlabel('x')
plt.ylabel('y')
plt.grid(True)
plt.show()
```

## Common Errors and Debugging

### Shape Mismatch Errors
```python
# Common error: incompatible shapes
try:
    arr1 = np.array([[1, 2, 3]])      # Shape: (1, 3)
    arr2 = np.array([[1], [2], [3]])  # Shape: (3, 1)
    result = arr1 + arr2  # This works due to broadcasting
    print("Success:", result.shape)   # (3, 3)
    
    arr3 = np.array([[1, 2], [3, 4]])  # Shape: (2, 2)
    result2 = arr1 + arr3  # This will fail
except ValueError as e:
    print("Error:", e)

# Debug shapes
print(f"arr1 shape: {arr1.shape}")
print(f"arr2 shape: {arr2.shape}")
print(f"arr3 shape: {arr3.shape}")
```

### Data Type Issues
```python
# Integer division vs float division
arr_int = np.array([1, 2, 3, 4], dtype=int)
arr_float = np.array([1, 2, 3, 4], dtype=float)

print(arr_int / 2)      # [0.5, 1.0, 1.5, 2.0] (float result)
print(arr_int // 2)     # [0, 1, 1, 2] (integer division)

# Overflow issues
small_int = np.array([100], dtype=np.int8)  # Max value: 127
print(small_int * 2)    # Overflow! Results in -56

# Safe operations
large_int = np.array([100], dtype=np.int32)
print(large_int * 2)    # 200 (no overflow)
```