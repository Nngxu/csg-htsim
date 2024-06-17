# 框架结构
框架中主要包含两部分代码:
- `sim`文件夹(不包含子文件夹)中是htsim仿真器和各种各种算法实现(如`EQDS`, `NDP`等)实现；
- `sim/datacenter`是使用`EQDS`, `NDP`等算法在数据中心的实现

```bash
❯ tree # sim文件夹下的文件结构
.
├── Makefile # 编译规则
├── cbr.cpp # 限制发送端的速率
├── cbr.h
├── compositequeue.cpp # 实现packet trimming的功能
├── eqds.cpp # EQDS实现
├── eqds.h
├── ndp.cpp # 实现NDP
├── ndp.h
├── network.cpp # 网络行为模拟(转发、路由)
├── network.h
├── parse_output.cpp
├── pipe.cpp    # 模拟传输媒介的延时
├── pipe.h
├── qcn.cpp
├── qcn.h
├── queue.cpp # 实现基本队列(模拟switch和NIC)
├── queue.h
├── queue_lossless.cpp # 实现ROCEV2中的PFC
├── queue_lossless.h
├── README.md
├── roce.cpp # 实现vanilla Roce(GBN)
├── roce.h
├── swift.cpp # 实现Swift
├── swift.h
├── switch.cpp # 模拟交换机
├── switch.h
├── tcp.cpp   # 实现TCP NewReno
├── tcp.h
├── ..... 
├── datacenter # 数据中心仿真需要的其他组件
│   ├── bcube_topology.cpp
│   ├── bcube_topology.h
│   ├── camcubetopology.cpp
│   ├── camcubetopology.h
│   ├── connection_matrices # 生成各种流量模式的方法allreduce ...
│   │   ├── bidir.cm
│   │   ├── gen_allreduce_butterfly.py
│   │   ├── ..... # 省略
│   │   └── perm_32n_32c_2MB.cm
│   ├── fat_tree_switch.cpp # fat_tree下支持ECMP的交换机
│   ├── fat_tree_switch.h
│   ├── fat_tree_topology.cpp # fat_tree拓扑
│   ├── fat_tree_topology.h
│   ├── main_eqds.cpp # main_* 仿真入口程序
│   ├── main.h
│   ├── main_hpcc.cpp
│   ├── main_ndp.cpp
│   ├── main_roce.cpp
│   ├── main_swift.cpp
│   ├── main_tcp.cpp
│   ├── Makefile
│   ├── ...
│   ├── old_tests
│   │   ├── main_dctcp_incast_collateral.cpp
│   │   ├── ....
│   │   ├── Makefile
│   │   └── README
│   ├── oversubscribed_fat_tree_topology.cpp
│   ├── oversubscribed_fat_tree_topology.h
│   ├── shortflows.cpp
│   ├── shortflows.h
│   ├── star_topology.cpp
│   ├── star_topology.h
│   ├── subflow_control.cpp
│   ├── subflow_control.h
│   ├── topologies
│   │   ├── fat_tree_1024.topo
│   │   ├── leaf_spine_tiny.topo
│   │   └── test.topo
│   ├── topology.h
│   ├── validate.py
│   ├── validate.txt
│   ├── vl2_topology.cpp
│   └── vl2_topology.h
├── EXAMPLES # 相关示例
├── tests
│   ├── htsim-tests
│   │   ├── bidir_ndp
│   │   │   ├── default.ref
│   │   │   └── test_config.json
│   │   ├── bidir_swift
│   │   │   ├── default.ref
│   │   │   └── test_config.json
│   │   ├── dumbell_hpcc
│   │   │   ├── default.ref
│   │   │   └── test_config.json
│   │   ├── dumbell_ndp
│   │   │   ├── default.ref
│   │   │   └── test_config.json
│   │   ├── dumbell_ndptunnel
│   │   │   ├── default.ref
│   │   │   └── test_config.json
│   │   ├── dumbell_roce
│   │   │   ├── default.ref
│   │   │   └── test_config.json
│   │   ├── dumbell_strack
│   │   │   ├── default.ref
│   │   │   └── test_config.json
│   │   ├── dumbell_swift
│   │   │   ├── default.ref
│   │   │   └── test_config.json
│   │   ├── dumbell_tcp
│   │   │   ├── default.ref
│   │   │   └── test_config.json
│   │   ├── multihop_swift
│   │   │   ├── default.ref
│   │   │   └── test_config.json
│   │   ├── multihop_swift2
│   │   │   ├── default.ref
│   │   │   └── test_config.json
│   │   ├── multipath_swift
│   │   │   ├── default.ref
│   │   │   └── test_config.json
│   │   └── trigger_test
│   │       ├── default.ref
│   │       └── test_config.json
│   ├── ........
│   ├── Makefile
│   └── tests.py
├── trigger.cpp
└── trigger.h
```

# 编译方法
- 在sim目录中运行`make`命令将编译`TCP、NDP、htsim、network`等源文件。这些代码与数据中心无关。此阶段的输出是:
    - `libhtsim.a`，这是构建数据中心代码所需的库（该库静态链接到`sim/datacenter/`目录下所有的`htsim_*`可执行文件中）。
    - 输出解析工具`（parse_output）`。
- 进入`sim/datacenter`目录并运行`make`。这将编译htsim支持的所有数据中心拓扑，并创建多个可执行文件（名称为`htsim_...`），用于运行不同的实验。每个`htsim_X`可执行文件对应于一个`main_X`源文件，负责设置和运行相应的实验。

## 试验
- 在`sim/datacenter`文件夹下进行NDP试验:
```bash
# 运行试验
./htsim_ndp -nodes 16 -tm connection_matrices/perm_16n_16c_2MB.cm -cwnd 50 -strat perm -log sink -q 50 -end 1000
# 解析试验结果
../parse_output logout.dat -ndp -show
```

# htsim Network Simulator

htsim is a high performance discrete event simulator, inspired by ns2, but much faster, primarily intended to examine congestion control algorithm behaviour.  It was originally written by [Mark Handley](http://www0.cs.ucl.ac.uk/staff/M.Handley/) to allow [Damon Wishik](https://www.cl.cam.ac.uk/~djw1005/) to examine TCP stability issues when large numbers of flows are multiplexed.  It was extended by [Costin Raiciu](http://nets.cs.pub.ro/~costin/) to examine [Multipath TCP performance](http://nets.cs.pub.ro/~costin/files/mptcp-nsdi.pdf) during the MPTCP standardization process, and models of datacentre networks were added to [examine multipath transport](http://nets.cs.pub.ro/~costin/files/mptcp_dc_sigcomm.pdf) in a variety of datacentre topologies.  [NDP](http://nets.cs.pub.ro/~costin/files/ndp.pdf) was developed using htsim, and simple models of DCTCP, DCQCN were added for comparison.  Later htsim was adopted by Correct Networks (now part of Broadcom) to develop [EQDS](http://nets.cs.pub.ro/~costin/files/eqds.pdf), and switch models were improved to allow a variety of forwarding methods.  Support for a simple RoCE model, PFC, Swift and HPCC were added.

## Getting Started

There are some limited instructions in the [wiki](https://github.com/Broadcom/csg-htsim/wiki).  

htsim is written in C++, and has no dependencies.  It should compile and run with g++ or clang on MacOS or Linux.  To compile htsim, cd into the sim directory and run make.

To get started with running experiments, take a look in the experiments directory where there are some examples.  These examples generally require bash, python3 and gnuplot.
