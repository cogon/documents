# lwip in freertos add tap driver
### api:

1. 挂载tap驱动到lwip
***************************************
````
* netif_add(struct netif *netif, ip4_addr_t *ipaddr, ip4_addr_t *netmask, ip4_addr_t *gw,
          void *stat, netif_init_fn init, netif_input_fn input);
* init中打开tap句柄,  打开os的sys_thread_new 
    设置netif->output = etharp_output
       netif->linkoutput =  send_packet ;  lwip 出口最终调用 
       netif->input  = input ;    input一般是`tcpip_input()`  是receive_packet后，调用的协议栈的入口。
````

2. lwip 中启动接受服务
******************************************
> sys_thread_new("tcpecho_thread", tcpecho_thread, NULL, DEFAULT_THREAD_STACKSIZE, DEFAULT_THREAD_PRIO);

3. 操作系统需要提供的接口
**************************************************
* sys_thread_new

  * 创建线程
* sys_sem_new  sys_arch_sem_wait

  * 信号量（select通过这个实现）
* sys_mbox_new  sys_arch_mbox_tryfetch


4.lwip 流程
 **************************************************
 
### * 主线程
*   tcpip_thread->tcpip_thread_handle_msg() :收到邮件就处理.调用msg中设置的回调函数   
*   tcpip_input()接受的包  和   lwip_send() 应用线程发送的包  都会将包放到主线程的mailbox里. sys_mbox_post()

*   tcpip_input() -> sys_mbox_trypost() 注册的回调函数是 : ethernet_input()   
*   lwip_send() ->  netconn_send(){ 注册的回调函数是 lwip_netconn_do_send()  }-> netconn_apimsg() -> tcpip_send_msg_wait_sem() -> sys_mbox_post()


  

