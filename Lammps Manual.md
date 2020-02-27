<!--
 * @Descripttion: 
 * @version: 
 * @Author: Yuanshuo
 * @Date: 2020-02-21 17:13:46
 * @LastEditors: Yuanshuo
 * @LastEditTime: 2020-02-27 13:03:14
 -->

<!-- TOC -->

- [LAMMPS输入文件](#lammps输入文件)
- [LAMMPS输出文件](#lammps输出文件)
- [LAMMPS简单MD模拟——Fe的弛豫过程](#lammps简单md模拟fe的弛豫过程)
- [初始模型系统设置](#初始模型系统设置)
- [初始模型构建(读取模型数据)](#初始模型构建读取模型数据)
- [定义原子间相互作用势](#定义原子间相互作用势)
- [定义原子/体系某些信息的计算(ke/pe...)](#定义原子体系某些信息的计算kepe)
- [定义输出原子(坐标)/体系(热力学信息)](#定义输出原子坐标体系热力学信息)
- [模拟环境设定并运行](#模拟环境设定并运行)

<!-- /TOC -->

### LAMMPS输入文件
- in.file(in文件，必备)
- data.file(模型文件，可有可无)
- 势文件或力场文件(可有可无)
### LAMMPS输出文件
默认输出log文件帮助查看运行过程是否正常，其他输出则需要在in文件中自行定义，否则无输出。  
一般，in文件内容的主要框架包括以下几个部分：  
- 初始模型系统设置
- 初始模型构建(读取模型数据)
- 定义原子间相互作用势(势文件)
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
经常用于初始模拟系统设置的只有3个，分别为 `units`, `boundary`, `atom_style`, 还有一些比较不常用的（不常用的就是设置为默认值）有dimension，neighbor等
- `units`命令是用来定义整个模拟系统中的单位的，`units  metal`使用这条命令，则计算中一切与时间挂钩的量都是以皮秒为单位；`units  real`使用这条命令，则计算中一切与时间挂钩的量都是以飞秒为单位；units命令使用不当会导致后期数据处理需要进行繁杂的单位换算。所以计算前要思考清楚选取何种单位体系最合适。
- `boundary`命令是用来定义边界条件的，LAMMPS提供了四种边界条件分别为：
    - p：周期性边界条件
    - f：非周期性边界条件，采用这种边界条件，当有原子运动到盒子以外的区域时，该原子便会被系统删除。需与`thermo_modify  lost  ignore`这条命令结合使用。
    - s：该方向的大小会随着原子的运动而改变（以保证不丢失原子）。
    - m：该方向的大小会随着原子的运动而改变，但有最小值。
- `atom_style`命令是用来告诉LAMMPS模型中有些什么东西比如说原子，键角，电荷之类的，若模型中只有Fe原子：`atom_style  atomic`(一般金属体系)
### 初始模型构建(读取模型数据)
对于初始模型的构建，可以自己编写建模，可以利用建模软件进行建模(Materials  Studio)，将MS得到的模型文件导入OVITO，利用OVITO导出LAMMPS软件可读的格式，再利用`read_data`命令读取模型文件即可。  
Lammps提供一些命令帮助构建简单模型：
- `lattice`主要是用来定义晶格类型，晶格常数，以及晶向方向的，例如：`lattice bcc 2.8552 orient x 1 0 0 orient y 0 1 0 orient z 0 0 1`表示为构建一个晶格常数为2.8552，晶格类型为bcc的模型，其x方向的矢量为 [100]... ...`orient x 1 0 0 orient y 0 1 0 orient z 0 0 1`是默认值，不改变晶向的话，可不写。
- `region`用来构建模拟盒子大小以及划分模拟区域，例如(前提是定义了lattice命令)：`region box block 0 10 0 10 0 10 units lattice`表示构建一个10a*10a*10a大小的模拟盒子，无论模拟的构型有多么复杂，一般都是选择先构建一个矩形盒子，让模型在盒子中进行模拟。`units lattice`为默认值。`region  1 block 1 9 1 9 1 9`表示将盒子中x（1a-9a），y（1a-9a），z（1a-9a）的区域选中，定义为区域1，用于后续建模模拟。  
注：一般box只用于与盒子相关的地方，只选取部分区域时，不要将其定义为box。
- `create_box`告诉LAMMPS模型中有几类原子，几类键等等。`create_box 2 box`表示盒子中有2种原子；`create_box 2 box bond/type 2`表示盒子中有2种原子以及2种键长。
- `create_atoms`向盒子中添加原子，例如：`create_atoms 1 box`表示将类型1的原子按照lattice命令的设定填满盒子。`create_atoms 2 single 5 5 5`表示在坐标为（5a, 5a, 5a）的地方添加一个类型为2的原子。
- `group`将选中的原子定义为一个组，用于后续建模模拟，例如：`group 1 type 1`表示将所有类型为1的原子设置为1组；`group 2 region 2`表示将处于区域2中的所有原子设置为2组；`group 3 union 1 2`表示将1，2组合并为3组。
- `delete_atoms`表示删除不需要的原子，例如：`delete_atoms group 1`表示将1组的原子删掉；`delete_atoms region 2`
表示将区域2中的原子删掉。
### 定义原子间相互作用势
告诉LAMMPS如何去计算原子间的相互作用力,Lammps的in文件里面定义原子间相互作用势涉及到两个命令：
- `pair_style`表示告诉LAMMPS相互作用势的类型
- `pair_coeff`给出势函数中的参数或者数值列表
```
pair_style              meam
pair_coeff              * * CoNiCrFeMn.meam Fe Mn
```
\* \* 表示考虑任意的两个原子间的相互作用，Fe表示元素类型，CoNiCrFeMn.meam为势文件，对于含多元的势文件，如果只用到其中一部分元素，则其它的元素就不需要写出来，例如只用到Fe、Mn，则Co、Cr、Ni就不必写，而且Fe，Mn的书写顺序要与定义的原子类型序号对应。
### 定义原子/体系某些信息的计算(ke/pe...)
`compute`命令可以帮助计算所想得出的原子/体系信息。  
例如：`compute 1 all pe/atom`计算每个原子的势能，`compute 2 all ke/atom`计算每个原子的动能，`compute 3 all coord/atom cutoff 3.0`计算截断半径3.0内的原子近邻数，`compute 4 allporperty/atom fx fy fz`计算每个原子在x、y、z方向上受到的力。  
以Fe简单弛豫的动态模拟为例：  
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
compute                 4 allporperty/atom fx fy fz
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
`dump 1 all custom 1000 dump.xyz id x y z c_1 c_2 c_3`  
c为compute缩写，1为compute的id。若写为`compute x all pe/atom`则输出写为`c_x`。
### 定义输出原子(坐标)/体系(热力学信息)
`thermo`命令表示每运行多少步输出一次热力学信息。例如：`thermo 100`，表示每运行100步输出一次热力学信息。  
`thermo_style`命令表示输出所需要的热力学信息。例如：`thermo_style custom step temp ke pe`，其中custom表示所需的热力学信息自定义，step表示输出运行的步数是多少，temp表示体系的温度，ke表示体系的动能，pe表示体系的势能，需要输出的热力学信息都可以写在custom后。  
`dump`命令表示输出所需要的数据。例如：`dump 1 all custom 1000 dump.xyz id type x y z c_1 c_2 c_3`，其中all表示对于所有原子，也可只输出一部分原子的表示方式，与之前group命令相关，1000表示每运行1000步输出一次dump.xyz为输出文件名的文件，而其中的内容有，id：原子的序号，type：原子的类型，x,y,z：原子的坐标，c_1,c_2,c_3：compute计算数据。
### 模拟环境设定并运行
- `velocity`命令表示模拟初期给与原子初速度。例如：`velocity all create 273.15 4928459 dist gaussian`，all表示赋予所有原子，273.15为273.15K，4928459为随机正整数，dist gaussian为原子速度分布满足高斯分布。  
- `timestep`命令表示设置模拟步长。例如：`timestep 0.005`与`units`命令有关。  
- `fix`命令表示设置模拟系综。例如：`fix 1 all nvt temp 273.15 273.15 0.5`控制体系在温度为300K左右t的nvt环境下模拟，0.5为时间步的100倍。`fix 2 bottom setforce 0.0 0.0 0.0`设定bottom区域(region命令)的原子的受力为0(固定原子用)。  
- `run`命令表示设置运行步数。例如：`run 100000`表示运行100000步。