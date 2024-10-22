# Linux-IPC-Semaphores
Ex05-Linux IPC-Semaphores

# AIM:
To Write a C program that implements a producer-consumer system with two processes using Semaphores.

# DESIGN STEPS:

### Step 1:

Navigate to any Linux environment installed on the system or installed inside a virtual environment like virtual box/vmware or online linux JSLinux (https://bellard.org/jslinux/vm.html?url=alpine-x86.cfg&mem=192) or docker.

### Step 2:

Write the C Program using Linux Process API - Sempahores

### Step 3:

Execute the C Program for the desired output. 

# PROGRAM:

## Write a C program that implements a producer-consumer system with two processes using Semaphores.
```c
#include <stdio.h>     
#include <stdlib.h>    
#include <unistd.h>    
#include <time.h>      
#include <sys/types.h> 
#include <sys/ipc.h>   
#include <sys/sem.h>   

#define NUM_LOOPS 20

#if defined(__GNU_LIBRARY__) && !defined(_SEM_SEMUN_UNDEFINED)
// union semun is defined by including <sys/sem.h>
#else
// Define the union semun if not defined
union semun {
    int val;                    
    struct semid_ds *buf;       
    unsigned short int *array;  
    struct seminfo *__buf;      
};
#endif

int main(int argc, char* argv[]) {
    int sem_set_id;         
    union semun sem_val;    
    int child_pid;         
    int i;                 
    struct sembuf sem_op;  
    int rc;                
    struct timespec delay; 

    // Create a private semaphore set with one semaphore
    sem_set_id = semget(IPC_PRIVATE, 1, 0600);
    if (sem_set_id == -1) {
        perror("main: semget");
        exit(1);
    }
    printf("Semaphore set created, ID: '%d'.\n", sem_set_id);

    // Initialize the semaphore to '0'
    sem_val.val = 0;
    rc = semctl(sem_set_id, 0, SETVAL, sem_val);

    // Fork a child process
    child_pid = fork();
    switch (child_pid) {
        case -1: // fork() failed
            perror("fork");
            exit(1);
        
        case 0: // Child process (Consumer)
            for (i = 0; i < NUM_LOOPS; i++) {
                // Wait for the semaphore to be non-negative
                sem_op.sem_num = 0;
                sem_op.sem_op = -1; // Decrement semaphore
                sem_op.sem_flg = 0;
                semop(sem_set_id, &sem_op, 1);

                printf("Consumer: consumed item '%d'\n", i);
                fflush(stdout);
            }
            break;
        
        default: // Parent process (Producer)
            for (i = 0; i < NUM_LOOPS; i++) {
                printf("Producer: produced item '%d'\n", i);
                fflush(stdout);

                // Increment the semaphore to signal the consumer
                sem_op.sem_num = 0;
                sem_op.sem_op = 1; // Increment semaphore
                sem_op.sem_flg = 0;
                semop(sem_set_id, &sem_op, 1);

                // Pause execution randomly to simulate work
                if (rand() > 3 * (RAND_MAX / 4)) {
                    delay.tv_sec = 0;
                    delay.tv_nsec = 10; // Adjust delay time as needed
                    nanosleep(&delay, NULL);
                }
            }
            // Remove the semaphore after production is done
            semctl(sem_set_id, 0, IPC_RMID, sem_val);
            break;
    }
    return 0;
}

```



## OUTPUT
![image](https://github.com/user-attachments/assets/443e0d5f-5f21-41cd-a94d-6c1189fc3ad0)






# RESULT:
The program is executed successfully.
