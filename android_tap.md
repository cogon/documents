Androio  VpnService 
--------------
创建一个类extends VpnService 

onStartCommand方法中 val builder = Builder()
  
    用c语言实现  startvpn(builder.establish.getFd(), server_ip ,server_port)
                stopvpn()
           
                
jni 接口
---------------

     startvpn()
     {  创建一个线程handle_event()
     }
     handle_event()
     {
          struct arguments *args = (struct arguments *) a;
          log_android(ANDROID_LOG_WARN, "Start events tun=%d thread %x", args->tun, thread_id);

          // Attach to Java
         JNIEnv *env;
          jint rs = (*jvm)->AttachCurrentThread(jvm, &env, NULL);
     }
