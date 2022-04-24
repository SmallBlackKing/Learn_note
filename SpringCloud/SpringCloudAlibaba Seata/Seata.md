





![image-20220423165436034](../SpringCloud Alibaba Sentinel/image/image-20220423165436034.png)



![image-20220423165412501](../SpringCloud Alibaba Sentinel/image/image-20220423165412501.png)





![image-20220423173521928](../SpringCloud Alibaba Sentinel/image/image-20220423173521928.png)









# 问题

解决seata不能使用mysql8版本的问题

可能原因：[seata](https://so.csdn.net/so/search?q=seata&spm=1001.2101.3001.7020)不支持mysql8最主要的原因就是连接驱动是5版本的，所以将mysql8版本的连接驱动替换掉或者两个都保留也可以。

![image-20220423172600913](../SpringCloud Alibaba Sentinel/image/image-20220423172600913.png)



![image-20220423172624167](../SpringCloud Alibaba Sentinel/image/image-20220423172624167.png)