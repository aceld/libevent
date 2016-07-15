#4 libevent常用设置

 
 libevent 有一些被整个进程共享的、影响整个库的全局设置。
  
  
  

>### **必须在调用libevent 库的任何其他部分之前修改这些设置，否则，libevent 会进入不一致的状态。**

