---
title: "OpenVPN分析"
date: "2011-04-26"
categories: 
  - "技"
tags: 
  - "openvpn"
  - "vpn"
  - "translate"
---

OpenVPN 从架构上来看，OpenVPN在某种程度上和 `tinc` 或者和`VTun` 比较相近，它是一个基于用户模式（user-mode）的程序，通过 `TUN/TAP` 接口与 `TCP/IP` 栈进行通信。

作为用户程序运行的 `OpenVPN`，带来了移动性和易维护性的优点，正如我们在 `VTun` 和 `tinc` 中看到的那样。和 `tinc` 一样，OpenVPN 在VPN服务中使用两种通道：一个携带用户的IP数据报文的数据通道，一个处理“密钥交互和配置(key negotiation and configuration)这种协议事务的控制通道。 

OpenVPN 把两个通道都封装在UDP数据包中。两个通道使用相同的端口，所以一个给定的数据报既可以包含数据通道数据也可以包含控制通道数据。因为OpenVPN使用TLS协议进行认证和密钥交换，而TLS需要一个可靠的传输层，所以OpenVPN在控制通道中添加了一个可靠的层。这样保证了TLS所需要的可靠性，但是在数据通道中没有高可靠性的层（ but that there will not be competing reliability layers on the data channel），所以我们在SSL和SSH VPN 中看到的干扰现象不会发生。

正如我们在本章中其他VPN中看到的，OpenVPN可以携带IP数据报或者物理数据帧，它可以在IP层或者数据链路层进行操作。我们将把注意力集中在它在IP数据中的应用，但是其他模式是相近的。 OpenVPN 安全模式 OpenVPN 可以通过两种安全模式运行。两种模式各有利弊，但是正如我们将要看到的，只有一种使用与需要高于常规安全(more than casual security)需求的情形。 
我们在本节简明地介绍一下两种模式，但是把详细分析留给相对更安全的那一个。 

我们把第一种模式叫做“静态密钥方法”(static key method)，在这种模式下，两个节点利用预先分享的密钥进行加密和认证。这些密钥在节点使用VPN之前协商配置。当然，这就意味着，必须使用其他的安全措施来在节点之间通知(inform)这些密钥。 

默认情况下，有两个密钥：一个用来加解密，一个用来进行HMAC认证。在这种情况下，两个节点使用相同的密钥。配置OpenVPN使用四个密钥—可以这么做，而且这样更安全——每一个节点都有一个HMAC发送密钥和一个接收密钥，一个加密密钥和一个解密密钥。就是说，每一个节点都有一组发送密钥和一组接收密钥。 
这样做的好处是，增加了密钥猜解的难度，并且降低了单个密钥被猜解之后的危害。 

抛开使用四个还是两个密钥的问题，我们都应该警惕这种方式下，重复使用事先协商的密钥带来的弊端。同样的密钥将会用在VPN的整个生命周期，并且用在VPN的每一个请求，知道人为的改变或者VPN重启。尽管在不同的传输方向上使用不同的密钥可以减缓”使用相同密钥加密的数据量的增长速度“，但它能做的也就是减缓而已。最终，加密数据积累，事先商定的密钥变得越来越脆弱和敏感。一旦密钥被猜解，使用这个密钥的所有请求中的所有被传输的数据都变得可读。 

静态密钥方法的优势在于配置方便。VPN的管理员不许要被证书或者证书认证困扰，这正是第二种方法所需要的。（……） 第二种方法，被称作TLS方法，使用SSL协议来使每一个VPN节点和对应节点进行认证和交换密钥和其他控制信息。在这中方法中，OpenVPN为控制通道在它的节点之间建立一个SSL/TLS对话。在认证状态下，节点之间交换被信任的第三方CA签名的证书。这样保证了双方都确定都是在和他们想要的终端进行通信，同时也阻止了“中间人攻击”。 

一旦认证完成，而且SSL对话已经在节点之间建立，OpenVPN就使用这个连接去协商数据通道的对话密钥。管理员可以配置OpenVPN基于任意传输的字节，报文或者时间去重新协商这些密钥。这样可以避免像头密钥加密的数据积累，而且提供了一个近乎完美的保密策略：猜解一个给定的密钥，不会影响到使用这个密钥之前和之后的数据。 TLS方法提供了一个十分健壮的认证和密钥交换机制。除了我们上面提到的短期的VPN服务，我们总是应该选择这种方法。下面的讨论都限制在这种方法上。 数据通道 正如我们前面提到过的，OpenVPN使用UDP作为他的传输协议，提供了一个良好的数据通道。 OpenVPN 可以选择TCP连接来代替UDP。尽管这在特定的环境下十分方便，但它重新引入了竞争性可靠层的问题，所以应该尽量避免。 通过UDP，我们得到一个类似的封装，如下图8.25：

![](https://raw.githubusercontent.com/catbaron0/pic/main/images/2023219151602.png)

OpenVPn 把负载头部分为两个部分：报文头部(packet header)，标识了报文的类型和密钥数据；和数据负载头部(data payload header)，包含了认证，IV,和数据报的序列号。 图8.26：

![](https://raw.githubusercontent.com/catbaron0/pic/main/images/2023219151713.png)

图8.26显示了TCP和UDP的报文头部结构。TCP版本中的packet length域显示了接下来数据报的长度。在UDP中不需要这此域，因为UDP十一个基于报文(packet-oriented)的协议，而TCP是基于流(stream-oriented)的协议。Op code域标识了数据报的类型。图8.27显示了所有可能的值。 图8.27

![](https://raw.githubusercontent.com/catbaron0/pic/main/images/2023219151828.png)

key Id域表示了数据报使用的密钥类型。我们很快会测试这个特性。 从8.25中可以看到，加密和认证的数据有一部分扩展到了数数据负载头部中。准确的机构在图8.28中有清晰的描述，结合了数据负载头部和数据负载。 图8.28 

![](https://raw.githubusercontent.com/catbaron0/pic/main/images/2023219151911.png)

HMAC域基于标准的SHA-1或者MD5HMAC。

正如图示，这个数据域对负载，数据报ID和IV进行了认证。IV是为CBC-mode加密生成的一个随机初始向量。 除了CBC加密模式，OpenVPN还支持CFG和OFB加密模式，都需要IV。 IV通常是64位或者128位。 packet ID 域作为序列号来防止重放攻击。

OpenVPN为这些序列号保留了滑动窗口机制，如果接收的报文在窗口的左边，或者在窗口中但是已经接收过像头序列的报文，这个报文将被丢弃。如果接收的报文在窗口的右边，窗口将向右滑动，使得右边界位于新接收的报文序列处。这个滑动窗口机制和我们在tinc中看到的很类似。 除了滑动窗口，OpenVPn 还提供了时间测试，丢弃那些在接收到更高序列号的报文之后t时间的乱序报文( rejects out-of-order packets that are received more than t seconds after any packet containing a higher sequence number)。

这个可配置的t参数默认为15s。窗口的宽度也是可以配置的，默认为64个序列长度。 图8.28显示了packet Id为32位，是它使用CBC加密模式时的长度。使用CFB，OFB，胡子哦和静态密钥模式时，他的长度都是64位。在CFB/OFB加密模式中，这个包含了时间戳和增量计数的64位数据包含在IV中，作为一种空间节省策略。 如我们所见，OpenVPN 的数据通道完美地满足了我们对VPN的需求：一个存在域主机或者网络之间的加密和认证通道。

我们讨论安全性时将会看到，数据通道的设计十分优秀，不存在任何明显的安全缺陷。事实上，它精确模仿了IPsec ESP协议，而且在此协议在发展和部署中学到很多优点。 Ping和OCC协议 除了传输用户数据，数据通道还携带限定数量的控制信息。OpenVPN可以设置为使节点发送“维持(keep-alive)”消息，如果他们没有在规定时间内接收到这个消息，则可以关闭或者放重启VPN连接。尽管OpenVPN把维持消息看作ping包，但是他们并不是ICMP下的ping包。接收到这个ping，节点会重置接收时间并丢弃ping包。这个ping包作为一个携带一个唯一序列号的普通数据通道数据被传输。

应该注意到，这个协议只是要求每一个节点不要太久不在数据通道中发送数据。如果一个节点在超时之前没有接收到数据，它就可一根据配置来重启或者关掉VPN。一个节点并不期望得到ping包的回应，甚至不知到对方是否收到了自己的ping包。 在通道初始化的时候，节点使用OCC加密模式相互加换配置信息。和ping包一样，这些信息作为普通数据通过数据通道传输，并且通过一个唯一的序列号来和其他数据包区分。通常，这些信息的格式如图8.29所示。 图8.29 

![](https://raw.githubusercontent.com/catbaron0/pic/main/images/2023219152123.png)

OCC magic域是一个16位的唯一序列号，用来标识OCC数据包。这是必需的，因为数据通道的数据报没有message type 域。OCC op code域标识了OCC信息的类型。最新定义的op code值如图8.30所示。 图8.30 
 
![](https://raw.githubusercontent.com/catbaron0/pic/main/images/2023219152249.png) 
 
可选的OCC data域包含了任何OCC 信息所需要的数据。比如，在OCC\_REPLY消息中，OCC data域包含了节点的option string。 控制通道 正如我们在SSL，SSH，和其他轻量级的VPN一样，提供一个安全的数据通道的两大难点是，密钥管理和节点认证。这两个方面任意一个出现错误都可能到值数据通道的不安全，不论数据通道本身的设计如何优秀。在这一节，我们会看到OpenVPN如何处理这两大方面。 OpenVPN的一个优势在于，它使用SSL/TLS协议为节点提供密钥管理和认证。因为SSL协议在多年的深入发展和研究中，其安全性已经被专家们普遍认可，OpenVPN可以利用它的安全性保证VPN的安全。 图8.31显示了控制通道数据的格式。 
 
![](https://raw.githubusercontent.com/catbaron0/pic/main/images/2023219152337.png)
 
session ID 域是一个64位的随机数，用来标识VPN会话。如同我们在其他协议中所见，VPN通信的每一端都有自己的session ID，在此域中出现的是发送放的session ID。 可选的HMAC域用来帮助防止DOS攻击。当使用它时，它对整个数据报进行认证，并且允许节点不需花费任何执行资源就可以丢弃一个伪造的数据包。它还可一防止一个伪造的数据包到达SSL层以探测安全缺陷，或者造成潜在的信息泄漏。此域只有在—tls--auth选项开启的时候才会出现。
  
packet Id域用来防止重放攻击。它和在数据通道中扮演的角色相同。当使用TLS方法时，长度位32位，否则就是64位。 可靠层使用ACK buffer 来获知一个节点的数据包。ACK buffer length域是单字节，用来标识后面信息的长度。如果是0，则ACK buffer不存在。如果大于0，则包含了ACK buffer中32位信息序列的个数。
  
在ACK buffer 的最后是节点的session ID。节点通过这个session ID 把ACK buffer的信息和一个特定的VPN会话绑定。此session ID 也只在ACK buffer length不为0时出现。 如果这个数据包的op code是P\_ACK\_V1，那么ACK buffer是数据包中最后一个域。如果是P\_CONTROL\_V1,message sequence number域，也就是可靠层使用的序列号，包含了常规SSL数据记录的还有TLS payload 域，都会出现。 
  
当VPN启动时，节点执行标准的SSL客户端认证握手行为。在这个行为中，通信双方都会位对方提供自己的证书。经过认证，双方都确认他们在和预订的节点对话，而且拥有了一个安全的SSL通道来位数据通道提供密钥信息的交换。 有两种密钥交换的信息。当节点根据opcode V1设定使用方法1（图8.27），它们使用的消息格式如图8.32所示。 图8.32： 

![](https://raw.githubusercontent.com/catbaron0/pic/main/images/2023219152551.png)

cipher key length 域包含了cipher key 域的长度，cipher key中包含了一个随机生成的密钥供接收方解密使用。类似的，HMAC key length域包含了HMAC key域的长度。HMAC key包含了一个随机生成的密钥，供接收方对接收的数据包进行认证。应该注意的是，这里将会有四个密钥，双方各有两个。再次强调，这种方法并不完善，因为所有的密钥都由一个单独的节点完全决定。 最后的域十一个不定长的option string。option string必须和本地的option string匹配，保证两个节点的配置一致。 方法2必方法1要更加完善。每一个节点都提供了双方生成密钥的密钥数据。当方法2被使用的时候，如8.27的V2设定，节点间的交换数据，使用如下的消息。 图8.33： 
   
   ![](https://raw.githubusercontent.com/catbaron0/pic/main/images/2023219152701.png)
   
如图所示，这个消息以4个字节的0开始，跟着是一个字节的key-method域。现在，这个域通常是2，标识这密钥交换的方法2，但是这个域允许以后添加更多的方法。 

premaster secret域是由客户端生成的48位随机数据。此域提供的服务和SSL中的同名域相同：提供一个可以生成master secret的生成数据。premaster srcret域在服务器间的密钥交换中不被使用。 

接下来，random1和random2两个域也在密钥生成的操作中使用。他们是32位的随机数据。因为通信双方都提供两个随机域，所以两边都不能当方面决定密钥。 

optionstring length 域包含了不定长数据option string的长度。如同方法1描述的那样，节点使用option string来判别双方的配置是否一致。 

user name length域包含了不定长的user name域的长度。相似的，password length域包含了不定长的password域的长度。user name和password 用在OpenVPN 运行在HTTP代理，而代理需要认证的情况。

这些域可选，并且只在我们使用HTTP代理的时候使用。 密钥生成阶段和TLS使用的方法十分接近。

首先，两边都通过OpenVpn master secret,客户端的random1,服务器端的random1来生成maste rsecret，使用结果和premaster secret作为输入来计算HMAC。 

PRF(premaster secret, OpenVPN master secret client random 1 server random 1) PRF函数对参数进行MD5和SHA-1 HMACs运算，并对结果进行或运算。 实际的PRF的细节比这里介绍的复杂一点。PRF函数首先把secret分成两部分，一半用来生成MD5的HMAC，一半用来生成SHA-1的HMAC。所需的输出长度是通过重复把种子数据反馈给HMACs来的到的。 一旦节点拥有了master secret，他们将利用OpenVPN key expansion(客户端random2，服务端random2，客户端session Id和服务端session ID)生成四个输入密钥（加密，解密，输入认证，输出认证）。结果和master secret用来作为PRF的输入得到密钥。

PRF(master secret, "OpenVPN key expansion" client random 2 server random 2 client SID server SID) 在密钥交换之后，数据可以在数据通道中传输了。数据通道中的数据包都用刚才生成的密钥进行了认证和加密。当需要更换密钥的时候，就重新进行密钥交换。如果不关心是方法1还是方法2，它看起来就像是最初的交换一样。为了帮助新密钥的传输，OpenVPN提供了下面三组密钥： 1.active keys 2.lame-duck keys，收回的密钥 3.另一个lame-duck keys，当密钥协商失败的时候使用。 这三组密钥解释了数据报头部的key ID 域。key Id标识了使用三组中的那一组密钥。 OpenVPN 安全性分析 不写了睡觉去。总之安全性很好。有兴趣的看原文去。
