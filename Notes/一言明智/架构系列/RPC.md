
RPC和Socket的区别
http://blog.163.com/fanning_7213/blog/static/249650520113124540501/

RPC(Remote Procedure Call，远程过程调用)是建立在Socket之上的,出于一种类比的愿望,在一台机器上运行的主程序,可以调用另一台机器上准备好的子程序,就像LPC(本地过程调用).

    越底层，代码越复杂、灵活性越高、效率越高；越上层，抽象封装的越好、代码越简单、效率越差。Socket和RPC的区别再次说明了这点。

不论是程序员在编写基于C/S(客户端服务器)的程序时,还是网络工程师在处理RPC问题时,他们问的最多的就是RPC和Socket有什么区别和联系? 
　 　RPC(Remote Procedure Call，远程过程调用)是建立在Socket之上的,出于一种类比的愿望,在一台机器上运行的主程序,可以调用另一台机器上准备好的子程序,就像 LPC(本地过程调用).RPC带来了开发C/S程序的简单可靠的手段,它通过一种叫XDR的数据表达方法描述数据,程序员书写伪代码,然后由 rpcgen程序翻译为真正的可编译的C语言源代码,再编译成真正的Client端和Server端程序。 
　　RPC作为普遍的C/S开发方 法,开发效率高效,可靠.但RPC方法的基本原则是－－以模块调用的简单性忽略通讯的具体细节,以便程序员不用关心C/S之间的通讯协议,集中精力对付实 现过程.这就决定了 RPC生成的通讯包不可能对每种应用都有最恰当的处理办法,与Socket方法相比,传输相同的有效数据,RPC占用更多的网络带宽. 
　　RPC是在Socket的基础上实现的,它比socket需要更多的网络和系统资源.另外,在对程序优化时,程序员虽然可以直接修改由rpcgen产生的令人费解的源程序,但对于追求程序设计高效率的RPC而言,获得的简单性则被大大削弱. 
RPC与是Socket的类比
[HTTP和RPC的优缺点](https://www.jianshu.com/p/b61695e6b473)
[RPC服务和HTTP服务对比](https://blog.csdn.net/wangyunpeng0319/article/details/78651998)
[为什么需要RPC，而不是简单的HTTP接口](https://www.cnblogs.com/winner-0715/p/5847638.html   )