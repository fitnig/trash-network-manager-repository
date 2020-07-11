# BGP选路规则

## 2020.06.24

| 序号 | 规则                                | 释义                                                         | 备注                                                         |
| ---- | ----------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 1    | weight                              | 管理权重越大越优先。这个参数本地有效。虽然Weight属性是Cisco私有的，但是很多厂商也支持该属性，这样就保证了本地始发的路由是最悠闲的，因为本地始发路由的Weight为32768，从其他BGP Peer学习过来的路由的Weight为0。 |                                                              |
| 2    | local-pref                          | 本地优先级（越大越优先），这个参数在本AS内传递。Local Preference属性只能在IBGP Peer之间传递，如果在EBGP Peer之间收到的路由的路径属性中携带了Local  Preference，则会触发Notifaction报文，造成会话中断。 |                                                              |
| 3    | 本地始发的路径优先                  | 本地始发的路径特点是next-hop为0.0.0.0，weight为32768。可以使用不同的方式比如network或redistribute等，那么这些方式之间是存在优先顺序的原则：network>redistribuet>aggregate，但该原则是不会作为BGP路由选路策略的。 |                                                              |
| 4    | AS-path                             | 一般情况下，AS-PATH最短的优先。但是可以配置bgp bestpath as-path ignore来忽略这一步。注意：在做聚合路由时，使用as-set后产生的AS-Path列表中的{}录得AS号长度只算一个AS号的长度；而在联盟内的AS-Path列表中的()的AS号长度不做计算依据！不同方向的route-map对于插入的AS号的位置是不同的。 | 如果要形成负载的关系，AS-PATH必须一致。如果AS-PATH长度相同，但是AS-PATH经过的AS不同，则不会形成敷在关系。这时，BGP会选择BGP邻居简历时间成唱的peer发送的路由。 |
| 5    | origin                              | 三种不同的Origin属性的优先顺序：IGP>EGP>incomplate，Origin属性会一直在BGP路由中携带。很少使用设置Origin属性作为BGP路由选路策略。 |                                                              |
| 6    | MED                                 | MED值最小的路径优选。默认情况下，只比较来自同一AS的BGP路由的MED值（就是AS-sequence中第一个AS相同才比较）。命令bgp always-compare-med对于所有路径都比较MED，不考虑他们是否来自同一个AS。如果使用了这个选项要在AS内都这么配置（避免路由选择环路）。（任何开头为as-confed-sequence的都被忽略比较MED值，如果配置了bgp always-compare-med那么会进行比较）。 |                                                              |
| 7    | EBGP优于IBGP；EBGP优于联邦EBGP      | （联邦EBGP和联邦IBGP不具有可比性，不比较。因为联邦ebgp和ibgp都被看做内部路径没有差别）。如果都是EBGP对等体收到的条目或者都是从IBGP对等体收到的的条目或者分别从联邦EBGP和联邦IBGP对等体收到的条目则继续向下一步进行。 |                                                              |
| 8    | 优先选择到下一跳IGP度量值最低的路径 | 详细解释                                                     |                                                              |
| 8    | cost comminuty                      | BGP Cost Community（BGP成本团体）的扩展团体属性提供了自定义最佳路径选择过程的方式。这个自动路径选择过程插入在BGP13条选路原则的第8条之后（优先到下一跳IGP-cost最低的路径）、第9条之前，也可以成为8.5选路原则。 |                                                              |
| 9    | 等价负载均衡                        | 当前面8条选路原则都无法选出最优路由时，并且在BGP进程下配置了maximum-paths [ibgp] <1-16>，那么将执行等加负载均衡，如果没有ibgp关键字，那么只会对EBGP对等体收到的路由执行等价负载均衡，如果不配置maximum-paths那么将进行到下一条选路原则。 |                                                              |
| 10   | EBGP优先使用最先收到的路由条目      | 该条针对的是多条EBGP路径的情况。所谓最先收到的路由也就是路由表里存活时间最长的路由（最老的路由）。这能最小化路由抖动。如果BGP进程下使用bgp bestpath compare-routerid命令，则忽略本原则，跳到第11条选路原则；当多条路由具有相同的router-id时也胡月本原则，当没有当前最佳路由时，也忽略本原则，例如提供最佳路径的邻居down掉。（仅ebgp路由） |                                                              |
| 11   | router-id小者优选                   | 如果路径包含RR属性，那么在路径选择过程中就用originator-id来代替router-id进行比较（就是originator-id之间进行比较）。 |                                                              |
| 12   | 优选cluster-list长度最短的路径      | 存在在部署了RR的网络环境                                     |                                                              |
| 13   | 优选建立邻居地址最小的路径          | 这个地址指的是我们配置邻居时neighbor后指定的邻居地址         |                                                              |

