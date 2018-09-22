#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <fcntl.h> 
#include <sys/types.h>
#include <sys/stat.h>
#include <ctype.h>
#include <dirent.h>
#include <libgen.h>

// function prototypes
void sys(char *path);
void pids(char *path, int names);
int is_proc(char *name);
void sort_found_contents(char **found_contents, int size_contents);
int str2int(char *pid_str, int digit_count);
int get_pid(char *content);
void swap_contents(char **contents, int index_1, int index_2);
void sort_found_pids(char **found_contents, int size_contents);
void ps(char *path, int extended_action, char *parent_id);
char *extract_pid(char *contents);
char *extract_ppid(char *contents);
char *extract_state(char *contents);
char *extract_name(char *contents);
void sort_found_contents_ps(char **found_contents, int size_contents);
int contains_id(int *parent_ids_list, int parent_ids_list_size, int ppid);

// main function
int main(int argc, char **argv)
{
	// parsing path to directory
	// default path is /proc
	// char *path = "/home/jernej/proc-demo";
	char *path = "/proc";
	// If path argument is given and action is not equal to "forktree"
	if(argc >= 3 && strcmp(argv[1], "forktree") != 0) 
	{
		path = argv[2];
	}

	if(argc > 1) 
	{
		if(strcmp(argv[1], "sys") == 0)
		{
			sys(path);
		}

		else if(strcmp(argv[1], "pids") == 0)
		{
			int names = 0;
			pids(path, names);
		}
		if(strcmp(argv[1], "names") == 0)
		{
			int names = 1;
			pids(path, names);
		}
		
		if(strcmp(argv[1], "ps") == 0)
		{
			int extended_action = 0;
			char *parent_id = NULL;
			if(argc >= 4) {
				extended_action = 1;
				parent_id = argv[3];
			}
			ps(path, extended_action, parent_id);
		}	
	}
	return 0;
}	

void sys(char *path)
{
// sys action //////////////////////
	
	// linux version ///////////////////

	// compute working path and file descriptor
	char *working_path = (char*)malloc((strlen(path) + strlen("/version"))*sizeof(char));
	working_path = strcpy(working_path, path);
	working_path = strcat(working_path, "/version");

	// get file descriptor
		int fd = open(working_path, O_RDONLY);

	// Allocate string buffer
	char *buff = (char*)malloc(1024*sizeof(char));
	// read into buffer
	read(fd, buff, 1024);

	// move to first digit
	int str_index = 0;
	while(!isdigit(buff[str_index]))
	{
		str_index++;
	}

	// print linux version
	printf("Linux: ");
	while(!isspace(buff[str_index]))
	{
		putchar(buff[str_index]);
		str_index++;
	}
	putchar('\n');

	// GCC version //////////////////////

	// get to "version" substring
	while(strncmp(buff + str_index, "version", 7) != 0)
	{
		str_index++;
	}
	
	// get to digits
	while(!isdigit(buff[str_index]))
	{
		str_index++;
	}

	// print result 
	printf("gcc: ");
	while(!isspace(buff[str_index]))
	{
		putchar(buff[str_index]);
		str_index++;
	}

	putchar('\n');

	// name of first swap partition /////////

	// compute new working path
	free(working_path);
	working_path = (char*)malloc((strlen(path) + strlen("/swaps"))*sizeof(char));
	working_path = strcpy(working_path, path);
	working_path = strcat(working_path, "/swaps");

	// get file descriptor
	close(fd);
	fd = open(working_path, O_RDONLY);
	read(fd, buff, 1024);

	// get to first '/' character
	str_index = 0;
	while(buff[str_index] != '/')
	{
		str_index++;
	}

	// print result
	printf("Swap: ");

	while(!isspace(buff[str_index]))
	{
		putchar(buff[str_index]);
		str_index++;
	}
	putchar('\n');

	// number of kernel modules ///////////////

	// compute new working path
	free(working_path);
	working_path = (char*)malloc((strlen(path) + strlen("/swaps"))*sizeof(char));
	working_path = strcpy(working_path, path);
	working_path = strcat(working_path, "/modules");

	// get file descriptor
	fd = open(working_path, O_RDONLY);

	// module descriptions are stored in separate lines - count lines
	int mod_count = 0;
	char temp[1];
	while(read(fd, temp, 1))
	{
		if(temp[0] == '\n') {
			mod_count++;
		}
	}
	// print result
	printf("Modules: %d\n", mod_count);
	close(fd);
	///////////////////////////////////////////
}

void pids(char *path, int names)
{

	// pids //////////////////////////////////
	DIR *dir;
	DIR *sub_dir;
    struct dirent *dir_ptr;
    char *buff = calloc(128, sizeof(char));

    char **found_contents = NULL;
    int size_contents = 0;

    if((dir = opendir(path)) == NULL) 
    {
        perror("Error");
        exit(1);
    }
    
    while ((dir_ptr = readdir(dir)) != NULL)
    {
    	char *pid = dir_ptr->d_name;
    	if(is_proc(pid))
    	{
    		// enter dir and find comm file
    		char *sub_path = (char*)calloc((strlen(path) + strlen(pid) + 1 + 1), sizeof(char));

    		strcpy(sub_path, path);
    		strcat(sub_path, "/");
    		strcat(sub_path, pid);


    		if((sub_dir = opendir(sub_path)) == NULL) 
    		{
        		perror("Error");
        		exit(1);
    		}
    		struct dirent *sub_dir_ptr;
    		printf("Scanning dir %s\n", sub_path);
    		while((sub_dir_ptr = readdir(sub_dir)) != NULL)
    		{
    			
    			// if file is comm
    			char *name = sub_dir_ptr->d_name;
 	   			if(!strcmp(name, "stat")) {
 	   				char *status_path = calloc((strlen(sub_path) + 5), sizeof(char));
 	   				strcpy(status_path, sub_path);
 	   				strcat(status_path, "/");
 	   				strcat(status_path, "stat");
    				int fd = open(status_path, O_RDONLY);
 					read(fd, buff, 128);
 					printf("READ %s FROM %s\n", buff, status_path);


 					if(strlen(buff) >= 1){
	    				found_contents = (char**)realloc(found_contents, (++size_contents)*sizeof(char*));
	    				found_contents[size_contents - 1] = (char*)calloc((strlen(pid) + strlen(buff) + 1), sizeof(char));
	    				strcpy(found_contents[size_contents - 1], pid);
	    				strcat(found_contents[size_contents - 1], " ");
	    				strncat(found_contents[size_contents - 1], buff, strlen(buff));
    				}
					buff = NULL;
 					buff = calloc(128, sizeof(char));
    			}
    		}
    		closedir(sub_dir);
    	}
    }
    closedir(dir);
    if(!names) {
    	sort_found_pids(found_contents, size_contents);
    	for(int i = 0; i < size_contents; i++) 
    	{
    	printf("%d\n", get_pid(found_contents[i]));
   		}
    } 
    else if(names) 
    {
    	sort_found_contents(found_contents, size_contents);
    	for(int i = 0; i < size_contents; i++) 
    	{
	    	printf("%s", found_contents[i]);
   		}	
    }
    
}

void ps(char *path, int extended_action, char *parent_id)
{
	// pids //////////////////////////////////
	DIR *dir = NULL;
	DIR *sub_dir = NULL;
    struct dirent *dir_ptr = NULL;
    char *buff = calloc(65536, sizeof(char));

    char **found_contents = NULL;
    int size_contents = 0;

    int size_pid = 0;
    int size_ppid = 0;
    int size_stat = 0;
    int size_name = 0;

    int parent_ids_list[1024];
    if(extended_action)
    {
    	parent_ids_list[0] = atoi(parent_id);	
    }
    int parent_ids_list_size = 1;

    if((dir = opendir(path)) == NULL)
    {
        perror("Error");
        exit(1);
    }
    
    while ((dir_ptr = readdir(dir)) != NULL)
    {
    	char *pid = dir_ptr->d_name;
    	if(is_proc(pid))
    	{
    		// enter dir and find comm file
    		char *sub_path = (char*)calloc((strlen(path) + strlen(pid) + 1 + 1), sizeof(char));

    		strcpy(sub_path, path);
    		strcat(sub_path, "/");
    		strcat(sub_path, pid);

    		if((sub_dir = opendir(sub_path)) == NULL) 
    		{
        		perror("Error");
        		exit(1);
    		}

    		struct dirent *sub_dir_ptr;
    		while((sub_dir_ptr = readdir(sub_dir)) != NULL)
    		{
    			// if file is comm
    			char *name = sub_dir_ptr->d_name;
 	   			if(!strcmp(name, "status")) {
 	   				char *status_path = calloc((strlen(sub_path) + 8), sizeof(char));
 	   				strcpy(status_path, sub_path);
 	   				strcat(status_path, "/");
 	   				strcat(status_path, "status");

    				int fd = open(status_path, O_RDONLY);
 					read(fd, buff, 65535);
 					close(fd);

 					// STRING PROCESSING
 					int buff_index = 0;
 					int buff_index_temp = 0;
 					while(strncmp(buff + buff_index, "Name:", 5) != 0) {
 						buff_index++;
 					}

 					buff_index += 6;

 					char *name  = NULL;
 					size_name = 0;
 					buff_index_temp= buff_index;
 					while(*(buff + buff_index_temp) != '\n') {
 						size_name++;
 						buff_index_temp++;
 					}

 					name = (char*)calloc(size_name, sizeof(char));
 					strncpy(name, buff + buff_index, size_name);

 					
 					while(strncmp(buff + buff_index, "State:", 6) != 0) {
 						buff_index++;
 					}

					
 					buff_index += 7;

 					char *status  = NULL;
 					size_stat = 0;
 					buff_index_temp = buff_index;
 					while(*(buff + buff_index_temp) != ' ') {
 						size_stat++;
 						buff_index_temp++;
 					}
 					
 					status = calloc(size_stat, sizeof(char));
 					strncpy(status, buff + buff_index, size_stat);

 					while(strncmp(buff + buff_index, "Pid:", 4) != 0) {
 						buff_index++;
 					}

					buff_index += 5;

 					char *pid  = NULL;
 					size_pid = 0;
 					buff_index_temp = buff_index;

 					while(*(buff + buff_index_temp) != '\n') {
 						size_pid++;
 						buff_index_temp++;
 					}

 					pid = (char*)calloc(size_pid, sizeof(char));
 					strncpy(pid, buff + buff_index, size_pid);

 					
 					while(strncmp(buff + buff_index, "PPid:", 5) != 0) {
 						buff_index++;
 					}

 					buff_index += 6;

 					char *ppid  = NULL;
 					buff_index_temp = buff_index;
 					size_ppid = 0;
 					while(*(buff + buff_index_temp) != '\n') {
 						size_ppid++;
 						//ppid = (char*)realloc(ppid, (size_ppid)*sizeof(char));
 						//ppid[size_ppid - 1] = *(buff + buff_index);
 						buff_index_temp++;
 					}
 					ppid = (char*)calloc(size_pid, sizeof(char));
 					strncpy(ppid, buff + buff_index, size_ppid);		 					
 					if(extended_action)
 					{
 						if(strcmp(parent_id, pid) != 0 && !contains_id(parent_ids_list, parent_ids_list_size, atoi(ppid)))
 						{
 							continue;
 						}
 						
 						parent_ids_list[parent_ids_list_size] = atoi(pid);
 						parent_ids_list_size++;
 					}

    				found_contents = (char**)realloc(found_contents, (++size_contents)*sizeof(char*));
    				found_contents[size_contents - 1] = (char*)calloc(strlen(name) + 1 + strlen(status) + 1 + strlen(pid) + 1 + strlen(ppid) + 1, sizeof(char));
    				strcpy(found_contents[size_contents - 1], pid);
    				strcat(found_contents[size_contents - 1], " ");
    				strcat(found_contents[size_contents - 1], ppid);
    				strcat(found_contents[size_contents - 1], " ");
    				strcat(found_contents[size_contents - 1], status);
    				strcat(found_contents[size_contents - 1], " ");
    				strcat(found_contents[size_contents - 1], name);

					buff = NULL;
 					buff = calloc(65536, sizeof(char));
    			}
    		}
    		closedir(sub_dir);
		   	sub_dir = NULL;	
		
    	}
    		
    }
    
    closedir(dir);
    dir = NULL;

	sort_found_contents_ps(found_contents, size_contents);
	printf("  PID  PPID STANJE IME\n");
	for(int i = 0; i < size_contents; i++) {
		printf("%5s ", extract_pid(found_contents[i]));
		printf("%5s ", extract_ppid(found_contents[i]));	
		printf("%6s ", extract_state(found_contents[i]));
		printf("%s\n", extract_name(found_contents[i]));
	}
}

int is_proc(char *name)
{
	return isdigit(name[0]) ? 1 : 0;
}

void sort_found_contents(char **found_contents, int size_contents)
{
	for(int i = 0; i < size_contents - 1; i++)
	{
		int min_index = i;
		for(int j = i + 1; j < size_contents; j++)
		{
			int index_start_name_fst = 0;
			while(found_contents[min_index][index_start_name_fst] != ' ') {
				index_start_name_fst++;
			}
			index_start_name_fst++;

			int index_start_name_snd = 0;
			while(found_contents[j][index_start_name_snd] != ' ') {
				index_start_name_snd++;
			}
			index_start_name_snd++;

			if(strcasecmp(found_contents[min_index] + index_start_name_fst, found_contents[j] + index_start_name_snd) > 0)
			{
				min_index = j;
			}
			else if(strcasecmp(found_contents[min_index] + index_start_name_fst, found_contents[j] + index_start_name_snd) == 0)
			{
				int pid_1 = get_pid(found_contents[min_index]);
				int pid_2 = get_pid(found_contents[j]);
				if(pid_2 < pid_1) {
					min_index = j;
				}
			}
		}
		if(min_index != i)
		{
			swap_contents(found_contents, i, min_index);
		}
		
	}
}

void sort_found_pids(char **found_contents, int size_contents) {
	for(int i = 0; i < size_contents - 1; i++) {
		int index_min = i;
		for(int j = i + 1; j < size_contents; j++) {
			int pid_1 = get_pid(found_contents[index_min]);
			int pid_2 = get_pid(found_contents[j]);
			if(pid_2 < pid_1) {
				index_min = j;
			}
		}

		if(index_min != i) {
			swap_contents(found_contents, i, index_min);
		}
	}
}

void sort_found_contents_ps(char **found_contents, int size_contents)
{
	for(int i = 0; i < size_contents - 1; i++)
	{
		int index_min = i;
		for(int j = i + 1; j < size_contents; j++)
		{
			int pid_1 = get_pid(found_contents[index_min]);
			int pid_2 = get_pid(found_contents[j]);
			if(pid_2 < pid_1) {
				index_min = j;
			}
		}
		if(i != index_min)
		{
			swap_contents(found_contents, i, index_min);
		}
	}
}

int get_pid(char *content)
{
	int digit_count = 0;
	int pid_index = 0;
	while(content[pid_index] != ' ')
	{
		digit_count++;
		pid_index++;
	}

	char *pid_str = (char*)calloc(digit_count + 1, sizeof(char));
	strncpy(pid_str, content, digit_count);
	int pid_int = str2int(pid_str, digit_count);
	return pid_int;
}

//TODO
int str2int(char *pid_str, int digit_count)
{
	return atoi(pid_str);
}

void swap_contents(char **contents, int index_1, int index_2) {
	char *temp = contents[index_1];
	contents[index_1] = contents[index_2];
	contents[index_2] = temp;
}


char *extract_pid(char *contents)
{

	int size = 0;
	int temp_index = 0;
	while(contents[temp_index] != ' ')
	{
		size++;
		temp_index++;
	}

	char *res = (char*)calloc(size + 1, sizeof(char));
	strncpy(res, contents, size);
	return res;
}


char *extract_ppid(char *contents)
{
	int space_counter = 0;
	int contents_index = 0;
	while(space_counter < 1)
	{
		if(contents[contents_index] == ' ')
		{
			space_counter++;
		}
		contents_index++;
	}

	int size = 0;
	int temp_index = contents_index;
	while(contents[temp_index] != ' ')
	{
		size++;
		temp_index++;
	}

	char *res = (char*)calloc(size + 1, sizeof(char));
	strncpy(res, contents + contents_index, size);
	return res;
}

char *extract_state(char *contents)
{
	int space_counter = 0;
	int contents_index = 0;
	while(space_counter < 2)
	{
		if(contents[contents_index] == ' ')
		{
			space_counter++;
		}
		contents_index++;
	}

	int size = 0;
	int temp_index = contents_index;
	while(contents[temp_index] != ' ')
	{
		size++;
		temp_index++;
	}

	char *res = (char*)calloc(size + 1, sizeof(char));
	strncpy(res, contents + contents_index, size);
	return res;
}

char *extract_name(char *contents)
{
	int space_counter = 0;
	int contents_index = 0;
	while(space_counter < 3)
	{
		if(contents[contents_index] == ' ')
		{
			space_counter++;
		}
		contents_index++;
	}

	int size = 0;
	int temp_index = contents_index;
	while(contents[temp_index] != ' ')
	{
		size++;
		temp_index++;
	}

	char *res = (char*)calloc(size + 1, sizeof(char));
	strncpy(res, contents + contents_index, size);
	return res;
}

int contains_id(int *parent_ids_list, int parent_ids_list_size, int ppid)
{
	for(int i = 0; i < parent_ids_list_size; i++)
	{
		if(parent_ids_list[i] == ppid)
		{
			return 1;
		}
	}
	return 0;
}