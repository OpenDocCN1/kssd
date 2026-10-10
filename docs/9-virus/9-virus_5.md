

# Billy Belceb 病毒编写教程---DOS 篇

> 原文：[`www.pediy.com/kssd/pediy06/pediy64061.htm`](https://www.pediy.com/kssd/pediy06/pediy64061.htm)

 Billy Belceb 病毒编写教程---DOS 篇(基础理论)    
                                  翻译：onlyu\[FCG\]\[DFCG\]                
  
【译者声明】   
       这是一篇关于病毒基础知识的教程，作者 Billy Belceb，西班牙人，在 16 岁时写的这篇教程，曾创建了病毒组织 DDT，当时为著名病毒组织 iKX(International Knowledge eXchange)的成员。大家都知道，DOS 病毒早已成为过去，但这篇教程介绍的关于病毒的基础知识、思想很重要，是学习 Win32 病毒和网络病毒的基础。翻译这篇教程的目的是想揭开病毒的神秘面纱，从编写病毒的角度来学习病毒，希望对大家有用。由于原文为西班牙人写的英文，译者也是第一次尝试翻译英文教程，有些术语翻译得不好，错误之处还请大家原谅，大家也可对照原文学习。(原文在 29A#3 中)。下面为原文译文，祝你好运！   
  
【声明】   
 ~~~~~~   
    作者对因对此文档使用不当而造成的任何损失概不负责。这篇教程的目的是教会人们编写病毒和防护一些破坏力大的病毒的破坏。这篇教程仅作为教学目的。所以，如果有人利用这篇文章编写了破坏力很大的病毒，我可不负责任。如果通过这篇文章你看到我鼓励人们破坏数据的字眼，先去买副眼镜再说。   
  
【介绍】   
\~~~~~~   
    欢迎阅读 Billy Belceb 的病毒编写教程，来到病毒的世界。这篇文章献给我的导师 zAxOn,他真是我的良师啊，是他不厌其烦的解释当我请教他什么是 ARJ，借给我 TP5，把他的全部所知教给了我，直到迈出了汇编的第一步。这篇文章还献给那些想摆脱蹩脚病毒编写者称号，加入“出色者”行列的人。我还要提写了很多经典文章的 Dark Angel(Phalcon/Skism 组织的成员)，是他的许多教程带我入了门。当然了，还要献给 Offspring,Marilyn Manson,Blind Guardian,Stratovarius 和 Metallica(我听说过另外一些组织，但是上面这些是最出色的)，还有他们的作品。希望你们喜欢这篇为初学者准备的教程，可能这也是我的最后一篇 DOS 病毒教程了。   
  
说明：英语不是我的母语(西班牙语才是)\[译者注：所以这篇西班牙式英语的病毒教程很难翻译，不当之处还请原谅\]，所以原谅我的许多拼写错误，请告知我，我会修正的。   
  
\----------跟我联系   
 ?E-mail        billy\_belcebu@hotmail.com                billy\_belcebu@cryogen.com   
 ?ICQ #           22290500   
个人主页                  http://members.xoom.com/billy\_bel            http://www.cryogen.com/billy\_belcebu   
组织主页                  http://sourceofkaos.com/homes/ddt   
IRC \[Billy\_Bel\]            Undernet #vir, Irc-Hispano #virus   
祝玩得快乐！   
Billy Belceb  4dd466d0e1e0cc123afbe504da2462ca