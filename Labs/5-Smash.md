# Smash

## First Example

```
#include <stdio.h>
#include <string.h>

char * longString = "AAAAAAAAAAAAAAAA";
int i; // Declare i here so it doesn't go on stack in copy()

void copy(char *s) {
    char buffer[12];
    strcpy(buffer, s);
    printf("Address of buffer = %p\n\n", buffer);
    
    // Print the contents of the stack starting from the buffer
    // Memory address    Bytes past buffer start     Hex value
    printf("Contents of memory starting at buffer:\n");
    for (i = 0; i < 48; i++) {
        printf("%p\t%d\t%x\n", buffer + i, i, *(buffer + i));
    }

    return;
}


int main(int argc, char *argv[1]) {
    printf("Address of main: %p\n", main);
    copy(longString);
    return 0;
}
```

## `execve` Example

```
#include <stdio.h>
#include <unistd.h>

// execve replaces the address space of the current process with a new 
// address space to run a new process

// execve takes three arguments:
// 1. a string containing the name of the new process
// 2. an array of strings containing the command line arguments to the 
//    new process, ordered just as they would be in the argv array --- by
//    convention, the first argument in this array is a repeat of the name
//    of the process and the last item must be NULL
// 3. the third argument is almost always NULL

int main() {
   char *name[2];

   name[0] = "/bin/sh";
   name[1] = NULL;
   
   execve(name[0], name, NULL);
   
   // A call to execve never returns so this code should never execute
   //
   // If it does, something went wrong in execve
   printf("I shouldn't be here!\n");
   perror("execve");
   return -1;
}
```

## Shell Smash

```
// Example function call
//
// Taken from "Smashing the Stack for Fun and Profit" by Aleph One
//
// Compile with -fno-stack-protector option

#include <stdio.h>

void run_sh() {
  char *exec_args[2];
  exec_args[0] = "/bin/sh";
  exec_args[1] = '\0';

  // Use execvp to turn this process into a shell
  execvp("/bin/sh", exec_args);
}


void function(int a, int b, int c) {
  char buffer[12];

  printf("Return address on stack = %p\n", *(&a  + 7));

  // Modify the return address to jump to a method that runs a shell
  *(&a + 7) = run_sh;
  printf("Modified return address on stack = %p\n", *(&a  + 7));
}


void main () {
  printf("Address of main = %p\n", main);
  function(1, 2, 3);
}
```

## Array Smash

```
// Stack smashing using a true buffer overflow attack
// Buffer contains code that prints "Hax!" to the console
//
// Adapted from "Stack Smashing on a Modern Linux System" by jip@soldierx.com
//
// Compile using gcc -z execstack -fno-stack-protector ...

#include <string.h>
#include <stdio.h>

// THIS WORKS!
// Run it on Cloud 9
// Compile using gcc -z execstack -fno-stack-protector -o array_smash array_smash.c
char printcode[] = "\xeb\x22\x48\x31\xc0\x48\x31\xff\x48\x31\xd2\x48\xff\xc0\x48\xff\xc7\x5e\x48\x83\xc2\x04\x0f\x05\x48\x31\xc0\x48\x83\xc0\x3c\x48\x31\xff\x0f\x05\xe8\xd9\xff\xff\xff\x48\x61\x78\x21\xff\xff\xff\xff\xff\xff\xff\xff\xff\xff\xff\xff\xff\xff\xff\xff\xff\xff\xff\xff\xff\xff\xff\xff\xff\xff\xff\x40\xde\xff\xff\xff\x7f";

void go(char *data) {
    char name[64];
    printf("Return address on stack:%p\n", (void *) *( (int*) name + 18));
    printf("Address of name buffer: %p\n", name);

    memcpy(name, data, 82);
    printf("Return address on stack:%p\n", (void*) *( (int*) name + 18));
}

int main() {
    printf("Address of main: %p\n", main);
    go(printcode);
}
```
