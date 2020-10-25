---
title: "Standard Inputs of C"
tags: question c
last_modified_at: 2020-10-26
---

Review of common inputs: `scanf`, `getchar`, `gets` (`fgets`).

Remember that, when functions in the standard library, or some code snippets on the internet seems confusing, you can always refer to the following sites:
- [cplusplus.com](https://www.cplusplus.com/reference/)
- [cppreference.com](https://en.cppreference.com/w/c)
- [ISO Standard Drafts](https://stackoverflow.com/a/17015061)

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

## Basic Formats

1. What is the result of the following code?
   ```c
   #include <stdio.h>

   int main(void) {
       int a, b;
       scanf("%d%d", a, b);
       printf("%d %d", a, b);
       return 0;
   }
   ```

   Input:
   ```
   12 34

   ```

   <details><summary>Click to expand Answer</summary><div class="notice--success" markdown="1">
   ✔️ **Answer**
   <!--Answer Begin-->

   Undefined Behavior.
   <!--Answer End-->
   <!--Description Begin-->

   `scanf` requires **the memory address** of the variable.
   <!--Description End-->
   </div></details>


1. What is the result of the following code?
   ```c
   #include <stdio.h>

   int main(void) {
       int a, b;
       scanf("%1d%2d", &a, &b);
       printf("%d %d", a, b);
       return 0;
   }
   ```

   Input:
   ```
   1234 5678
   ```

   <details><summary>Click to expand Answer</summary><div class="notice--success" markdown="1">
   ✔️ **Answer**
   <!--Answer Begin-->
   ```
   1 23
   ```
   <!--Answer End-->
   <!--Description Begin-->
   The `%2d` part means to read in 2 digits.
   <!--Description End-->
   </div></details>

1. What is the result of the following code?

   ```c
   #include <stdio.h>

   int main(void) {
       int a, b;
       scanf("(%d,%d)", &a, &b);
       printf("%d %d", a, b);
       return 0;
   }
   ```

   Input:
   ```
   (12,34)

   ```

   <details><summary>Click to expand Answer</summary><div class="notice--success" markdown="1">
   ✔️ **Answer**
   <!--Answer Begin-->
   ```
   12 34
   ```
   <!--Answer End-->
   </div></details>
   <br/>
   What if the input becomes:
   ```
   (12, 34)

   ```

   <details><summary>Click to expand Answer</summary><div class="notice--success" markdown="1">
   ✔️ **Answer**
   <!--Answer Begin-->
   ```
   12 34
   ```
   <!--Answer End-->
   <!--Description Begin-->
   If we specify a certain character to match in `scanf`, the input must include that specific character.
   For example, using `(` in `scanf` indicates the next character must be `(`, the call will fail if the next character is space or `\n`, ...
   However, using `%d`, `%f`, ... automatically ignores every whitespace characters (space, `\n`, ...) before the number digits.
   `%c` is an exception that does read in whitespace characters.
   <!--Description End-->
   </div></details>

1. What is the result of the following code?
   ```c
   #include <stdio.h>

   int main(void) {
       float f;
       double d;
       scanf("%f%f", &f, &d);
       printf("%f %f", f, d);
       return 0;
   }
   ```

   <details><summary>Click to expand Answer</summary><div class="notice--success" markdown="1">
   ✔️ **Answer**
   <!--Answer Begin-->

   Undefined Behavior.
   <!--Answer End-->
   <!--Description Begin-->

   The input format should be `%lf` for double variables, using `%f` will leave half of the variable's memory undefined.
   <!--Description End-->
   </div></details>

## Usages of `scanf("%c", ...)` and `getchar`

1. What is the result of the following code?
   ```c
   #include <stdio.h>

   int main(void) {
       int i;
       char a[6];
       for (i = 0; i < 6; i++)
           scanf("%c", &a[i]);
       for (i = 0; i < 6; i++)
           printf("%d ", (int)a[i]);
       return 0;
   }
   ```

   Input:
   ```
   A B
   C

   ```
   Hint: `'A'` is `65`, `' '` is `32`, `'\n'` is `10`.

   <details><summary>Click to expand Answer</summary><div class="notice--success" markdown="1">
   ✔️ **Answer**
   <!--Answer Begin-->
   ```
   65 32 66 10 67 10
   ```
   <!--Answer End-->
   <!--Description Begin-->
   Using `%c` as the input format automatically reads in every character, including whitespace characters.
   The visualization of variable `a` is: `{'A', ' ', 'B', ' ', '\n', 'C'}`.
   For each variable, they are stored in its binary format in the memory, regardless of its type. Take `char` as an example, the characters are stored by their ASCII code. It should be straightforward to see that: `'A' + ' ' is 'a'`. (`65 + 32 = 97`)

   The `int` cast before printing out the character is used to extend the 1-byte variable to 4-bytes to avoid undefined behavior.
   <!--Description End-->
   </div></details>

1. What is the result of the following code?
   ```c
   #include <stdio.h>

   int main(void) {
       int i;
       char a[6];
       for (i = 0; i < 6; i++)
           scanf(" %c", &a[i]);
       for (i = 0; i < 6; i++)
           printf("%d ", (int)a[i]);
       return 0;
   }
   ```

   Input:
   ```
   AB C
   DEF G

   ```
   Hint: `'A'` is `65`, `' '` is `32`, `'\n'` is `10`.

   <details><summary>Click to expand Answer</summary><div class="notice--success" markdown="1">
   ✔️ **Answer**
   <!--Answer Begin-->
   ```
   65 66 67 68 69 70
   ```
   <!--Answer End-->
   <!--Description Begin-->
   The space character before `%c` ignores all whitespace characters.
   <!--Description End-->
   </div></details>

1. What is the result of the following code?
   ```c
   #include <stdio.h>

   int main(void) {
       int i;
       char a[6];
       for (i = 0; i < 6; i++)
           a[i] = getchar();
       for (i = 0; i < 6; i++)
           printf("%d ", (int)a[i]);
       return 0;
   }
   ```

   Input:
   ```
   AB C
   DEF G

   ```
   Hint: `'A'` is `65`, `' '` is `32`, `'\n'` is `10`.

   <details><summary>Click to expand Answer</summary><div class="notice--success" markdown="1">
   ✔️ **Answer**
   <!--Answer Begin-->
   ```
   65 66 32 67 10 68
   ```
   <!--Answer End-->
   <!--Description Begin-->
   `c = getchar();` is same as `scanf("%c", &c);`.
   <!--Description End-->
   </div></details>

## Usages of `scanf("%s", ...)`

1. What is the result of the following code?
   ```c
   #include <stdio.h>

   int main(void) {
       int i;
       char a[5] = {-1, -1, -1, -1, -1};
       scanf("%s", a);
       for (i = 0; i < 5; i++)
           printf("%d ", (int)a[i]);
       return 0;
   }
   ```

   Input:
   ```
    ABC DE
   FG

   ```
   Hint: `'A'` is `65`, `' '` is `32`, `'\n'` is `10`.

   <details><summary>Click to expand Answer</summary><div class="notice--success" markdown="1">
   ✔️ **Answer**
   <!--Answer Begin-->
   ```
   65 66 67 0 -1
   ```
   <!--Answer End-->
   <!--Description Begin-->
   The value of an array variable is its memory address, so using `scanf("%s", a)` should be enough. Of course, using `scanf("%s", &a)` is also valid.

   Using `%s` as the input format automatically ignores leading whitespace characters (like `%d`). It'll continuously read input characters until a whitespace character, and then add a `'\0'` (ASCII code 0) character indicating the termination of the string.
   The values after `'\0'` aren't affected.
   <!--Description End-->
   </div></details>

1. What is the result of the following code?
   ```c
   #include <stdio.h>

   int main(void) {
       int i;
       char a[3];
       scanf("%s", a);
       for (i = 0; i < 3; i++)
           printf("%d ", (int)a[i]);
       return 0;
   }
   ```

   Input:
   ```
   ABC

   ```
   Hint: `'A'` is `65`, `' '` is `32`, `'\n'` is `10`.

   <details><summary>Click to expand Answer</summary><div class="notice--success" markdown="1">
   ✔️ **Answer**
   <!--Answer Begin-->

   Undefined Behavior. (May result in a segmentation fault (`SEGFAULT`))
   <!--Answer End-->
   <!--Description Begin-->

   After reading `ABC`, `scanf` adds an additional `'\0'` at the end of the input. However, `a[3]` shouldn't be accessed in this case. If you tried to compile and run this code, the program may not crash most of the time, but the behavior is not defined in the C standard.
   This is the notorious Null-terminated string in C, which can easily lead to many bugs if the programmer is not careful enough.
   <!--Description End-->
   </div></details>

## `gets` vs. `fgets`

1. What is the result of the following code?
   ```c
   #include <stdio.h>

   int main(void) {
       int i;
       char s[6] = {-1, -1, -1, -1, -1, -1};
       fgets(s, 6, stdin);
       for (i = 0; i < 6; i++)
           printf("%d ", (int)s[i]);
       return 0;
   }
   ```

   Input:
   ```
   ABCDEF

   ```
   Hint: `'A'` is `65`, `' '` is `32`, `'\n'` is `10`.

   <details><summary>Click to expand Answer</summary><div class="notice--success" markdown="1">
   ✔️ **Answer**
   <!--Answer Begin-->
   ```
   65 66 67 68 69 0
   ```
   <!--Answer End-->
   </div></details>
   <br/>
   What if the input becomes:
   ```
   AB C

   ```

   <details><summary>Click to expand Answer</summary><div class="notice--success" markdown="1">
   ✔️ **Answer**
   <!--Answer Begin-->
   ```
   65 66 32 67 10 0
   ```
   <!--Answer End-->
   <!--Description Begin-->
   `fgets` reads in the ending newline character, so as other whitespace characters, including leading whitespaces!
   <!--Description End-->
   </div></details>

1. What is the result of the following code?
   ```c
   #include <stdio.h>

   int main(void) {
       int i;
       char s[6] = {-1, -1, -1, -1, -1, -1};
       gets(s);
       for (i = 0; i < 6; i++)
           printf("%d ", s[i]);
       return 0;
   }
   ```

   Input:
   ```
   AB C

   ```
   Hint: `'A'` is `65`, `' '` is `32`, `'\n'` is `10`.

   <details><summary>Click to expand Answer</summary><div class="notice--success" markdown="1">
   ✔️ **Answer**
   <!--Answer Begin-->
   ```
   65 66 32 67 0 -1
   ```
   <!--Answer End-->
   </div></details>
   <br/>
   What if the input becomes:
   ```
   ABCDEF

   ```

   <details><summary>Click to expand Answer</summary><div class="notice--success" markdown="1">
   ✔️ **Answer**
   <!--Answer Begin-->

   Undefined Behavior. (May result in a segmentation fault)
   <!--Answer End-->
   <!--Description Begin-->

   `gets` is like `fgets` but does not read in the newline character.
   `scanf("%[^\n]", s)` works the same as `gets` but the syntax may be difficult to memorize. `%[...]` tells `scanf` to read any number of the characters specified in the brackets. `^` inside the brackets indicates negation. So `%[^\n]` tells `scanf` to read all characters that aren't the newline character `\n`.
   <!--Description End-->
   </div></details>

## `scanf`’s Return Value

1. What is the result of the following code?
   ```c
   #include <stdio.h>

   int main(void) {
       int a, b, ret;
       a = b = -1;
       ret = scanf("%d%d", &a, &b);
       printf("%d %d %d", ret, a, b);
       return 0;
   }
   ```

   Input:
   ```
   100 abc 1000

   ```

   <details><summary>Click to expand Answer</summary><div class="notice--success" markdown="1">
   ✔️ **Answer**
   <!--Answer Begin-->
   ```
   1 100 -1
   ```
   <!--Answer End-->
   <!--Description Begin-->
   The return value of `scanf` indicates the number of variables that has been read successfully.
   Take the above code and input as an example, it reads in `100` and stores it into `a`, then it sees `abc`, which is not a number, so the matching fails. The value of `b` isn't modified due to the failed matching.
   In C, the return value of many library functions indicates whether that function ran without error.
   <!--Description End-->
   </div></details>

1. What is the result of the following code?
   ```c
   #include <stdio.h>

   int main(void) {
       int a, b, ret;
       a = b = -1;
       ret = scanf("(%d,%d)", &a, &b);
       printf("%d %d %d", ret, a, b);
       return 0;
   }
   ```

   Input:
   ```
   ( 1 , 2 )
   ```

   <details><summary>Click to expand Answer</summary><div class="notice--success" markdown="1">
   ✔️ **Answer**
   <!--Answer Begin-->
   ```
   1 1 -1
   ```
   <!--Answer End-->
   <!--Description Begin-->
   For `,`, whitespace characters are invalid, so the match fails.
   <!--Description End-->
   </div></details>

## End of File `EOF`

`EOF` is defined to be `-1` in most of the compiler implementations. When the input ends, `scanf` and `getchar` returns `EOF`.

For `gets` and `fgets`, they return `NULL`, which is `0` in most of the compiler implementations.

After interacting with a program on the command line / terminal, the user can tell the program that there will be no further inputs by signaling `EOF`. `EOF` is sent by pressing `Ctrl+Z` and `Enter` on Windows or pressing `Ctrl+D` and `Enter` on macOS or Linux.

For some common usages:

```c
while (scanf("%d", &x) != EOF) {
    // Do something.
}
while (scanf("%d%d", &a, &b) == 2) {
    // Do something.
}
while ((c = getchar()) != EOF) {
    // Do something.
}
// Change 'MAX_STRLEN' to a constant (max string length including null-terminator)
while (fgets(s, MAX_STRLEN, stdin) != NULL) {
    // Do something.
}
```

## A Toy Example

For reading a particular input format, there exist many different code snippets that can achieve the same result. However, one might be simpler than another.

For example, if we want to read a 5-digit number, with each of its digit stored in a different variable:

```
12345

```

And store them into 5 variables `a`, `b`, `c`, `d`, `e` respectively.

1. Math
   ```c
   int n;
   scanf("%d", &n);
   a = n / 10000 % 10;
   b = n / 1000 % 10;
   c = n / 100 % 10;
   d = n / 10 % 10;
   e = n / 1 % 10;
   ```

2. Scan 1-digit at a time
   ```c
   scanf("%1d%1d%1d%1d%1d", &a, &b, &c, &d, &e);
   ```

3. Scan by char
   ```c
   a = getchar() - '0';
   b = getchar() - '0';
   c = getchar() - '0';
   d = getchar() - '0';
   e = getchar() - '0';
   ```

4. Scan by string
   ```c
   char str[6];
   scanf("%s", &str);
   a = str[0] - '0';
   b = str[1] - '0';
   c = str[2] - '0';
   d = str[3] - '0';
   e = str[4] - '0';
   ```

5. And more...

For the next practice, we'll review the standard outputs of C.

## Epilogue

A piece of code that works on your computer doesn't necessarily mean that it will also run successfully on another person's computer. You should be able to identify and fix codes with undefined behaviors after finishing this practice.

![]({{site.imgs}}{{page.id}}/always-has-been.png)

Photo Credit: [Created on Meme Creator](https://meme-creator.com/editor).
