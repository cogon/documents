Androio  VpnService 
--------------
创建一个类extends VpnService 

    public int onStartCommand(){
        val builder = Builder()
        builder.establish()
        jni_start(builder.establish.getFd(), server_ip ,server_port)
   }
  
                
jni 接口
---------------

     jni_start()
     { 
        pthread_create(handle_event())
     }
     handle_event()
     {
          struct arguments *args = (struct arguments *) a;
          log_android(ANDROID_LOG_WARN, "Start events tun=%d thread %x", args->tun, thread_id);
          service_main();
          // Attach to Java
         JNIEnv *env;
          jint rs = (*jvm)->AttachCurrentThread(jvm, &env, NULL);
     }
