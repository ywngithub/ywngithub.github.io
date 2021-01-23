---
title: MII接口
date: 2018-12-25 10:53:35
tags: 知识扩展
categories: 技术
---

一、简介
========
&emsp;&emsp;MII(Media Independent Interface),即媒体独立接口，它是IEEE-802.3规定的以太网行业标准，它包括一个数据接口和一个MAC和PHY之间的管理接口。“媒体独立”表明在不对MAC硬件重新设计或替换的情况下，任何类型的PHY设备都可以正常工作。MII数据接口的类型有很多，常用的有MII、RMII、SMII、SSMII、SSSMII、GMII、RGMII、SGMII,管理接口一般为MDIO。

二、MII管理接口(MDIO)
======================
&emsp;&emsp;MDIO是一种简单的双线串行接口，是一个PHY的管理接口，用来读/写PHY的寄存器，以控制PHY的行为或获取PHY的状态。有2根线，分别定义为：  
>MDIO_CLK：时钟线  
>MDIO_DATA：数据线  

三、MII接口
===========
&emsp;&emsp;MII接口一共有16根线，分别定义为：  
>TX_CLK：发送参考时钟，100Mbps速率下，时钟频率为25MHz，10Mbps速率下，时钟频率为2.5MHz;  
>TXD[3:0]：数据发送信号，共4根信号线;  
TX_ER：发送数据错误提示信号，同步于TX_CLK，高电平有效，表示TX_ER有效期内传输的数据无效。对于10Mbps速率下，TX_ER不起作用;  
>TX_EN：发送使能信号，只有在TX_EN有效期内传的数据才有效;  
> CRS：载波侦测信号，不需要同步于参考时钟，只要有数据传输，CRS就有效，另外，CRS只有PHY在半双工模式下有效;  
>COL：冲突检测信号，不需要同步于参考时钟，只有PHY在半双工模式下有效;

>RX_CLK：接收数据参考时钟，100Mbps速率下，时钟频率为25MHz，10Mbps速率下，时钟频率为2.5MHz;  
>RXD[3:0]：数据接收信号，共4根信号线;  
>RX_ER：接收数据错误提示信号，同步于RX_CLK，高电平有效，表示RX_ER有效期内传输的数据无效。对于10Mbps速率下，RX_ER不起作用;
>RX_DV(Reveive Data Valid)：接收数据有效信号，作用类型于发送通道的TX_EN。

四、RMII接口
===========
&emsp;&emsp;RMII(Reduced MII)，是MII的简化版，连线数量由MII的16根减少为8根。
>TXD[1:0]：数据发送信号线，数据位宽为2，是MII接口的一半;  
>RXD[1:0]：数据接收信号线，数据位宽为2，是MII接口的一半;  
>TX_EN(Transmit Enable)：数据发送使能信号，与MII接口中的该信号线功能一样;  
>RX_ER(Receive Error)：数据接收错误提示信号，与MII接口中的该信号线功能一样;  
>CLK_REF：是由外部时钟源提供的50MHz参考时钟，与MII接口不同，MII接口中的接收时钟和发送时钟是分开的，而且都是由PHY芯片提供给MAC芯片的。这里需要注意的是，由于数据接收时钟是由外部晶振提供而不是由载波信号提取的，所以在PHY层芯片内的数据接收部分需要设计一个FIFO，用来协调两个不同的时钟,在发送接收的数据时提供缓冲。PHY层芯片的发送部分则不需要FIFO，它直接将接收到的数据发送到MAC就可以了;  
>CRS_DV：此信号是由MII接口中的RX_DV和CRS两个信号合并而成。当介质不空闲时，CRS_DV和RE_CLK相异步的方式给出。当CRS比RX_DV早结束时(即载波消失而队列中还有数据要传输时)，就会出现CRS_DV在半位元组的边界以25MHz/2.5MHz的频率在0、1之间的来回切换。因此，MAC能够从 CRS_DV中精确的恢复出RX_DV和CRS。 


五、SMII接口
===========
&emsp;&emsp;SMII(Serial MII)，串行MII的意思，跟RMII相比，连线进一步减少到4根。   
>TXD：发送数据信号，位宽为1；   
>RXD：接收数据信号，位宽为1；   
>SYNC：收发数据同步信号，每10个时钟周期置1次高电平，指示同步;  
>CLK_REF：所有端口共用的一个参考时钟，频率为125MHz。  

六、SSMII接口
===========
&emsp;&emsp;SSMII即Serial Sync MII，叫串行同步接口，跟SMII接口很类似，只是收发使用独立的参考时钟和同步时钟，不再像SMII那样收发共用参考时钟和同步时钟，传输距离比SMII更远。

七、SSSMII接口
===========
&emsp;&emsp;SSSMII即Source Sync Serial MII，叫源同步串行MII接口，SSSMII与SSMII的区别在于参考时钟和同步时钟的方向，SSMII的TX/RX参考时钟和同步时钟都是由PHY芯片提供的，而SSSMII的TX参考时钟和同步时钟是由MAC芯片提供的，RX参考时钟和同步时钟是由PHY芯片提供的，所以顾名思义叫源同步串行。

八、GMII接口
===========
&emsp;&emsp;与MII接口相比，GMII的数据宽度由4位变为8位，GMII接口中的控制信号如TX_ER、TX_EN、RX_ER、RX_DV、CRS和COL的作用同MII接口中的一样，发送参考时钟GTX_CLK和接收参考时钟RX_CLK的频率均为125MHz(1000Mbps/8=125MHz)。  
　　在这里有一点需要特别说明下，那就是发送参考时钟GTX_CLK，它和MII接口中的TX_CLK是不同的，MII接口中的TX_CLK是由PHY芯片提供给MAC芯片的，而GMII接口中的GTX_CLK是由MAC芯片提供给PHY芯片的。两者方向不一样。   
　　在实际应用中，绝大多数GMII接口都是兼容MII接口的，所以，一般的GMII接口都有两个发送参考时钟：TX_CLK和GTX_CLK(两者的方向是不一样的，前面已经说过了)，在用作MII模式时，使用TX_CLK和8根数据线中的4根。

九、RGMII接口
============
&emsp;&emsp;RGMII即Reduced GMII，是RGMII的简化版本，将接口信号线数量从24根减少到14根(COL/CRS端口状态指示信号，这里没有画出)，时钟频率仍旧为125MHz，TX/RX数据宽度从8为变为4位，为了保持1000Mbps的传输速率不变，RGMII接口在时钟的上升沿和下降沿都采样数据。在参考时钟的上升沿发送GMII接口中的TXD[3:0]/RXD[3:0]，在参考时钟的下降沿发送GMII接口中的TXD[7:4]/RXD[7:4]。RGMI同时也兼容100Mbps和10Mbps两种速率，此时参考时钟速率分别为25MHz和2.5MHz。   
　　TX_EN信号线上传送TX_EN和TX_ER两种信息，在TX_CLK的上升沿发送TX_EN，下降沿发送TX_ER；同样的，RX_DV信号线上也传送RX_DV和RX_ER两种信息，在RX_CLK的上升沿发送RX_DV，下降沿发送RX_ER。










