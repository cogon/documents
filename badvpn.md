Reactor 
--------
* BReactor_Exec()  -> fd_handler -> read() :  Event Loop
* BTap_Init2() 
    *           -> BFileDescriptor_Init(bfd, fd ,fd_handler)
    *           -> BReactor_AddFileDescriptor(bfd)
    *           -> PacketRecvInterface_Init( Btap->output, output_handler_recv)
                  *  -> BPending_Init(&Btap->output->job_operation, pg, (BPending_handler)_PacketRecvInterface_job_operation, i);
                  *  -> BPending_Init(&Btap->output->job_done, pg, (BPending_handler)_PacketRecvInterface_job_done, i);
    *         fd_handler 与 output_handler_recv 功能似乎是一样的,都是从Btap的fd读数据
    
BTap
------
`````
typedef struct {
    BReactor *reactor;
    PacketRecvInterface output;
    uint8_t *output_packet;
    
    int fd;
    BFileDescriptor bfd;
    
} BTap;
`````

Interface
----------
````
typedef struct {
    // provider data
    int mtu;
    PacketRecvInterface_handler_recv handler_operation;
    void *user_provider;
    
    // operation job
    BPending job_operation;       //在这个pinging里执行handler_operation()
    uint8_t *job_operation_data;
     
    // done job
    BPending job_done; 
    int job_done_len;
 } PacketRecvInterface  , PacketPassInterface;
`````



BPending
----------
管理待指向作业队列
````
typedef {
  BPending_handler handler;
  void *user;
} BSmallPending;
BPendingGroup_ExecuteJob()->  handler(user)
`````


SinglePacketBuffer
------

SinglePacketBuffer_Init (SinglePacketBuffer *o, PacketRecvInterface *input, PacketPassInterface *output, BPendingGroup *pg)



数据流:
----------
BTap->RecvInterface-> SinglePacketBuffer-> PassInterface ->  pbuf_alloc-> netif.input()
