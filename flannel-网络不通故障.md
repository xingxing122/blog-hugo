









Master 节点与pod ip  ping 不通了

三台机器 

172.16.70.12 

172.16.70.16 

172.16.70.58 



![image-20210201170339555](https://xing-blog.oss-cn-beijing.aliyuncs.com/2021-02-01-090340.png)



在172.16.70.58 nginx-php 192.168.14.5 

![image-20210201170440112](https://xing-blog.oss-cn-beijing.aliyuncs.com/2021-02-01-090440.png)



在172.16.70.16上ping 192.168.14.5 

![image-20210201170530892](https://xing-blog.oss-cn-beijing.aliyuncs.com/2021-02-01-090531.png)

在172.16.70.12 ping 192.168.14.5 

​                        ![image-20210201170715858](https://xing-blog.oss-cn-beijing.aliyuncs.com/2021-02-01-090716.png)



解决

在172.16.70.12 上查看网络

 ![image-20210201170852295](https://xing-blog.oss-cn-beijing.aliyuncs.com/2021-02-01-090852.png)



在172.16.70.12抓包看一下是否从flannel 包出去  

tcpdump   -i   flannel.1   -nn host  192.168.72.0  

![image-20210201171420959](https://xing-blog.oss-cn-beijing.aliyuncs.com/2021-02-01-091421.png)

![image-20210201181813570](https://xing-blog.oss-cn-beijing.aliyuncs.com/2021-02-01-101813.png)



在172.16.70.12抓一下eth0 的网卡包目标ip ping 的包

![image-20210201180513907](https://xing-blog.oss-cn-beijing.aliyuncs.com/2021-02-01-100514.png)

​       IP 192.168.72.0 > 192.168.14.5: ICMP echo request, id 47084, seq 948, length 64  

![image-20210201181844265](https://xing-blog.oss-cn-beijing.aliyuncs.com/2021-02-01-101844.png)



