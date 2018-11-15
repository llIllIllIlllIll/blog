## Binarization of noisy gray-scale character images by thin line modeling
<script type="text/javascript" src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=default"></script>
原论文资源：[Binarization of noisy gray-scale character images by thin line modeling](https://ac.els-cdn.com/S0031320398000193/1-s2.0-S0031320398000193-main.pdf?_tid=6eb1a8a7-d174-4510-80b0-c435ae89c3fd&acdnat=1542117798_c80c35850ff58a98fce6360f9162cafa).
最近在准备Intel的一个涉及深度学习的程序设计大赛，我们小组选的课题是......手写汉字识别。然后这个汉字识别最基础的必然涉及图片的降噪、二值化处理，在网上找了蛮多相关资料。再mark一篇后续准备要看的论文：
- [ ] [A SURVEY OF METHODS AND STRATEGIES IN CHARACTER SEGMENTATION ——一篇探讨字符分割方案的文章](http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.158.220&rep=rep1&type=pdf)

看完了要记得回来✔哦hhhh。
然后是这篇论文有两个算法。现在我就看完了第一篇然后就想先记录一下，后半部分等今天上完课做完作业再看。
### ALGORITHM I
算法思想不是很复杂，但是好像有几个量你需要预先知道。先定义一下术语：
术语 | 含义
------|--------
stoke|这个指的应该就是笔画，论文是针对英文字母写的，但是中文也是一样的。
edge|他这里edge，如果我没弄错的话，指的是笔画的edge。从围观一点的角度分析好像一笔一划这样子不存在一个edge，但你如果从计算机存储的角度来看每个笔画他其实可以看成一个很大的封闭图形，他就会有edge。
intensity|指的应该是灰度图某个像素的亮度。
好了，这些白痴概念浪费了我很久时间。这篇论文开始根本没有术语介绍，当然也应该是因为我太菜了，估计了解这方面内容的人根本不需要这种术语介绍。
接下来先放一张他论文里给的algorithmI的流程图。
![流程图1](https://github.com/llIllIllIlllIll/blog/blob/master/bin_al1.png)
解释一下：
1. 输入灰度图。
2. 第二点就有点奇怪...原论文里对第一步第二步的描述是，先用Gaussian filter处理原图像I，然后找到经过处理后得到图像I~gf~。接下来的第一步是从I~gf~中导出一个图像I<sub>edge</sub>，其中这个I<sub>edge</sub>是用来标记原图中的edge的。只要是位于edge上的像素，在I<sub>edge</sub>中被标记为1，其余点都被标记为0.
这里我认为他奇怪的原因是......他没告诉我具体怎么去找那个边。但想来（？）也不难，看大家具体实现吧。
3. 突然很想吐槽这个论文图片的质量，搞得这么模糊不清。为什么不用你们自己的算法处理一下再发上来？一点说服力都没有......
4. 构造一个和原图大小一样的数组S(score array)。这个数组是用在后续步骤对每个像素的score计算上的，然后最后的最后的binarization基于这个score array的结果。
5. 这里还要引入几个值。W<sub>in</sub>和W<sub>out</sub>是两个矩形的window，一会儿要用来对于一个window内的部分数据进行处理。说实话这种window的形式让我想到了CNN里的filter......然后一个W<sub>in</sub>的宽度N<sub>in</sub>要大致等于stroke的宽度，一个W<sub>out</sub>的宽度N<sub>out</sub>要大致等于stroke的宽度*2。 然后这里搞笑的地方就是，这个算法也没告诉我怎么去确定stroke的宽度大小......
6. 引入完这个概念之后，介绍一下这两个window的用途。每一个在I<sub>edge</sub>中被标记为1的点(i,j)，都将有一个以这个点为中心的W<sub>in</sub>和一个W<sub>out</sub>。而每一个W<sub>in</sub>和W<sub>out</sub>中的数据如下：
**W<sub>in</sub>(i,j)[k,l]=I<sub>gf</sub>[i+k,j+l]**
**for -N<sub>in</sub>/2<=k,l<=N<sub>in</sub>/2**
**W</sub>out</sub>(i,j)[m,n]=I<sub>gf</sub>[i+m,j+n]**
**for -N<sub>out</sub>/2<=m,n<=N<sub>out</sub>/2**
接下来找到每一个W<sub>in</sub>(i,j)中的最大值和最小值，记为P<sub>max</sub>(i,j)和P<sub>min</sub>(i,j)。这里需要解释一下，在我的理解中，**因为我们是以一条边的边界作为每一个window的中心，并且一个W~in~的宽度和一条stroke相等，那么理想的情况下，这个W~in~中必然含有最亮和最暗的像素，分别代表stroke中的点以及字符间空白区域的点**。
再对于每一个window确定一个threshold，分界值t(i,j)=(P<sub>max</sub>(i,j)+P<sub>min</sub>(i,j))/2。现在就可以用这个分界值对于整个W<sub>out</sub>里的对象对应的来进行打分加权，具体说就是对于每一个W<sub>out</sub>中的对象对应的S中的值来进行加减：
**if(W<sub>out</sub>(i,j)[m,n]>t(i,j))
S[i+m,j+n]++;**
遍历。
7.最后最后有了一个打完分的S，就可以进行二值化分割了。论文里给出的结论是如果N<sub>out</sub>取的是stroke宽的两倍，最后的global threshold取3N<sub>out</sub>/2。
### ALGORITHM II
好了，今天下午看了一下ALGORITHM I。由于已经读过理解过AlGORITHM I，所以相对来说读这半部分的时候没有遇到读前半部分时候的那么多阻碍。
在读前面部分的时候我也提到了，上一个算法要求我们去求出图片中的stroke宽度以及edge的具体位置，但是他并没有给出一种特别好的侦测这些信息的方式。然后论文中告诉我们了，在上一个算法中，edge的侦测非常重要。
>It is evident that the performance of ALGORITHM
I depends heavily on the performance of the edge detector
used. It was observed in the experiment that thin
lines (strokes) were detected broken when some edge
pixels were missing.

然后算法II相对应的好处就是，他不需要进行edge detection。

![流程图2](https://github.com/llIllIllIlllIll/blog/blob/master/bin_al2.png)
算法II的主要方法是通过迭代被称为background equalization的过程，加大图像中前景和背景间的区别，使得binarization的过程变得非常简单。
具体步骤如下：（这个算法就涉及一些写起来比较复杂的公式，我也正好可以锻炼一下我的markdown使用技巧~）
1. 输入灰度图。
2. 这一步叫做local binarization。就是通过局部的像素值比较，来加大局部区域间像素值的分化。对于图像中的任意一个点(i,j)，都需要执行这一步。
这一步和之前的AlGORITHM I中的部分内容非常类似，所以长话短说。首先创造一个和原灰度图一样大的score array S。对于图像中的每一个像素(i,j)都加一个W<sub>in</sub>和一个W<sub>out</sub>，宽度取值仍然是取决于stroke的宽度。找到W<sub>in</sub>中的最大值和最小值P<sub>max</sub>和P<sub>min</sub>，然后下一步内容基于t(t=(P<sub>max</sub>+P<sub>min</sub>)/2)。下一步的操作是，将W<sub>out</sub>中的每一个像素值来和t进行比较：
if(I[x,y]<t)
I[x,y]+=S<sup>+</sup>;
else
I[x,y]-=S<sup>-</sup>;
这里，S<sup>+</sup>=$\frac{1}{1+exp(-\lambda(x-x_{0}))}$,S<sup>-</sup>=1-S<sup>+</sup>。
可以观察到S<sup>+</sup>代表了一个取值范围在0~1之间的S型曲线。当$\lambda$足够大的时候，S<sup>+</sup>这个函数可以起到一个thresholding function的作用。我的理解就是，这个函数的值会在X<sub>0</sub>两端形成一个非常明显的两极分化。并且，S<sup>+</sup>是与前面的t相关的，但同时也被局限于(0,1)的区间内。
当将第二步对于I中所有点执行之后，要注意如果存在小于零的S值，要把它调整至0。（应该是因为后面要取平方，如果不归零...10和-10就没有区别了）
3. 现在要根据S的结果得到一个W数组。
$W[i,j]=exp(\frac{-S^2[i,j]}{2k^2})$。(这里W是一个单调函数，关于$S[i,j]$在1~0上单调递减)
4. 引入一个新的图像，$\overline{I}$。在计算$\overline{I}$之前，还有以下几组数据需要计算,对于新的$\overline{I}$中的每一个像素(i,j)来说：
$P=\sum_{k,l=-M/2}^{M/2} W[i+k，j+l]$;
$m(i,j)=\frac{1}{P}\sum_{k,l=-M/2}^{M/2} W[i+k，j+l]*I[i+k，j+k]$;
$\sigma^2(i,j)=(\frac{1}{P}\sum_{k,l=-M/2}^{M/2} W[i+k，j+l]*I^2[i+k，j+k])-m^2(i,j)$;
$b[i,j]=m[i,j]+\mu\sigma[i,j]$
然后
$ \overline{I}[i,j]=\left\{
\begin{aligned}
b[i,j] &  & if(I[i,j]<b[i,j]) \\
I[i,j] &  & otherwise \\
\end{aligned}
\right.
$
我突然发现markdown里写latex真好看。
这里我理解P是通过W对于S进行一个度量。对应点在S中的值越大，在W中越接近于0，后续的系数$\frac{1}{P}$就越大。通过I->S->W到最后的b的过程，发生了下列变化：
- I->S:原I中的像素间的两级分化加强。
- S->W:将S中的值在W中两端分布于0、1. 也就是说，尽量使得W值为1或0.(S中小值在W中为1，S中大值在W中为0)
- P的大小可以近似看成原图中像素值较小的像素点的数量，也就是背景像素的数量。（这个论文中针对的图片是黑底白字的）
- m(i,j)可以看成是背景像素的平均值。
- $\sigma$函数是I的标准方差。
- b中的值=平均值+标准方差*一个系数。这里就很好理解了，b中的值确实是一个equalization的过程。
- $\overline{I}$里的值如果比b小，则被b替换。我们的前景一定是极大的，背景才会存在极小的score，然后被b替换~也就是背景的像素值朝着一个统一平均的方向前进，当然最后总体应该是比原先的背景要亮一点。论文后面有实验结果，确实是这样的。
**接下来需要多次迭代2-4步，直到背景有一个比较高的统一度。**
5. 经过了良好的Background equlization，这一步就轻轻松松，可以非常简单地找到一个threshold来进行二分。
6. 后续的post processing，暂时不考虑了。


