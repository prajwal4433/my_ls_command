# my_ls_command
<pre>
Project Title: Implementation of a Basic ls -l Command in C

Objective: The objective of this project is to develop a simplified version of the Unix ls -l command using C programming.
The program aims to list the contents of a directory in a long-format style, displaying detailed information about each file,
such as:

File permissions (read/write/execute)
Number of hard links
File owner and group names
File size in bytes
Last modification time
Filename (with symbolic link target if applicable)
This project enhances the understanding of:

File and directory manipulation in Unix-like systems.
Usage of system calls (opendir(), readdir(), lstat(), readlink()).
File metadata handling via struct stat.
Formatting and parsing binary data using bitwise operations.
Displaying human-readable output similar to standard Unix utilities.
</pre>
<pre>
/*my ls command basic level */

#include<stdio.h>
#include<stdlib.h>
#include<dirent.h>
#include<sys/stat.h>
#include<sys/types.h>
#include<pwd.h>
#include<grp.h>
#include<time.h>
#include<string.h>
#include<unistd.h>

void print_permissions(mode_t mode);
void print_file_info(const char *name, const struct stat *file_stat);

int main(int argc, char *argv[])
{
    DIR *dir;
    struct dirent *entry;
    struct stat file_stat;
    const char *path = "."; //Default to current directory

    //Handle command line argument
    if(argc>1)
    {
        path=argv[1];
    }

    //Open directory
    if((dir=opendir(path))==NULL)
    {
        perror("opendir");
        exit(EXIT_FAILURE);
    }

    //Read directory entries
    while((entry=readdir(dir))!=NULL)
    {
            char full_path[1024];

        //Skip "." and ".." entries
        if(strcmp(entry->d_name, ".")==0||strcmp(entry->d_name, "..")==0)
        {
            continue;
        }

        //Construct full path
        snprintf(full_path, sizeof(full_path), "%s/%s", path, entry->d_name);

        // Get file status
        if(lstat(full_path, &file_stat)==-1)
        {
            perror("lstat");
            continue;
        }

        // Print file information
        print_file_info(entry->d_name, &file_stat);
    }

    closedir(dir);
    return EXIT_SUCCESS;
}

void print_permissions(mode_t mode)
{
    printf((S_ISDIR(mode)) ? "d" : (S_ISLNK(mode)) ? "l" : "-");
    printf((mode & S_IRUSR) ? "r" : "-");
    printf((mode & S_IWUSR) ? "w" : "-");
    printf((mode & S_IXUSR) ? "x" : "-");
    printf((mode & S_IRGRP) ? "r" : "-");
    printf((mode & S_IWGRP) ? "w" : "-");
    printf((mode & S_IXGRP) ? "x" : "-");
    printf((mode & S_IROTH) ? "r" : "-");
    printf((mode & S_IWOTH) ? "w" : "-");
    printf((mode & S_IXOTH) ? "x" : "-");
}

void print_file_info(const char *name, const struct stat *file_stat)
{
    struct passwd *pwd;
    struct group *grp;
    char time_str[256];
    struct tm *tm_info;

    //File permissions
    print_permissions(file_stat->st_mode);
    printf(" ");

    //Number of links
    printf("%ld ", file_stat->st_nlink);

    //Owner name
    if((pwd = getpwuid(file_stat->st_uid))!=NULL)
    {
        printf("%s ", pwd->pw_name);
    }
    else
    {
        printf("%d ", file_stat->st_uid);
    }

    //Group name
    if((grp=getgrgid(file_stat->st_gid))!=NULL)
    {
        printf("%s ", grp->gr_name);
    }
    else
    {
        printf("%d ", file_stat->st_gid);
    }

    //File size
    printf("%8ld ", file_stat->st_size);

    //Modification time
    tm_info = localtime(&file_stat->st_mtime);
    strftime(time_str, sizeof(time_str), "%b %d %H:%M", tm_info);
    printf("%s ", time_str);

    //File name
    printf("%s", name);

    //For symbolic links, show the target
    if(S_ISLNK(file_stat->st_mode))
    {
        char link_target[1024];
        ssize_t len=readlink(name, link_target, sizeof(link_target)-1);
        if(len!=-1)
        {
            link_target[len]='\0';
            printf(" -> %s", link_target);
        }
    }

    printf("\n");
}
</pre>
