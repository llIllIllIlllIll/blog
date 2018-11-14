## Binarization of noisy gray-scale character images by thin line modeling
原论文资源：[Binarization of noisy gray-scale character images by thin line modeling](https://ac.els-cdn.com/S0031320398000193/1-s2.0-S0031320398000193-main.pdf?_tid=6eb1a8a7-d174-4510-80b0-c435ae89c3fd&acdnat=1542117798_c80c35850ff58a98fce6360f9162cafa).
最近在准备Intel的一个涉及深度学习的程序设计大赛，我们小组选的课题是......手写汉字识别。然后这个汉字识别最基础的必然涉及图片的降噪、二值化处理，在网上找了蛮多相关资料。再mark一篇后续准备要看的论文：
- [ ] [A SURVEY OF METHODS AND STRATEGIES IN CHARACTER SEGMENTATION ——一篇探讨字符分割方案的文章](http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.158.220&rep=rep1&type=pdf)

看完了要记得回来✔哦hhhh。
然后是这篇论文有两个算法。现在我就看完了第一篇然后就想先记录一下，后半部分等今天上完课做完作业再看。
算法思想不是很复杂，但是好像有几个量你需要预先知道。先定义一下术语：
术语 | 含义
------|--------
stoke|这个指的应该就是笔画，论文是针对英文字母写的，但是中文也是一样的。
edge|他这里edge，如果我没弄错的话，指的是笔画的edge。从围观一点的角度分析好像一笔一划这样子不存在一个edge，但你如果从计算机存储的角度来看每个笔画他其实可以看成一个很大的封闭图形，他就会有edge。
intensity|指的应该是灰度图某个像素的亮度。
好了，这些白痴概念浪费了我很久时间。这篇论文开始根本没有术语介绍，当然也应该是因为我太菜了，估计了解这方面内容的人根本不需要这种术语介绍。
接下来先放一张他论文里给的algorithmI的流程图。
![Alt](https://github.com/llIllIllIlllIll/blog/blob/master/bin_al1.png)
解释一下：
1. 输入灰度图。
2. 第二点就有点奇怪...原论文里对第一步第二步的描述是，先用Gaussian filter处理原图像I，然后找到经过处理后得到图像I~gf~。接下来的第一步是从I~gf~中导出一个图像I~edge~，其中这个I~edge~是用来标记原图中的edge的。只要是位于edge上的像素，在I~edge~中被标记为1，其余点都被标记为0.
这里我认为他奇怪的原因是......他没告诉我具体怎么去找那个边。但想来（？）也不难，看大家具体实现吧。
3. 突然很想吐槽这个论文图片的质量，搞得这么模糊不清。为什么不用你们自己的算法处理一下再发上来？一点说服力都没有......
4. 构造一个和原图大小一样的数组S(score array)。这个数组是用在后续步骤对每个像素的score计算上的，然后最后的最后的binarization基于这个score array的结果。
5. 这里还要引入几个值。W~in~和W~out~是两个矩形的window，一会儿要用来对于一个window内的部分数据进行处理。说实话这种window的形式让我想到了CNN里的filter......然后一个W~in~的宽度N~in~要大致等于stroke的宽度，一个W~out~的宽度N~out~要大致等于stroke的宽度*2。 然后这里搞笑的地方就是，这个算法也没告诉我怎么去确定stroke的宽度大小......
6. 引入完这个概念之后，介绍一下这两个window的用途。每一个在I~edge~中被标记为1的点(i,j)，都将有一个以这个点为中心的W~in~和一个W~out~。而每一个W~in~和W~out~中的数据如下：
**W~in~(i,j)[k,l]=I~gf~[i+k,j+l]**
**for -N~in~/2<=k,l<=N~in~/2**
**W~out~(i,j)[m,n]=I~gf~[i+m,j+n]**
**for -N~out~/2<=m,n<=N~out~/2**
接下来找到每一个W~in~(i,j)中的最大值和最小值，记为P~max~(i,j)和P~min~(i,j)。这里需要解释一下，在我的理解中，==因为我们是以一条边的边界作为每一个window的中心，并且一个W~in~的宽度和一条stroke相等，那么理想的情况下，这个W~in~中必然含有最亮和最暗的像素，分别代表stroke中的点以及字符间空白区域的点==。
再对于每一个window确定一个threshold，分界值t(i,j)=(P~max~(i,j)+P~min~(i,j))/2。现在就可以用这个分界值对于整个W~out~里的对象对应的来进行打分加权，具体说就是对于每一个W~out~中的对象对应的S中的值来进行加减：
**if(W~out~(i,j)[m,n]>t(i,j))
S[i+m,j+n]++;**
遍历。
7.最后最后有了一个打完分的S，就可以进行二值化分割了。论文里给出的结论是如果N~out~取的是stroke宽的两倍，最后的global threshold取3N~out~/2。
