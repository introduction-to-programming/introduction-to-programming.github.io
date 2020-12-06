---
title: "Basic Array and Pointers in C"
tags: question c
last_modified_at: 2020-12-06
---

Review of array and pointers.

<!--more-->

Shortcut for Quick Review:
- <a id="toggle-all-summary">Toggle all answers (Click to expand/collapse all Answers)</a>
<script>
// Reference the toggle link
var toggle = document.getElementById('toggle-all-summary');
toggle.addEventListener('click', function(e) {
  e.target.classList.toggle('toggle-state');
  var details = document.querySelectorAll('details');
  Array.from(details).forEach(function(obj, idx) {
    if (e.target.classList.contains('toggle-state')) {
      obj.open = true;
    } else {
      obj.open = false;
    }
  });
}, false);
</script>

<span style="color:red">Update (2020.12.05): Fix typo in description, (`&a` and `&a[0]` have different types, so `&a` is not an alias of `&a[0]`) (More on Pointers and Array)</span>

## Structure

We'll start by reviewing some basics about structures.

If we want to make a 2D game with different objects, we may need to have its position coordinates and size stored in arrays. Something like:

```c
float enemy_x[MAX];
float enemy_y[MAX];
float enemy_w[MAX];
float enemy_h[MAX];
float player_x;
float player_y;
float player_w;
float player_h;
```

However, it gets confusing once the variable count increases. So Structures come into rescue:

```c
struct object {
    float x, y, w, h;
};

struct object enemy[MAX];
struct object player;
```

and then, we can access the coordinates easily like:

```c
player.x = 0;
```

The `struct object` declaration is quite long and kind of redundant, so we can use `typedef`:

```c
struct object {
    float x, y, w, h;
};
typedef struct object Object;
Object enemy[MAX];
Object player;
```

We can even combine the declaration of `struct` with `typedef`.

```c
typedef struct object {
    float x, y, w, h;
} Object;
Object enemy[MAX];
Object player;
```

and even omit the `object` type:

```c
typedef struct {
    float x, y, w, h;
} Object;
Object enemy[MAX];
Object player;
```

By using Structures, we can define a special type for variable declaration afterward. More advanced usages such as [bit-fields](https://en.cppreference.com/w/c/language/bit_field), [union](https://en.cppreference.com/w/c/language/union), and [enumeration](https://en.cppreference.com/w/c/language/enum) can provide more flexible controls on Structures.

## Arrays

For using static arrays, we can only declare it with constant length.

1. Static array initialization

   What is the result of:

   ```c
   #include <stdio.h>
   #define MAX 3

   int a[MAX];
   int b[MAX] = {};
   int c[MAX] = {0};
   int d[MAX] = {1};
   int e[MAX] = {1, 1, 1};

   int main(void) {
       for (int i = 0; i < MAX; i++)
           printf("%d ", a[i]);
       for (int i = 0; i < MAX; i++)
           printf("%d ", b[i]);
       for (int i = 0; i < MAX; i++)
           printf("%d ", c[i]);
       for (int i = 0; i < MAX; i++)
           printf("%d ", d[i]);
       for (int i = 0; i < MAX; i++)
           printf("%d ", e[i]);
       return 0;
   }
   ```

   <details><summary>Click to expand Answer</summary><div class="notice--success" markdown="1">
   ✔️ **Answer**
   <!--Answer Begin-->
   ```
   0 0 0 0 0 0 0 0 0 1 0 0 1 1 1
   ```
   <!--Answer End-->
   <!--Description Begin-->
   All entries are initialized into zero when the bracket `= {}` initialization is used. Using `= {1}` is kind of like initialize all entries into zero but the first entry should be 1. So something like `= {0}` is actually a redundant version of `= {}`.
   <!--Description End-->
   </div></details>


2. Multi-dimensional Static Array

   {% raw %}
   <!-- since "{{1}.." is jekyll syntax -->

   What is the result of:

   ```c
   #include <stdio.h>
   #define N 3
   #define M 2

   int a[N][M] = {{1}, {2, 3}};

   int main(void) {
       for (int i = 0; i < N; i++) {
           for (int j = 0; j < M; j++) {
               printf("a[%d][%d] is %d\n", i, j, a[i][j]);
           }
       }
       return 0;
   }
   ```

   {% endraw %}

   <details><summary>Click to expand Answer</summary><div class="notice--success" markdown="1">
   ✔️ **Answer**
   <!--Answer Begin-->
   ```
   a[0][0] is 1
   a[0][1] is 0
   a[1][0] is 2
   a[1][1] is 3
   a[2][0] is 0
   a[2][1] is 0
   ```
   <!--Answer End-->
   <!--Description Begin-->
   This declaration can be seen as declaring a size-3 array, and each array cell is a size-2 array.

   ```c
   int (a[3])[2];
   ```
   <!--Description End-->
   </div></details>


3. Variable-Length Arrays (VLA)

   ```c
   #include <stdio.h>
   int main(void) {
       int N;
       scanf("%d", &N);
       int a[N];
       return 0;
   }
   ```

   You may see some code that declares an array as above, where `N` is not a constant value. They are called Variable-Length Arrays and are only supported after C99. There are some drawbacks to using VLAs, to avoid confusion, you can use the dynamic array as shown in the next section.

## Pointers

Misusing pointers can pose serious security threats to programs. You'll often see Buffer Overflow security fixes in Google Chrome, Mozilla Firefox, or any other applications. These security issues are mostly caused by pointers. Thus, in many high-level languages, the direct manipulation of pointers is forbidden.

If pointers are so dangerous, then why use pointers in the first place? This is because some cases require using pointers:

- Function returning more than one value
- Send arguments more efficiently
- Access dynamic allocated memory
- Implement data structures like linked lists, trees, etc.
- Call functions through a variable
- etc.

1. Function returning more than one value

   Before starting let's take a look at the snippet below:

   ```c
   int a, b;
   a = 1;
   b = a;
   ```

   For the second line, `a` is on the left-hand side of the assignment operator. The value `1` is written to `a`'s memory address.

   For the third line, `a` is on the right-hand side of the assignment operator. The value of `a` is read from its memory address and written to `b`'s memory address (I'll ignore registers and caches here for the sake of simplicity).

   If we want to change `a`'s value through a function call, we can only assign it by follows:

   ```c
   int a;
   a = func_a();
   ```

   If we want to change multiple variables' value, we may want to type the following:

   ```py
   a, b = func()
   ```

   which works in Python, but not in C. So, we need to find a way to send the variables' address into the function, and let the code inside the function change its value. However, something like:

   ```c
   int a, b;
   func(a, b);
   ```

   doesn't work since the value is read from the memory addresses before sending into the function, if we change the variables inside the function, it only changes the value of the local variables inside that function. So, we send their memory addresses instead:

   ```c
   int a, b;
   func(&a, &b);
   ```

   which is how `scanf` works, and the return value can now be used to indicate whether the function runs successfully.

   In the case above, we wanted to change multiple variables' value. What if the value we want to change is a memory address?

   ```c
   int* a;
   a = malloc(sizeof(int));
   free(a);
   ```

   If we don't want to use the return value of the function, then it'll become something like:

   ```c
   #include <stdio.h>
   #include <stdlib.h>

   void my_malloc(int** a, size_t size) {
       *a = malloc(size);
   }

   int main(void) {
       int* a;
       my_malloc(&a, sizeof(int));
       free(a);
       return 0;
   }
   ```

   Sending the pointer `a` doesn't work here since we can only change the value stored in the memory address stored in `a` (i.e., `*a`). If we want to change the memory address stored in `a` (`a`), we need to send `a`'s memory address into the function.

   This double-pointer allocation is commonly used in CUDA programming and also used a lot when interacting with Windows native APIs.

2. Send arguments more efficiently

   What's the output of:

   ```c
   #include <stdio.h>
   #define MAX 100

   typedef struct {
       int content[MAX];
   } LargeArray;

   void func(LargeArray la) {
       printf("sizeof la is %zu\n", sizeof la);
   }

   void funcp(LargeArray* pla) {
       printf("sizeof pla is %zu\n", sizeof pla);
       printf("sizeof la is %zu\n", sizeof *pla);
   }

   int main(void) {
       LargeArray la;
       func(la);
       funcp(&la);
       return 0;
   }
   ```

   <details><summary>Click to expand Answer</summary><div class="notice--success" markdown="1">
   ✔️ **Answer**
   <!--Answer Begin-->
   ```
   sizeof la is 400
   sizeof pla is 8
   sizeof la is 400
   ```
   <!--Answer End-->
   <!--Description Begin-->
   You can see that the variable `la` is quite large and requires an additional copy step to send it into `func`. But we can send the structure's pointer instead, so we only need to copy `la`'s address, which is much smaller.
   <!--Description End-->
   </div></details>


3. Access dynamic allocated memory

   We can allocate dynamic resources for later usages. For example a 1d dynamic array:

   ```c
   #include <stdio.h>
   #include <stdlib.h>
   #define MAX 100

   int main(void) {
       int* a;
       a = malloc(sizeof(int) * MAX);
       free(a);
       return 0;
   }
   ```

   Freeing (`free`) any allocated (`malloc`) resources is a good coding practice. Forgetting to free allocated resources results in memory leaks, which is a major issue for products and applications.

   When developing applications/games, we may want to allocate some resources when doing a certain operation, where this operation is repetitively called. If we forget to free these resources, the program will keep asking for more memory spaces from the OS, and eventually use up all memory and crashes. (Sometimes the OS kills the program, but sometimes the OS crashes due to Out Of Memory issue (OOM)) A dangerous code is shown below, which may crash your OS:

   ```c
   #include <stdio.h>
   #include <stdlib.h>
   #define MAX 1000000

   void game_update() {
       int* a;
       a = malloc(sizeof(int) * MAX);
       // Do something...
       // and forget to free this resource.
       // free(a);
   }

   int main(void) {
       while (1) {
           game_update();
       }
       return 0;
   }
   ```

4. Implement data structures like linked lists, trees, etc.

   Linked List:

   ```c
   typedef struct _node {
       int value;
       struct _node* next;
   } Node;
   ```

   Binary Tree:

   ```c
   typedef struct _node {
       int value;
       struct _node *left, *right;
   } Node;
   ```

5. Call functions through a variable

   What is the result of:

   ```c
   #include <stdio.h>
   #include <stdlib.h>
   #define MAX 1000000

   void add(int a, int b) {
       printf("add : %d\n", a+b);
   }
   void mult(int a, int b) {
       printf("mult: %d\n", a*b);
   }

   int main(void) {
       void (*fptr) (int, int);
       fptr = &add;
       fptr(2, 3);
       fptr = &mult;
       fptr(2, 3);
       return 0;
   }
   ```

   <details><summary>Click to expand Answer</summary><div class="notice--success" markdown="1">
   ✔️ **Answer**
   <!--Answer Begin-->
   ```
   add : 5
   mult: 6
   ```
   <!--Answer End-->
   <!--Description Begin-->
   This increases a lot of flexibility! The native function `qsort` in `stdlib.h` uses a function pointer to let the programmer define its own comparison function for sorting arrays of different types/structures.
   <!--Description End-->
   </div></details>

## More on Pointers and Array

Arrays are not equal to Pointers, they are similar but not the same. In most of the time, they can be used in the same way.

This part is the most interesting, we'll start by reviewing basic operators.

Operator review:

- `&var` returns `var`'s memory address.
- `*ptr = c` assign `c` to the address where `ptr` is pointing.
- `r = *ptr` takes out the value where `ptr` is pointing and store it in `r`.
- `ptr++` Move the pointer forward for `sizeof(*ptr)`-byte, that is, to move n-bytes, where n is the size of the type that `ptr` is pointing to.

  e.g. `int* ptr`, if `ptr == 0x00`, `ptr+1 == 0x04`, assuming `sizeof(int)` is 4.
- The arithmetic of `a` is the same with `&a[0]`. (However, they have different types, the former is an array, the latter is a pointer)

  e.g. `int a[100]`, if `a == 0x00`, `&a[0] == 0x00`, `a+1 == 0x04`, `&a[0]+1 == 0x04` , assuming `sizeof(int)` is 4.
- `a[i]` is an alias of `*(a+i)`.

  So, `i[a]` is valid! (which is equal to `a[i]`)
- `a[i][j]` is an alias of `*(*(a+i)+j)`.
- `struct_ptr->var` is an alias of `(*struct_ptr).var`.

1. Size of Pointers and Arrays.

   What is the output of:

   ```c
   #include <stdio.h>
   #include <stdlib.h>
   #define N 2
   #define M 3

   int main(void) {
       // Declaration
       int i, j, *tmp;
       int a[N*M];
       int b[N][M];
       int c[M][N];
       int *pa = malloc(sizeof(int) * M * N);
       int **pb = malloc(sizeof(int*) * N);
       int **pc = malloc(sizeof(int*) * M);
       // Allocate second level resources all at once to reduce allocation overhead.
       tmp = malloc(sizeof(int) * M * N);
       for (i = 0; i < N; i++)
           pb[i] = tmp + i * M;
       tmp = malloc(sizeof(int) * M * N);
       for (i = 0; i < M; i++)
           pc[i] = tmp + i * N;
       // Print
       printf("sizeof(int)       : %2zu\n", sizeof(int));
       printf("sizeof(int[M])    : %2zu\n", sizeof(int[M]));
       printf("sizeof(int[N])    : %2zu\n", sizeof(int[N]));
       printf("sizeof(int[N][M]) : %2zu\n", sizeof(int[N][M]));
       printf("sizeof(int[M][N]) : %2zu\n", sizeof(int[M][N]));
       printf("sizeof(void*)     : %2zu\n", sizeof(void*));
       printf("sizeof(int*)      : %2zu\n", sizeof(int*));
       printf("sizeof(int**)     : %2zu\n", sizeof(int**));
       printf("sizeof   a        : %2zu\n", sizeof a);
       printf("sizeof  *a        : %2zu\n", sizeof *a);
       printf("sizeof   b        : %2zu\n", sizeof b);
       printf("sizeof  *b        : %2zu\n", sizeof *b);
       printf("sizeof **b        : %2zu\n", sizeof **b);
       printf("sizeof   c        : %2zu\n", sizeof c);
       printf("sizeof  *c        : %2zu\n", sizeof *c);
       printf("sizeof **c        : %2zu\n", sizeof **c);
       printf("sizeof   pa       : %2zu\n", sizeof pa);
       printf("sizeof  *pa       : %2zu\n", sizeof *pa);
       printf("sizeof   pb       : %2zu\n", sizeof pb);
       printf("sizeof  *pb       : %2zu\n", sizeof *pb);
       printf("sizeof **pb       : %2zu\n", sizeof **pb);
       printf("sizeof   pc       : %2zu\n", sizeof pc);
       printf("sizeof  *pc       : %2zu\n", sizeof *pc);
       printf("sizeof **pc       : %2zu\n", sizeof **pc);
       // Free
       free(pa);
       free(pb[0]);
       free(pb);
       free(pc[0]);
       free(pc);
       return 0;
   }
   ```

   <details><summary>Click to expand Answer</summary><div class="notice--success" markdown="1">
   ✔️ **Answer**
   <!--Answer Begin-->
   ```
   sizeof(int)       :  4
   sizeof(int[M])    : 12
   sizeof(int[N])    :  8
   sizeof(int[N][M]) : 24
   sizeof(int[M][N]) : 24
   sizeof(void*)     :  8
   sizeof(int*)      :  8
   sizeof(int**)     :  8
   sizeof   a        : 24
   sizeof  *a        :  4
   sizeof   b        : 24
   sizeof  *b        : 12
   sizeof **b        :  4
   sizeof   c        : 24
   sizeof  *c        :  8
   sizeof **c        :  4
   sizeof   pa       :  8
   sizeof  *pa       :  4
   sizeof   pb       :  8
   sizeof  *pb       :  8
   sizeof **pb       :  4
   sizeof   pc       :  8
   sizeof  *pc       :  8
   sizeof **pc       :  4
   ```
   <!--Answer End-->
   <!--Description Begin-->
   First, remember, pointers are just variables that store memory addresses, so `void*`, `int*`, `int**`, or even `int*****` should be 8-bytes, assuming using a 64-bit computer.
   Let's explain each results:

   |Target|Type|Size|
   |--------------------|--|--|
   |`sizeof   a    : 24`|`int[6]`|`sizeof(int[6])`|
   |`sizeof  *a    : 4`|`int`|`sizeof(int)`|
   |`sizeof   b    : 24`|`int[2][3]`|`sizeof(int[2][3])`|
   |`sizeof  *b    : 12`|`int[3]`|`sizeof(int[3])`|
   |`sizeof **b    : 4`|`int`|`sizeof(int)`|
   |`sizeof   c    : 24`|`int[3][2]`|`sizeof(int[3][2])`|
   |`sizeof  *c    : 8`|`int[2]`|`sizeof(int[2])`|
   |`sizeof **c    : 4`|`int`|`sizeof(int)`|
   |`sizeof   pa   : 8`|`int*`|`sizeof(int*)`|
   |`sizeof  *pa   : 4`|`int`|`sizeof(int)`|
   |`sizeof   pb   : 8`|`int**`|`sizeof(int**)`|
   |`sizeof  *pb   : 8`|`int*`|`sizeof(int*)`|
   |`sizeof **pb   : 4`|`int`|`sizeof(int)`|
   |`sizeof   pc   : 8`|`int**`|`sizeof(int**)`|
   |`sizeof  *pc   : 8`|`int*`|`sizeof(int*)`|
   |`sizeof **pc   : 4`|`int`|`sizeof(int)`|

   <!--Description End-->
   </div></details>


2. Array as Arguments

   What is the output of:

   ```c
   #include <stdio.h>
   #include <stdlib.h>

   void func1(int a[]) {
       printf("%zu\n", sizeof a);
   }
   void func2(int *a) {
       printf("%zu\n", sizeof a);
   }

   int main(void) {
       int a[100];
       printf("%zu\n", sizeof a);
       func1(a);
       func2(a);
       return 0;
   }
   ```

   <details><summary>Click to expand Answer</summary><div class="notice--success" markdown="1">
   ✔️ **Answer**
   <!--Answer Begin-->
   ```
   400
   8
   8

   ```
   <!--Answer End-->
   <!--Description Begin-->
   Note that `sizeof a` is determined by the compiler [when compiling to assembly](https://stackoverflow.com/q/671790/).

   The `int a[]` is just syntactic sugar for `int* a`, they are both pointers (array decayed into pointer).
   <!--Description End-->
   </div></details>

3. Global array vs. Local array

   What is the difference between

   ```c
   #include <stdio.h>
   #define MAX 100000000

   int main(void)
   {
       int a[MAX];
       a[MAX-1] = 10;
       printf("%d\n", a[MAX-1]);
       return 0;
   }
   ```

   and

   ```c
   #include <stdio.h>
   #define MAX 100000000

   int a[MAX];

   int main(void)
   {
       a[MAX-1] = 10;
       printf("%d\n", a[MAX-1]);
       return 0;
   }
   ```

   <details><summary>Click to expand Answer</summary><div class="notice--success" markdown="1">
   ✔️ **Answer**
   <!--Answer Begin-->
   The first code segment (`a` defined in `main`) may result in Stack Overflow (which shows Segmentation Fault), while the second one (`a` in global) will not cause Stack Overflow.
   <!--Answer End-->
   <!--Description Begin-->

   More details on this can be learned in the Computer Architecture course.
   <!--Description End-->
   </div></details>

3. Pointer Compatibility

   ```c
   int * pt;
   int (*pa)[3];
   int ar1[2][3];
   int ar2[3][2];
   int **p2;
   int *arp[3];
   ```

   Which of the operations below are valid?

   ```c
   pt = &ar1[0][0];
   pt = ar1[0];
   pt = ar1;
   pa = ar1;
   pa = ar2;
   p2 = &pt;
   *p2 = ar2[0];
   p2 = ar2;
   pa = arp;
   ```

   <details><summary>Click to expand Answer</summary><div class="notice--success" markdown="1">
   ✔️ **Answer**
   <!--Answer Begin-->

   |Target|Valid?|L-Type|R-Type|
   |---|---|---|---|
   |`pt = &ar1[0][0];`|$\color{green} ✔$ Valid|pointer-to-int|pointer-to-int|
   |`pt = ar1[0];`|$\color{green} ✔$ Valid|pointer-to-int|pointer-to-int|
   |`pt = ar1;`|$\color{red} ✘$ Invalid|pointer-to-int|pointer-to-int[3]|
   |`pa = ar1;`|$\color{green} ✔$ Valid|pointer-to-int[3]|pointer-to-int[3]|
   |`pa = ar2;`|$\color{red} ✘$ Invalid|pointer-to-int[3]|pointer-to-int[2]|
   |`p2 = &pt;`|$\color{green} ✔$ Valid|pointer-to-int*|pointer-to-int*|
   |`*p2 = ar2[0];`|$\color{green} ✔$ Valid|pointer-to-int|pointer-to-int|
   |`p2 = ar2;`|$\color{red} ✘$ Invalid|pointer-to-int*|pointer-to-int[2]|
   |`pa = arp;`|$\color{red} ✘$ Invalid|pointer-to-int[3]|pointer-to-int*|

   <!--Answer End-->
   <!--Description Begin-->

   The pointer can only be assigned if the type matches. Something like `ar1[2][3]` cannot be seen as `int**`, but should be seen as pointer-to-int[3].

   For the difference of `pa` and `arp` here is that `pa` is a pointer-to-int[3], while `arp` is an array of pointers that can be seen as pointer-to-int*.
   <!--Description End-->
   </div></details>

4. Pointer Arithmetic

   What is the output of:

   ```c
   #include <stdio.h>

   int main(void)
   {
       int a[3] = {0, 1, 2};
       int *p;
       for (int i = 0; i < 3; i++) {
           p = a + i;
           printf("%td\n", (char*)p-(char*)a);
           printf("%td\n", (p-a));
       }
       return 0;
   }
   ```

   <details><summary>Click to expand Answer</summary><div class="notice--success" markdown="1">
   ✔️ **Answer**
   <!--Answer Begin-->
   ```
   0
   0
   4
   1
   8
   2

   ```
   <!--Answer End-->
   <!--Description Begin-->
   Note the difference between types.
   <!--Description End-->
   </div></details>

4. Pointing one past the last element of the array

   What is the output of:

   ```c
   #include <stdio.h>

   int main(void)
   {
       int a[3] = {0, 1, 2};
       int *p;
       for (int i = 0; i <= 3; i++) {
           p = a + i;
           printf("%td\n", (char*)(p)-(char*)a);
           printf("%td\n", (p-a));
       }
       return 0;
   }
   ```

   <details><summary>Click to expand Answer</summary><div class="notice--success" markdown="1">
   ✔️ **Answer**
   <!--Answer Begin-->
   ```
   0
   0
   4
   1
   8
   2
   12
   3

   ```
   <!--Answer End-->
   <!--Description Begin-->
   Pointing to one past the last element of the array is valid, as long as you do not dereference the pointer.
   <!--Description End-->
   </div></details>

5. Pointer Out-of-bounds

   What is the output of:

   ```c
   #include <stdio.h>

   int main(void)
   {
       int a[3] = {0, 1, 2};
       int *p;
       for (int i = 0; i <= 4; i++) {
           p = a + i;
           printf("%td\n", (char*)(p)-(char*)a);
           printf("%td\n", (p-a));
       }
       return 0;
   }
   ```

   <details><summary>Click to expand Answer</summary><div class="notice--success" markdown="1">
   ✔️ **Answer**
   <!--Answer Begin-->

   Undefined Behavior.
   <!--Answer End-->
   <!--Description Begin-->

   Pointing out-of-bounds is undefined behavior, even without dereferencing the pointer, according to C's standard.
   <!--Description End-->
   </div></details>

## Constants

- `const int* ptr` is the same as `int const* ptr`, which protects the value pointed.
   ```
   *ptr = 0; // Not allowed
   ptr = &a; // Allowed
   ```
- `int * const ptr`, which protects the pointer variable itself.
   ```
   *ptr = 0; // Allowed
   ptr = &a; // Not allowed
   ```
- `const int * const ptr` is the same as `int const * const ptr`.
   ```
   *ptr = 0; // Not allowed
   ptr = &a; // Not allowed
   ```

## Interpretation on Syntaxes

- Arrays

  For `int a[2][3]`, it can be interpreted from the inside to the outside. So `a` is an array with 2 cells. Each of the cells contains an `int[3]` element, that is, an array with 3 cells storing int.

- Pointers

  You may interpret it in 2 ways:
  1. `int* ptr`, `ptr` is a variable, and its type is `int*`, that is, a variable storing memory address.
  2. `int (*ptr)`, `ptr` is a pointer, and the pointed memory address represents an `int`.

  For `int (*pa)[3]`, it can also be interpreted from the inside to the outside. So `pa` is a pointer, pointing to an `int[3]` element.

  For `int *ap[3]`, it is an array of 3 cells. Each cell stores an `int*`, that is, a pointer-to-int. ([Left-Right Rule](http://cseweb.ucsd.edu/~ricko/rt_lt.rule.html))

There are much more tricky problems on pointers and arrays, but by thinking hard on the questions above and understanding the underlying reason for the results should be quite enough.

If you have trouble understanding a C pointer statement, try out this website:
- [cdecl](https://cdecl.org/)

Note that the result of `sizeof` operator in the questions above may have different outputs on different computers.

This should be the last assignment for this semester, I hope you have better understandings of the C language through finishing these six assignments.

## Epilogue

If you feel frustrated when trying to understand these concepts, it should be a good sign. Since it means that you reinforced these important concepts that cannot be learned just by writing codes for Online Judges.

![]({{site.imgs}}{{page.id}}/pointers.png)

Photo Credit: [Pointers](https://xkcd.com/138/) on [XKCD](https://xkcd.com/).