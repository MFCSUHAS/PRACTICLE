[9:23 PM, 12/11/2023] Suhas😇:

 #include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <unistd.h>

void get_file_info(const char *file_path) {
    struct stat file_stat;

    // Get file information
    if (stat(file_path, &file_stat) == 0) {
        // Print inode number and file type
        printf("%s: Inode %lu, Type %o\n", file_path, (unsigned long)file_stat.st_ino, file_stat.st_mode & S_IFMT);
    } else {
        perror("Error");
    }
}

int main(int argc, char *argv[]) {
    // Check if file paths are provided
    if (argc < 2) {
        printf("Usage: %s file1 file2 ...\n", argv[0]);
        return 1;
    }

    // Process each file path
    for (int i = 1; i < argc; ++i) {
        get_file_info(argv[i]);
    }

    return 0;
}

[10:30 PM, 12/11/2023] Suhas😇: Take multiple files as Command Line Arguments and print their inode numbers and file types


 #include <stdio.h>
#include <stdlib.h>
#include <signal.h>
#include <unistd.h>
#include <sys/types.h>

// Function to handle the SIGALRM signal
void alarmHandler(int signum) {
    if (signum == SIGALRM) {
        printf("Alarm is fired!\n");
    }
}

int main() {
    // Register the alarmHandler function for SIGALRM signal
    signal(SIGALRM, alarmHandler);

    pid_t pid = fork();

    if (pid == -1) {
        perror("Error in fork");
        exit(EXIT_FAILURE);
    }

    if (pid > 0) {
        // Parent process
        sleep(2); // Allow child process to send the signal

        // Send SIGALRM signal to the child process
        kill(pid, SIGALRM);
    } else {
        // Child process
        // Sleep for a while to allow the parent process to set up the signal handler
        sleep(1);

        // Send SIGALRM signal to the parent process
        kill(getppid(), SIGALRM);
    }

    return 0;
}
[10:33 PM, 12/11/2023] Suhas😇: Write a C program to send SIGALRM signal by child process to parent process and parent process 
make a provision to catch the signal and display alarm is fired.(Use Kill, fork, signal and sleep 
system call)



 #include <stdio.h>
#include <sys/stat.h>
#include <unistd.h>
#include <stdlib.h>
#include <time.h>

int main(int argc, char *argv[]) {
    if (argc != 2) {
        fprintf(stderr, "Usage: %s <filename>\n", argv[0]);
        exit(EXIT_FAILURE);
    }

    struct stat fileStat;

    if (stat(argv[1], &fileStat) == -1) {
        perror("stat");
        exit(EXIT_FAILURE);
    }

    printf("File Properties for: %s\n", argv[1]);
    printf("Inode number: %ld\n", (long)fileStat.st_ino);
    printf("Number of hard links: %ld\n", (long)fileStat.st_nlink);
    printf("File permissions: %o\n", fileStat.st_mode & 0777);
    printf("File size: %ld bytes\n", (long)fileStat.st_size);

    // Access time
    printf("Last access time: %s", ctime(&fileStat.st_atime));

    // Modification time
    printf("Last modification time: %s", ctime(&fileStat.st_mtime));

    return 0;
}
[10:36 PM, 12/11/2023] Suhas😇: Write a C program to find file properties such as inode number, number of hard link, File 
permissions, File size, File access and modification time and so on of a given file using stat() 
system call
[10:37 PM, 12/11/2023] Suhas😇:

 #include <stdio.h>
#include <signal.h>
#include <stdlib.h>

// Global variable to count the number of times Ctrl-C is pressed
int ctrlCCount = 0;

// Signal handler function
void sigintHandler(int signum) {
    if (signum == SIGINT) {
        ctrlCCount++;

        if (ctrlCCount == 1) {
            printf("\nCtrl-C pressed. Press again to exit.\n");
        } else {
            printf("\nExiting...\n");
            exit(EXIT_SUCCESS);
        }
    }
}

int main() {
    // Registering the signal handler
    if (signal(SIGINT, sigintHandler) == SIG_ERR) {
        perror("signal");
        exit(EXIT_FAILURE);
    }

    printf("Press Ctrl-C to test the signal handler.\n");

    // Infinite loop to keep the program running
    while (1) {
        // Do nothing, just wait for signals
    }

    return 0;
}
[10:37 PM, 12/11/2023] Suhas😇: Write a C program that catches the ctrl-c (SIGINT) signal for the first time and display the 
appropriate message and exits on pressing ctrl-c again.
[10:43 PM, 12/11/2023] Suhas😇:

 #include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <unistd.h>

int main(int argc, char *argv[]) {
    if (argc != 2) {
        fprintf(stderr, "Usage: %s <filename>\n", argv[0]);
        exit(EXIT_FAILURE);
    }

    char *filename = argv[1];
    struct stat file_info;

    if (stat(filename, &file_info) == -1) {
        perror("Error");
        exit(EXIT_FAILURE);
    }

    printf("File Type: ");

    if (S_ISREG(file_info.st_mode))
        printf("Regular File\n");
    else if (S_ISDIR(file_info.st_mode))
        printf("Directory\n");
    else if (S_ISCHR(file_info.st_mode))
        printf("Character Device\n");
    else if (S_ISBLK(file_info.st_mode))
        printf("Block Device\n");
    else if (S_ISFIFO(file_info.st_mode))
        printf("FIFO/Named Pipe\n");
    else if (S_ISLNK(file_info.st_mode))
        printf("Symbolic Link\n");
    else if (S_ISSOCK(file_info.st_mode))
        printf("Socket\n");
    else
        printf("Unknown\n");

    printf("Inode Number: %ld\n", (long)file_info.st_ino);

    return 0;
}

[10:43 PM, 12/11/2023] Suhas😇: Write a c program for Print the type of file and inode number where file name accepted through Command Line
[10:45 PM, 12/11/2023] Suhas😇:

 #include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <signal.h>
#include <sys/wait.h>

void child_handler(int signo) {
    if (signo == SIGCHLD) {
        printf("Child process terminated.\n");
    }
}

void alarm_handler(int signo) {
    if (signo == SIGALRM) {
        printf("Child process execution time exceeded 5 seconds. Killing the child.\n");
        exit(EXIT_FAILURE);
    }
}

int main(int argc, char *argv[]) {
    if (argc < 2) {
        fprintf(stderr, "Usage: %s <command>\n", argv[0]);
        exit(EXIT_FAILURE);
    }

    pid_t pid;
    signal(SIGCHLD, child_handler);
    signal(SIGALRM, alarm_handler);

    pid = fork();

    if (pid == -1) {
        perror("Error forking process");
        exit(EXIT_FAILURE);
    }

    if (pid == 0) { // Child process
        execvp(argv[1], &argv[1]);
        perror("Error in execvp");
        exit(EXIT_FAILURE);
    } else { // Parent process
        alarm(5); // Set an alarm for 5 seconds

        int status;
        waitpid(pid, &status, 0);

        alarm(0); // Cancel the alarm

        if (WIFEXITED(status)) {
            printf("Child process exited with status %d.\n", WEXITSTATUS(status));
        } else if (WIFSIGNALED(status)) {
            printf("Child process terminated by signal %d.\n", WTERMSIG(status));
        }
    }

    return 0;
}


[10:45 PM, 12/11/2023] Suhas😇: Write a C program which creates a child process to run linux/ unix command or any user defined 
program. The parent process set the signal handler for death of child signal and Alarm signal. If 
a child process does not complete its execution in 5 second then parent process kills child process.
[10:46 PM, 12/11/2023] Suhas😇: 

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <dirent.h>

int fileExists(const char *filename) {
    FILE *file = fopen(filename, "r");
    if (file) {
        fclose(file);
        return 1; // File exists
    }
    return 0; // File does not exist
}

int main(int argc, char *argv[]) {
    if (argc < 2) {
        fprintf(stderr, "Usage: %s <file1> <file2> ... <fileN>\n", argv[0]);
        exit(EXIT_FAILURE);
    }

    for (int i = 1; i < argc; ++i) {
        if (fileExists(argv[i])) {
            printf("%s exists in the current directory.\n", argv[i]);
        } else {
            printf("%s does not exist in the current directory.\n", argv[i]);
        }
    }

    return 0;
}
[10:46 PM, 12/11/2023] Suhas😇: Write a C program to find whether a given files passed through command line arguments are 
present in current directory or not
[10:49 PM, 12/11/2023] Suhas😇:

 #include <stdio.h>
#include <stdlib.h>
#include <signal.h>
#include <unistd.h>

void child_signal_handler(int signo) {
    switch (signo) {
        case SIGHUP:
            printf("Child: Received SIGHUP\n");
            break;
        case SIGINT:
            printf("Child: Received SIGINT\n");
            break;
        case SIGQUIT:
            printf("Child: Received SIGQUIT. My Papa has Killed me!!!\n");
            exit(EXIT_SUCCESS);
    }
}

int main() {
    pid_t pid;

    pid = fork();

    if (pid == -1) {
        perror("Error forking process");
        exit(EXIT_FAILURE);
    }

    if (pid == 0) { // Child process
        signal(SIGHUP, child_signal_handler);
        signal(SIGINT, child_signal_handler);
        signal(SIGQUIT, child_signal_handler);

        while (1) {
            // Child process continues running
            sleep(1);
        }
    } else { // Parent process
        // Send SIGHUP or SIGINT to the child every 3 seconds
        for (int i = 0; i < 5; ++i) {
            sleep(3);
            kill(pid, (i % 2 == 0) ? SIGHUP : SIGINT);
        }

        // Send SIGQUIT to the child after 15 seconds
        sleep(6);
        kill(pid, SIGQUIT);

        // Wait for the child to terminate
        wait(NULL);
    }

    return 0;
}
[10:49 PM, 12/11/2023] Suhas😇: Write a C program which creates a child process and child process catches a signal SIGHUP, 
SIGINT and SIGQUIT. The Parent process send a SIGHUP or SIGINT signal after every 3 
seconds, at the end of 15 second parent send SIGQUIT signal to child and child terminates by 
displaying message "My Papa has Killed me!!!”
[10:51 PM, 12/11/2023] Suhas😇:

 #include <stdio.h>
#include <stdlib.h>
#include <dirent.h>

int main() {
    DIR *directory;
    struct dirent *entry;
    int fileCount = 0;

    // Open the current directory
    directory = opendir(".");

    if (directory == NULL) {
        perror("Error opening current directory");
        exit(EXIT_FAILURE);
    }

    // Read and display the names of files in the current directory
    while ((entry = readdir(directory)) != NULL) {
        if (entry->d_type == DT_REG) {  // Check if it's a regular file
            printf("%s\n", entry->d_name);
            fileCount++;
        }
    }

    // Display the total number of files in the current directory
    printf("\nTotal number of files: %d\n", fileCount);

    // Close the directory
    closedir(directory);

    return 0;
}
[10:51 PM, 12/11/2023] Suhas😇: Write c program Read the current directory and display the name of the files, no of files in current directory
[10:52 PM, 12/11/2023] Suhas😇:


 #include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

#define MESSAGE1 "Hello World"
#define MESSAGE2 "Hello SPPU"
#define MESSAGE3 "Linux is Funny"

int main() {
    int pipe_fd[2];
    pid_t pid;

    if (pipe(pipe_fd) == -1) {
        perror("Error creating pipe");
        exit(EXIT_FAILURE);
    }

    pid = fork();

    if (pid == -1) {
        perror("Error forking process");
        exit(EXIT_FAILURE);
    }

    if (pid == 0) { // Child process
        close(pipe_fd[0]); // Close the read end

        // Write messages to the pipe
        write(pipe_fd[1], MESSAGE1, sizeof(MESSAGE1));
        write(pipe_fd[1], MESSAGE2, sizeof(MESSAGE2));
        write(pipe_fd[1], MESSAGE3, sizeof(MESSAGE3));

        close(pipe_fd[1]); // Close the write end
    } else { // Parent process
        close(pipe_fd[1]); // Close the write end

        char buffer[256];

        // Read and display messages from the pipe
        while (read(pipe_fd[0], buffer, sizeof(buffer)) > 0) {
            printf("%s\n", buffer);
        }

        close(pipe_fd[0]); // Close the read end

        wait(NULL); // Wait for the child to complete
    }

    return 0;
}
[10:53 PM, 12/11/2023] Suhas😇: Write a C program to create an unnamed pipe. The child process will write following three 
messages to pipe and parent process display it. 
Message1 = “Hello World” 
Message2 = “Hello SPPU” 
Message3 = “Linux is Funny
[10:56 PM, 12/11/2023] Suhas😇:


 #include <stdio.h>
#include <stdlib.h>
#include <dirent.h>
#include <sys/stat.h>
#include <time.h>

void displayFilesByMonth(const char *targetMonth) {
    DIR *directory;
    struct dirent *entry;
    struct stat fileStat;
    
    // Open the current directory
    directory = opendir(".");

    if (directory == NULL) {
        perror("Error opening current directory");
        exit(EXIT_FAILURE);
    }

    // Read and display files created in the target month
    while ((entry = readdir(directory)) != NULL) {
        if (stat(entry->d_name, &fileStat) == -1) {
            perror("Error getting file information");
            exit(EXIT_FAILURE);
        }

        // Check if the file was created in the target month
        struct tm *timeinfo = localtime(&fileStat.st_ctime);
        char month[10];
        strftime(month, sizeof(month), "%b", timeinfo);

        if (strcmp(month, targetMonth) == 0) {
            printf("%s\n", entry->d_name);
        }
    }

    // Close the directory
    closedir(directory);
}

int main(int argc, char *argv[]) {
    if (argc != 2) {
        fprintf(stderr, "Usage: %s <target_month>\n", argv[0]);
        exit(EXIT_FAILURE);
    }

    displayFilesByMonth(argv[1]);

    return 0;
}
[10:56 PM, 12/11/2023] Suhas😇: Write c program Read the current directory and display the name of the files, no of files in current directory
[10:59 PM, 12/11/2023] Suhas😇:


 #include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <sys/wait.h>
#include <sys/times.h>
#include <unistd.h>

int main(int argc, char *argv[]) {
    if (argc != 2) {
        fprintf(stderr, "Usage: %s <number_of_children>\n", argv[0]);
        exit(EXIT_FAILURE);
    }

    int numChildren = atoi(argv[1]);
    pid_t pid;
    clock_t startTime, endTime;
    struct tms startTMS, endTMS;

    startTime = times(&startTMS);

    for (int i = 0; i < numChildren; ++i) {
        pid = fork();

        if (pid == -1) {
            perror("Error forking process");
            exit(EXIT_FAILURE);
        }

        if (pid == 0) { // Child process
            // Child-specific code
            exit(EXIT_SUCCESS);
        }
    }

    // Parent process waits for all children to terminate
    for (int i = 0; i < numChildren; ++i) {
        wait(NULL);
    }

    endTime = times(&endTMS);

    printf("Total User Time: %f seconds\n", (double)(endTMS.tms_utime - startTMS.tms_utime) / sysconf(_SC_CLK_TCK));
    printf("Total Kernel Time: %f seconds\n", (double)(endTMS.tms_stime - startTMS.tms_stime) / sysconf(_SC_CLK_TCK));

    return 0;
}
[10:59 PM, 12/11/2023] Suhas😇: Write a C program to create n child processes. When all n child processes terminates, Display 
total cumulative time children spent in user and kernel mode
[11:00 PM, 12/11/2023] Suhas😇: #include <stdio.h>
#include <stdlib.h>

int main() {
    // Save the current standard output file descriptor
    int original_stdout = dup(fileno(stdout));

    // Open a file for writing (or create if it doesn't exist)
    FILE *file = fopen("output.txt", "w");

    if (file == NULL) {
        perror("Error opening file");
        exit(EXIT_FAILURE);
    }

    // Redirect standard output to the file
    dup2(fileno(file), fileno(stdout));

    // Now, any printf statements will be written to the file
    printf("This is redirected to the file.\n");

    // Restore the original standard output
    fflush(stdout); // Ensure any buffered data is written to the file
    dup2(original_stdout, fileno(stdout));

    // Close the file
    fclose(file);

    // Now, printf statements go back to the console
    printf("This is back to the console.\n");

    return 0;
}
[11:00 PM, 12/11/2023] Suhas😇: Write a C Program that demonstrates redirection of standard output to a file
[1
[11:02 PM, 12/11/2023] Suhas😇:


 #include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/wait.h>

int main() {
    int pipe_fd[2];
    pid_t pid;

    if (pipe(pipe_fd) == -1) {
        perror("Error creating pipe");
        exit(EXIT_FAILURE);
    }

    pid = fork();

    if (pid == -1) {
        perror("Error forking process");
        exit(EXIT_FAILURE);
    }

    if (pid == 0) { // Child process (ls -l)
        // Close the read end of the pipe
        close(pipe_fd[0]);

        // Redirect standard output to the write end of the pipe
        dup2(pipe_fd[1], STDOUT_FILENO);

        // Close unused file descriptors
        close(pipe_fd[1]);

        // Execute the "ls -l" command
        execlp("ls", "ls", "-l", NULL);
        perror("Error executing ls -l");
        exit(EXIT_FAILURE);
    } else { // Parent process (wc -l)
        // Close the write end of the pipe
        close(pipe_fd[1]);

        // Redirect standard input to the read end of the pipe
        dup2(pipe_fd[0], STDIN_FILENO);

        // Close unused file descriptors
        close(pipe_fd[0]);

        // Execute the "wc -l" command
        execlp("wc", "wc", "-l", NULL);
        perror("Error executing wc -l");
        exit(EXIT_FAILURE);
    }

    return 0;
}
[11:02 PM, 12/11/2023] Suhas😇: Write a c program Implement the following unix/linux command (use fork, pipe and exec system call) 
ls –l | wc –l
[11:03 PM, 12/11/2023] Suhas😇:


 #include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <fcntl.h>

int main() {
    // Save the current standard output file descriptor
    int original_stdout = dup(fileno(stdout));

    // Open or create a file for writing
    int file_descriptor = open("output.txt", O_WRONLY | O_CREAT | O_TRUNC, 0666);

    if (file_descriptor == -1) {
        perror("Error opening file");
        exit(EXIT_FAILURE);
    }

    // Redirect standard output to the file
    dup2(file_descriptor, fileno(stdout));

    // Now, any printf statements will be written to the file
    printf("This is redirected to the file.\n");

    // Restore the original standard output
    fflush(stdout); // Ensure any buffered data is written to the file
    dup2(original_stdout, fileno(stdout));

    // Close the file descriptor
    close(file_descriptor);

    // Now, printf statements go back to the console
    printf("This is back to the console.\n");

    return 0;
}
[11:04 PM, 12/11/2023] Suhas😇: Write a C program that redirects standard output to a file output.txt. (use of dup and open system 
call).
[11:07 PM, 12/11/2023] Suhas😇: 


#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/wait.h>

int main() {
    int pipe_fd[2];
    pid_t pid;

    // Create an unnamed pipe
    if (pipe(pipe_fd) == -1) {
        perror("Error creating pipe");
        exit(EXIT_FAILURE);
    }

    // Fork to create a child process
    pid = fork();

    if (pid == -1) {
        perror("Error forking process");
        exit(EXIT_FAILURE);
    }

    if (pid == 0) { // Child process
        // Close the write end of the pipe
        close(pipe_fd[1]);

        char buffer[256];
        ssize_t bytesRead;

        // Read from the pipe
        bytesRead = read(pipe_fd[0], buffer, sizeof(buffer));

        // Check for read errors
        if (bytesRead == -1) {
            perror("Error reading from pipe");
            exit(EXIT_FAILURE);
        }

        // Display the read data
        printf("Child Process: Read from pipe: %.*s\n", (int)bytesRead, buffer);

        // Close the read end of the pipe
        close(pipe_fd[0]);

        exit(EXIT_SUCCESS);
    } else { // Parent process
        // Close the read end of the pipe
        close(pipe_fd[0]);

        // Write data into the pipe
        const char *message = "Hello, Child Process!";
        write(pipe_fd[1], message, strlen(message) + 1);

        // Close the write end of the pipe
        close(pipe_fd[1]);

        // Wait for the child to complete
        wait(NULL);

        exit(EXIT_SUCCESS);
    }

    return 0;
}
[11:07 PM, 12/11/2023] Suhas😇: Write a c program Generate parent process to write unnamed pipe and will read from it
[11:08 PM, 12/11/2023] Suhas😇:


 #include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <unistd.h>

void identifyFileType(const char *filename) {
    struct stat file_info;

    // Use stat() to obtain information about the file
    if (stat(filename, &file_info) == -1) {
        perror("Error getting file information");
        exit(EXIT_FAILURE);
    }

    printf("File Type of %s: ", filename);

    if (S_ISREG(file_info.st_mode))
        printf("Regular File\n");
    else if (S_ISDIR(file_info.st_mode))
        printf("Directory\n");
    else if (S_ISCHR(file_info.st_mode))
        printf("Character Device\n");
    else if (S_ISBLK(file_info.st_mode))
        printf("Block Device\n");
    else if (S_ISFIFO(file_info.st_mode))
        printf("FIFO/Named Pipe\n");
    else if (S_ISLNK(file_info.st_mode))
        printf("Symbolic Link\n");
    else if (S_ISSOCK(file_info.st_mode))
        printf("Socket\n");
    else
        printf("Unknown\n");
}

int main(int argc, char *argv[]) {
    if (argc != 2) {
        fprintf(stderr, "Usage: %s <filename>\n", argv[0]);
        exit(EXIT_FAILURE);
    }

    const char *filename = argv[1];
    identifyFileType(filename);

    return 0;
}
[11:08 PM, 12/11/2023] Suhas😇: Write a C program to Identify the type (Directory, character device, Block device, Regular file, 
FIFO or pipe, symbolic link or socket) of given file using stat() system call
[11:11 PM, 12/11/2023] Suhas😇: 


#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/wait.h>

int main() {
    int pipe_fd[2];
    pid_t pid;

    // Create a pipe
    if (pipe(pipe_fd) == -1) {
        perror("Error creating pipe");
        exit(EXIT_FAILURE);
    }

    // Fork to create a child process
    pid = fork();

    if (pid == -1) {
        perror("Error forking process");
        exit(EXIT_FAILURE);
    }

    if (pid == 0) { // Child process (ls -l)
        // Close the read end of the pipe
        close(pipe_fd[0]);

        // Redirect standard output to the write end of the pipe
        dup2(pipe_fd[1], STDOUT_FILENO);

        // Close unused file descriptors
        close(pipe_fd[1]);

        // Execute the "ls -l" command
        execlp("ls", "ls", "-l", NULL);
        perror("Error executing ls -l");
        exit(EXIT_FAILURE);
    } else { // Parent process (wc -l)
        // Close the write end of the pipe
        close(pipe_fd[1]);

        // Redirect standard input to the read end of the pipe
        dup2(pipe_fd[0], STDIN_FILENO);

        // Close unused file descriptors
        close(pipe_fd[0]);

        // Execute the "wc -l" command
        execlp("wc", "wc", "-l", NULL);
        perror("Error executing wc -l");
        exit(EXIT_FAILURE);
    }

    return 0;
}
[11:11 PM, 12/11/2023] Suhas😇: Write a c program that illustrates how to execute two commands concurrently with a pipe
[11:12 PM, 12/11/2023] Suhas😇: 

#include <stdio.h>
#include <stdlib.h>
#include <sys/resource.h>

void printResourceLimits() {
    struct rlimit limits;

    // Get and print current resource limits
    if (getrlimit(RLIMIT_NOFILE, &limits) == -1) {
        perror("Error getting RLIMIT_NOFILE");
        exit(EXIT_FAILURE);
    }
    printf("Current Maximum Number of Open Files (RLIMIT_NOFILE): soft=%lu, hard=%lu\n", limits.rlim_cur, limits.rlim_max);

    if (getrlimit(RLIMIT_AS, &limits) == -1) {
        perror("Error getting RLIMIT_AS");
        exit(EXIT_FAILURE);
    }
    printf("Current Address Space Limit (RLIMIT_AS): soft=%lu, hard=%lu bytes\n", limits.rlim_cur, limits.rlim_max);
}

void setResourceLimits() {
    struct rlimit new_limits;

    // Set new resource limits
    new_limits.rlim_cur = 10000; // New soft limit for RLIMIT_NOFILE
    new_limits.rlim_max = 20000; // New hard limit for RLIMIT_NOFILE
    if (setrlimit(RLIMIT_NOFILE, &new_limits) == -1) {
        perror("Error setting RLIMIT_NOFILE");
        exit(EXIT_FAILURE);
    }

    new_limits.rlim_cur = 1024 * 1024 * 10; // New soft limit for RLIMIT_AS (10 MB)
    new_limits.rlim_max = RLIM_INFINITY;    // Set to unlimited
    if (setrlimit(RLIMIT_AS, &new_limits) == -1) {
        perror("Error setting RLIMIT_AS");
        exit(EXIT_FAILURE);
    }

    printf("\nResource limits have been set.\n");
}

int main() {
    printf("Before setting resource limits:\n");
    printResourceLimits();

    setResourceLimits();

    printf("\nAfter setting resource limits:\n");
    printResourceLimits();

    return 0;
}
[11:13 PM, 12/11/2023] Suhas😇: Write a C program to get and set the resource limits such as files, memory associated with a 
process
[11:16 PM, 12/11/2023] Suhas😇:


 #include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <fcntl.h>

int main() {
    // Save the current standard output file descriptor
    int original_stdout = dup(fileno(stdout));

    // Open or create a file for writing
    int file_descriptor = open("output.txt", O_WRONLY | O_CREAT | O_TRUNC, 0666);

    if (file_descriptor == -1) {
        perror("Error opening file");
        exit(EXIT_FAILURE);
    }

    // Redirect standard output to the file
    dup2(file_descriptor, fileno(stdout));

    // Now, any printf statements will be written to the file
    printf("This is redirected to the file.\n");

    // Restore the original standard output
    fflush(stdout); // Ensure any buffered data is written to the file
    dup2(original_stdout, fileno(stdout));

    // Close the file descriptor
    close(file_descriptor);

    // Now, printf statements go back to the console
    printf("This is back to the console.\n");

    return 0;
}
[11:16 PM, 12/11/2023] Suhas😇: Write a C program that redirects standard output to a file output.txt. (use of dup and open system 
call).
[11:16 PM, 12/11/2023] Suhas😇: 

#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <sys/wait.h>
#include <unistd.h>

int main() {
    pid_t pid = fork();

    if (pid == -1) {
        perror("Error forking process");
        exit(EXIT_FAILURE);
    }

    if (pid == 0) { // Child process
        printf("Child Process: PID=%d\n", getpid());
        exit(EXIT_SUCCESS);
    } else { // Parent process
        int status;
        pid_t terminated_pid = wait(&status);

        if (terminated_pid == -1) {
            perror("Error waiting for child process");
            exit(EXIT_FAILURE);
        }

        if (WIFEXITED(status)) {
            printf("Parent Process: Child process %d exited with status %d\n", terminated_pid, WEXITSTATUS(status));
        } else if (WIFSIGNALED(status)) {
            printf("Parent Process: Child process %d terminated by signal %d\n", terminated_pid, WTERMSIG(status));
        } else {
            printf("Parent Process: Child process %d did not exit normally\n", terminated_pid);
        }
    }

    return 0;
}
[11:17 PM, 12/11/2023] Suhas😇: Write a C program that print the exit status of a terminated child process
[11:18 PM, 12/11/2023] Suhas😇: 

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <unistd.h>

// Structure to hold file information
struct FileInfo {
    char *filename;
    off_t size;
};

// Function to compare two FileInfo structures based on file sizes
int compareFileInfo(const void *a, const void *b) {
    return ((struct FileInfo *)a)->size - ((struct FileInfo *)b)->size;
}

int main(int argc, char *argv[]) {
    if (argc < 2) {
        fprintf(stderr, "Usage: %s <file1> <file2> ... <fileN>\n", argv[0]);
        exit(EXIT_FAILURE);
    }

    // Allocate memory for an array of FileInfo structures
    struct FileInfo *fileInfos = malloc((argc - 1) * sizeof(struct FileInfo));

    if (fileInfos == NULL) {
        perror("Error allocating memory");
        exit(EXIT_FAILURE);
    }

    // Populate fileInfos array with file names and sizes
    for (int i = 1; i < argc; ++i) {
        struct stat fileStat;

        if (stat(argv[i], &fileStat) == -1) {
            perror("Error getting file information");
            exit(EXIT_FAILURE);
        }

        fileInfos[i - 1].filename = argv[i];
        fileInfos[i - 1].size = fileStat.st_size;
    }

    // Sort fileInfos array based on file sizes
    qsort(fileInfos, argc - 1, sizeof(struct FileInfo), compareFileInfo);

    // Display filenames in ascending order of sizes
    printf("Filenames in ascending order of sizes:\n");
    for (int i = 0; i < argc - 1; ++i) {
        printf("%s\t%ld bytes\n", fileInfos[i].filename, fileInfos[i].size);
    }

    // Free allocated memory
    free(fileInfos);

    return 0;
}
[11:18 PM, 12/11/2023] Suhas😇: Write a C program which receives file names as command line arguments and display those 
filenames in ascending order according to their sizes. I) (e.g $ a.out a.txt b.txt c.txt, …)
[11:31 PM, 12/11/2023] Suhas😇:


 #include <stdio.h>
#include <stdlib.h>
#include <signal.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/wait.h>

int childSuspended = 0;

void handleSIGUSR1(int signo) {
    if (signo == SIGUSR1) {
        printf("Child process suspended.\n");
        childSuspended = 1;
    }
}

void handleSIGUSR2(int signo) {
    if (signo == SIGUSR2) {
        printf("Child process resumed.\n");
        childSuspended = 0;
    }
}

int main() {
    pid_t pid;

    // Register signal handlers
    signal(SIGUSR1, handleSIGUSR1);
    signal(SIGUSR2, handleSIGUSR2);

    pid = fork();

    if (pid == -1) {
        perror("Error forking process");
        exit(EXIT_FAILURE);
    }

    if (pid == 0) { // Child process
        while (1) {
            printf("Child process is running...\n");
            sleep(1);
            if (childSuspended) {
                pause(); // Suspend the child process until a signal is received
            }
        }
    } else { // Parent process
        sleep(3); // Allow some time for child process to start

        // Send SIGUSR1 to suspend the child process
        kill(pid, SIGUSR1);
        sleep(3); // Simulate some work being done while child is suspended

        // Send SIGUSR2 to resume the child process
        kill(pid, SIGUSR2);

        // Wait for the child to complete
        wait(NULL);
    }

    return 0;
}
[11:31 PM, 12/11/2023] Suhas😇: Write a C program that illustrates suspending and resuming processes using signals
[11:32 PM, 12/11/2023] Suhas😇:

 #include <stdio.h>
#include <stdlib.h>
#include <dirent.h>
#include <string.h>

void findFilesWithPrefix(const char *prefix) {
    DIR *directory;
    struct dirent *entry;

    // Open the current directory
    directory = opendir(".");

    if (directory == NULL) {
        perror("Error opening current directory");
        exit(EXIT_FAILURE);
    }

    // Iterate over directory entries
    while ((entry = readdir(directory)) != NULL) {
        // Check if the entry is a regular file and starts with the specified prefix
        if (entry->d_type == DT_REG && strncmp(entry->d_name, prefix, strlen(prefix)) == 0) {
            printf("%s\n", entry->d_name);
        }
    }

    // Close the directory
    closedir(directory);
}

int main(int argc, char *argv[]) {
    if (argc != 2) {
        fprintf(stderr, "Usage: %s <prefix>\n", argv[0]);
        exit(EXIT_FAILURE);
    }

    const char *prefix = argv[1];
    findFilesWithPrefix(prefix);

    return 0;
}
[11:33 PM, 12/11/2023] Suhas😇: Write a C program that a string as an argument and return all the files that begins with that name 
in the current directory. For example > ./a.out foo will return all file names that begins with foo
[11:34 PM, 12/11/2023] Suhas😇:


 #include <stdio.h>
#include <stdlib.h>
#include <dirent.h>
#include <sys/stat.h>

void displayFilesGreaterThanNBytes(long n) {
    DIR *directory;
    struct dirent *entry;
    struct stat fileStat;

    // Open the current directory
    directory = opendir(".");

    if (directory == NULL) {
        perror("Error opening current directory");
        exit(EXIT_FAILURE);
    }

    // Iterate over directory entries
    while ((entry = readdir(directory)) != NULL) {
        // Get file information
        if (stat(entry->d_name, &fileStat) == -1) {
            perror("Error getting file information");
            exit(EXIT_FAILURE);
        }

        // Check if the entry is a regular file and its size is greater than n bytes
        if (entry->d_type == DT_REG && fileStat.st_size > n) {
            printf("%s\t%ld bytes\n", entry->d_name, fileStat.st_size);
        }
    }

    // Close the directory
    closedir(directory);
}

int main() {
    long n;

    // Accept n from the user
    printf("Enter the size threshold (in bytes): ");
    scanf("%ld", &n);

    // Display files greater than n bytes
    displayFilesGreaterThanNBytes(n);

    return 0;
}
[11:34 PM, 12/11/2023] Suhas😇: Write a c program Display all the files from current directory whose size is greater that n Bytes Where n is accept Write a c program Display all the files from current directory whose size is greater that n Bytes Where n is accept