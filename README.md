### ——不尽长江滚滚流片

记录学习流片的过程

# 1. [Ansys lumerical FDTD 入门](https://courses.ansys.com/index.php/learning-track/ansys-lumerical-fdtd/)

## 1.1. Intro

FDTD is finite difference time domain 时域有限差分算法

使用 lumerical FDTD 的优势有：

1. 可以模拟任意的几何形状（可以通过FDTD方法求解任意的 Maxwell's equations）。
2. 在波长尺度的求解较为精确。
3. 受益于时域分析的性质，单次模拟就可以给出宽频带的结果。

使用 lumerical FDTD 的 workflow：

1. setup simulation

   1.1. define meterials (use default database or import manually)

   1.2. add structures

   1.3. add simulation region and mesh

   1.4. add sources and monitors

2. before simulation

   2.1. check meterials properties

   2.2. check memory requirements

3. run simulation

4. analyse

   4.1. collect monitor data

   4.2. plot

   4.3. export data

   4.4. post-process

5. iteration design

6. convergence test

    e.g. increase mesh accuracy

## 1.2. Setup Simulation

### 1.2.1. Set Boundary (simulation region)

#### 1.2.1.1. General settings

设置2D/3D，背景折射率，仿真时间(注：run过程中，如果process一直跑到100%才结束，说明仿真时间太短了，需修改)；FDTD Solutions默认设置的模拟时间是1000fs(必须保证有足够时间使结果收敛)，而模拟会在场衰减到小于用户定义的电场强度时(默认设置是1E-5)自动结束；仿真时间一般至少是光经过高折射率材料的仿真区域所用时间的两倍(choose a simulation time that is at least as twice as long as the time it would take light to travel across the entire simulation area in the material with the highest index of refraction), eg: t＞2nL/c
参考：<https://blog.csdn.net/HECHUANWANG/article/details/95976174>

#### 1.2.1.2. Mesh settings

设置网格类型、网格精度、亚网格处理(材料界面处理)，推荐使用默认的自适应网格，精度使用默认2(初始使用，如有需要再提高精度)，材料界面处理选择默认0(注意：conformal variant 1通常只适合在低折射率对比的情况下使用，并不一定适合所有的金属材料)；FDTD Solutions还提供mesh，在仿真区域内设置，即局部网格设置。一般情况下，这些局部网格（当然如果必要也可以将整个仿真区覆盖，不过一般不建议这样做，因为有其它的方法可以设置）都是为了进一步将某局部网格细化，例如分辨几何细节，高掺杂区，感兴趣所的量局部变化梯度很大等。也有个别时候反而是将网格加粗，这种粗化网格的使用需要慎重，除非你清楚知道原因和后果。mesh的典型应用是dx=dy＜lamda/(10n)，n是材料中的最高折射率

#### 1.2.1.3. Boundary conditions

1. **PML** 反射率为0
   - perfectly match layer 完美匹配层。边界相当于黑体。电磁场传输到边界后被完全吸收。
   - 具体而言，这种边界通常用在仿真区边缘，能与周围材料阻抗匹配，最大限度地减少反射，同时完全吸收仿真区内的光场。理想状况下，PML边界产生零反射，但是实际上由于基础PML方程的离散性，总会存在一定程度的反射，同时，用有限差分算法中对于PML方程的离散近似，也会带来一定程度的数值不稳定性。
   - 参考：<https://zhuanlan.zhihu.com/p/560832768>
   - 设置时要确保PML边界**离开物体至少半个波长**左右(有必要一个波长也可以)，因为PML不只会吸收入射光源，也会吸收倏逝场(evanescent field)
   - **PML settings**
     - details in: <https://optics.ansys.com/hc/en-us/articles/360034382674>
     - PML profiles:
       - steep angles 可以用来最大限度的吸收具有一定角度的光
  
2. **Metal** 完全反射
   - Metal边界条件模拟了一个全反射的边界，即要求场在仿真边缘处趋近于0.
   - 如果在仿真边缘处的模场非常小 ≈ 0 那么边界条件的选择将变得不再那么重要。在这种情况下，我们可以选择最高效的数值边界，而不是最符合物理情况的边界。
   - Metal边界相对于PML边界具有以下优势：
     1. Metal边界条件的仿真速度比PML边界条件更快。Metal边界条件模拟了一个全反射的边界，类似于一个理想的金属表面。由于Metal边界条件不引入额外的计算开销，因此在仿真中使用Metal边界条件可以提高计算速度。
     2. PML边界条件可能引入非物理的“PML”模式。 PML边界条件是一种用于减少波在计算区域边界处反射的方法。它通过引入吸收层来模拟吸收边界，以吸收边界处的波能量。然而，PML边界条件可能会引入一些非物理的模式，这些模式并不能真实地反映系统的行为。
     3. PML边界条件可能在本来是无损耗系统中引入非常小的增益或损耗。 这是因为PML边界条件的引入会改变波的传播特性，从而导致能量的微小损耗或增益。
      - 只要模场在仿真区域的边缘非常小，Metal边界条件通常是最好的选择。它们与PML具有相同的结果，但是仿真不会受到上述问题的困扰。对于扩展到仿真区域边缘的模式，需要使用PML边界条件。这最有可能发生在模式不完全限制在结构内部并具有一定辐射损耗的情况下。例如，在研究弯曲波导时，应该使用PML，因为弯曲可能导致辐射损耗。
      - 参考：<https://zhuanlan.zhihu.com/p/647137255>

3. **Periodic**
   - 只使用于周期结构，平面波入射情况下(以免光源被切断而产生衍射边缘效应)。光源正入射，如果光源与坐标轴有一定角度，必须在有角度的平面使用Bloch边界条件。Periodic是Bloch的特殊情况.

4. **Bloch**
   - 是Periodic的一般形式；它是一种普遍的边界条件，由于数学上要求它只能针对指定的波长有指定的入射角，其它波长的实际入射角将不同于指定的那个入射角，因此一般情况下，它适合单波长计算。

5. **Symmetric/anti-symmetric**
   - 对称/反对称边界条件。要求：结构对称性，光源的偏振也要对称。

6. **PMC**
   - PMC boundary conditions are the magnetic equivalent of the PEC boundaries. The component of the magnetic field parallel to a PMC boundary is zero and the component of the electric field perpendicular to a PMC boundary is also zero.
   - [reference](https://optics.ansys.com/hc/en-us/articles/360045467453-Simulation-Tips-Accuracy-and-Performance-Tips-Boundary-Conditions)

### 1.2.2. Set Mesh

Mesh step need to be finer when doing convergence testing

### 1.2.3. Set Source

Can only choose pulse light, may because of time domain...

### 1.2.4. Set Monitor

It is fine for the monitors to extend outside of the simulation region.
1. Refractive index
   - Measures refractive index and surface conductivity over space.
   - Used to check mesh order and meshed structure cross-section.
2. Field time
   - Measures E, H fields over time by default.
   - Returns E, H, the spectrum of E and H.
   - Record and return P if specified.
3. Movie
   - Generates an mp4 movie file in the current working directory showing the specified field component vs. time over the monitor area.
4. Frequency-domain monitors (power and profile)
   - Shows frequency-domain E, H data by default.
   - Returns E, H and T.
   - T gives net transmission and is only available for linear or 2D monitors in 2D simulations or 2D monitors in 3D simulations.
   - Can return P if specified.
   - Power uses the nearest mesh cell interpolation, whereas profile uses the specified position.
5. Mode expansion
   - Performs overlap calculations between calculated modes and recorded fields from a frequency domain monitor to get the power traveling in selected modes of a waveguide or fiber.
   - Can be used to extract S-parameters of a device, although using Port objects is simpler.
6. ports
   - Added using the Ports button in the top toolbar.
   - Acts as a mode source as well as a power monitor and mode expansion monitor.
   - Returns the same results as mode source, power monitor, and mode expansion monitor in addition to S result which gives the S-parameter.
   - The port group, which is a child of the FDTD simulation region, contains all port objects and sets the active port and mode to use as the source in the simulation.
   - Ports can be used in conjunction with the S-parameter sweep tool to extract the full S-parameters of a device and export the S-parameters to INTERCONNECT for circuit simulations.
## 1.3. Running the simulation

### 1.3.1. Check meterials properties

无论是储存在 database 里现成的数据还是我们导入的数据都是点集，而模拟需要使用连续的函数，因此需要先对数据集进行拟合得到拟合方程。正确的数据和有效的拟合是仿真准确的必要条件。因此每当更换材料或仿真波长范围都应检查拟合是否合适。

- [提升拟合效果的方法](https://optics.ansys.com/hc/en-us/articles/360034915053)：
默认情况下，采样数据材质的容差为0.1，最大系数为6。在许多情况下，这些都是合理的值。但是，在运行模拟之前检查配合始终是一个好的做法。过拟合和欠拟合都不是好的选择。
  1. Fit tolerance: 可以将该值设为0，在这种情况下程序将在 Max coefficients 的限制下尽可能找到最小的 RMS
  2. Max coefficients

### 1.3.2. Check memory requirements

If memory requirements are too high, to reduce memory requirements:

   1. use a coarser simulation mesh
   2. use symmetry in the boundary conditions if the source and structure have symmetry
   3. reduce monitor size and number of recorded frequency points

### 1.3.3. Job manager during simulation

Status: initialization -> meshing -> running -> saving


## 1.4. export data(Frequency-domain Profile and Power monitor )
参考:https://optics.ansys.com/hc/en-us/articles/360034902393-Frequency-domain-Profile-and-Power-monitor-Simulation-object

### 1.4.1. E: Electric field data as a function of position and frequency/wavelength.
 - Vactor OPeration： XYZ三个分量和Magnitude.其中Magnitude=sqrt(abs(Ex)^2+abs(Ey)^2+abs(Ez)^2)
 - Scalar OPeration:
 - Re： 是显示量的实部，一般用于复数，比如Ex，Ey和Ez等；或者透射率T（它没有虚部）;
 - -Re： 将实部取负号，这个操作一般仅适用于透射率，这是因为透射率根据颇印廷矢量与监视器法线方向点乘后积分得到的，而监视器法线按规定是沿轴正向为正，沿负向为负，因此当能流沿轴负向传播时，得到的透射率是负的，需要在前面加负号才能为正。
 - Abs：取绝对值。
 - Abs^2：取绝对值平方。
 - angle：相位


   参考https://forum.ansys.com/forums/topic/ansys-insight-%e6%9c%89%e5%85%b3visualizer%e7%9a%84%e7%9b%b8%e5%85%b3%e9%97%ae%e9%a2%98/

### 1.4.2. H. Magnetic field data as a function of position and frequency/wavelength.
### 1.4.3. P. Poynting vector as a function of position and frequency/wavelength.
### 1.4.4. T. Transmission as a function of frequency/wavelengt. Returns the amount of power transmitted through power monitors and profile monitors, normalized to the source power.(Negative values mean the power is flowing in the negative direction.)
$T(f)=\frac{\frac{1}{2}\int Re(P(f))dS }{sourcepower(f)}$ (T(f) is the normalized transmission as a function of frequency，P(f) is the Poynting vector and dS is the surface normal.)

参考:https://optics.ansys.com/hc/en-us/articles/360034405354-transmission-Script-command


