1. 两个指针`pointer1, pointer2`指向同一块内存空间，使用`free(pointer1)`释放了内存，那么`pointer2也无法再访问这块呢噢村空间`。
```C
#include <stdio.h>
#include "stdlib.h"

typedef struct step{
    int *a;
    int b;
}step;

typedef struct readsList{
    step *data;
    int a;
    struct readsList *next;
}readsList;

typedef struct resultList{
    int *data;
    struct resultList *next;
}resultList;

int main() {
    readsList *read = (readsList *) malloc(sizeof(readsList));
    read->a = 1;
    readsList *test = read;
    printf("%d\n", test->a);
    free(read);
    printf("%d\n", test->a);
}

/*******运行结果******/
1
1589481488

进程已结束,退出代码0
```
2. 链表：当声明了一个链表但没有初始化（没有分配指向的内存空间），那么该链表本身是有一个地址的，但是无法访问链表的数据结构，因为它还未初始化。
```C
#include <stdio.h>
#include "stdlib.h"

//声明节点结构
typedef struct Link {
    int  elem;//存储整形元素
    struct Link *next;//指向直接后继元素的指针
}link;

int main(){
    link *head = (link*) malloc(sizeof(link));
    printf("%d\n", head == NULL);  //有指向的内存空间，所以不等于NULL
    printf("%d\n", head->next == NULL);  //head->next没有指向的内存空间，所以等于NULL
    link *end = head->next;
    printf("%d\n", end == NULL);  //由于head指向的内存空间没变，所以end指向的和head->next指向的空间为“同一块”，但都没有分配
    link *test;
    printf("%d\n", test == NULL);  
    printf("%p\n", &test);
    printf("%d\n", end->next == NULL);  //由于test没有分配空间，所以end->next根本还不存在，因此会报段错误
}

/*******运行结果******/
0
1
1
1
0x7ffd243cd660

进程已结束,退出代码139 (interrupted by signal 11: SIGSEGV)  //对应最后一条printf()语句
```
3. 链表头尾指针问题：可以使用`head`、`end`来指向`list`，但是`list`本身不进行移动，那么三者之间互不干扰。
```c
#include <stdio.h>
#include "stdlib.h"

//声明节点结构
typedef struct Link {
    int  elem;//存储整形元素
    struct Link *next;//指向直接后继元素的指针
}link;

int main(){
    link *list = (link*) malloc(sizeof(link));
    link *head = list;
    link *end = list;
    for(int i = 0; i < 5; i++){
        end->elem = i + 1;
        end->next = (link*) malloc(sizeof(link));
        end = end->next;
    }
    for(int i = 0; i < 5; i++){
        printf("%d\n", head->elem);
        head = head->next;
    }
    for(int i = 0; i < 5; i++){
        printf("%d\n", list->elem);
        list = list->next;
    }
}

/*******运行结果******/
1
2
3
4
5
1
2
3
4
5

进程已结束,退出代码0
```
4. 