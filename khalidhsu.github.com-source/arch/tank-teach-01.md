Title: 一个Pygame版坦克大战的诞生(01)  
Date: 2013-05-14 07:43  
Tags: python,pygame  
Category: 2013-05  
Slug:  tank-teach-01   
Author: Khalid Hsu  
  
  
  
  
这个学期软件工程课上要求制作一个游戏，一个经典得不能再经典的游戏--坦克大战。然后还要求写N篇实验报告。  
说实话，花那么多时间写实验报告，到最后留它在某个角落里发霉，还不如写成教程发布到网上。要是有人能够从中得到一点点帮助，那就是一件有意义的事情。所以就有了这个教程。    
这是一个入门教程，希望能给刚学python又想写游戏的人提供一点帮助。大神请直接无视此教程。    
关于什么是pygame，在此不再赘述。废话少说，开门见山。    
  
先来个简单的，建立一个窗口：    
  
```python  
#!/usr/bin/env python    
#-*-coding:utf-8-*-  
import pygame  
pygame.init()    #pygame初始化  
size=[700,500]    #指定窗口尺寸  
screen=pygame.display.set_mode(size)    #新建一个窗口  
pygame.display.set_caption("坦克大战")    #设置窗口标题  
done=False  
while done==False:  
    for event in pygame.event.get():   
        if event.type == pygame.QUIT:   
            done=True   
    pygame.display.flip()#刷新屏幕  
pygame.quit ()  
```  
前面几句是常规初始化用的。而对于后面的循环，则是从pygame的事件列表(event queue)中取出事件，判断事件类型并做相应操作。  
  
什么是事件？例如你移动了鼠标，按下了键盘按键，甚至松开了键盘按键等，这些都是事件。  
而事件则有其相应的类型，例如按下按键，则是pygame.KEYDOWN，释放按键，则是pygame.KEYUP。  
pygame将一大堆发生了的事件储存起来，使用pygame.event.get()方法能获取到这些事件，然后你就可以做相应的处理。  
所以以上这个循环的作用是判断获取的事件是否为pygame.QUIT，是则设置done为True，结束循环。  

你可以试试把done=True这句改成pass（即什么也不干），当你再点击窗口的关闭按钮时候，你会发现窗口无法关闭了~~哈哈  
知道怎么关闭不？windows用户请按Ctrl-Alt-Delete呼出任务管理器，找到那个python的进程并干掉，linux用户也可以用System Monitor之类的软件来搞定，不过我更喜欢打命令:  
  
```bash  
ps -ef|grep python   #找到进程的PID  
kill -s 9 PID     #干掉它,PID换成上面得到的数字号  
```  


再干点别的吧，画个红色的圆。  
红色怎样表示：  
red =   [255,  0,  0]  
这是RGB表示法嘛，R为255，G为0，B也为0。（注意数字的范围是0~255）  
色光的三原色是RGB，如今G和B都为0，只有R，还是所以就是红色咯~~还是正红呢！  


那怎么画圆：  
pygame.draw.circle(screen,red,[200,200],50)  
pygame.draw.circle是画圆的函数，你可以到 [这里](http://www.pygame.org/docs/ref/draw.html) 看到pygame.draw支持画的各种图形和相应的参数 。  
这个screen就是刚刚新建的窗口，也就是说，在这个窗口中画。当然你也可以画在别的看不见的地方，例如内存中“画布”，这个后面再说吧。  
red就是上面定义的红色  
[200,200]是圆的坐标，50则是半径  
  
所以增加两行代码，长这样了：  
  
```python  
#!/usr/bin/env python    
#-*-coding:utf-8-*-  
import pygame  
pygame.init()    #pygame初始化  
red =   [255,  0,  0]  
size=[700,500]    #指定窗口尺寸  
screen=pygame.display.set_mode(size)    #新建一个窗口  
pygame.display.set_caption("坦克大战")    #设置窗口标题  
done=False  
while done==False:  
    for event in pygame.event.get():   
        if event.type == pygame.QUIT:   
            done=True  
    pygame.draw.circle(screen,red,[200,200],50)  
    pygame.display.flip()#刷新屏幕  
pygame.quit ()  
```  
运行看看你的红色的圆出来没？    


(本文涉及的代码已经上传到[Github中](https://github.com/khalidhsu/TankWar))   


  
  
