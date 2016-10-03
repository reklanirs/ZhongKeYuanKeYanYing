协议定义

通信方式为socket.

裸的socket通信端口为4004

ssl加密的socket(如果实现的话)通信端口为4399 (from OpenSSL import SSL)



以下每个表格皆表示一个json

Key栏中表示方式为: 键名称; 键含义

Value栏中表示方式为: 值类型; 值限制



注意

以下所有功能为所有可实现功能的并, 除明确标明外, 并非必须实现功能/实现全部特性. 所有标准皆在制定中, 将会在10.3日持续更新.

整个名称都被括号括起来的为明确并非必须实现的部分.

Command编号(每个表格第二行的Value)后序可能有改动.

现协议为β版,仅供参考,欢迎提交改进.



登陆

概述

服务器基本功能.

游客模式登陆时, 客户端发送客户设定的用户名给服务器(表1), 登陆成功情况下, 服务器返回给登录用户登录成功标志及所有在线用户列表(表2), 服务器广播给其它所有在线用户新登陆的用户用户名(及其是否为游客);   登陆失败情况下, 服务器返回给登录用户登陆失败标识(在线用户列表为空列表). 

限制

服务器必须实现无注册无密码登陆, 即以游客身份登陆. 用户名不可重复.

流程

socket建立连接后的信息依次发送顺序为:

1. Client —> Server
     Key                 	Value                                   
     Command; 表示该信息实际表达含义	string; 定值01                            
     Name; 用户名           	string; 仅包含所有大小写字母和数字, 长度须>3, 不包括保留字(见附录1)
     (Password; 密码)      	string; 仅包含大小写字母和数字, 密码为空则为游客登录         
   
2. Server —> Client
     Key         	Value                                   
     Command     	11                                      
     Flag, 登陆是否成功	0/1 int; 失败/成功, 该标志位为0时, 以下全部键值对信息无效    
     Friends     	list; 当前所有在线用户列表, list中项为元组: (在线用户用户名,该在线用户是否为游客)



当2中Flag值为1时

1. Server —> All other Clients
     Key                    	Value            
     Command                	string; 定值30     
     LogOnUserName; 上线用户名   	string; 用户名类型    
     (IsUserGuest; 该用户是否为游客)	int; 0/1  用户是否为游客
   



(注册)

概述

由客户端发送注册信息(表1), 服务器内部判定注册是否合法并返回注册是否成功(表2) 

该功能为非必须功能

流程

1. Client —> Server
     Key                 	Value                                   
     Command, 辨别该信息含义的标志位	string; 定值02                            
     Name,用户名            	string; 仅包含所有大小写字母和数字, 长度须>3, 不包括保留字(见附录1)
     Password, 密码        	string; 仅包含大小写字母和数字                     
2. Server —> Client
     Key          	Value               
     Command      	string; 定值12        
     Flag, 注册是否成功 	int;  0/1,表示 失败/成功  
     Message, 错误信息	string; Flag为0时才有该信息
   

用户正常登出

概述

用户登出分为两种, 正常登出与程序崩溃. 正常登出过程为客户端向服务器发送登出命令(表1), 客户端接受后更新在线用户列表并向其它全部在线用户广播下线用户信息(表2)



流程

1. Client —> Server
     Key    	Value
     Command	03   
   
   用户非正常登出时从此开始
2. Server —> All online Clients
     Key                  	Value        
     Command              	31           
     LogOutUserName; 下线用户名	string, 用户名类型
   

聊天[通过服务器单对单]

概述

用户之间的聊天是通过服务器进行的.首先由用户A将所要发送信息及对方用户名发送给服务器(表1), 服务器转发给用户B(表2). [为防止网络问题导致发送失败, 可以给用户A发送回执(表3). 发送成功的判断依据为表2操作socket没有报异常]

该功能为必须功能.



流程

假设用户A要向用户B发送信息:

1. Client A—>Server
     Key              	Value              
     Command          	04                 
     FriendName; 对方用户名	string, 用户名类型      
     Message; A发送给B的信息	string; 字符串格式为UTF-8
   
2. Server —> Client B
     Key                	Value              
     Command            	13                 
     FriendName; 对方用户名  	string, 用户名类型      
     Message; B得到的来自A的信息	string; 字符串格式为UTF-8

1. (Server —> Client A)
     Key        	Value      
     Comment, 回执	14         
     Flag       	0,1; 发送是否成功
   

(群聊/聊天室)

概述

发送信息可以让所有在线用户看到的聊天室. 首先由用户发送给服务器信息(表1), 服务器广播给所有在线用户发言用户名及信息(表2)

该功能为非必须功能

流程

1. Client —> Server
     Key        	Value 
     Command    	05    
     Message; 信息	string
   

1. Server —> All Online Clients
     Key         	Value              
     Command     	32                 
     Speaker; 说话人	string; 用户名类型      
     Message; 信息 	string; 字符串格式为UTF-8
   



(Server—>Client的特殊消息)

心跳探测

概述

服务器通过发送给客户端探测信息, 根据发送是否成功判断用户是否已非正常下线. 

  Key    	Value
  Command	14   









附录1

不能使用的用户名保留字

admin, administrator, guest, user, table, 



附录2

特殊符号解析表(该表非json)


