## Reference vs copy

Investigate the behavior of the statements below by looking at the values of
the arrays a and b after assignments:

```
a = np.arange(5)
b = a
b[2] = -1
b = a[:]
b[1] = -1
b = a.copy()
b[0] = -1
```


## Array creation

1. Start from a Python list containing both integers and floating point values,
and construct then a NumPy array from the list.

2. Generate a 1D NumPy array containing all numbers from -2.0 to 2.0 with a
spacing of 0.2. Use optional start and step arguments of the `np.arange()`
function.

3. Generate another 1D NumPy array containing 11 equally spaced values between
0.5 and 1.5. 

4. Take some Python string and construct from it NumPy array consisting of 
single characters (a character array).


import numpy as np

a = [4, 6, 21.12, 11, 0.23]
b = np.array(a)
print(b)

c = np.arange(-2.0, 2.01, 0.2)
print(c)

d = np.linspace(0.5, 1.5, 11)
print(d)

dna = 'ACGAATGCAACCGATC'
e = np.array(dna, dtype='c')
print(e)



## Array slicing

First, create a 4x4 array with arbitrary values, then

1. Extract every element from the second row.
2. Extract every element from the third column.
3. Assign a value of 0.21 to upper left 2x2 subarray.

Next, create a 8x8 array with checkerboard pattern, i.e. alternating zeros
and ones:

```
1 0 1 ...
0 1 0 ...
1 0 1 ...
 ...
```


import numpy as np

my_list = [[1.1, 1.2, 1.3, 1.4], 
           [2.1, 2.2, 2.3, 2.4],
           [3.1, 3.2, 3.3, 3.4],
           [4.1, 4.2, 4.3, 4.4],
           ]

arr = np.array(my_list)
print(arr)
print()
print(arr[1,:])
print()
print(arr[:,2])
print()
arr[:2, :2] = 0.21
print(arr)
print()

checker = np.zeros((8,8))
checker[::2, ::2] = 1
checker[1::2, 1::2] = 1
print(checker)


## Split and combine arrays

Create a new 8x8 array with some values (or continue with the one created in
the [Array slicing](../array-slicing) exercise).

1. Use `np.split()` function for splitting the array into two new 4x8 arrays.
   Reconstruct the original 8x8 array by using `np.concatenate()`.
2. Repeat the above exercise but create now 8x4 subarrays and then combine
   them.
   
   
import numpy as np

my_list = [[j +1 + (i + 1) / 10 for i in range(8)] for j in range(8)]

arr = np.array(my_list)
print(arr)
print()

sub1, sub2 = np.split(arr, 2)
print(sub1)
print()
print(sub2)
print()

orig = np.concatenate((sub1, sub2))
print(orig)
print()

sub1, sub2 = np.split(arr, 2, axis=1)
print(sub1)
print()
print(sub2)
print()

orig = np.concatenate((sub1, sub2), axis=1)
print(orig)
print()


## Subdiagonal matrix

Create a 6x6 matrix with 1’s above and below the diagonal and zeros
otherwise:
```
0 1 0 0 0 0
1 0 1 0 0 0
0 1 0 1 0 0
0 0 1 0 1 0
0 0 0 1 0 1
0 0 0 0 1 0
```
Use the `numpy.eye()` function.


import numpy as np

mat = np.eye(6, k=1) + np.eye(6, k=-1)
print(mat)
