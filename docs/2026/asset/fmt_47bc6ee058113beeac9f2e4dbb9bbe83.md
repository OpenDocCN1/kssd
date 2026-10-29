
```c
#include <linux/sched.h>
#include <linux/kernel.h>
#include <linux/sys.h>
#include <linux/fdreg.h>
#include <asm/system.h>
#include <asm/io.h>
#include <asm/segment.h>
#include <signal.h>
#define \_S(nr) (1<<((nr)-1))
#define \_BLOCKABLE (~(\_S(SIGKILL) | \_S(SIGSTOP)))
void show\_task(int nr,struct task\_struct \* p) //nr就是pid
{
int i,j = 4096-sizeof(struct task\_struct);
printk("%d: pid=%d, state=%d, ",nr,p->pid,p->state); //打印pid与state
i=0;
while (i<j && !((char \*)(p+1))[i])
i++;
printk("%d (of %d) chars free in kernel stack\n\r",i,j); //打印栈
}
void show\_stat(void)
{
int i;
for (i=0;i<NR\_TASKS;i++)
if (task[i])
show\_task(i,task[i]);
}
#define LATCH (1193180/HZ)
extern void mem\_use(void);
extern int timer\_interrupt(void);
extern int system\_call(void);
union task\_union {
struct task\_struct task;
char stack[PAGE\_SIZE];
};
static union task\_union init\_task = {INIT\_TASK,};
long volatile jiffies=0;
long startup\_time=0;
struct task\_struct \*current = &(init\_task.task);
struct task\_struct \*last\_task\_used\_math = NULL;
struct task\_struct \* task[NR\_TASKS] = {&(init\_task.task), };
long user\_stack [ PAGE\_SIZE>>2 ] ;
struct {
long \* a;
short b;
} stack\_start = { & user\_stack [PAGE\_SIZE>>2] , 0x10 };
/\*
\* 'math\_state\_restore()' saves the current math information in the
\* old math state array, and gets the new ones from the current task
\*/
void math\_state\_restore() //进程切换时完成协处理器中寄存器的切换
{
if (last\_task\_used\_math == current)
return;
\_\_asm\_\_("fwait");
if (last\_task\_used\_math) {
\_\_asm\_\_("fnsave %0"::"m" (last\_task\_used\_math->tss.i387));
}
last\_task\_used\_math=current;
if (current->used\_math) {
\_\_asm\_\_("frstor %0"::"m" (current->tss.i387));
} else {
\_\_asm\_\_("fninit"::);
current->used\_math=1;
}
}
/\*
\* 'schedule()' is the scheduler function. This is GOOD CODE! There
\* probably won't be any reason to change this, as it should work well
\* in all circumstances (ie gives IO-bound processes good response etc).
\* The one thing you might take a look at is the signal-handler code here.
\*
\* NOTE!! Task 0 is the 'idle' task, which gets called when no other
\* tasks can run. It can not be killed, and it cannot sleep. The 'state'
\* information in task[0] is never used.
\*/
void schedule(void)
{
int i,next,c;
struct task\_struct \*\* p;
/\* check alarm, wake up any interruptible tasks that have got a signal \*/
/\*
#define TASK\_RUNNING 0 只有state是0时，该进程才会被运行，或进入就绪队列
#define TASK\_INTERRUPTIBLE 1 可中断睡眠状态 可以被信号中断，变成running状态
#define TASK\_UNINTERRUPTIBLE 2 不可中断睡眠状态 只能被wakeup函数唤醒，变成running状态
#define TASK\_ZOMBIE 3 僵死状态 进程停止运行，但是其task\_struct未被清空
#define TASK\_STOPPED 4 暂停状态
\*/
for(p = &LAST\_TASK ; p > &FIRST\_TASK ; --p) //从后往前遍历
if (\*p) { //若进程存在
if ((\*p)->alarm && (\*p)->alarm < jiffies) { //若alarm不为空且小于jiffies(此处是0)
(\*p)->signal |= (1<<(SIGALRM-1));
(\*p)->alarm = 0;
}
if (((\*p)->signal & ~(\_BLOCKABLE & (\*p)->blocked)) && //进程不理会某些信号；并且进程是可中断睡眠状态
(\*p)->state==TASK\_INTERRUPTIBLE)
(\*p)->state=TASK\_RUNNING;
}
/\* this is the scheduler proper: \*/
while (1) { //进行counter的比较，来决定进程的调用
c = -1;
next = 0;
i = NR\_TASKS;
p = &task[NR\_TASKS];
while (--i) {
if (!\*--p)
continue;
if ((\*p)->state == TASK\_RUNNING && (\*p)->counter > c)
c = (\*p)->counter, next = i; //遍历之后，会将counter的最大值赋给c，并且next存着最大counter的pid
}
if (c) break;
for(p = &LAST\_TASK ; p > &FIRST\_TASK ; --p)
if (\*p)
(\*p)->counter = ((\*p)->counter >> 1) + //counter = counter/2 + priority
(\*p)->priority;
}
switch\_to(next); //进程切换
}
/\*
这部分代码的目的是在所有就绪状态的任务进程中筛选出counter值最大的进程ID。之后如果counter值不为0则进入调度这个进程执行，如果counter值为0，则说明所有就绪状态的进程的时间片都已用完，需要重新调整所有进程的时间片。
\*/
/\*#define switch\_to(n) {\
struct {long a,b;} \_\_tmp; \
\_\_asm\_\_("cmpl %%ecx,\_current\n\t" \
"je 1f\n\t" \
"movw %%dx,%1\n\t" \
"xchgl %%ecx,\_current\n\t" \
"ljmp %0\n\t" \
"cmpl %%ecx,\_last\_task\_used\_math\n\t" \
"jne 1f\n\t" \
"clts\n" \
"1:" \
::"m" (\*&\_\_tmp.a),"m" (\*&\_\_tmp.b), \
"d" (\_TSS(n)),"c" ((long) task[n])); \
}
\*/
int sys\_pause(void)
{
current->state = TASK\_INTERRUPTIBLE;
schedule();
return 0;
}
void sleep\_on(struct task\_struct \*\*p) //当p进程想访问cpu的某个资源，但是该资源被占用；
{
struct task\_struct \*tmp;
if (!p)
return;
if (current == &(init\_task.task)) //如果当前进程为0号进程时，就返回，不能sleep
panic("task[0] trying to sleep");
tmp = \*p;
\*p = current; //将p赋为当前进程
current->state = TASK\_UNINTERRUPTIBLE;
schedule();
if (tmp)
tmp->state=0;
}
void interruptible\_sleep\_on(struct task\_struct \*\*p)
{
struct task\_struct \*tmp;
if (!p)
return;
if (current == &(init\_task.task))
panic("task[0] trying to sleep");
tmp=\*p;
\*p=current;
repeat: current->state = TASK\_INTERRUPTIBLE;
schedule();
if (\*p && \*p != current) {
(\*\*p).state=0;
goto repeat;
}
\*p=NULL;
if (tmp)
tmp->state=0;
}
void wake\_up(struct task\_struct \*\*p)
{
if (p && \*p) {
(\*\*p).state=0;
\*p=NULL;
}
}
/\*
\* OK, here are some floppy things that shouldn't be in the kernel
\* proper. They are here because the floppy needs a timer, and this
\* was the easiest way of doing it.
\*/
static struct task\_struct \* wait\_motor[4] = {NULL,NULL,NULL,NULL};
static int mon\_timer[4]={0,0,0,0};
static int moff\_timer[4]={0,0,0,0};
unsigned char current\_DOR = 0x0C;
int ticks\_to\_floppy\_on(unsigned int nr)
{
extern unsigned char selected;
unsigned char mask = 0x10 << nr;
if (nr>3)
panic("floppy\_on: nr>3");
moff\_timer[nr]=10000; /\* 100 s = very big :-) \*/
cli(); /\* use floppy\_off to turn it off \*/
mask |= current\_DOR;
if (!selected) {
mask &= 0xFC;
mask |= nr;
}
if (mask != current\_DOR) {
outb(mask,FD\_DOR);
if ((mask ^ current\_DOR) & 0xf0)
mon\_timer[nr] = HZ/2;
else if (mon\_timer[nr] < 2)
mon\_timer[nr] = 2;
current\_DOR = mask;
}
sti();
return mon\_timer[nr];
}
void floppy\_on(unsigned int nr)
{
cli();
while (ticks\_to\_floppy\_on(nr))
sleep\_on(nr+wait\_motor);
sti();
}
void floppy\_off(unsigned int nr)
{
moff\_timer[nr]=3\*HZ;
}
void do\_floppy\_timer(void)
{
int i;
unsigned char mask = 0x10;
for (i=0 ; i<4 ; i++,mask <<= 1) {
if (!(mask & current\_DOR))
continue;
if (mon\_timer[i]) {
if (!--mon\_timer[i])
wake\_up(i+wait\_motor);
} else if (!moff\_timer[i]) {
current\_DOR &= ~mask;
outb(current\_DOR,FD\_DOR);
} else
moff\_timer[i]--;
}
}
#define TIME\_REQUESTS 64
static struct timer\_list {
long jiffies;
void (\*fn)();
struct timer\_list \* next;
} timer\_list[TIME\_REQUESTS], \* next\_timer = NULL;
void add\_timer(long jiffies, void (\*fn)(void))
{
struct timer\_list \* p;
if (!fn)
return;
cli();
if (jiffies <= 0)
(fn)();
else {
for (p = timer\_list ; p < timer\_list + TIME\_REQUESTS ; p++)
if (!p->fn)
break;
if (p >= timer\_list + TIME\_REQUESTS)
panic("No more time requests free");
p->fn = fn;
p->jiffies = jiffies;
p->next = next\_timer;
next\_timer = p;
while (p->next && p->next->jiffies < p->jiffies) {
p->jiffies -= p->next->jiffies;
fn = p->fn;
p->fn = p->next->fn;
p->next->fn = fn;
jiffies = p->jiffies;
p->jiffies = p->next->jiffies;
p->next->jiffies = jiffies;
p = p->next;
}
}
sti();
}
void do\_timer(long cpl)
{
extern int beepcount;
extern void sysbeepstop(void);
if (beepcount)
if (!--beepcount)
sysbeepstop();
if (cpl)
current->utime++;
else
current->stime++;
if (next\_timer) {
next\_timer->jiffies--;
while (next\_timer && next\_timer->jiffies <= 0) {
void (\*fn)(void);
fn = next\_timer->fn;
next\_timer->fn = NULL;
next\_timer = next\_timer->next;
(fn)();
}
}
if (current\_DOR & 0xf0)
do\_floppy\_timer();
if ((--current->counter)>0) return;
current->counter=0;
if (!cpl) return;
schedule();
}
int sys\_alarm(long seconds)
{
int old = current->alarm;
if (old)
old = (old - jiffies) / HZ;
current->alarm = (seconds>0)?(jiffies+HZ\*seconds):0;
return (old);
}
int sys\_getpid(void)
{
return current->pid;
}
int sys\_getppid(void)
{
return current->father;
}
int sys\_getuid(void)
{
return current->uid;
}
int sys\_geteuid(void)
{
return current->euid;
}
int sys\_getgid(void)
{
return current->gid;
}
int sys\_getegid(void)
{
return current->egid;
}
int sys\_nice(long increment)
{
if (current->priority-increment>0)
current->priority -= increment;
return 0;
}
void sched\_init(void)
{
int i;
struct desc\_struct \* p;
if (sizeof(struct sigaction) != 16)
panic("Struct sigaction MUST be 16 bytes");
set\_tss\_desc(gdt+FIRST\_TSS\_ENTRY,&(init\_task.task.tss));
set\_ldt\_desc(gdt+FIRST\_LDT\_ENTRY,&(init\_task.task.ldt));
p = gdt+2+FIRST\_TSS\_ENTRY;
for(i=1;i<NR\_TASKS;i++) {
task[i] = NULL;
p->a=p->b=0;
p++;
p->a=p->b=0;
p++;
}
/\* Clear NT, so that we won't have troubles with that later on \*/
\_\_asm\_\_("pushfl ; andl $0xffffbfff,(%esp) ; popfl");
ltr(0);
lldt(0);
outb\_p(0x36,0x43); /\* binary, mode 3, LSB/MSB, ch 0 \*/
outb\_p(LATCH & 0xff , 0x40); /\* LSB \*/
outb(LATCH >> 8 , 0x40); /\* MSB \*/
set\_intr\_gate(0x20,&timer\_interrupt);
outb(inb\_p(0x21)&~0x01,0x21);
set\_system\_gate(0x80,&system\_call);
}
```
