

**Lab 11 Report – Inter-Process Communication and Pipes**  
 **Student:** Ibragimov Bobir  
 **Student ID:** 210095  
 **Email:** bjbobur@icloud.com

---

### **Objective of the Assignment**

The purpose of this lab is to gain hands-on experience with inter-process communication in Linux, using unnamed pipes and `popen()`. By completing the exercises and tasks, we learn to send and receive data between processes, use `fork()`, and integrate external commands into C programs.

---

## **Q1: Reading Output from an External Program Using `popen()`**

In this task, we used `popen()` with read mode to capture output from `ls -la` and `find . -type f -name "*.c"`.

\#include \<stdio.h\>  
\#include \<stdlib.h\>  
\#include \<string.h\>

int main() {  
    FILE \*read\_fp;  
    char buffer\[BUFSIZ \+ 1\];  
    int chars\_read;

    memset(buffer, '\\0', sizeof(buffer));

    // Run 'ls \-la'  
    read\_fp \= popen("ls \-la", "r");  
    if (read\_fp \!= NULL) {  
        chars\_read \= fread(buffer, sizeof(char), BUFSIZ, read\_fp);  
        if (chars\_read \> 0\) {  
            printf("Output of ls \-la:\\n%s\\n", buffer);  
        }  
        pclose(read\_fp);  
    }

    // Run 'find . \-type f \-name "\*.c"'  
    memset(buffer, '\\0', sizeof(buffer));  
    read\_fp \= popen("find . \-type f \-name \\"\*.c\\"", "r");  
    if (read\_fp \!= NULL) {  
        chars\_read \= fread(buffer, sizeof(char), BUFSIZ, read\_fp);  
        if (chars\_read \> 0\) {  
            printf("Output of find:\\n%s\\n", buffer);  
        }  
        pclose(read\_fp);  
    }

    return 0;  
}

---

## **Q2: Writing to External Programs Using `popen()`**

This program demonstrates how to send data to commands like `sort`, `wc`, and `od`.

\#include \<stdio.h\>  
\#include \<stdlib.h\>  
\#include \<string.h\>

int main() {  
    FILE \*write\_fp;

    // Example 1: Sort  
    write\_fp \= popen("sort", "w");  
    if (write\_fp) {  
        fputs("zebra\\napple\\norange\\nbanana\\n", write\_fp);  
        pclose(write\_fp);  
    }

    // Example 2: Word Count  
    write\_fp \= popen("wc", "w");  
    if (write\_fp) {  
        const char \*text \= "Line one.\\nLine two.\\nLine three.\\n";  
        fwrite(text, sizeof(char), strlen(text), write\_fp);  
        pclose(write\_fp);  
    }

    // Example 3: Octal dump  
    write\_fp \= popen("od \-c", "w");  
    if (write\_fp) {  
        const char \*data \= "Hello, world\!";  
        fwrite(data, sizeof(char), strlen(data), write\_fp);  
        pclose(write\_fp);  
    }

    return 0;  
}

---

## **Q3: Parent to Child Communication Using Pipe**

We use `pipe()` and `fork()` to demonstrate one-way communication between a parent and its child.

\#include \<stdio.h\>  
\#include \<stdlib.h\>  
\#include \<unistd.h\>  
\#include \<string.h\>  
\#include \<sys/wait.h\>

int main() {  
    int pipefd\[2\];  
    pid\_t pid;  
    char buffer\[BUFSIZ\];  
    const char msg\[\] \= "Message from parent.";

    if (pipe(pipefd) \== \-1) {  
        perror("pipe");  
        exit(EXIT\_FAILURE);  
    }

    pid \= fork();  
    if (pid \== 0\) {  
        close(pipefd\[1\]);  
        read(pipefd\[0\], buffer, BUFSIZ);  
        printf("Child received: %s\\n", buffer);  
        close(pipefd\[0\]);  
    } else {  
        close(pipefd\[0\]);  
        write(pipefd\[1\], msg, strlen(msg) \+ 1);  
        close(pipefd\[1\]);  
        wait(NULL);  
    }

    return 0;  
}

---

## **Q4: Two Children Communicating via Pipes**

The parent spawns two children. Child 1 receives a message from the parent and forwards a modified version to Child 2\.

\#include \<stdio.h\>  
\#include \<stdlib.h\>  
\#include \<unistd.h\>  
\#include \<string.h\>  
\#include \<sys/wait.h\>

int main() {  
    int pipe1\[2\], pipe2\[2\];  
    char buffer\[BUFSIZ\];  
    pipe(pipe1);  
    pipe(pipe2);  
    pid\_t c1 \= fork();

    if (c1 \== 0\) {  
        close(pipe1\[1\]);  
        close(pipe2\[0\]);  
        read(pipe1\[0\], buffer, BUFSIZ);  
        sprintf(buffer, "Child1 modified: %s", buffer);  
        write(pipe2\[1\], buffer, strlen(buffer) \+ 1);  
        exit(0);  
    } else {  
        pid\_t c2 \= fork();  
        if (c2 \== 0\) {  
            close(pipe2\[1\]);  
            read(pipe2\[0\], buffer, BUFSIZ);  
            printf("Child2 received: %s\\n", buffer);  
            exit(0);  
        } else {  
            close(pipe1\[0\]);  
            write(pipe1\[1\], "Original message from parent", 29);  
            close(pipe1\[1\]);  
            wait(NULL);  
            wait(NULL);  
        }  
    }

    return 0;  
}

---

## **Q5: Pipe Between Separate Programs**

The parent executes another binary (`pipe_reader`) to read from a pipe passed as a file descriptor argument.

**pipe3.c**

\#include \<stdio.h\>  
\#include \<stdlib.h\>  
\#include \<unistd.h\>  
\#include \<string.h\>

int main() {  
    int pipefd\[2\];  
    pid\_t pid;  
    const char \*msg \= "123";

    pipe(pipefd);  
    pid \= fork();

    if (pid \== 0\) {  
        char fd\_str\[10\];  
        sprintf(fd\_str, "%d", pipefd\[0\]);  
        execl("./pipe\_reader", "pipe\_reader", fd\_str, (char \*)0);  
        perror("execl failed");  
        exit(1);  
    } else {  
        write(pipefd\[1\], msg, strlen(msg) \+ 1);  
        wait(NULL);  
    }

    return 0;  
}

**pipe\_reader.c**

\#include \<stdio.h\>  
\#include \<stdlib.h\>  
\#include \<unistd.h\>  
\#include \<string.h\>

int main(int argc, char \*argv\[\]) {  
    char buffer\[BUFSIZ\];  
    int fd;

    if (argc \!= 2\) {  
        fprintf(stderr, "Usage: %s \<fd\>\\n", argv\[0\]);  
        return 1;  
    }

    sscanf(argv\[1\], "%d", \&fd);  
    read(fd, buffer, BUFSIZ);  
    printf("pipe\_reader received: %s\\n", buffer);

    return 0;  
}

---

## **Conclusion**

This lab provided practical exposure to IPC using pipes and external command execution. We practiced both one-way and two-way communication using pipes, worked with multiple processes, and learned how to capture or send data using `popen()`. The examples show how low-level Unix features can be used to coordinate data flow between processes effectively.

