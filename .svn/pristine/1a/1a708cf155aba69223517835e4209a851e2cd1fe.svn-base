// See Assignment instructions for tree description syntax!
#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <unistd.h>
#include <wait.h>

// Node structure - represents a node in the tree "skeleton" of the process hierarchy.
typedef struct Node {
	struct Node **children;
	struct Node *parent;
	int num_children;
} *Node;

// function prototypes
void construct_tree(Node root, Node prev, int *input_data, int input_data_size, int num_children, int child_parse_index);
void traversal(Node root, int root_pid, char *root_pid_str);

// main function
int main(void)
{
	// Define array for storing data parsed from stdin.
	int *input_data = NULL;
	// Define array size counter.
	int input_data_size = 0;
	// Define temporary buffer for storing the current parsed value.
	int temp_buff;

	// See Assignment instructions for tree description syntax!
	// While scanner has read some bytes from stdin...
	while(scanf("%d", &temp_buff) > 0)
	{
		// Allocate memory for the next entry in the input_data array.
		input_data = (int*)realloc(input_data, (++input_data_size) * sizeof(int));
		// Put data from temp_buff into the newly allocated memory
		input_data[input_data_size - 1] = temp_buff;
	}

	/*
    for(int i = 0; i < input_data_size; i++)
    {
        printf((i == input_data_size - 1) ? "%d\n" : "%d ", input_data[i]);
    }
    printf("\n");
	*/

	// Initialize root node.
	Node root = (Node)malloc(sizeof(Node));
	// Get number of children of root.
	int num_children = input_data[0];
	// Construct tree on root.
	construct_tree(root, NULL, input_data, input_data_size, num_children, 1);

	// Get pid of root process.
	int root_pid = getpid();

	// Get string representation for root_pid (For exec call).
	char *root_pid_str;
	sprintf(root_pid_str, "%d", root_pid);

	// Traverse built tree and create process hierarchy that mimics the tree's structure.
	traversal(root, root_pid, root_pid_str);	
	
	return 0;
}

// construct_tree: Constructs tree form parsed input data stored in input_data array and returns pointer to root
// The input is formated according to the forktree function specification.
void construct_tree(Node root, Node prev, int *input_data, int input_data_size, int num_children, int child_parse_index)
{
	// Set parent to calling Node.
	root->parent = prev;
	root->num_children = num_children;
	root->children = NULL;
	// Base case - Node has no children
	if(num_children <= 0)
	{
		return;
	}

	// Initialize array of pointer to child nodes.
	root->children = (Node*)malloc(num_children*sizeof(Node));

	// Add children nodes.
	for(int i = 0; i < num_children; i++)
	{
		(root->children)[i] = (Node)malloc(sizeof(Node));
	}

	// If tree data for children does not exist there are no more children.
	if(child_parse_index >= input_data_size)
	{
		return;
	}

	// Compute starting point where children will parse their children info.
	

	// Build trees on children.
	for(int i = 0; i < num_children; i++)
	{

		int num_children_child = (child_parse_index + i >= input_data_size) ? 0 : input_data[child_parse_index + i];
		int follow_up = 0;
		for(int j = child_parse_index; j < child_parse_index + i; j++)
		{
			follow_up += input_data[j];
		}

		int parse_start_index = child_parse_index + num_children + follow_up;
		
		printf("processing index %d, parse_start_index == %d\n", child_parse_index + i, parse_start_index);

		construct_tree((root->children[i]), root, input_data, input_data_size, num_children_child, parse_start_index);
		//parse_start_index += num_children_child;
	}
}

// Traversal: Traverse a tree of Nodes and create a process structure that mimics its structure.
void traversal(Node root, int root_pid, char *root_pid_str)
{

	// If in parent process...
	if(getpid() == root_pid)
	{
		// Create a new process for executing the pstree command.
		pid_t exec_pid = fork();
		// If in child process...
		if(exec_pid == 0)
		{
			// Wait for all branches to fully extend.
			sleep(2);
			// Execute the pstree command.
			execlp("pstree", "pstree", "-c", root_pid_str, NULL);
			// Kill this process.
			_exit(0);
		}
	}

	// If leaf process... pause (base case)
	if(root->children == NULL)
	{
		pause();
	}

	// Go over children of root.
	for(int i = 0; i < root->num_children; i++)
	{
		// Create new process.
		pid_t pid = fork();

		// If child, recur for its children
		if(pid == 0)
		{
			traversal((root->children)[i], root_pid, root_pid_str);	
		}
	}
	// If root process...
	if(getpid() == root_pid)
	{
		// Wait for the pstree calling process to die.
		wait(NULL);
		// Exit program.
		exit(0);
	}
	// Else pause.
	pause();
}