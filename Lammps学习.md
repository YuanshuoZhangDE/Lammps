<!--
 * @Descripttion: 
 * @version: 
 * @Author: Yuanshuo
 * @Date: 2020-02-21 17:13:46
 * @LastEditors: Yuanshuo
 * @LastEditTime: 2020-02-21 18:41:19
 -->
### LAMMPS输入文件
- in.file(in文件，必备)
- data.file(模型文件，可有可无)
- 势文件或力场文件(可有可无)
### LAMMPS输出文件
默认输出log文件帮助查看运行过程是否正常，其他输出则需要在in文件中自行定义，否则无输出。  
一般，in文件内容的主要框架包括以下几个部分：  
- 初始模型系统设置
- 初始模型构建(读取模型数据)
- 定义原子间相互作用势或设置力场(势文件或力场文件)
- 定义原子/体系某些信息的计算(ke/pe...)
- 定义输出原子(坐标)/体系(热力学信息)
- 模拟环境设定并运行

### LAMMPS简单MD模拟——Fe的弛豫过程
弛豫过程：一般进行模拟前必备的过程，目的是使模型达到热力学平衡状态。  
1. 输入文件in.file
```
#初始模型系统设置
echo                    screen 
units                   metal
boundary                p p p
atom_style              atomic
#初始模型构建(读取模型数据)
lattice                 bcc 2.8552
region                  box block 0 10 0 10 0 10 units lattice
create_box              1 box
create_atoms            1 box
#定义原子间相互作用势
pair_style              eam/fs
pair_coeff              * * Fe_mm.eam.fs Fe 
#设定计算原子势能、动能及相邻原子个数
compute                 1 all pe/atom
compute                 2 all ke/atom
compute                 3 all coord/atom cutoff 3.0
#定义输出原子(坐标)/体系(热力学信息)
thermo_style            custom step pe ke
thermo                  1000
dump                    1 all custom 1000 dump.xyz id x y z c_1 c_2 c_3
#模拟环境设定并运行
velocity                all create 273.15 4928459 dist gaussian
fix                     1 all nvt temp 273.15 273.15 0.5                   
timestep                0.005
run                     100000
```  
2. 由于直接在in文件中构建好了模型，所以不需要data文件。
### 初始模型系统设置
经常用于初始模拟系统设置的只有3个，分别为 units, boundary, atom_style, 还有一些比较不常用的（不常用的就是设置为默认值）有dimension，neighbor等
- units命令是用来定义整个模拟系统中的单位的，	units          metal使用这条命令，则计算中一切与时间挂钩的量都是以皮秒为单位；     	units           real 使用这条命令，则计算中一切与时间挂钩的量都是以飞秒为单位；units命令使用不当会导致后期数据处理需要进行繁杂的单位换算。所以计算前要思考清楚选取何种单位体系最合适。
