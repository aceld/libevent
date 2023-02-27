# 《Libevent深入浅出》

本教程要求有一定的服务并发编程基础，了解select和epoll等多路I/O复用机制。

教程目的主要是快速建立libevent的认知，了解libevent的常用数据结构和编程方法。

达到可以使用libevent写出自己的高并发服务器处理模型。

> ## 作者：刘丹冰
>
> 邮箱：danbing\_at@163.com

![](/assets/libevent深入浅出封面.jpg)

---
# 目录

* [Libevent深入浅出](README.md)
* [1 Libevent官方](chapter1.md)
* [2 epoll](2-epoll.md)
   * [2.1 流-IO操作-阻塞](21-流-io.md)
   * [2.2 解决阻塞死等待的办法](21-解决阻塞死等待的办法.md)
   * [2.3 什么是epoll](23-什么是epoll.md)
   * [2.4 epollAPI](24-epollapi.md)
   * [2.5 触发模式](25hong_fa_mo_5f0f_md.md)
   * [2.6 简单的epoll服务器](26-简单的epoll服务器.md)
* [3 epoll和reactor](3-epoll和reactor.md)
   * [3.1 reactor反应堆模式](31_reactorfan_ying_dui_mo_shi.md)
   * [3.2 epoll的反应堆模式实现](32_epollde_fan_ying_dui_mo_shi_shi_xian.md)
* [4 event_base](5-libevent编程api.md)
   * [4.1 创建event_base](41_jian_li_mo_ren_de_event_base.md)
   * [4.2 检查event_base后端](42_jian_cha_event_base_hou_duan.md)
   * [4.3 释放event_base](43_shi_fang_event_base.md)
   * [4.4 event_base优先级](44_eventbase_you_xian_ji.md)
   * [4.5 event_base和fork](45_eventbase_he_fork.md)
* [5 事件循环event_loop](5_eventloop_shi_jian_xun_huan.md)
   * [5.1 运行循环](51_yun_xing_xun_huan.md)
   * [5.2 停止循环](52_ting_zhi_xun_huan.md)
   * [5.3 转储event_base的状态](53_zhuan_chu_event_base_de_zhuang_tai.md)
* [6 事件event](6_shi_jian.md)
   * [6.1 创建事件](61_chuang_jian_shi_jian.md)
   * [6.2 事件的未决和非未决](62_shi_jian_de_wei_jue_he_fei_wei_jue.md)
   * [6.3 事件的优先级](63_shi_jian_de_you_xian_ji.md)
   * [6.4 检查事件状态](64_jian_cha_shi_jian_zhuang_tai.md)
   * [6.5 一次触发事件](65_yi_ci_hong_fa_shi_jian.md)
   * [6.6 手动激活事件](66_shou_dong_ji_huo_shi_jian.md)
   * [6.7 事件状态之间的转换](67_shi_jian_zhuang_tai_zhi_jian_de_zhuan_huan.md)
* [7 数据缓冲Bufferevent](7_bufferevent.md)
   * [7.1 回调和水位](71_hui_diao_he_shui_wei.md)
   * [7.2 延迟回调](72_yan_chi_hui_diao.md)
   * [7.3 bufferevent 选项标志](73_bufferevent_xuan_xiang_biao_zhi.md)
   * [7.4 使用bufferevent](74_shi_yong_bufferevent.md)
   * [7.5 通用bufferevent操作](75_tong_yong_bufferevent_cao_zuo.md)
       * [7.5.1 释放bufferevent操作](751_shi_fang_bufferevent_cao_zuo.md)
       * [7.5.2 操作回调、水位和启用/禁用](752_cao_zuo_hui_diao_3001_shui_wei_he_qi_7528_jin_.md)
       * [7.5.3 操作bufferevent中的数据](753_cao_zuo_bufferevent_zhong_de_shu_ju.md)
       * [7.5.4 bufferevent的清空操作](755_buffereventde_qing_kong_cao_zuo.md)
* [8 数据封装evBuffer](8_evbuffer.md)
   * [8.1 创建和释放evbuffer](81_chuang_jian_he_shi_fang_evbuffer.md)
   * [8.2 evbuffer与线程安全](82_evbufferyu_xian_cheng_an_quan.md)
   * [8.3 检查evbuffer](83_jian_cha_evbuffer.md)
   * [8.4 向evbuffer添加数据](84_xiang_evbuffer_tian_jia_shu_ju.md)
   * [8.5 evbuffer数据移动](85_evbuffershu_ju_yi_dong.md)
   * [8.6 添加数据到evbuffer前](86_tian_jia_shu_ju_dao_evbuffer_qian.md)
* [8 链接监听器evconnlistener](8_lian_jie_jian_ting_qi_evconnlistener.md)
   * [8.1 创建和释放 evconnlistener](81_chuang_jianhe_shi_fang_evconnlistener.md)
   * [8.2 启用和禁用 evconnlistener](82_qi_yong_he_jin_yong_evconnlistener.md)
   * [8.3 调整 evconnlistener 的回调函数](83_diao_zheng_evconnlistener_de_hui_diao_han_shu.md)
   * [8.4 检测 evconnlistener](84_jian_ce_evconnlistener.md)
   * [8.5 侦测错误](85_zhen_ce_cuo_wu.md)
* [9 libevent常用设置](4-libevent简单服务器.md)
   * [9.1 日志消息回调设置](41_ri_zhi_xiao_xi_hui_diao_she_zhi.md)
   * [9.2 致命错误回调设置](42_zhi_ming_cuo_wu_hui_diao_she_zhi.md)
   * [9.3 内存管理回调设置](43_nei_cun_guan_li_hui_diao_she_zhi.md)
   * [9.4 锁和线程的设置](43_suo_he_xian_cheng_de_she_zhi.md)
   * [9.5 调试事件的使用](45_diao_shi_shi_jian_de_shi_yong.md)
* [10 基于libevent服务器](10_ji_yu_libevent_fu_wu_qi.md)
   * [10.1 Hello_World服务器(基于信号)](101_helloworld_fu_wu_qi.md)
   * [10.2 基于事件服务器](102_ji_yu_shi_jian_fu_wu_qi.md)
   * [10.3 回显服务器](102_hui_xian_fu_wu_qi.md)
   * [10.3 libevent实现http服务器](103_libeventshi_xian_http_fu_wu_qi.md)
   * 10.4 libevent实现TCP/IP服务器
   
   ---
### About the author

`name`：`Aceld(刘丹冰)`

`mail`:
[danbing.at@gmail.com](mailto:danbing.at@gmail.com)

`github`:
[https://github.com/aceld](https://github.com/aceld)

`原创技术文章作品`:
[https://www.yuque.com/aceld](https://www.yuque.com/aceld)

### 技术讨论社区
|  **WeChat**   | **WeChat Public Account**  | **QQ Group**  |
|  ----  | ----  | ----  |
| <img src="https://s1.ax1x.com/2020/07/07/UF6rNV.png" width = "150" height = "180" alt="" align=center />  | <img src="https://s1.ax1x.com/2020/07/07/UFyUdx.th.jpg" height = "150"  alt="" align=center /> | <img src="https://s1.ax1x.com/2020/07/07/UF6Y9S.th.png" width = "150" height = "150" alt="" align=center /> |
