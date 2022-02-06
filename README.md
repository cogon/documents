# lwip in freertos add tap driver
### * api:

1. 挂载tap驱动到lwip


   * netif_add(struct netif *netif, ip4_addr_t *ipaddr, ip4_addr_t *netmask, ip4_addr_t *gw,
          void *stat, netif_init_fn init, netif_input_fn input);
   * init中打开tap句柄,  打开os的sys_thread_new 
     设置netif->output = etharp_output
    
       netif->linkoutput =  send_packet ;  lwip 出口最终调用 
       
       netif->input  = input ;    input一般是`tcpip_input()`  是receive_packet后，调用的协议栈的入口。


2. lwip 中启动接受服务

   * sys_thread_new("tcpecho_thread", tcpecho_thread, NULL, DEFAULT_THREAD_STACKSIZE, DEFAULT_THREAD_PRIO);

3. 操作系统需要提供的接口

   * sys_thread_new

     * 创建线程
   * sys_sem_new  sys_arch_sem_wait

     * 信号量（select通过这个实现）
   * sys_mbox_new  sys_arch_mbox_tryfetch


4.lwip 流程
   `````
     etharp_output(){
          if (多播和广播)
                    ethernet_output()
          if (静态arp)
              etharp_output_to_arp_index()
          etharp_query(){
               查询arp表 
               if(is_new_entry)
                    etharp_requst()
               if(state==ETHARP_STATE_PENDING){
                  拷贝当前包放入etharp_q_entry 队列
               }
   `````                 
 
### * 主线程
*   tcpip_thread->tcpip_thread_handle_msg() :收到邮件就处理.调用msg中设置的回调函数   
*   tcpip_input()接受的包  和   lwip_send() 应用线程发送的包  都会将包放到主线程的mailbox里. sys_mbox_post()

*   tcpip_input() -> sys_mbox_trypost() 
     `````
          带mac注册的回调函数是 : 
          ethernet_input(){
                      if(ip包)
                         ip_input()
                      if(arp包)
                         etharp_intpu()
              }
         不带mac回调函数是:     ip_intpu(){
                                 if(inet_chksum()){
                                    return 
                                 }
                                 if(不是本地接受地址){
                                     ip4_forward()
                                     return
                                 }
                                 if(分片数据){
                                     ip4_reass()
                                     return
                                 }
                                 往上层送
                                 if(udp) upd_input
                                 if(tcp) tcp_input
                                 if(icmp) icmp_intpu
                                 if(igmp) igmp_intpu
                            }     
                                    
    ``````
              
*   lwip_send() ->  netconn_send(){ 注册的回调函数是 lwip_netconn_do_send()  }-> netconn_apimsg() -> tcpip_send_msg_wait_sem() -> sys_mbox_post()


  

