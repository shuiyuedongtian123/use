# Xyce指导手册

## 1.介绍

XYCE 包括几个独特的功能。一个重要的驱动因素是需要在晶体管级别上模拟非常大规模的电路（100，000个器件或更多）。为此，已经开发了用于并行模拟大型电路的可扩展算法。此外，Xyce 还包括数值核的新方法，包括模型降阶、延续算法、时间积分、非线性和线性求解器。此外，与大多数基于 SPICE 的代码不同，Xyce 使用微分代数方程 （DAE） 公式，可以更好地将设备模型包与求解器算法隔离开来。

```mermaid
graph LR
A[Xyce] --> B[支持大型平行计算]
A[Xyce] --> C[使用DAE方程组求解器, 求解器具有高效, 准确, 稳定等优点]
A[Xyce] --> D[可以支持多种不同类型的电路元件模型,且自定义设备模型]
```

## 2. Xyce安装与使用

Xyce是一个没有GUI的界面，在里面是运行命令语句，生成文件。安装链接https://xyce.sandia.gov/downloads/, Mac和linux很难装，因为要下载Xyce配套的版本包，可以选择window10的软件安装包，直接点击安装就可以了，很方便。

<img src="./assets/image-20230615233736840.png" alt="image-20230615233736840" style="zoom:50%;" />

![image-20230614105318496](/Users/linjing/Documents/papers_with_github/papers_for_Typora/assets/image-20230614105318496.png)

## 3.XYCE 仿真示例

本章提供了几个简单的 Xyce 用法示例。为每种可用的分析类型提供了一个示例电路。

```Mermaid
graph LR
A[Xyce 仅支持通过网表编辑创建电路] --> B[Xyce 网表.cir] --> C[生成输出文件]

```

- Xyce网表格式：

<img src="./assets/image-20230615233919875.png" alt="image-20230615233919875" style="zoom:50%;" />

<img src="./assets/image-20230615233954163.png" alt="image-20230615233954163" style="zoom:50%;" />

此时只是给出了电路对应的网表结果，还不能运行Xyce，还没有加入分析语句。

- 加入分析语句，查看**DC Sweep Analysis**

<img src="./assets/image-20230615234026865.png" alt="image-20230615234026865" style="zoom:50%;" />

​		通过以一伏步长将直流电压源 （Vin） 从 -10 伏扫描到 15 伏来获得限速器电路的直流响应，输出节点2,3,4处的电压。

前期给出一个小例子：编写**a.cir**， 运行**Xyce a.cir**, 本地生成**a.cir.prn**:

<img src="./assets/image-20230616000204231.png" alt="image-20230616000204231" style="zoom: 33%;" />/><img src="./assets/image-20230616000228514.png" alt="image-20230616000228514" style="zoom: 33%;" />



- 加入分析语句，查看**Transient Analysis**

  在这个 Xyce 网表文件中，V1 是一个正弦电压源，其幅值为 1V，频率为 1kHz。R1 和 C1 分别是一个电阻和电容。

  .tran 1us 100us 命令定义了一个瞬态分析，在 0 到 100us 的时间范围内，以步长为 1us 对电路进行分析，查看节点2的电压变化。

  

## 4.网表基础

包含有关网表语法和用法的介绍性材料，Xyce支持的网表格式，包括SPICE、Spectre、HSPICE和Verilog-A等。

## 5.使用子电路和模型

![image-20230615234419397](./assets/image-20230615234419397.png)

## 6.模拟行为建模

通过创建自定义的模型来描述电路的行为，可以设计一些非线性的元件，从而更好地模拟和预测电路的性能。

Ex: 一个SPICE电路模拟器中的电路描述语句，用来描述一个由一个叫做Bcrtl的器件和一个叫做OUTA的端口组成的电路。该电路的功能是将输入端口IN的电压与3.5V进行比较，如果大于3.5V，则输出5V，否则输出0V。其中，V={}表示的是一个表达式，用来计算输出值，这里使用了IF函数来进行条件判断。

![image-20230615234821487](./assets/image-20230615234821487.png)

EX: 一个简单的二极管整流电路，其中VIN是一个5V的直流电压源，R1是一个2K的电阻，D1是一个二极管，DMOD是一个模拟器件，用于模拟二极管的非线性特性。

![image-20230615234927844](./assets/image-20230615234927844.png)

## 7.分析类型

### 7.1 介绍

### 7.2 DC分析

| 常见的DC命令：                                               |
| ------------------------------------------------------------ |
| .DC V1 7m 5m -1m：扫描V1电源的值，并从7m逐渐降低到5m，再降低到-1m，计算每个值下电路的直流响应。 |
| .DC I1 5u 10u 1u：扫描I1电流源的值，并从5u逐渐增加到10u，每次增加1u，计算每个值下电路的直流响应。 |
| .DC M1:L 7u 5u -1u：扫描M1的L参数，并从7u逐渐降低到5u，再降低到-1u，计算每个值下电路的直流响应。 |
| .DC OCT V0 0.125 64 2：扫描V0电压源的值，并在对数尺度上从0.125逐渐增加到64，共扫描64个值，每个值之间的比率为2，计算每个值下电路的直流响应。 |
| .DC DEC R1 100 10000 3：扫描R1电阻的值，并在对数尺度上从100逐渐增加到10000，共扫描3个值，计算每个值下电路的直流响应。 |
| .DC TEMP LIST 10.0 15.0 18.0 27.0 33.0：扫描电路的温度值，并分别计算在10.0、15.0、18.0、27.0和33.0度下电路的直流响应。 |
| .DC data=table：将直流分析结果以表格形式输出：xyce运行有误   |



### 7.3 瞬态分析

![image-20230615235129736](./assets/image-20230615235129736.png)

- 独立瞬态源

  <img src="./assets/image-20230615235212910.png" alt="image-20230615235212910" style="zoom:50%;" />

  <img src="./assets/image-20230615235225019.png" alt="image-20230615235225019" style="zoom: 67%;" />

  - 步长

    ![image-20230616000648352](./assets/image-20230616000648352.png)

    - 时间积分方法

    ![image-20230616000904830](./assets/image-20230616000904830.png)

    ![image-20230616000932329](./assets/image-20230616000932329.png)

    注意：Xyce也可以根据电路中的基本元件和连接关系**自动生成微分方程**，并用数值方法求解微分方程，得到电路的响应。在Xyce中，用户只需要在输入文件中描述电路的拓扑结构和元件参数，然后通过Xyce进行仿真计算即可。Xyce支持多种数值方法对微分方程进行离散化处理，包括后向欧拉法、梯形法、龙格-库塔法等。用户可以根据需要选择适合的数值方法进行仿真计算。

    - 错误控制等

      Xyce 中有两种基本的时间步长错误控制方法 — 基于本地截断错误 （LTE） 和非基于 LTE 的方法。主要介绍.OPTIONS命令可以用来设置各种仿真选项，包括时间步长、误差限制、迭代步长等，控制仿真的精度等，使得仿真结果更加准确。

### 7.4 STEP 参数分析

.STEP命令对电路的所有分析执行参数扫描。当 .调用STEP命令，典型的分析，如.直流，交流电和 .对阶梯参数的每个值执行 TRAN。

![image-20230616001153609](./assets/image-20230616001153609.png)

<img src="./assets/image-20230616001310602.png" alt="image-20230616001310602" style="zoom:50%;" />

运行上面的代码，有2个变量，分别是电路中的电阻R4和输入信号的电压值VIN。在网表中，R4是通过.STEP命令实现的参数扫描，而VIN则是通过在输入电压源VIN的定义中设置不同的电压值来实现的。在仿真过程中，Xyce会根据.STEP命令的设置和输入电压源VIN的电压值，对电路进行多次仿真，得到的结果是不同电阻下，节点2,3,4随着时间变化的电压。

### 7.5 Sampling分析

在随机采样分析中，Xyce会在指定的参数范围内随机抽样多个点，并对每个点进行电路仿真，以获取电路性能的统计信息。这些统计信息可以帮助电路设计师评估电路的鲁棒性，即在参数变化或噪声影响下，电路是否能够保持期望的性能。

![image-20230616001833969](./assets/image-20230616001833969.png)

- param=M1:L, M1:W：指定要进行采样的参数为M1:L和M1:W，即分别为场效应管M1的长度和宽度。

- type=uniform,uniform：指定采样类型为均匀分布，即每个参数的取值在指定的范围内等概率随机分布。

- lower_bounds=5u,5u：指定参数的下限范围为5u，即5微米。

- upper_bounds=7u,7u：指定参数的上限范围为7u，即7微米。

  ![image-20230616001911452](./assets/image-20230616001911452.png)

  - samples numsamples=30：指定要进行的样本数为30，即进行30次仿真。

  - covmatrix=1e6,1.0e-3,1.0e-3,4e-14：指定协方差矩阵，其中第一个元素为1e6，表示R1:R和C1:C两个参数的方差；第二个和第三个元素为1.0e-3，表示R1:R和C1:C两个参数的相关系数；第四个元素为4e-14，表示maxSine量测的方差。

  - OUTPUTS=R1:R,C1:C：指定要输出的节点为R1:R和C1:C。

  - MEASURES=maxSine：指定测量类型为maxSine，即以最大正弦输出电压作为测量结果。

  - SAMPLE_TYPE=LHS：指定采样类型为LHS，即拉丁超立方采样。

  - SEED=743190581：指定随机数种子为743190581，用于生成随机数序列。

    ![image-20230616002003751](./assets/image-20230616002003751.png)

    这个电路是一个简单的电压分压器，其中包括一个电压源VS1、两个电阻R1和R2，以及一个地电位。

    具体来说，这个电路的模拟和分析过程如下：

    - 定义了电路的拓扑结构，包括两个电阻R1和R2、一个电压源VS1、以及一个地电位。
    - 定义了一个0.1ms到1ms的瞬态分析，用来模拟电路的行为。
    - 使用.meas命令来测量V(1)的最大正弦输出电压，将其命名为maxSine。
    - 使用.SAMPLING命令来进行蒙特卡洛采样分析，其中我们对电路中的R1参数进行采样，采样类型为正态分布，均值为1K，标准差为0.1K。
    - 使用.options命令来设置采样参数，其中我们要进行1000次采样（numsamples=1000），输出的结果为R1参数的值（OUTPUTS={R1:R}），测量类型为maxSine，采样类型为MC（SAMPLE_TYPE=MC），并将结果输出到标准输出中（stdoutput=true）。

### 7.6 多项式混沌分析

Xyce支持多种风格的多项式混沌展开（PCE）方法，这些方法是一种随机展开方法，通过多项式展开来近似模拟响应对不确定模型参数的函数依赖性[17，18]。在Xyce中，Stokhos库[19]已被用于使用Wiener-Askey方案实现广义PCE方法。

为了使用PCE通过模型传播输入不确定性，Xyce执行以下步骤：（1）将输入不确定性转换为一组不相关的随机变量，（2）选择诸如Hermite多项式之类的基，以及（3）确定函数近似的参数。响应 O 的一般 PCE 具有以下形式

<img src="./assets/image-20230616002115484.png" alt="image-20230616002115484" style="zoom:67%;" />

!![image-20230616002133612](./assets/image-20230616002133612.png)

- Regression-based Polynomial Chaos Expansion是通过回归分析的方法，将系统的响应函数表示为多项式函数的线性组合。通常使用最小二乘法来进行回归分析，得到每个多项式系数的值。最后，通过计算多项式函数的值，可以得到系统响应函数在随机变量不同取值下的表现，进而对随机变量进行建模和分析。

  <img src="./assets/image-20230616002226982.png" alt="image-20230616002226982" style="zoom:50%;" />

- Non-Intrusive Spectral Projection (NISP)

  <img src="./assets/image-20230616002242241.png" alt="image-20230616002242241" style="zoom:50%;" />

  ...

- 代码解析Xyce

  ![image-20230616003101603](./assets/image-20230616003101603.png)

  .param testNorm={aunif(2k,1k)}：定义了一个随机变量testNorm，其取值范围在(2k, 1k)之间，即服从均匀分布。

  .param R1value={testNorm*2.0}：定义了另一个参数R1value，它是testNorm的两倍。这样，R1value也是一个服从均匀分布的随机变量。

  .SAMPLING useExpr=true：启用Regression PCE分析。

  .options SAMPLES ：设置采样点数目为12，使用拉丁超立方采样方法（lhs）进行采样，使用五次多项式函数进行建模，分析的输出为节点1的电压v(1)。设置resample=true，表示需要重新采样以获得更准确的结果。

  .end：结束XYCE输入文件。

  <img src="/Users/linjing/Documents/papers_with_github/papers_for_Typora/assets/image-20230615154251663.png" alt="image-20230615154251663" style="zoom:50%;" />

### 7.7 Harmonic Balance 分析

### 7.8 AC分析

<img src="./assets/image-20230616002529407.png" alt="image-20230616002529407" style="zoom: 50%;" />

- 第一行设置了两个全局参数mag和phase，分别为1和0.1。

- 第二行定义了一个交流电源Isrc，其幅值为mag，相位为phase。

- 第三行定义了一个电阻R1和一个电容C1，分别连接到Isrc的正极和负极。

- 第四行定义了一个数据表格，其中包括mag、phase、频率和电阻r1四列数据。

- 第六行使用.AC命令进行AC分析，并使用data=table选项指定数据表格作为输入。

- 第八行使用.print命令输出AC分析的结果，包括mag、phase、Isrc的幅值和相位、电阻r1和节点1的电压。

  

### 7.9噪声分析

### 7.10 敏感性分析

.SENS 命令指示 Xyce 计算输出表达式相对于指定电路参数列表的灵敏度。此功能适用于稳定状态 （.DC）、瞬态 （.TRAN）和小信号（.AC）分析。

![image-20230616003309456](./assets/image-20230616003309456.png)

- objfunc参数指定了需要计算灵敏度的输出量表达式，可以是一个或多个输出量表达式。例如，objfunc=v(2)表示计算节点2的电压对电路参数的灵敏度。
- param参数指定了需要计算灵敏度的电路参数，可以是一个或多个电路参数。例如，param=r1表示计算电阻r1对输出量的灵敏度。
- options SENSITIVITY选项指定了进行灵敏度分析。
- direct选项指定了是否使用直接法进行灵敏度计算，1表示使用，0表示不使用。直接法计算速度较慢，但对于复杂电路和大量参数的情况，可以获得更准确的结果。
- adjoint选项指定了是否使用伴随法进行灵敏度计算，1表示使用，0表示不使用。伴随法计算速度较快，但对于某些电路和参数组合，可能无法获得准确的结果。

运行这个命令之后，Xyce将计算每个指定的电路参数对每个指定的输出量的灵敏度，并将结果输出到文件中。输出文件中将包含每个电路参数和输出量的灵敏度值，可以使用Xyce提供的工具或其他软件进行可视化和分析。

#### 7.10.1. Steady-State (DC) sensitivities

稳态（DC）指的是在电路中所有电荷和电流都达到稳定状态，不再随时间变化的情况。

<img src="/Users/linjing/Documents/papers_with_github/papers_for_Typora/assets/image-20230615161319853.png" alt="image-20230615161319853" style="zoom: 33%;" />

<img src="/Users/linjing/Documents/papers_with_github/papers_for_Typora/assets/image-20230615161417322.png" alt="image-20230615161417322" style="zoom:33%;" />

#### 7.10.2. Transient sensitivities

可以计算灵敏度以进行瞬态分析。与稳态 （DC） 分析一样，支持直接和伴随灵敏度。

##### 7.10.2.1. Transient Direct Sensitivities

<img src="/Users/linjing/Documents/papers_with_github/papers_for_Typora/assets/image-20230615163626042.png" alt="image-20230615163626042" style="zoom:50%;" />

.print tran命令用于输出电路的传统文件，其中包含电容上的电压随时间变化的数据。

.print sens命令用于输出灵敏度文件，其中包含电容上的电压随时间变化的数据，以及每个参数对电容电压的灵敏度随时间变化的数据。其中，analytic solution表示理论解，dV(1)/dR和dV(1)/dC分别表示电容电压对电阻R1和电容C1的灵敏度。

##### 7.10.2.2. Transient Adjoint Sensitivities 

瞬态伴随灵敏度对于大量参数以及目标函数数量适中时是一个不错的选择。对于瞬态计算，每个时间点都被视为一个单独的目标函数，因此当感兴趣的灵敏度仅涉及一个或几个时间点时，最好使用伴随。

<img src="/Users/linjing/Documents/papers_with_github/papers_for_Typora/assets/image-20230615165416282.png" alt="image-20230615165416282" style="zoom:50%;" />

.options timeint method=gear reltol=1.0e-6 abstol=1e-6：

这个命令用于设置瞬态分析的选项。其中，timeint表示时间积分选项，method=gear表示使用Gear方法进行时间积分，reltol=1.0e-6和abstol=1e-6分别表示相对误差和绝对误差的容忍度。这个命令需要在.tran命令之前使用，用于设置瞬态分析的选项。这些选项将直接影响仿真结果的准确性和速度，因此需要根据具体情况进行调整。

<img src="/Users/linjing/Documents/papers_with_github/papers_for_Typora/assets/image-20230615200337751.png" alt="image-20230615200337751" style="zoom:50%;" />

在这个例子中，`.SENS` 语句中的 `adjointTimePoints=0.1e-2, 0.2e-2, 0.3e-2, 0.4e-2` 指定了四个时间节点，分别为 0.1e-2 s、0.2e-2 s、0.3e-2 s 和 0.4e-2 s。这意味着 OpenFOAM 将在这些时间节点上计算电路的伴随状态和灵敏度，从而得到更准确的结果。

### need to know

瞬态伴随敏感度分析和瞬态直接敏感度计算都是用于计算电路在瞬态响应过程中各个参数对目标函数的影响程度的方法，但它们的计算方式不同。

在瞬态伴随敏感度分析中，需要对电路进行时间域分析，即瞬态分析，来获得电路在时间上的响应。在进行瞬态分析时，需要对时间进行离散化，即将时间分成若干个小时间步长，并在每个时间步长内对电路进行分析，并计算目标函数对电路中各个参数的偏导数，从而获得电路在瞬态响应过程中各个参数对目标函数的影响程度。

而在瞬态直接敏感度计算中，不需要进行时间域分析，也不需要时间积分。它是基于电路中各个元件的导纳矩阵和电荷、电流、电压之间的关系，通过解析求导的方法，直接计算电路中各个参数对目标函数的影响程度。

因此，瞬态伴随敏感度需要时间积分，而瞬态直接敏感度计算不用。



#### 7.10.3. AC Sensitivities 

敏感度也可以计算小信号交流分析的灵敏度。与稳态(DC)和瞬态分析一样,支持直接和伴随灵敏度。

<img src="/Users/linjing/Documents/papers_with_github/papers_for_Typora/assets/image-20230615203040640.png" alt="image-20230615203040640" style="zoom:50%;" />

<img src="/Users/linjing/Documents/papers_with_github/papers_for_Typora/assets/image-20230615203148178.png" alt="image-20230615203148178" style="zoom:50%;" />

<img src="/Users/linjing/Documents/papers_with_github/papers_for_Typora/assets/image-20230615204138885.png" alt="image-20230615204138885" style="zoom: 50%;" />



#### 7.10.4. Notes about .SENS accuracy and formulation

##### 7.10.4.1. Direct Sensivity 

##### 7.10.4.2. Adjoint Sensivity 

##### 7.10.4.3. Analytical vs Numerical derivatives 

##### 7.10.4.4. Time integration error 

##### 7.10.4.5. AC Sensitivities 

#### 7.10.5. Output .

### 7.11 S-参数分

![image-20230616021218122](./assets/image-20230616021218122.png)
