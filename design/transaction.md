>#交易系统#
>>文件编号
>文件状态	[ √ ]草稿  [ ] 正式发布  [ ]正在修改
>当前版本	v1.0
>拟 制  王  磊	
>日期	2014-04-11         
> 审核日期	
> 批准日期	
>发布日期

>***
>>1.	文档控制
>* 文档更新记录
>A - 增加  M - 修订  D - 删除
>日期	更新人	版本	变更类型（A*M*D）	备注
>***2014-04-11	王磊	V1.0	A	***
>2. 文档审核记录
>日期	审核人	职务	备注
>3. 文档发行范围
>分发单位	说明
>***厦门极致互动网络技术有限公司	***
>
>***
>>####概述
>本篇文档主要关系到交易系统UI界面布局划分及其功能描述。

　　交易界面布局示意图
　　显示规格：
项目	字体	字号	颜色
交易方名称	幼圆		
交易方等级	黑体		
交易方关系	黑体		
金钱数额	幼圆		详见金钱显示规则
1.交易判断流程
　　点击交易邀请按钮，如果通过交易判断流程，则直接打开交易界面进行交易：
判断对方是否在线：
  ☉如果对方不在线，则在重要信息区进行提示：
         <对方已下线，交易失败>；
  ☉如果对方在线，则继续进行下一步判断。
判断对方是否在交易范围内：
  ☉如果对方不在交易范围内，则在重要信息区进行提示：
         <对方距离你太远了，交易失败>；
  ☉如果对方在交易范围内，则继续进行下一步判断。
判断自身角色是否死亡：
  ☉如果自身角色已经死亡，则在重要信息区进行提示：
         <死亡无法进行交易，交易失败>；
  ☉如果自身角色未死亡，则继续进行下一步判断。
判断对方是否死亡：
　☉如果对方角色已经死亡，则在重要信息区进行提示：
         <对方已经死亡，交易失败>；
　☉如果对方角色未死亡，则继续进行下一步判断。
判断自身是否在物品交互中：
  ☉如果自身处于物品交互中，则根据所进行的操作进行对应提醒。
        对方正在<某操作名称>中，交易取消；
  ☉如果自身未处于物品交互中，则继续进行下一步判断。
判断对方是否在物品交互中：
  ☉如果对方处于物品交互中，则在重要信息区提示：
　　　　　　　　　<对方正在忙碌中>；
          ☉如果对方未处于物品交互中，则继续下一步判断。
判断对方是否设置了拒绝所有交易：
  ☉如果对方设置了拒绝所有交易，则在重要信息区进行提示：
         <对方设置了拒绝交易，交易失败>
  ☉如果对方没有设置拒绝所有交易，则进行下一步判断。
为响应方发送交易邀请提示，并判断响应方是否接受交易邀请：
          ☉响应方主动拒绝邀请：
                ☉请求方收到重要信息提示：
　　　　　　　　　<被请求的玩家名>拒绝与您进行交易；
                ☉响应方收到重要信息提示：
　　　　　　　　你拒绝了<提出请求玩家>的交易邀请；
          ☉响应方主动接受邀请，进行下一步流程。
响应方是否在时限内对邀请进行处理：
          ☉响应方没有及时对邀请做出处理：
                ☉超过60s依然没有做出回应，请求方收到重要信息提示：
                 <对方拒绝了你的邀请>；
          ☉响应方未对邀请做处理便下线：
                ☉请求方收到重要信息提示：
　　　　　　　　　<对方已下线，交易取消>。
如果通过以上各步判断，则进入交易界面。
1.1交易判断流程图
对应交易判定流程图如下所示：

2相关操作
2.1交易前相关准备
　　☉在交易双方相距半屏之内，选中目标，右键点击目标头像，在弹出的二级菜单中选择
　　　【交易邀请】，即可对目标发起交易邀请。
☉交易申请成功发出后，响应方主界面下方弹出交易的交互提示框（叹号按钮显示），点击后主界面右下角弹出交易选择提示框：
          “ <请求方玩家姓名>玩家请求与你进行交易，确定吗？”
2.2放入物品
　　☉交易界面开启时默认同时打开背包界面。两界面需要水平下边框对其以保持美观度。
　　☉交易界面格子每一方最多允许同时放入10格物品。每一页交易界面为2行5列（列
　　　数与背包界面保持统一）。
　　☉玩家可以通过鼠标左键或右键点击目标物品向交易界面进行提交：
　　      ☉左键或右键点击时，若该格物品数量为1，则自动移动该物品至目标单元格。
　　      ☉左键或右键点击时，若该格物品数量超过1，则提交时弹出数量输入框。输入
　　　　　　框默认显示1，可以通过键盘输入或鼠标“+”、“-”进行数量修改，修改范围
　　　　　　为1~99。
          ☉无法交易的物品其icon在背包道具栏中需要显示红色蒙板作为标示，同时给
　　　　　　予玩家重要信息提示：
　　　　　　    “该物品无法进行交易。”
　　　　　　物品的交易属性可以在物品ID配置表中进行配置处理。
　　　　　☉已经提交至交易界面的物品，在原来所在的单元格依然保留显示其虚化的残
　　　　　　影，残影隐藏右下角的数字显示，残影化的物品无法再次进行提交。中途放弃
　　　　　　交易，则已提交的物品返回原来的所在单元格中。
          ☉鼠标左键点击自身界面的金钱输入框可以进行金钱数额的输入：
                 ☉金钱默认输入数额为0。
　　　　　　　　　☉输入金额若大于持有金钱数量总数时，则显示金钱总数。
                 ☉提交金额输入为0（最多仅可输入1个0），则为清空之前的提交金额。
          ☉交易过程中点击交易界面右上角的【X】或右下角的【取消】可以中断交易，
　　　　　　重要信息区提示：
　　　　　　     <取消方玩家名称>取消了交易；
　　　　　　     你取消了交易；
　　　　　☉交易过程中，若双方距离超过预设值、任意一方死亡则交易关闭，双方都收到
　　　　　　重要信息提示：
　　　　　　     双方距离太远，交易取消；
　　　　　　     对方已死亡，交易取消；
　　　　　　　　　死亡状态下无法进行交易；
2.3锁定物品
         ☉提交物品后需要锁定才能继续点击【交易】按钮以完成交易，否则【交易】按
　　　　　　钮置灰显示无法点击；
         ☉锁定后，锁定方的交易界面出现红色蒙板；同时锁定按钮置灰，【交易】按钮变
　　　　　　为高亮可点击状态。
         ☉锁定后除非取消交易并再次开启交易否则无法增减物品状态或修改金钱数额；
         ☉锁定状态下，点击交易界面右上角的【X】或右下角的【取消】可以中断交易，
　　　　　　重要信息区提示：
                <取消方玩家名称>取消了交易；
                你取消了交易；
　　　　　☉交易过程中，若双方距离超过预设值、任意一方死亡则交易关闭，双方都收到
　　　　　　重要信息提示：
                双方距离太远，交易取消；
                对方已死亡，交易取消；
　　　　　　　　死亡状态下无法进行交易；
2.4确认交易
         ☉双方都锁定了交易界面之后，点击【交易】按钮进行确认。此时【锁定】按钮
　　　　　　和【交易】按钮都处于置灰状态。
         ☉若有A方点击了【交易】确认按钮，B方尚未点击时，A方可以通过取消交易来退出交易状态，双方已提交的物品和金钱全部退还。
           重要信息区提示：
                <取消方玩家名称>取消了交易；
                你取消了交易；
         ☉若其中一方背包空间不足以放下交易成功后所得的全部物品，则交易不成功，物品金钱全额退还至双方，重要信息区提示：
                <背包不足玩家>背包空间不足，交易失败；
                你的背包空间不足，交易失败；
         ☉若其中一方背包负重不足以支持交易成功后所得的全部物品，则交易不成功，物品金钱全额退还至双方，重要信息区提示：
                <负重不足玩家>背包空间不足，交易失败；
                你的背包负重不足，交易失败；
         ☉若其中一方交易后所得金钱总数额超过20亿显示上限，则交易不成功，物品金钱全额返还，并根据《物品界面功能设计》中“金钱上限管理”进行提示。
☉若【锁定】后【交易】前双方距离超过预设值、任意一方死亡则关闭交易，双方都收到重要信息提示：
                双方距离太远，交易取消；
                对方已死亡，交易取消；
　　　死亡状态下无法进行交易；
         ☉交易成功后，进行聊天区提示：
　　　　　　　　你将<物品名>*<数量>交易给了<名称>；
　　　　　　　　你将<数量>金钱交易给了<名称>；
　　　　　　　　你从<名称>处交易获得<数量>金钱；
　　　　　　　　你从<名称>处交易获得<物品名>*<数量>； 
         ☉交易成功后，判断身上金钱数量，若超过携带上限，则给予相应提示。
3互斥规则
         ☉进行交易的过程中，无法同时进行其他有可能增减包裹中物品数量的操作
　　　　　☉若进行上述操作时，显示重要信息区提示信息 
　　　　　　　　“交易中无法进行此操作。”
3.1互斥操作归纳
和交易互斥的操作	物品拆分
	物品丢弃
	物品使用
	购买物品
	与npc交谈
	装备/卸下道具
	打开仓库
	存取物品、金钱
	捡取物品
	打开在线礼包
	摆摊
	收取信件附件
4.其它位置【交易】按钮的判定规则
　　除了目标头像选择区域外，还可以发起交易的方式有：1.通过队友头像区域；2.好友列表二级菜单。
　　☉处理流程：
　　   玩家点击出现在以上位置的“交易”按钮时，判断目标是否在可交易范围：
　　  ☉是，则进入正常的交易流程，给目标发送交易申请，进行后续的双方交易意向判断；
　　  ☉否，则在重要信息区提示：
　　       “距离太远，交易失败。”
　　        


> Written with [StackEdit](https://stackedit.io/).