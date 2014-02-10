Title:   
Date: 2013-08-14 19:30  
Tags: wd,hardisk  
Category: 2013-08  
Slug:  wdidle3   
Author: Khalid Hsu  
  
新买的WD蓝盘，用了半年左右，Load_Cycle_Count居然达到了8W多。  
之前希捷的硬盘，设置了APM参数就可以搞定这个问题，不过同样的方法对这WD无效。  
后来发现，WD官方提供WDIDLE3（纯DOS软件），可以修改硬盘的计时器。  
将计时器修改为240发现情况好转。  
不过貌似黑盘和蓝盘的是要区别对待的，[这里是别人的测试](http://dell.benyouhui.it168.com/thread-1238605-1-1.html)  
  
