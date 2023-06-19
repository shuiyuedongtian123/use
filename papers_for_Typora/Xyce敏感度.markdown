# 敏感性分析 Xyce

![image-20230615222809182](/Users/linjing/Documents/papers_with_github/papers_for_Typora/assets/image-20230615222809182.png)

![image-20230615222841528](/Users/linjing/Documents/papers_with_github/papers_for_Typora/assets/image-20230615222841528.png)

![image-20230615222908281](/Users/linjing/Documents/papers_with_github/papers_for_Typora/assets/image-20230615222908281.png)

## DC (Steady-State) Analysis

![image-20230615223205530](/Users/linjing/Documents/papers_with_github/papers_for_Typora/assets/image-20230615223205530.png)

![image-20230615225043516](/Users/linjing/Documents/papers_with_github/papers_for_Typora/assets/image-20230615225043516.png)

![image-20230615231520685](/Users/linjing/Documents/papers_with_github/papers_for_Typora/assets/image-20230615231520685.png)

![image-20230615231530593](/Users/linjing/Documents/papers_with_github/papers_for_Typora/assets/image-20230615231530593.png)

## 项目分析准备：

<img src="./assets/image-20230616021735905.png" alt="image-20230616021735905" style="zoom:50%;" />

![image-20230616021134712](./assets/image-20230616021134712.png)

- `.tran` 命令定义了瞬态分析的时间步长和仿真时间等参数。其中，`1.0e-6` 表示时间步长为 1 微秒，`0.5e-2` 表示仿真时间为 50 毫秒。
- `.options` 命令设置了求解器的选项参数，包括时间积分方法、相对误差容限和绝对误差容限等。
- `.print tran` 命令用于将仿真结果输出到文件中，输出的内容包括节点 A 和节点 B 的电压值。
- `.print TRANADJOINT` 命令用于将瞬态伴随灵敏度分析结果以 Tecplot 格式输出到文件中。
- `.SENS` 命令用于启用瞬态伴随灵敏度分析，并设置敏感性分析所需的参数。其中，`objfunc=V(B)` 表示目标函数为节点 B 的电势，`param=R1:R,C1:C` 表示需要对电阻 R1 的阻值和电容 C1 的容值进行灵敏度分析。
- `.options SENSITIVITY` 命令用于设置灵敏度分析的选项参数，包括直接法和伴随法等方法。在这个网表中，我们启用了伴随法计算灵敏度。
- `.end` 命令用于结束分析。

通过这个网表，我们可以对电路的瞬态响应进行仿真，并计算出电阻 R1 和电容 C1 对节点 B 电势的灵敏度。

## 如果修改上面的网表的目标为SNDR:

将上面的网表中的敏感性分析目标函数改成 SNDR，即

```python
.SENS objfunc=snrd param=R1:R,C1:C .options SENSITIVITY direct=0 adjoint=1
```

在编写中定义函数：

![image-20230616024324490](./assets/image-20230616024324490.png)

另一个参考：

```pytho
* Example circuit for SNDR sensitivity analysis
* V1 is the input signal, R1 is the load resistance
* C1 and C2 are coupling capacitors, D1 is a diode

.subckt diode anode cathode
D1 anode cathode D
.ends diode

* Input signal
V1 in 0 PWL(0 0 1n 1 2n 0)

* Load resistance
R1 out 0 1k

* Coupling capacitors
C1 in 1 1n
C2 1 out 1n

* Diode
X1 in 1 diode
.model D D(IS=1e-14)

* Transient analysis
.tran 0 10u 0 1n

* SNDR measurement
.measure SNDR TRIG V(out) VAL=0.5 RISE=1 FALL=1
.measure THD TRIG V(out) VAL=0.5 RISE=1 FALL=1 HARM=2
.measure SNR PARAM '20*log10(V(out)/sqrt((SNDR-THD)/2))'

* Sensitivity analysis
.sens adjoint
.meas S_V1 SNDR SENS V(V1)
.meas S_C1 SNDR SENS V(C1)
.meas S_C2 SNDR SENS V(C2)
.meas S_D1 SNDR SENS V(D1)

* Output
.print TRAN V(out) I(R1) SNDR THD SNR S_V1 S_C1 S_C2 S_D1

```

```
在上面的网表中，使用了 `.sens adjoint` 语句来启用瞬态伴随敏感度分析。在 `.meas` 语句中，我们使用了 `SNDR SENS` 参数来计算SNDR的瞬态伴随敏感度。我们还计算了输入信号和三个电容的瞬态伴随敏感度，以了解它们对SNDR的影响。

在 `.print` 语句中，我们输出了瞬态响应、电阻电流、SNDR、THD、SNR和瞬态伴随敏感度。根据你的需要，可以更改输出量。
```

--------------------------------------------------------------------------------------------------------------

--------------------------------------------------------------------------------------------------------------

## 叶老师：

Step1:对波形分段，每段用3次样本插值，可以参考的例子：

![image-20230616032332281](./assets/image-20230616032332281.png)

先得到对应的响应函数的输出：

```
.PRINT TRAN SDNR
```

再考虑对这些输出进行插值计算：

```
* Input signal
V1 in 0 PWL(0 0 1n 1 2n 0)

* Load resistance
R1 out 0 1k

* Segmenting the target function
.param x1=0
.param y1=0
.param x2=1n
.param y2=1
.param x3=2n
.param y3=0

* Cubic spline interpolation
.param h1=x2-x1
.param h2=x3-x2
.param a=y1
.param b=(y2-y1)/h1 - h1/3*(2*y1+y2)
.param c=(y3-y2)/h2 - (y2-y1)/h1 + h1/3*(y1+2*y2)
.param d=(y3-y2)/h2 - h2/3*(2*y3+y2)

* Saving the spline parameters
.save SplineParams a b c d

* Transient analysis
.tran 0 10u 0 1n

* Output
.print TRAN V(out) I(R1)
```

```
在这个示例电路中，我们首先定义了目标函数的三个分段点和对应的值。然后，我们计算了每个分段之间的距离（`h1` 和 `h2`），并使用三次样条插值计算了近似波形的参数（`a`，`b`，`c`，`d`）。我们使用 `.save` 语句将这些参数保存到文件中，以便以后使用。

在 `.print` 语句中，我们输出了瞬态响应和电阻电流。根据你的需要，可以更改输出量。

请注意，这只是一个简单的示例，用于演示如何在Xyce中对目标函数进行分段和三次样条插值。在实际应用中，你需要更改参数和模型以适应你的应用。

```

未来：分成多段，根据delat自己设置

另一个更合适的：

```
* 定义节点信息
.param x1 = 0
.param x2 = 1
.param x3 = 2
.param x4 = 3
.param y1 = 0
.param y2 = 1
.param y3 = -1
.param y4 = 2
.param yp1 = 0
.param yp4 = 0

* 将节点信息封装到SplineParams对象中，无导数None
.param spline_params = new SplineParams(x=[x1,x2,x3,x4],y=[y1,y2,y3,y4],yp=[yp1,None,None,yp4])
* 计算插值多项式系数
.param a = cubicSpline(1.5,x1,x4,y1,y4,spline_params)
即考虑在1.5处的插值
* 注意用save 保存参数
```

Step2: 为了求得伴随敏感度要开始一系列的计算了：

主要的方程：

![image-20230616040522223](./assets/image-20230616040522223.png)

![image-20230616040545919](./assets/image-20230616040545919.png)

最终方程为上式子：

通过![image-20230616040918408](./assets/image-20230616040918408.png)

![image-20230616040836374](./assets/image-20230616040836374.png)









分析以上：先要得到 alpha

第一种可能的编写：

任意时刻：
	将第一步的参数带入.param a = cubicSpline(x,x1,x4,y1,y4,spline_params)，自己打公式，计算，利用.measure

![image-20230616043001509](./assets/image-20230616043001509.png)

![image-20230616043252988](./assets/image-20230616043252988.png)

另一种方式：

存在直接计算傅里叶积分的函数，应该是如下（还需再确认），得到所求项。

<img src="./assets/截屏2023-06-16 04.39.53.png" alt="截屏2023-06-16 04.39.53" style="zoom: 67%;" />













