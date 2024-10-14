## Background

This file contains a walkthrough of my solutions for the WebGPU Puzzles. I have also created a walkthrough for the [official solutions]().

## Puzzle 1

```WGSL
@group(0) @binding(0) var<storage, read_write> a : array<f32>;
@group(0) @binding(1) var<storage, read_write> out : array<f32>;

@compute @workgroup_size({{workgroupSize}})
fn main(@builtin(local_invocation_id) lid: vec3<u32>) {
  out[lid.x] = a[lid.x] + 10;
}
```

```
Test case 1

Workgroup Size       ( 6, 1, 1 )
Total Workgroups     ( 1, 1, 1 )
Input a  [  0  1  2  3  4  5 ]

Test case 2

Workgroup Size       ( 10, 1, 1 )
Total Workgroups     ( 1, 1, 1 )
Input a  [  0  1  2  3  4  5  6  7  8  9 ]
```

For both test cases, `lid.x` sufficiently indexes into the input array `a` as it has 6 elements for Test case 1 and 10 elements for Test case 2.

## Puzzle 2

```WGSL
@group(0) @binding(0) var<storage, read_write> a : array<f32>;
@group(0) @binding(1) var<storage, read_write> b : array<f32>;
@group(0) @binding(2) var<storage, read_write> out : array<f32>;

@compute @workgroup_size({{workgroupSize}})
fn main(@builtin(local_invocation_id) lid: vec3<u32>) {

  if(lid.x < arrayLength(&a)) {
    out[lid.x] = a[lid.x] + b[lid.x]; 
  }
  
}
```

```
Test case 1

Workgroup Size       ( 4, 1, 1 )
Total Workgroups     ( 1, 1, 1 )

Input a  [  0  1 ]
Input b  [  2  3 ]


Test case 2

Workgroup Size       ( 10, 1, 1 )
Total Workgroups     ( 1, 1, 1 )

Input a  [  0  1  2  3  4 ]
Input b  [  5  6  7  8  9 ]
```

Similar to Puzzle #1, `lid.x` is sufficient to index into each input array and perform a elementwise sum operation. I added a guard rail (`if (lid.x < arrayLength(&a))`) since the workgroup size is larger than the array lengths in each Test case and I don't want to index out of bounds.

## Puzzle 3

```WGSL
@group(0) @binding(0) var<storage, read_write> a : array<f32>;
@group(0) @binding(1) var<storage, read_write> out : array<f32>;

@compute @workgroup_size({{workgroupSize}})
fn main(@builtin(local_invocation_id) lid: vec3<u32>) {
  let len = arrayLength(&a);
  
  if (lid.x < len) {
    out[lid.x] = a[lid.x] + 10;
  }
}
```

```
Test case 1

Workgroup Size       ( 10, 1, 1 )
Total Workgroups     ( 1, 1, 1 )

Input a  [  0  1  2  3  4  5 ]

Test case 2

Workgroup Size       ( 12, 1, 1 )
Total Workgroups     ( 1, 1, 1 )

Input a  [  0  1  2  3  4  5  6  7  8  9 ]
```

This is pretty much the same as Puzzle #1 except with a guard (`if (lid.x < len)`) so that we don't index out of bounds into the input array `a`.

## Puzzle 4

```WGSL
@group(0) @binding(0) var<storage, read_write> a : array<f32>;
@group(0) @binding(1) var<storage, read_write> out : array<f32>;

// wgs is a vec3 containing workgroup size in x, y, z.
// the x component is wgs.x, y component is wgs.y, and z component
// is wgs.z

const wgs = vec3({{workgroupSize}});

@compute @workgroup_size({{workgroupSize}})
fn main(@builtin(local_invocation_id) lid: vec3<u32>) {
  let i = wgs.x; 
  out[lid.x] = a[lid.x] + 10;
  out[lid.x + wgs.x] = a[lid.x + wgs.x] + 10;
  out[lid.x + wgs.x + 1] = a[lid.x + wgs.x + 1] + 10;
}
```

```
Test case 1

Workgroup Size       ( 3, 3, 1 )
Total Workgroups     ( 1, 1, 1 )

Input a  [  0  1  2  3 ]

Test case 2

Workgroup Size       ( 4, 4, 1 )
Total Workgroups     ( 1, 1, 1 )

Input a  [  0  1  2  3  4  5  6  7  8 ]
```

This was a hacky solution as at this point in my WebGPU journey I had no idea how to utilize the workgroup size or invocation IDs to cleverly index into an array when the number of positions in the input was larger than the number of positions in lid.x or lid.y.

This line:

```WGSL
out[lid.x] = a[lid.x] + 10;
```

Only populates the first three elements of `out`.

The next line:

```WGSL
out[lid.x + wgs.x] = a[lid.x + wgs.x] + 10;
```

Populates up to 6 elements for Test case 1 (lid.x has three elements + workgroup size x = 3) which passes the test. Test case 2 has 9 elements and lid.x + wgs.x only goes up to 8 elements so I needed to add `1` to pass Test case 2:

```WGSL
out[lid.x + wgs.x + 1] = a[lid.x + wgs.x + 1] + 10;
```

Ugly solution but that's what I was able to do at that point.

## Puzzle 5

```WGSL
@group(0) @binding(0) var<storage, read_write> a : array<f32>;
@group(0) @binding(1) var<storage, read_write> b : array<f32>;
@group(0) @binding(2) var<storage, read_write> out : array<f32>;

// workgroup sizes: wgs.x, wgs.y, wgs.z 
const wgs = vec3({{workgroupSize}});

@compute @workgroup_size({{workgroupSize}})
fn main(@builtin(local_invocation_id) lid: vec3<u32>) {
  //out[lid.x] = a[lid.x];
  //out[lid.x + wgs.x] = b[lid.x] + 1;
  //out[lid.x + 2 * wgs.x] = b[lid.x] + 2;
  //out[lid.x + 3 * wgs.x] = b[lid.x] + 3;
  out[lid.x + lid.y * wgs.x] = a[lid.x] + b[lid.y];
}
```

After reading the official solution to Puzzle #4, this was the first puzzle where I felt like I was actually utilizing workgroup size and location invocation IDs correctly. See the commented lines for how I built up this solution intuitively. The following screenshot shows how I visualized the indexing in Excel before arriving at the final solution:

![](screenshots/my_solution_puzzle_5.jpeg)