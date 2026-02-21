# Digital-IC_learn
基于Picorvi的数字后端实践

-- 进行指令集测试 -- make test
-  修改脚本文件tcl，主要修改输入输出文件，以及工程路径
-

目录结构
-- picorv32-main
  -- tests
    --
  -- DC
    -dc_syn.tcl
    --OUT
  -- PR
    -- INPUT
      - netlist.v .sdc  这两个文件是上一步骤DC跑出的输出文件
      - Init_globals Init_io_file.io MMMC.view PowerDEfine.1801  固定输入设置，同样要修改工程路径以及项目名称和输入输出。
    -- PR_DIR （仿真项目的根目录，dbs、out、rot这三个文件夹必须在仿真前提前创建，打开innovus进行Place以及Routing）
      -- DBS
      -- OUT
      -- RPT
    -- SCRIPT （主要是存放脚本 ）
      - floorPlan.tcl innovus.tcl padPlan.tcl powerPlan.tcl #实现布局规划（芯片长宽面积等整体布局信息）、引脚规划（根据IO端口进行设计）、功耗设计、工具参数（输出输入以及工程路径）
  -- FM
    -fm_post.tcl (修改工作目录以及项目名称，以及输入输出文件路径，对比verilog与gds版图布局后的逻辑进行vertification)
  -- PT
    -pt_postPR.tcl （修改相关参数，修改端口以及测试输入）
  -- VIRTUOSO
  -- work
  -- rtl
  -- scripts
  -- picosoc
  -- firmware
  - Makefile
  - picorv32.core/v
  - testbench.*

## 环境配置
## 前驱准备文件以及软件平台
## Design Compiler
1、修改DC中的tcl，主要实现端口修改以及控制信号如reset信号端口的重置
2、借助design_vision工具，首先启动lmg实现Synopsys的证书管理，然后再DesignVision中执行tcl命令，实现DC综合步骤，tcl+OUT文件夹
3、OUT文件夹中产出violators、timing、power、area.report  以及picorv.netlist.v  picorv.sdf/sdc
<img width="1061" height="644" alt="image" src="https://github.com/user-attachments/assets/f638950f-535d-49d9-995b-c2b8f65b6bcc" />
<img width="721" height="975" alt="image" src="https://github.com/user-attachments/assets/1918238f-bb2c-4d04-a768-f0247de5431b" />
综合部分首先是依赖于特定工艺节点的数字工艺库，包含门电路laytancy_time、time_setup、time_hold、版图面积功耗等信息。


## P&R 
### 项目结构构建
 -- PR
    -- INPUT
      - netlist.v .sdc  这两个文件是上一步骤DC跑出的输出文件
      - Init_globals Init_io_file.io MMMC.view PowerDEfine.1801  固定输入设置，同样要修改工程路径以及项目名称和输入输出。
    -- PR_DIR （仿真项目的根目录，dbs、out、rot这三个文件夹必须在仿真前提前创建，打开innovus进行Place以及Routing）
      -- DBS
      -- OUT
      -- RPT
    -- SCRIPT （主要是存放脚本 ）
      - floorPlan.tcl innovus.tcl padPlan.tcl powerPlan.tcl #实现布局规划（芯片长宽面积等整体布局信息）、引脚规划（根据IO端口进行设计）、功耗设计、工具参数（输出输入以及工程路径）
### 
使用innorvus进行布局布线，具体可以看innovus.tcl的具体的代码片段

1、首先加载SCRIPT配置的加载，完成 EDA 工具环境初始化、工艺库加载、低功耗意图（UPF）加载，建立后端设计的基础环境。
2、布局规划（Floorplan）与电源规划阶段
完成芯片的平面布局规划（Core 面积、IO 位置）、电源网络规划（Power Ring、Power Stripe），为后续标准单元布局和电源连接奠定基础。
3、电源网络布线（Sroute）与验证阶段
完成电源 / 地网络的特殊布线（Sroute），连接 Block Pin、Pad Pin、Core Pin、Floating Stripe，确保 VDD/VSS 网络的连通性，为标准单元供电。
4、标准单元布局（Placement）阶段
将标准单元（逻辑门、寄存器等）摆放到 Core 区域的 Row 中，满足时序驱动、拥塞控制、密度约束，同时完成衬底接触（Well Tap）和 Tie Cell 的插入。
5、预时钟树综合优化（Pre-CTS Optimization）阶段
在时钟树综合前，通过 GigaOpt 引擎优化组合逻辑，修复 Setup 时序违例，为 CTS 提供良好的起点。
6、时钟树综合（CTS）阶段
构建时钟树网络，通过插入时钟缓冲器和反相器，平衡时钟 skew，满足时钟树的时序和 DRC 要求。
7、后时钟树综合优化（Post-CTS Optimization）阶段
在时钟树完成后，分两次优化：先修复 Setup 时序，再修复 Hold 时序，确保时序收敛。
8、全局布线与详细布线（Route）阶段
完成所有信号网络的全局布线（Global Route）和详细布线（Detail Route），满足 DRC 规则，同时考虑时序驱动。
9、布线后优化（Post-Route Optimization）与收尾阶段
在布线完成后，进行最终的时序优化、DRC 修复、单元填充，确保设计满足所有签核（Signoff）要求。
10、数据输出与流片准备阶段
输出流片所需的 GDSII、网表、SDF 等数据，完成后端设计的最终交付。

最后产出GDS文件，是最终的版图信息，同时包含制造工艺库参数



## Formality形式验证（逻辑验证）
使用formality工具看DC综合后的电路逻辑功能是否缺失，对比innovus跑出的gds 与 前端设计RTL-verilog的逻辑是否一致，
在FM文件下加入fm_post.tcl并且修改工作目录以及项目名称，以及输入输出文件路径，对比verilog与gds版图布局后的逻辑进行vertification
<img width="1008" height="1185" alt="image" src="https://github.com/user-attachments/assets/2eb9f552-9191-4a52-b415-eb23c432f194" />


## PrimeTime（PT）时序上验证
属于STA静态时序分析的一环，利用PrimeTime检查布局后的电路的violation情况，如time_setup、time_hold
验证关键时序路径的延迟，显示对寄存器关键路径测试的覆盖率
<img width="350" height="247" alt="image" src="https://github.com/user-attachments/assets/712f9e5a-58bf-42b9-aef1-f862190f9024" />
<img width="945" height="854" alt="image" src="https://github.com/user-attachments/assets/08d8a457-93c0-489c-8acb-fbfefeaa1ec8" />
<img width="1890" height="846" alt="image" src="https://github.com/user-attachments/assets/0535e197-faf0-45cc-8d72-c07b2b889818" />

## PV 物理验证
通过前面阶段的测试后，导出GDSII文件（包含标准单元库版图信息以及参考库电路版图信息）。
用virtuoso进行DRC设计规则违例检查以及LVS版图layout与Sechematic一致性检查。
