---
date: 2024-12-09
categories:
  - triton
tags:
  - notes
---

Notes for [Lecture 14: Practitioners Guide to Triton](https://www.youtube.com/watch?v=DdTsX6DQk24)

<!-- more -->

:fontawesome-solid-code: ***indicates that code snippet is included in the [attached notebook.](https://github.com/gpu-mode/lectures/blob/main/lecture_014/A_Practitioners_Guide_to_Triton.ipynb)*** 

### Is your AI model not fast enough? 
1. Not fast enough? `torch.compile` it.
	- `torch.compile` makes your model faster by trying to use existing kernels more effectively and creating simple new (Triton) kernels. 
1. Rewrite your code to make it more suitable for `torch.compile`.
2. Check which parts are slow and write custom Triton kernel(s) for those.
3. Check which parts are slow and write custom CUDA kernel(s) for those.

### How to write Triton kernels
- Debug Triton kernels: Set environment variable `TRITON_INTERPRET = 1`
  → Triton runs on the CPU, but simulates that it runs on the GPU. 
- Utility functions for debugging: :fontawesome-solid-code:
	- `check_tensors_gpu_ready`: (i) assert all tensors are contiguous in memory and (ii) assert all tensors are on gpu (when not simulating)
	- `breakpoint_if`: set a breakpoint, depending on conditions on pids
	- `print_if`: print sth, depending on conditions on pids

### Programming model 
- **CUDA** computation unit: `block > thread`
	- All threads in a block run on the same SM and share the same Shared Memory.
	- Each thread computes on ==scalars==. 
- **Triton** computation unit: `block`
	- Can't manage shared memory directly. Triton does that automatically.
	-  All operations perform on ==vectors==. 
- **Jargon**: In triton lingo, each kernel is called a "program". Similarly, "block_id" is called "pid". 
- **Example**: Add `x` and `y`(vector size: 6) and save the output into `z` . Block size is 4, so we have `cdiv(6, 4) = 2` blocks.

??? example "cuda"
    ```c
    # x,y = input tensors, z = output tensors, n = size of x, bs = block size
    def add_cuda_k(x, y, z, n, bs):
        # locate which part of the computation this specific kernel is doing
        block_id = ... # in our example: one of [0,1] 
        thread_id = ... # in our example: one of [0,1,2,3] 

        # identify the location of the data this specific kernel needs
        offs = block_id * bs + thread_id
        
        # guard clause, to make sure we're not going out of bounds
        if offs < n:
            # read data
            x_value = x[offs]
            y_value = y[offs]
            
            # do operation
            z_value = x_value + y_value
            
            # write data
            z[offs] = z_value

        # Important: offs, x_value, y_value, x_value are all scalars!
        # The guard clause is also (kind of) scalar op.
    ```
??? example "triton"
    ```python
    # Note: this is for illustration, and not quite syntactically correct. See further below for correct triton syntax

    def add_triton_k(x, y, z, n, bs):
        # locate which part of the computation this specific kernel is doing
        block_id = tl.program_id(0)  # in our example: one of [0,1] 
        
        # identify the location of the data this specific kernel needs
        offs = block_id * bs + tl.arange(0, bs) # <- this is a vector!
        
        # the guard clause becomes a mask, which is a vector of bools
        mask = offs < n # <- this is a vector of bools!
        
        # read data
        x_values = x[mask] # <- a vector is read!
        y_values = y[mask] # <- a vector is read!
        
        # do operation
        z_value = x_value + y_value  # <- vectors are added!
        
        # write data
        z[offs] = z_value  # <- a vector is written!
    ```

### Example 1: Copying a tensor

