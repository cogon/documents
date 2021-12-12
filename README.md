# lwip in freertos add tap driver
api:

1. 挂载tap驱动到lwip

````
* netif_add(struct netif *netif, ip4_addr_t *ipaddr, ip4_addr_t *netmask, ip4_addr_t *gw,
          void *stat, netif_init_fn init, netif_input_fn input);
* init中打开tap句柄,  打开os的sys_thread_new 
    设置netif->output = etharp_output
       netif->linkoutput =  send_packet ;  lwip 出口最终调用 
       netif->input  = input ;    input一般是`tcpip_input()`  是receive_packet后，调用的协议栈的入口。
````

2. lwip 中启动接受服务

> sys_thread_new("tcpecho_thread", tcpecho_thread, NULL, DEFAULT_THREAD_STACKSIZE, DEFAULT_THREAD_PRIO);

3. 操作系统需要提供的接口

* sys_thread_new

  创建线程
* sys_sem_new  sys_arch_sem_wait

  信号量（select通过这个实现）
* sys_mbox_new  sys_arch_mbox_tryfetch
* 
