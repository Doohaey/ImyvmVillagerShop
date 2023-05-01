# Imyvm Villager Shop
***
**注意，该库仍在开发状态中，目前仅完成了部分指令的注册以及数据的存取，如有任何想法欢迎提issue，有能力者可以帮忙完善并提出pr。**
## 一、配置文件
文件位置：`config/imyvm_villagershop.conf`
```yaml
core {
  # 指定显示语言
  language="en_us"
  # 玩家补货时的税率
  "tax_rate"=0.01
  # 数据库地址，数据库目前仅支持Postgresql
  "database_url"="jdbc:postgresql://localhost:12346/imyvmvillagershop"
  # 数据库用户名
  "database_username"=root
  # 数据库密码
  "database_password"="1145141919810"
}
```
管理员可以使用`/villagershop config reload`来重载配置，使用`/villagershop taxrate set <taxrate>`来设置税率。
## 二、商店创建
### 1.玩家商店
玩家可以使用指令`/villagershop create shop <shopname> <x y z> <item> <count> <price>`创建商店[^1]，并扣除一定的货币。  
货币的花费会随着玩家拥有商店数量的增加而增加，函数表达式为:
```math
f(x)
\begin{cases}
40, &x\ \le\ 3\\
10\times2^{x-1}, &x\ \gt\ 3
\end{cases}
```
**注意**
- 商店刚创建或商品刚添加时商品的库存为0，你需要在创建完成后额外使用`/villagershop item addstock <shopname> <item> [count]`来进行“补货”，`count`是一个可选参数，若不填则默认消耗背包内所有特定物品。  
同时，补货需要消耗一定的货币，计算公式如下：
$$f(x) = p \times c \times t$$  
> $p$为设置价格，$c$为补货数量，$t$为税率，默认为0.1%
- 同一玩家创建的商店**不允许**名称相同，当个商店的商品**数量上限为7个物品**，价格下限**不低于**服务器设置的物价或系统商店物价。
- 创建商店时的`count`参数代表用`price`的钱可以买到`count`个数的物品  
举个栗子，比如我使用`/villagershop create shop air 0 0 0 minecraft:air 2 100`创建了一个出售`air`的商店。那么，我可以使用`100`货币买到`2`个`air`。而不是商店中`air`的库存为`2`。
- 所有商店都有一个唯一的`shopnumber`标识符，并且永远不可能重复。
### 2.系统商店
管理员可以使用指令`/villagershop create adminshop <shopname> <x y z> <item1> <count>:<price> <item2> <count>:<price> ... <item_n> <count>:<price>`来创建系统商店。  
系统商店库存**无限**，但是可以根据情况限制单个玩家的购买数量。  
### 3.商店管理
##### ①货物设置
在商店创建成功后，玩家可以使用`/villagershop item add <shopname> <item> <count> <price>`来添加商品，使用`/villagershop item change <shopname> <item> <count> <price>`来修改**已有**商品的价格以及出售数量，使用`/villagershop item delete <shopname> <item>`来删除已有的商品[^1]  
##### ②商店设置
玩家可以使用`/villagershop manager info set <shopname> {shopnamenew,x y z}`来修改商店的名称或者位置。当你不想要该商店时，可以使用`/villagershop delete {shopname,shopnumber}`[^1]来删除该商店，管理员可以使用`shopnumber`删除任意不符合规定的商店，无需提前通知拥有者。删除商店会返还所有的库存，但是并不会返还创建时消耗的货币。  
同时，管理员可以使用`/villagershop manager search {shopname:shopname, location:location, owner:owner}`，owner参数暂时只允许使用玩家的uuid，可以使用`/villagershop manager inquire <shopnumber>`获取特定商店的详细信息。    
管理员在征得玩家以及管理组同意的情况下，可以使用`/villagershop setAdmin <shopnumber>`[^1]来将该商店转化为系统商店。  
### 4.收购相关
服务器在特定时间（如每半小时）就系统商店中已在出售的一种物品发起一次收购，拥有物品的玩家可以使用`/villagershop sell <count>`出售指定数量的收购物品，达到收购数量后服务器停止收购。  
收购价格为系统商店在售价格的**80%**  

TODO：  
 - [ ] 商店创建时即消耗背包内的物品补货   
 - [ ] 补货时可以从玩家身上的潜影盒中获取所需物品   
 - [ ] 支持范围搜索商店，以及通过玩家名称搜索商店，搜索结果添加所满足条件，不输出重复结果。
 - [ ] 完成收购指令以及商店的实体创建。

[^1]: 对于这种指令，你需要使用`/villagershop confirm`来确定执行。