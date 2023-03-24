# 单向链表
```
// 想写一个队列，供IOCP线程写入。
typedef struct IOBUFFER
{
    char buffer[2048];
    int bufferLen;
    int readLen; //已读取的长度
    int socket;
    struct sockaddr_in sa;
    time_t ttl;
    struct IOBUFFER* next;
}DATA_QUEUE,* LPDATA_QUEUE;

LPDATA_QUEUE initDataQueue()
{
    LPDATA_QUEUE queue=(LPDATA_QUEUE)malloc(sizeof(DATA_QUEUE));
    memset(queue,0,sizeof(DATA_QUEUE));
    return queue;
}

BOOL dataQueuePushBack(LPDATA_QUEUE queue,LPDATA_QUEUE pNext)
{
    if(NULL==queue)
    {
        return false;
    }
    if(NULL==pNext)
    {
        return false;
    }
    //尾插法
    LPDATA_QUEUE tmp=queue;
    while(tmp->next)
    {
        tmp=tmp->next;
    }
    tmp->next=pNext;
    //printf("tmp is 0x%x\n",tmp);
    return true;
}

LPDATA_QUEUE dataQueuePopFront(LPDATA_QUEUE queue)
{
    if(NULL==queue||NULL==queue->next)
    {
        return NULL;
    }
    //返回第一个节点的指针，将第二个节点移动到第一个节点，返回的指针由外部负责析构
    LPDATA_QUEUE tmp=NULL;
    tmp=queue->next;
    //printf("tmp is %x\n",tmp);
    queue->next=tmp->next;
    return tmp;
}

BOOL deleteDataQueueNode(LPDATA_QUEUE queue,LPDATA_QUEUE node)
{
    if(NULL==queue)
    {
        return false;
    }
    if(NULL==node)
    {
        return false;
    }
    LPDATA_QUEUE tmp=queue;
    while(tmp->next)
    {
        if(tmp->next==node)
        {
            //当前节点的下一个节点是需要删除的节点
            tmp->next=node->next;
            free(node);
            node=NULL;
            break;
        }
        tmp=tmp->next; //指针后移
    }
    return true;
}

int getDataQueueLength(LPDATA_QUEUE queue)
{
    int i=0;
    while(queue->next)
    {
        i++;
        queue=queue->next;
    }
    return i;
}
```
## 注意事项
1. 无论是单向链表还是其他链表，一定要记住一件事情，最开始的空节点是很有必要的，即需要有一个不放数据的头节点。后续的删除和其他的操作，在头节点上操作会简单很多。