---
title: "Standard Outputs of C"
tags: question c
last_modified_at: 2020-11-01
---

Review of common inputs: `printf`, `putchar`, `puts`.

Comparing to Standard Inputs, Standard Outputs are much simpler.

<!--more-->

## Basic Formats

1. What is the result of the following code?

   ```c
   #include <stdio.h>

   int main(void) {
       char a[5] = {'A', 'B', '\0', 'C', 'D'};
       puts(a);
       return 0;
   }
   ```

   <details><summary>Click to expand Answer</summary><div class="notice--success" markdown="1">
   ✔️ **Answer**
   <!--Answer Begin-->
   ```
   AB

   ```
   <!--Answer End-->
   <!--Description Begin-->
   `puts` or `printf("%s", ...)` stops when the terminating character (`'\0'`) is seen. If you forget to add a `'\0'` at the end of your string, the program might crash due to memory access violation.
   <!--Description End-->
   </div></details>


1. Replace `<REPLACE_HERE>` with a string format that can get the expected output.
   ```c
   #include <stdio.h>

   int main(void) {
       printf("<REPLACE_HERE>");
       return 0;
   }
   ```
   Expected Output:
   ```
   printf("%d\n", x);
   ```

   <details><summary>Click to expand Answer</summary><div class="notice--success" markdown="1">
   ✔️ **Answer**
   <!--Answer Begin-->
   ```
   printf(\"%%d\\n\", x);
   ```
   <!--Answer End-->
   <!--Description Begin-->
   `printf` requires escape characters to print certain special characters.
   - `\\` becomes `'\'`
   - `\"` becomes `'"'`
   - `%%` becomes `'%'`

   For the example above, we can print out the expected output easily by `puts`. But if you use `printf`, it's a pain in the a**.
   <!--Description End-->
   </div></details>


1. What is the result of the following code?

   ```c
   #include <stdio.h>

   int main(void) {
       int a, b;
       a = 101;
       b = 8787887;
       printf("%8d\n", a);
       printf("%8d\n", b);
       printf("%08d\n", a);
       printf("%08d", b);
       return 0;
   }
   ```

   <details><summary>Click to expand Answer</summary><div class="notice--success" markdown="1">
   ✔️ **Answer**
   <!--Answer Begin-->
   ```
        101
    8787887
   00000101
   08787887
   ```
   <!--Answer End-->
   <!--Description Begin-->
   This is a easy way to pad outputs with whitespaces or zeros.
   <!--Description End-->
   </div></details>


1. What is the result of the following code?

   ```c
   #include <stdio.h>

   int main(void) {
       float f;
       f = 878722e-4;
       printf("%f\n", f);
       printf("%.2f\n", f);
       printf("%.1f\n", f);
       printf("%.0f\n", f);
       printf("%.f", f);
       return 0;
   }
   ```

   <details><summary>Click to expand Answer</summary><div class="notice--success" markdown="1">
   ✔️ **Answer**
   <!--Answer Begin-->
   ```
   87.872200
   87.87
   87.9
   88
   88
   ```
   <!--Answer End-->
   <!--Description Begin-->
   The `xey` in float representation means $x\cdot 10^y$.

   If the precision isn't specified, the default is 6 digits after the decimal point.

   `printf` does the rounding for you.
   <!--Description End-->
   </div></details>


1. Replace `<REPLACE_HERE>` with string formats that can get the expected output.

   ```c
   #include <stdio.h>

   int main(void) {
       long long x, y;
       scanf("%lld%lld", &x, &y);
       printf("<REPLACE_HERE>\n", 20, 2*(unsigned long long)x);
       printf("<REPLACE_HERE>\n", 20, 2*(unsigned long long)y);
       return 0;
   }
   ```
   Input:
   ```
   9100000000000000000
   12

   ```
   Expected Output:
   ```
   18200000000000000000
   00000000000000000024

   ```

   <details><summary>Click to expand Answer</summary><div class="notice--success" markdown="1">
   ✔️ **Answer**
   <!--Answer Begin-->
   ```
   %0*llu
   ```
   <!--Answer End-->
   <!--Description Begin-->
   - `%lld` for `long long`
   - `%llu` for `unsigned long long`.

   `printf("%*d", NUM, ...)` replaces `*` to `NUM`.
   <!--Description End-->
   </div></details>

1. What is the result of the following code?

   ```c
   #include <stdio.h>

   int main(void) {
       putchar('\a');
       return 0;
   }
   ```

   <details><summary>Click to expand Answer</summary><div class="notice--success" markdown="1">
   ✔️ **Answer**
   <!--Answer Begin-->

   - In Console/Terminal

     Does not output any visible characters.

     If your computer's sound is on, you should hear a bell ringing sound or a "beep!", or some other strange noises.

   - In text editor (which is used by OJ)

     ```
     BEL
     ```

     ![]({{site.imgs}}{{page.id}}/bel.png)

     `BEL` indicates the bell character.
   <!--Answer End-->
   <!--Description Begin-->
   `\a` is stored as a character if the output is redirected to a file. The bell sound is the result of the terminal's interpretation of `\a`.

   For more information, please refer to the [ASCII Table](https://computersciencewiki.org/index.php/File:Ascii_table.png).
   <!--Description End-->
   </div></details>

1. What is the result of the following code?

   ```c
   #include <stdio.h>

   int main(void) {
       puts("helloworld!\b\b\b\b\b\b hell");
       return 0;
   }
   ```

   <details><summary>Click to expand Answer</summary><div class="notice--success" markdown="1">
   ✔️ **Answer**
   <!--Answer Begin-->

   - In Console/Terminal

     ```
     hello hell!
     ```

   - In text editor (which is used by OJ)

     ```
     helloworld!BSBSBSBSBSBS hell
     ```

     ![]({{site.imgs}}{{page.id}}/bs.png)

     `BS` indicates the backspace character. `\b` is stored as a character if the output is redirected to a file. The terminal's interpretation of `\b` is to move the cursor left by one character.

     Since OJ uses I/O redirection, you should not expect `\b` to work as you expected.
   <!--Answer End-->
   <!--Description Begin-->
   <!--Description End-->
   </div></details>


## Standard I/O Review

1. `scanf("%c", ...)`, `getchar` does not ignore leading whitespace characters.
2. `gets` does not store the terminating newline character; `fgets` stores the terminating newline character (if the input is terminated by newline instead of `EOF`).
3. when reading `EOF`, `scanf`, `getchar` returns `EOF`; `gets`, `fgets` return `NULL`.
4. When reading strings, remember to save an additional space for the easily forgotten `'\0'`.
5. Strings should be null-terminated (end with `'\0'`) before outputting using `printf("%s", ...)` or `puts`.

The list above is some mistakes that I see a lot of beginners make. If you see other special usages, you can search for them online. (such as `%x`, `%#x`, `%hd`, ...)

If you forget some of the I/O formats above in your exam (such as leading zero paddings), most of them can be replaced with additional `if` statements and loops.

For the next assignment, we'll review some basic syntaxes of C.

## Epilogue

![]({{site.imgs}}{{page.id}}/c_hello_world.jpg)

Photo Credit: Posted on [Reddit](https://www.reddit.com/r/ProgrammerHumor/comments/6xrxbm/hello_world_using_c/)