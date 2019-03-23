Pingmesh：用于数据中心网络延迟测量和分析的大规模系统
====

概论
-
&emsp;&emsp;我们是否可以在大型数据中心网络中随时获得任意两台服务器之间的网络延迟？收集的延迟数据可用于解决一系列挑战：告知应用程序感知延迟问题是否由网络引起，定义和跟踪网络服务水平协议（SLA）以及自动网络故障排除。
   我们开发了用于大规模数据中心网络延迟测量和分析的Pingmesh系统，以肯定地回答上述问题。 Pingmesh已在Microsoft数据中心运行了四年多，每天收集数十TB的延迟数据。 Pingmesh不仅被网络软件开发人员和工程师广泛使用，还被应用程序和服务开发人员和运营商广泛使用。
    
CCS概念

网络->网络测量、云计算、网络监控、计算机系统架构->云计算

关键字

数据中心网络 网络故障排除 无声数据包丢失


1.介绍
-
&emsp;&emsp;在今天的数据中心，有数十万台服务器。这些服务器通过网络接口卡（NIC），交换机和路由器，电缆和光纤连接，形成大规模的内部和内部数据中心网络。 由于云计算的快速发展，数据中心网络（DCN）的规模正在扩大。 在物理数据中心基础设施之上，构建了各种大规模的分布式服务，例如Search [5]，分布式文件系统[17]和存储[7]，MapReduce [11]。<br>
&emsp;&emsp;这些分布式服务是庞大且不断发展的软件系统，具有许多组件并具有复杂的依赖性。 所有这些服务都是分布式的，许多组件需要通过网络在数据中心内或跨不同的数据中心进行交互。 在这样的大型系统中，软件和硬件故障是常态而非例外。因此，网络团队面临着一些挑战。<br>
&emsp;&emsp;第一个挑战是确定问题是否是网络问题?由于分布式系统的性质，许多故障显示为“网络”问题，例如，某些组件只能间歇性地到达，或者端到端延迟显示在第99百分位处突然增加，每台服务器的网络速度吞吐量从20MB/s降低到5MB/s。我们的经验表明，这些“网络”问题中约有50％不是由网络引起的。 然而，要判断“网络”问题是否确实是由网络故障引起的，这并不容易。<br>
&emsp;&emsp;第二个挑战是定义和跟踪网络服务水平协议（SLA）。许多服务需要网络提供某些性能保证。 例如，搜索查询可能会触及数千台服务器，搜索查询的性能由最慢服务器的最后一次响应确定。这些服务对网络延迟和数据包丢失敏感，他们关心网络SLA。需要针对不同的服务单独测量和跟踪网络SLA，因为它们可能使用不同的服务器集和网络的不同部分。 由于网络中的大量服务和客户，这成为一项具有挑战性的任务。<br>
&emsp;&emsp;第三个挑战是网络故障排除。当网络SLA因各种网络故障而中断时，会发生“实时站点”事件。 实时站点事件是指对客户，合作伙伴或收入产生影响的任何事件。 现场事件需要尽快被检测，缓解和解决。但数据中心网络拥有数十万到数百万台服务器，数十万台交换机以及数百万条电缆和光纤。因此，检测问题所在的位置是一个难题。<br>
&emsp;&emsp;为了应对上述挑战，我们设计并实施了Pingmesh，这是一个用于数据中心网络延迟测量和分析的大型系统。 Pingmesh利用所有服务器启动TCP或HTTP pings,以提供最大延迟测量覆盖率。Pingmesh形成多层次的完整图表。在数据中心内，Pingmesh允许机架内的服务器形成完整的图形，并使用架顶式（ToR）交换机作为虚拟节点，并让它们形成第二个完整的图形。在数据中心之间，Pingmesh通过将每个数据中心视为虚拟节点来形成第三个完整的图形。完整图形和相关ping参数的计算由中央Pingmesh控制器控制。<br>
&emsp;&emsp;测量的延迟数据由数据存储和分析管道收集和存储，汇总和分析。根据等待时间数据，在宏级别（即数据中心级别）和微级别（例如，每服务器和每个机架级别）定义和跟踪网络SLA。 通过将服务和应用程序映射到它们使用的服务器来计算所有服务和应用程序的网络SLA。<br>
&emsp;&emsp;Pingmesh已经在微软的数十个全球分布式数据中心运行了四年。 它每天产生24TB的数据和超过200亿的探测数据。 由于Pingmesh数据的普遍可用性，因为网络变得更容易回答实时站点事件：如果Pingmesh数据不表示网络问题，则实时站点事件不是由网络引起的。<br>
&emsp;&emsp;Pingmesh主要用于网络故障排除，以找出问题所在。 通过可视化和自动模式检测，我们能够回答数据包丢失和/或延迟增加的时间和地点，识别网络中的静默交换机数据包丢失和黑洞。 应用程序开发人员和服务运营商也使用Pingmesh生成的结果，通过考虑网络延迟和数据包丢弃率来更好地选择服务器。<br>
&emsp;&emsp;本文作出了以下贡献：我们通过设计和实施Pingmesh，展示了建立大规模网络延伸测量和分析系统的可行性。 通过让每个服务器参与，我们始终为所有服务器提供延迟数据。 我们展示了Pingmesh通过在宏观和微观范围内定义和跟踪网络SLA来帮助我们更好地理解数据中心网络，并且Pingmesh帮助揭示和定位交换机数据包丢弃，包括数据包黑洞和静默随机数据包丢弃，这在以前很少被了解。<br>

2.背景
-
2.1 数据中心网络

&emsp;&emsp;数据中心网络以高速连接服务器，并提供高服务器到服务器带宽。今天的大型数据中心网络是由商品以太网交换机和路由器构成的。
![Image text](https://github.com/akliy/pingmesh/blob/master/pingmesh-image/Figure%201-Date%20center%20network%20structure)<br>
&emsp;&emsp;图1显示了典型的数据中心网络结构。 网络有两部分：内部数据中心（Intra-DC）网络和内部数据中心（Inter-DC）网络。 DC内网络通常是几层Clos网络架构，类似于[1,12,2]中描述的网络。 在第一层，数十个服务器（例如40台）使用10GbE或40GbE以太网NIC连接到架顶式（ToR）交换机并形成一个Pod节点。 然后将数十个ToR交换机（例如，20台）连接到第二层节点（汇聚）交换机（例如，2-8）。 这些服务器和ToR和Leaf交换机组成一个Podset。 然后，多个Podset连接到第三层Spine交换机（数十到数百）。使用现有的以太网交换机，DC内部网络可以连接数万个或更多具有高网络容量的服务器。<br>
&emsp;&emsp;一个优秀的数据中心是多个Leaf和Spine交换机提供具有冗余的多路径网络。 ECMP（等价多路径）用于对所有路径上的流量进行负载均衡。 ECMP使用TCP / UDP五元组的哈希值进行下一跳选择。 因此，即使连接的五元组已知，TCP连接的确切路径在服务器端也是未知的。 因此，找到有故障的Spine交换机并不容易（因为路径无法确定？）。<br>
&emsp;&emsp;DC间网络用于互连DC内网络并将DC间网络连接到因特网。 DC间网络使用高速长途光纤连接不同地理位置的数据中心网络。 软件定义网络（SWAN [13]，B4 [16]）被进一步引入，以实现更好的广域网络流量工程。<br>
&emsp;&emsp;我们的数据中心网络是一个庞大，复杂的分布式系统。 它由数百个服务器，数万个交换机和路由器以及数百万个电缆和光纤组成。 它由我们自行开发的数据中心管理软件堆栈Au-topilot [20]管理，交换机和NIC运行由不同交换机和NIC提供商提供的软件和固件。 在网络上运行的应用程序可能会引入复杂的流量模式。<br>

2.2 网络延迟和数据包丢失

&emsp;&emsp;在本文中，我们从应用程序的角度看“网络延迟”。 当服务器上的应用程序A向对等服务器上的应用程序B发送消息时，网络延迟定义为从A发送消息到B接收消息的时间之间的时间间隔。实际上，我们测量往返时间（RTT），因为RTT测量不需要同步服务器时钟。<br>

RTT组成如下<br>
-
>1.应用程序处理延迟<br>
>2.OS内核<br>
>3.TCP/IP堆栈<br>
>4.驱动程序处理延迟<br>
>5.NIC引入的延迟（例如，DMA操作，中断调制）[22]，数据包传输延迟，传播延迟和排队延迟引入组成 通过在路径上的交换机处进行分组缓冲。<br>

&emsp;&emsp;有人可能会争论应用程序和内核堆栈引入的延迟并非真正来自网络。 在实践中，我们的经验告诉我们，我们的客户和服务开发人员并不关心。 一旦观察到延迟问题，它通常被称为“网络”问题。 网络团队有责任显示问题是否确实是网络问题，如果问题是，则缓解并解决问题。
&emsp;&emsp;用户感知的延迟可能由于各种原因而增加，例如，由于网络拥塞导致的排队延迟，繁忙的服务器CPU，应用程序错误，网络路由问题等。我们还注意到数据包丢失会增加用户感知的延迟，因为丢弃的数据包需要重新传输。 由于各种原因（例如，光纤FCS（帧校验序列）错误，切换ASIC缺陷，交换机结构缺陷，交换机软件错误，NIC配置问题，网络拥塞等），数据包丢失可能发生在不同的地方。我们在生产网络中看到了所有这些类型的问题。<br>

2.3 数据中心管理和数据处理系统

&emsp;&emsp;接下来我们介绍Autopilot [20]和Cosmos和SCOPE [15]。 数据中心由集中式数据中心管理系统管理，例如Autopilot [20]或Borg [23]。 这些管理系统提供有关如何管理包括物理服务器在内的资源，如何部署，计划，监控和管理服务的框架。 Pingmesh是在Autopilot的框架内构建的。<br>
&emsp;&emsp;Autopilot是Microsoft用于自动数据中心管理的软件堆栈。 其理念是运行软件以自动执行所有数据中心管理任务，包括故障恢复，尽可能减少人力资源。 使用自动驾驶仪术语，集群是由本地数据中心网络连接的一组服务器，由自动驾驶仪环境管理。自动驾驶仪环境具有一组自动驾驶仪服务，包括设备管理器（DM），用于管理机器状态 ，为自动驾驶仪和各种应用程序进行服务部署的部署服务（DS），安装服务器操作系统映像的供应服务（PS），监视和报告各种硬件和软件的健康状况的监视服务（WS），维修服务（RS） ）通过从DM等获取命令来执行修复动作。<br>
&emsp;&emsp;Autopilot提供共享服务模式。 共享服务是在每个自动驾驶托管服务器上运行的一段代码。 例如，Service Manager是管理其他应用程序的生命周期和资源使用情况的共享服务，Perfcounter Collector是一种共享服务，它收集本地perf计数器，然后将计数器上载到Autopilot。 共享服务必须重量轻，CPU，内存和带宽资源使用率低，并且它们需要可靠而不会造成资源泄漏和崩溃。<br>
&emsp;&emsp;Pingmesh使用我们自己开发的数据存储和分析系统Cosmos/SCOPE进行延迟数据存储和分析。Cosmos是微软的BigData（大数据）系统，类似于Hadoop [3]，它提供了像GFS [17]和MapReduce [11]这样的分布式文件系统。Cosmos中的文件仅附加，文件被拆分为多个“扩展区”，扩展区存储在多个服务器中以提供高可靠性。 Cosmos集群可能拥有数万台或更多服务器，并为用户提供几乎“无限”的存储空间。<br>
&emsp;&emsp;SCOPE [15]是一种声明性和可扩展的脚本语言，它构建在Cosmos之上，用于分析海量数据集。 SCOPE设计易于使用。 它使用户能够专注于他们的数据而不是底层存储和网络基础设施。 用户只需要编写类似于SQL的脚本，而不必担心并行执行，数据分区和故障处理。 所有这些复杂性都由SCOPE和Cosmos处理。<br>

3.设计与实施
-

3.1 设计目标

&emsp;&emsp;Pingmesh的目标是构建网络延迟测量和分析系统，以解决我们在第1节中描述的挑战.Pingmesh需要始终开启，并能够为所有服务器提供网络延迟数据。 它需要始终保持开启，因为我们需要始终跟踪网络状态。 它需要为所有服务器生成网络延迟数据，因为最大可能的网络延迟数据覆盖对于我们更好地理解，管理和排除网络基础架构故障至关重要。<br>
&emsp;&emsp;从一开始，我们就将Pingmesh与各种公共和专有网络工具（例如，跟踪器，TcpPing等）区分开来。 我们意识到网络工具对我们不起作用，原因如下。 首先，这些工具并不总是开启，它们只在我们运行时生成数据。 其次，他们生成的数据没有所需的覆盖范围。 由于这些工具并非始终处于开启状态，因此我们无法指望它们跟踪网络状态。 当已知源目标地址时，这些工具通常用于网络故障排除。 然而，这对于大规模数据中心网络并不适用：当网络事件发生时，我们甚至可能不知道源目地址（发生事故的地方）。 此外，对于瞬态网络问题，在运行工具之前问题可能已经消失（网络组的痛点）。<br>

3.2 Pingmesh架构

&emsp;&emsp;根据其设计目标，Pingmesh需要满足以下要求。 首先，因为Pingmesh旨在从应用程序的角度提供尽可能大的覆盖范围和测量网络延迟，`因此每台服务器都需要Pingmesh Agent`。 必须小心这样做，以便Pingmesh Agent引入的CPU，内存和带宽开销很小且价格合理。<br>
&emsp;&emsp;其次，Pingmesh Agent的行为应该受到控制和配置。 需要高度可靠的控制平面来控制服务器应如何执行网络延迟测量。<br>
&emsp;&emsp;第三，应该近乎实时地汇总，分析和报告延迟数据，并将其存储和存档以进行更深入的分析。 根据要求，我们设计了Pingmesh的体系结构，如图2所示.Pingmesh有三个组件，如下所述。<br>
![Image text](https://github.com/aprilmadaha/pingmesh/blob/master/pingmesh-image/Figure%202-Pingmesh%20architecture)<br>
`1.《Pingmesh控制器》。 它是整个系统的大脑，因为它决定了服务器应该如何相互探测。 在Pingmesh Controller中，Pingmesh Generator为每个服务器生成一个pinglist文件。 pinglist文件包含对等服务器列表和相关参数。 pinglist文件基于网络拓扑生成。 服务器通过RESTful Web界面获取相应的pinglist。`<br>
`2.《Pingmesh代理》。 每台服务器都运行Pingmesh代理。 代理从Pingmesh Controller下载pinglist，然后启动TCP / HTTP ping到pinglist中的对等服务器。 Pingmesh代理将ping结果存储在本地内存中。 一旦计时器超时或测量结果的大小超过阈值，Pingmesh Agent会将结果上载到Cosmos进行数据存储和分析。 Pingmesh代理还公开了一组性能计数器，这些计数器由Autopilot的Perfcounter聚合器（PA）服务定期收集。`<br>
`3.《数据存储和分析（DSA)》。 Pingmesh代理的延迟数据在数据存储和分析（DSA）管道中存储和处理。 保护数据存储在Cosmos中。 开发SCOPE工作来分析数据。 SCOPE作业是用类似于SQL的声明性语言编写的。 然后将分析的结果存储在SQL数据库中。 根据此数据库和PA计数器中的数据生成可视化，报告和警报。`<br>

3.3 Pingmesh控制器

3.3.1 pinglist生成算法

&emsp;&emsp;Pingmesh控制器的核心是它的Pingmesh 控制器。 Pingmesh 控制器运行一个算法来决定哪个服务器应该ping哪组服务器。 如前所述，我们希望Pingmesh拥有尽可能大的覆盖范围。 最大可能的覆盖范围是服务器级完整图，其中每个服务器都会探测其余服务器。 但是，服务器级完整图是不可行的，因为服务器需要探测n-1个服务器，其中n是服务器的数量。 在数据中心中，n可以达到数百个。 此外，服务器级完整图不是必需的，因为数十台服务器通过相同的ToR交换机连接到世界其他地方。<br>
&emsp;&emsp;然后，我们提出了多层次完整图形的设计。 在Pod中，我们让同一个ToR交换机下的所有服务器形成一个完整的图形。 在DC内部级别，我们将每个ToR交换机视为虚拟节点，并让ToR交换机形成完整的图形。 在DC间级别，每个数据中心充当虚拟节点，并且所有数据中心形成完整的图形。<br>
&emsp;&emsp;在我们的设计中，只有服务器执行ping操作。 当我们将ToR称为虚拟节点时，它是ToR下执行ping的服务器。 同样，对于作为虚拟节点的数据中心，数据中心中的所选服务器会启动探测。<br>
&emsp;&emsp;在DC内部，我们曾经认为我们只需要选择可配置数量的服务器来参与Pingmesh。 但是如何选择服务器成为一个问题。 此外，少量选择的服务器可能不能很好地代表其余的服务器。 我们终于想出让所有服务器都参与其中的想法。 DC内算法是：对于任何ToR对（ToRx，ToRy），让ToRx中的服务器i ping ToRy中的服务器i。 在Pingmesh中，即使两台服务器位于彼此的pinglist中，它们也会分别测量网络延迟。 通过这样做，每个服务器都可以在本地和独立地计算自己的丢包率和网络延迟。<br>
&emsp;&emsp;在DC间级别，所有DC形成另一个完整的图形。 在每个DC中，我们选择多个服务器（从每个Podset中选择多个服务器）。<br>
结合三个完整的图，Pingmesh中的服务器需要根据数据中心的大小ping 2000-5000对等服务器。 Pingmesh控制器使用阈值来限制服务器的探测总数以及源目标服务器对的两个连续探测的最小时间间隔。<br>

3.3.2 Pingmesh控制器实现

&emsp;&emsp;Pingmesh控制器实现为自动驾驶仪服务，并成为自动驾驶仪管理堆栈的一部分。 它通过运行Pingmesh生成算法为每个服务器生成Pinglist文件。 然后将这些文件存储在SSD中，并通过Pingmesh Web服务提供给服务器。 Pingmesh Controller为Pingmesh代理提供了一个简单的RESTful Web API，以分别检索其Pinglist文件。 Pingmesh代理需要定期向Controller询问Pinglist文件，Pingmesh Controller不会将任何数据推送到Pingmesh代理。 通过这样做，Pingmesh Controller变得无状态且易于扩展。<br>
&emsp;&emsp;作为整个Pingmesh系统的大脑，Pingmesh Controller需要服务数十万个Pingmesh Agent。因此，Pingmesh控制器需要具有容错性和可扩展性。我们使用软件负载均衡器（SLB）[14]来提供容错和可扩展性
Pingmesh控制器。有关SLB如何工作的详细信息，请参见[9,14]。 Pingmesh Controller在单个VIP（虚拟IP地址）后面有一组服务器。 SLB将来自Pingmesh代理的请求分发到Pingmesh Controller服务器。每个Pingmesh控制器服务器都运行相同的代码，并为所有服务器生成相同的Pinglist文件集，并且能够处理来自任何Pingmesh代理的请求。然后，Pingmesh Controller可以通过在同一VIP后面添加更多服务器来轻松扩展。此外，一旦Pingmesh控制器服务器停止运行，它将自动从SLB中旋转。我们在两个不同的数据中心集群中设置了两个Pingmesh控制器，使控制器在地理上更具容错能力。<br>

3.4 pingmesh 代理

3.4.1 Pingmesh 代理设计注意事项

&emsp;&emsp;Pingmesh代理程序在所有服务器上运行。 它的任务很简单：从Pingmesh控制器下载pinglist; ping pinglist中的服务器; 然后将ping结果上传到DSA。<br>
&emsp;&emsp;根据Pingmesh需要能够区分用户感知延迟增加是否由网络引起的要求，Pingmesh应使用应用程序生成的相同类型的数据包。 由于我们数据中心的几乎所有应用程序都使用TCP和HTTP，因此Pingmesh使用TCP和HTTP而不是ICMP或UDP进行探测。<br>
&emsp;&emsp;因为我们需要区分“网络”问题是由于网络还是应用程序本身，Pingmesh Agent不会使用应用程序使用的任何网络库。 相反，我们开发了自己的轻量级网络库，专门用于网络延迟测量。<br>
&emsp;&emsp;Pingmesh代理可以配置为发送和响应除TCP SYN / SYN-ACK数据包之外的不同长度的探测数据包。 结果，Pingmesh代理需要充当客户端和服务器。 客户端部分启动ping，服务器部分响应ping。<br>
&emsp;&emsp;每次探测都需要是新连接并使用新的TCP源端口。 这是为了尽可能地探索网络的多路径特性，更重要的是，减少Pingmesh创建的并发TCP连接的数量。<br>

3.4.2 Pingmesh代理实现

&emsp;&emsp;虽然任务很简单，但Pingmesh Agent是最具挑战性的部分之一。 它必须符合以下安全和性能要求。<br>
首先，Pingmesh代理必须是失效关闭的，不能创建实时站点事件。 由于Pingmesh代理在每台服务器上运行，因此如果它发生故障（例如，使用大部分CPU和内存资源，产生大量探测流量等），它有可能将所有服务器关闭。 为了避免发生不良事件，PingMesh代理实现了以下几个安全功能：<br>

1.Pingmesh Agent的CPU和最大内存使用量受操作系统的限制。 一旦最大内存使用量超过上限，Pingmesh代理将被终止。<br>
2.任何两台服务器之间的最小探测间隔限制为10秒，探测有效负载长度限制为64千字节。 这些限制在源代码中是硬编码的。 通过这样做，我们对Pingmesh可以带入网络的最坏情况流量进行了硬性限制。<br>
3.如果Pingmesh代理无法连接到其控制器3次，或者控制器已启动但没有可用的pinglist文件，则Pingmesh代理将删除其所有现有ping对等项并停止其所有ping活动。 （它仍然会对ping做出反应。）由于这个功能，我们可以通过简单地从控制器中删除所有pinglist文件来阻止Pingmesh Agent工作。<br>
4.如果服务器无法上传其延迟数据，它将重试几次。 之后它将停止尝试并丢弃内存中的数据。 这是为了确保Pingmesh代理使用有界内存资源。 Pingmesh代理还将延迟数据作为日志文件写入本地磁盘。 日志文件的大小限制为可配置的大小。<br>

&emsp;&emsp;其次，Pingmesh Agent需要通过设计启动ping到数千台服务器。 但作为共享服务，Pingmesh代理应尽量减少其资源（CPU，内存和磁盘空间）的使用。 它应该使用接近零的CPU时间和尽可能小的内存占用，以便最大限度地减少对客户应用程序的干扰。<br>
&emsp;&emsp;为了实现性能目标并提高Pingmesh的延迟测量精度，我们使用C ++而不是Java或C＃来编写Pingmesh Agent。 这是为了避免公共语言运行时或Java虚拟机开销。 我们专门为Pingmesh开发了一个网络库。 该库的目标仅用于网络延迟测量，它设计为轻量级并处理大量并发TCP连接。 该库直接基于Winsock API，它使用Windows IO完成端口编程模型进行高效的异步网络IO处理。 该库充当客户端和服务器，并将探测处理负载均匀地分配给所有CPU核心。<br>
&emsp;&emsp;我们已经进行了大量测量，以了解和优化Pingmesh Agent的性能。 图3显示了典型服务器上Pingmesh Agent的CPU和内存使用情况。 在测量过程中，这个Pingmesh Agent正在积极探测大约`2500台服务器`。 该服务器具有128GB内存和两个In-tel Xeon E5-2450处理器，每个处理器有8个内核。 平均内存占用量小于45MB，平均CPU使用率为0.26％。<br>
![Image text](https://github.com/aprilmadaha/pingmesh/blob/master/pingmesh-image/Figure%203-%20CPU%20and%20memory%20usages%20of%20Pingmesh%20Agent)<br>
&emsp;&emsp;我们注意到Pingmesh代理生成的探测流量很小，通常为几十Kb/s。 作为比较，我们的数据中心网络在数据中心的任何两台服务器之间提供几个Gb/s的吞吐量。<br>

3.5 数据存储和分析

&emsp;&emsp;对于Pingmesh数据存储和分析，我们使用完善的现有系统，Cosmos / SCOPE和Autopilot的Perfcounter聚合器（PA）服务，而不是重新发明轮子。<br>
Pingmesh代理会定期将聚合记录上传到Cosmos。 与Pingmesh控制器类似，Cosmos的前端使用负载平衡器和VIP进行扩展。 同时，Pingmesh 代理对延迟数据执行本地计算，并生成一组性能计数器，包括50%至90%的丢包率和网络延迟等。所有这些性能计数器都被收集，汇总和存储在Autopilot的PA服务。<br>
&emsp;&emsp;一旦Cosmos收到结果，我们就会运行一组SCOPE作业进行数据处理。 我们的定时任务为10分钟，1小时，1天。 10分钟的工作是我们近乎实时的工作。 对于10分钟的作业，从生成延迟数据到消耗数据的时间（例如，警报触发，仪表板图形生成）的时间间隔大约为20分钟。 1小时和1天的管道用于非实时任务，包括网络SLA跟踪，网络黑洞检测，丢包检测等。我们的所有工作都由作业管理员自动并定期提交给SCOPE 用户干预。 SCOPE作业的结果存储在SQL数据库中，从中生成可视化，报告和警报。
&emsp;&emsp;实际上，我们发现这20分钟的延迟对于系统级SLA跟踪工作正常。 为了进一步减少响应时间，我们并行使用Autopilot PA管道来收集和聚合一组Pingmesh计数器。 Autopilot PA管道是一种分布式设计，每个数据中心都有自己的管道。 PA计数器收集延迟为5分钟，比我们的Cosmos / SCOPE管道更快。 PA管道比Cosmos / SCOPE更快，而Cosmos / SCOPE比PA更具表现力以进行数据处理。 通过使用它们，我们为Pingmesh提供了比其中任何一个更高的可用性。<br>
&emsp;&emsp;我们将Pingmesh作为一个永远在线的服务与一组定期运行的脚本区分开来。 Pingmesh的所有组件都有监视器来监视它们是否正常运行，`例如，是否正确生成了pinglists，CPU和内存使用是否在预算范围内，是否报告和存储pingmesh数据，DSA是否报告 及时的网络SLA等。此外，Pingmesh Agent旨在以轻量级和安全的方式探测数千个对等体。`<br>
&emsp;&emsp;所有Pingmesh代理每天都会向Cosmos上传24 TB的延迟测量结果。 这超过了2Gb / s的上传速率。 虽然这些看起来像是大数字，但它们只是我们的网络和Cosmos提供的总容量中可忽略不计的一小部分。<br>

4.延迟数据分析

&emsp;&emsp;在本节中，我们将介绍Pingmesh如何帮助我们更好地了解网络延迟和数据包丢失，定义和跟踪网络SLA，以及确定实时站点事件是否是由于网络问题。 我们在本节中描述的所有数据中心都具有与我们在图1中介绍的类似的网络架构，尽管它们的大小可能不同，并且可能在不同的时间构建。<br>

4.1网络延迟

&emsp;&emsp;图4显示了美国西部的两个代表性数据中心DC1和美国中部的DC2的DC内延迟分布。 DC1由分布式存储使用，MapReduce和DC2由交互式搜索服务使用。 DC1中的服务器是吞吐量密集型的，平均服务器CPU利用率高达90％。 DC1中的服务器大量使用网络，并且始终平均发送和接收数百Mb / s数据。 DC2对延迟敏感，服务器具有高扇入和扇出，因为服务器需要与大量其他服务器通信才能为搜索查询提供服务。 DC2中的平均CPU利用率适中，平均网络吞吐量较低但流量突发。<br>
![Image text](https://github.com/aprilmadaha/pingmesh/blob/master/pingmesh-image/Figure%204)<br>
&emsp;&emsp;图4中的CDF是根据一个正常工作日的延迟数据计算的，此时没有检测到网络事件，也没有因网络引起的现场事件。 我们使用和不使用TCP有效负载跟踪pod内和pod间延迟分布。 如果没有特别提及，我们在本文中使用的延迟是无负载的pod-pod TCP SYN / SYN-ACK RTT。<br>
&emsp;&emsp;图4（a）显示了整体的pod间延迟分布，图4（b）显示了高百分位的pod间分布。 我们曾经预计DC1的延迟应该远大于DC2，因为DC1中的服务器和网络负载很高。 但事实证明，90％或更低百分位数的延迟并非如此。<br>
&emsp;&emsp;但是如图4（b）所示，DC1在高百分位数下确实具有更高的延迟。 在P99.9，DC1和DC2的内部延迟分别为23.35ms和11.07ms。 在P99.99，pod间延迟变为1397.63ms和105.84ms。 我们的测量结果表明，即使服务器和网络在宏时间尺度上都是轻载的，也难以在三或四个9s处提供低延迟（例如，亚毫秒级别）。 这是因为服务器操作系统不是实时操作系统，而且网络中的流量是突发的。 即使平均网络利用率低至中等，我们也会看到10-5内部网络通信的丢包率（第4.2节）。<br>
&emsp;&emsp;图4（c）比较了pod内和pod之间的分布，图4（d）研究了有无负载的荚间延迟，所有这些都在DC1中。 对于使用有效负载的延迟测量，在TCP连接设置之后，我们让客户端发送消息（通常在一个数据包中为800-1200字节）。 客户端收到来自服务器的回送消息后，测量有效负载延迟。<br>
&emsp;&emsp;如图4（c）所示，pod-pod延迟总是小于pod之间的延迟，如预期的那样。 DC1的第50（P50）和第99（P99）个荚内和荚间延伸为（216us，1.26ms）和（268us，1.34ms）。 P50和P99的差异分别为52us和80us。 这些数字表明，由于排队延迟，网络确实引入了数十微秒的延迟。 但排队延迟很小。 因此，我们可以确定网络提供足够的网络容量。<br>
