---
title: Ode to Pointers
description: Just me waxing nostalgic about using pointers.
date: 2025-09-10 13:45:00 +0000
categories: [Code, Other]
author: rholder
tags: [c,c++,pointers]
image:
  path: /assets/preview_images/Pointers-1200x630.jpg
---

Sometimes when I'm coding or trying to solve a problem using C# I have flashbacks from my days of C\C++ programming of using pointers. For the uninitiated, a *pointer* is simply a variable which stores the address of another variable. This address can then be used to access the value stored at that location through a process known as dereferencing.  

I have fond memories of learning to use the **malloc()** function and **sizeof()** operator to manually allocate a block of memory which would be referenced using a pointer. The code would look something like this:

```c
#include <stdio.h>
#include <stdlib.h>

#define MAX_LENGTH 20

int main()
{
  // pointer declaration
  char *buffer; 
  
  // memory allocation
  buffer = (char*) malloc(MAX_LENGTH * sizeof(char));

  // utilise buffer in code block if allocation was successful
  if(buffer != NULL)
  {
    // insert code which uses buffer here
  }
  else
  {
    printf("Memory allocation failed.\n");
    exit(0);
  }

  // release the memory after we're done
  free(buffer);

  return 0;
}

```


I also have fond memories of using pointers to traverse an array using a combination of the **indirection (*)** and **increment (++)** operators. The code snippet below demonstrates a simple example of this:

```c
#include <stdio.h>

int main()
{
  int *p;
  int array[5] = {100, 220, 210, 560, 780};
  
  // assign address of first element of array 
  p = &array[0];
  
  printf("Address  Value\n");
  
  for(int i=0; i < 5;i++)
  {
    // print address stored in pointer and the value stored at the address
    printf("%p  %d\n", p, *p);

    // set pointer to point to next contiguous memory location by incrementing it
    p++;
  }
  
  return 0;
}
```


In the code, the pointer p is assigned the address of the first element of the array using the **address(&)** operator. The **indirection (*)** operator is used to dereference the pointer in order to get the value stored at the address. Applying the increment operator to the pointer increments the address thereby causing it to point to the next element of the array. The address stored in the pointer can be viewed by applying the special C language format character **%p**. 

The syntax of course can be shortened to *p++, but this will involve some slight rejigging of the code to print out the correct address value at step i because the updated value of p will be shown if the code is left as is. This code produces the following output:

![ArrayPointerOutput](/assets/posts/20250910/ArrayPointerOutput.jpg){: width="268" height="117"}
_Addresses of the array elements and their dereferenced values_


Seeing pointer syntax again just makes me feel all warm and fuzzy on the inside. Books and other reference materials on C would always warn about pointers being hard to understand and master, but I found that once the core ideas and how they related back to the syntax were understood it was fairly plain sailing. What could really throw a spanner into the works was dealing with *pointers to pointers*. The syntax could become a bit hairy there.

The example below declares a pointer to a pointer and outputs the result of dereferencing it. The values of the other variables are also output to give a clearer understanding of what exactly is happening:

```c
#include <stdio.h>

int main()
{
  int *ptr;         // normal pointer
  int **ptr_to_ptr; // pointer to pointer declared using **
  int number = 7;
  
  // assign addresses to pointers
  ptr = &number;
  ptr_to_ptr = &ptr;
  
  // output values
  printf("\n number = %d", number);
  printf("\n &number = %p \n", &number);
  
  printf("\n ptr = %p", ptr);
  printf("\n &ptr = %p", &ptr);
  printf("\n ptr_to_ptr = %p \n", ptr_to_ptr);
  
  printf("\n *ptr = %d", *ptr);
  printf("\n *ptr_to_ptr = %p", *ptr_to_ptr);
  printf("\n **ptr_to_ptr = %d \n ", **ptr_to_ptr);
  
  return 0;
}
```

The first couple lines of the output show the variable called number and its address. As expected, the address of the variable number is contained in the pointer variable ptr. The fifth line of the output shows the address of ptr which is also stored in the pointer to pointer variable ptr_to_ptr.

![PointerToPointerOutput](/assets/posts/20250910/PointerToPointerOutput.jpg){: width="240" height="204"}
_Results of dereferencing the pointers_

The last three lines of output is where it gets interesting. Deferencing ptr gives the value 7 as expected. However, dereferencing ptr_to_ptr gives the address of the variable number. This makes sense because applying the indirection operator gets the value stored at the address the pointer contains. In this case, since we are pointing to a pointer, another address is returned. To obtain the value at the address stored by the pointer we're pointing to, the indirection operator has to be applied again. So the syntax **ptr_to_ptr is used, and the value 7 is obtained.

The concepts of passing by reference and passing by value are also closely linked to pointers in C. If you wanted to pass variables by reference to a function, the parameters would be declared as pointer variables and the variables would then be passed to the function prefixed by the address operator:

```c
#include <stdio.h>

void byreferencefunction(int *x, int *y)
{
  *x = *x + 3;
  *y = *y + 2;
}

int main()
{
  int k = 9;
  int l = 21;

  printf("Before function call - k = %d l = %d\n", k, l);
  
  byreferencefunction(&k, &l);
  
  printf("After function call - k = %d l = %d", k, l);
  
  return 0;
}
```

The variables k and l passed into the function by reference are permanently changed once the function is executed:

![PassingByReference](/assets/posts/20250910/PassingByReference.jpg){: width="342" height="66"}
_Results of passing values by reference_

The idea of passing by reference or by value is certainly nothing new or groundbreaking, but it is interesting to note that the same concepts are encountered in high-level languages without the use of pointers and addresses. Those pesky, little details are abstracted away. Being closer to the machine and playing with pointers can be fun, but it can also be a nightmare when an intractable bug is introduced due to a syntactic error. However, when pointers are used in the right situations, the result can be beautifully efficient and fast code.

<br>

{% include comment.html %}
<br>

|![HumanContent](/assets/posts/badges/HumanContent_08.png) ![MadeByAHuman](/assets/posts/badges/MadeByAHuman_07.png) ![NeverByAI](/assets/posts/badges/NeverByAi_01.png)| 







