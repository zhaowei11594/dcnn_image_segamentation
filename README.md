# 基于深度学习的图像分割方法研究与实现
### 卷积神经网络图像分割
![卷积神经网络](https://github.com/liao1995/DCNN-Image-Segmentation/blob/master/method.jpg)  

***

### 图像分割
![图像分割](https://github.com/liao1995/DCNN-Image-Segmentation/blob/master/fig1-1.png)  
图像分割（Image Segmentation）是指为了从图像中提取有意义的内容或便于分析从而将数字图像划分为多个部分的过程。图像分割的结果是若干个互不交叠的区域，并且单个区域在灰度、颜色、质地等上有相似的特征，不同区域在这些特征上差别较大。

***

### 边缘检测
![边缘检测](https://github.com/liao1995/DCNN-Image-Segmentation/blob/master/fig1-2.png)  
作为图像处理、机器视觉、计算机视觉尤其是特征检测和特征提取领域的基础工具，以边缘检测为基础的分割方法已被使用多年。边缘检测方法主要通过检测点、线或边缘的灰度不连续性来分割图像。点检测使用滤波模板对图像进行滤波操作，根据孤立点响应最强的特点来检测孤立点。简单的线检测通过设置特定方向参数比较大的方法，根据特定方向上响应最强来检测线条，原理是求得二维函数在(x,y)处最大变化率的方向，即梯度，这样便可以通过差分近似一阶导数的方法得到更为准确的模板。如Roberts边缘检测模板，Prewitt边缘检测模板，Sobel边缘检测模板等。Roberts边缘检测模板是一个2x2的模板，它通过计算对角方向上两个相邻像素的差来检测边缘；与之不同的是，Prewitt模板空间尺寸为3x3，它通过计算左右（垂直边缘）或上下（水平边缘）相邻像素的差来检测边缘；Sobel算子与Prewitt算子非常相似，不过在最接近中心的位置有着更强的滤波系数，这能够更加有效的近似梯度，并且实验表明这也能够改善检测效果。

***

### 线连接
![线连接](https://github.com/liao1995/DCNN-Image-Segmentation/blob/master/fig1-3.png)  
除此之外，使用高斯平滑函数G(x,y)=e^(-(x^2+y^2)/(2σ^2 ))的拉普拉斯算子对图像进行卷积，可以获得先平滑（从而降低噪声）再产生双边缘的效果，最后通过检测零交叉来定位边缘，这便是LoG边缘检测器；更进一步，使用滞后阈值处理方法的双阈值检测器Canny边缘检测器往往能取得更好的效果。最后，可以通过霍夫变换（Hough Transform）方法从xy空间变换到ρθ空间来连接图像中的线段。

***

### 卷积神经网络
![卷积神经网络](https://github.com/liao1995/DCNN-Image-Segmentation/blob/master/fig2-5.png)  
卷积神经网络是一种前馈神经网络。但是，不同于前面描述的传统前馈神经网络，卷积神经网络具有三维神经元、局部连接性、权值共享等特点，可以接收未经预处理的图像作为输入，这使得它在图像处理方面的优势得天独厚。卷积神经网络一般包括卷积层，池化（Pooling）层及全连接层。上图展示了典型的卷积神经网络结构，可以表示为：输入->[卷积->ReLU->池化]*2->全连接层->输出，其中下采样（Subsampling）即为池化操作，ReLU没有在图中体现出来。
![卷积层](https://github.com/liao1995/DCNN-Image-Segmentation/blob/master/fig2-6.png)  
卷积层是卷积神经网络的核心部分。它由一系列神经元组成，这些神经元都有可学习的权值和偏置参数，每个神经元接收一些输入，进行点乘操作并施加一个可选的非线性函数。和传统的神经网络不同，卷积神经网络具有三维神经元，除了宽度高度还有深度方向。卷积层的一系列可学习的滤波参数（或称为卷积核，Kernel）有一个小的感受野连接到前一层的局部区域，但是可以在深度上，对应于特征图堆叠的方向，拓展前一层的完整深度。这种性质叫做局部连接性。假设输入是一幅图片，不妨认为其宽高相等，规格为W×W×C，卷积核数量为N，卷积层神经元感受野大小为F，卷积步幅（Stride）为S，每个边界零填充的数量为P，那么经过卷积后得到的特征图的宽和高为：(W-F+2P)/S+1，深度为N。

***

### 所设计完整网络结构
![网络结构](https://github.com/liao1995/DCNN-Image-Segmentation/blob/master/fig3-1.png)  

***

### Prototxt文件
[prototxt](https://github.com/liao1995/DCNN-Image-Segmentation/blob/master/train_val.prototxt)  

***

### 训练细节
首先涉及到输入图像处理方面。Caffe提供图像数据层支持将原图作为网络的输入，也支持LEVELDB、LMDB或HDF5等存储方式。本文使用转换为LMDB储存方式进入网络进行处理，LMDB因其所有读取操作都是使用mmap将要访问的文件地址加载到地址空间直接访问，从而简化索引空间实现，减少数据复制次数，具有很高的效率。对于训练集中标签也进行以上同样的处理。
当准备好所需的数据之后，可以使用Caffe的Data Layer作为数据输入层。Data Layer的上层是一个数据Blob与一个标签Blob，数据Blob即为转换后的输入图像，但是由于Caffe主要是以用于图像分类的目的而设计，所以其可接受的标签为一个表示类别的数字，而图像分割任务的标签是与原图大小一致的分割后图像。这可以通过改写Caffe输入层来实现把二维图像作为标签。本文网络所采用的方法为定义两个Caffe数据层，一个提供输入数据，另一个提供与数据相对应的标签，只要保证对应关系，便解决了输入的问题。
数据Blob在网络中流动，首先进入卷积层。卷积层参数使用前文提到的考虑线性修正单元的初始化，在Caffe中体现为类型为“msra”的权值填充方式；之后进入批量标准化层、参数化线性修正单元层、池化层，在Caffe中分别为简单的类型为BatchNorm、PReLU及Pooling的层；经过7层这样的结构，数据流入全连接层，Dropout及反卷积层，反卷积层参数初始化方式使用双线性内插，在Caffe中体现为类型为“bilinear”的权值填充方式，经过8层反卷积层之后，数据流入输出层。
![数据集](https://github.com/liao1995/DCNN-Image-Segmentation/blob/master/%E5%9B%BE3-7_%E6%95%B0%E6%8D%AE%E9%9B%86%E5%A4%84%E7%90%86.png)  
输出层经Softmax输出分数图，该分数图尺寸与分割标签即Groudtruth有关，标签在PASCAL VOC图像分割数据集中体现为每个像素灰度值为该像素所属类别的标号，由于该数据集主要有包括飞机、狗、人等20类目标，所以它的标签灰度值范围从0到20。本文网络为了简化任务，不对目标进行类别区分，也就是说将所有类别的物体都看作是目标，标记为1，这样网络的输出也简化为只有两个分数图，即识别为目标和概率和识别为背景的概率，如上图所示。

***

### 训练日志
![日志](https://github.com/liao1995/DCNN-Image-Segmentation/blob/master/log.png)  

***

### 训练过程损失变化
![loss-lr](https://github.com/liao1995/DCNN-Image-Segmentation/blob/master/LOSS_LR.svg)  
网络性能对学习速率是十分敏感的，一个选择不合适的学习速率可能会导致网络具有非常差的表现，如果学习速率设置的过低，网络的收敛速度将会慢到不可接受；学习速率设置太高则可能导致不收敛甚至发散；合适的学习速率会使得网络的损失曲线光滑下降，本文使用前文定义的网络，对完整数据集上不同学习速率10000次迭代损失变化情况进行实验，结果如上图所示，可以看出，学习速率越小损失波动越小，说明训练过程越稳定，但是较小的学习速率很明显又会导致非常长的训练时间。  所以本文在训练过程中迭代一定次数之后给出网络损失随着迭代次数变化的曲线，从而分析网络当前学习速率是否合理，并进行适当调整。由于本文所设计的网络采用小批量随机梯度下降方法进行训练，所以在每个迭代纪元（epoch）会有很多个小批量训练数据，下图给出了本文所设计网络在一次300,000次的迭代中损失函数的变化曲线，图中绘制出了每个训练批次的损失。可以看出，损失在这个时期大致呈现光滑的下降趋势，这说明网络的学习速率设置的比较合适。另外，损失曲线的“宽度”表示训练批次的大小，如果太宽说明每批次训练数据之间的方差过大，这说明应该增加批次的大小，也就是说，需要一次训练更多的图像，这有利于改善网络性能或提升收敛速度，但是如果训练批次增大会大大增加对内存的占用，这对机器性能要求很高，否则可能会出现“Out of Memory”错误。  
![损失变化](https://github.com/liao1995/DCNN-Image-Segmentation/blob/master/%E5%9B%BE4-2_%E6%8D%9F%E5%A4%B1%E5%8F%98%E5%8C%96.png)  

***

### 精确度情况
除此之外，观察训练精确度和验证精确度两者变化曲线也有利于我们对网络当前所处的状态进行分析。下图给出了本文网络训练过程中训练精确度和在验证数据集上的验证精确度变化曲线，当验证精确度收敛的时候，两条精确度曲线之间的距离能体现出网络训练的有效性。如果两条曲线之间的距离很大，这就暗示网络在训练数据集上能够得到很好的精确度，然而在验证数据集上表现不行，也就是说，此时网络处于过拟合状态，这就应该增加网络的正则化强度，也即增大权值衰减参数，从而弱化训练样本梯度更新过程中所占的比重，减轻过拟合；然而，如果两条曲线之间虽然没有距离，但是都处于很低的精确度水平，这说明网络处于欠拟合状态，这样就需要增加网络的描述能力，或者说增加网络的深度，对于本文设计的网络来说，就是在原网络基础上再次增加卷积层和反卷积层。从图4-3曲线可以看出，本文所设计的网络在这个时期训练精确度和验证精确度大致呈上升趋势，都处于比较高的水平，并且它们之间有着很小的差距，训练精确度的波动是因为在数据集的不同部分所获得的损失会有差距。总体来说，在这个时期网络的状况是可以接受的。  
![精确度](https://github.com/liao1995/DCNN-Image-Segmentation/blob/master/%E5%9B%BE4-3_%E7%B2%BE%E7%A1%AE%E5%BA%A6%E6%83%85%E5%86%B5.png)  

***

### GUI
![GUI](https://github.com/liao1995/DCNN-Image-Segmentation/blob/master/%E5%9B%BE4-4_GUI.png)  
本文为方便使用训练后的模型来获得分割效果，基于MATLAB实现了一个跨平台的图形用户界面（GUI）程序，该程序包含了上述生成分割后图像的功能，并且可以以原图上对目标加边界、分割出目标、分割出背景、分割为二值图像形式、只显示边界等方式来显示分割结果，同时具备从Caffe训练日志中提取训练集损失、检验集精确度数据并绘出它们随网络迭代次数变化情况图像的功能，如果给定测试图像的Groudtruth，或者给定一对用其他方法分割后的图像及其Groudtruth，该程序还可以计算像素精确度、平均精确度、平均IoU（Intersection over Union）、加权IoU（Weighted IoU）这四项衡量分割效果的指标，并绘出分割后结果与Groudtruth之间的异同；最后，使用该程序还可以进行图像批量分割。  

***

### 分割结果
![结果](https://github.com/liao1995/DCNN-Image-Segmentation/blob/master/%E5%9B%BE4-5_%E5%88%86%E5%89%B2%E7%BB%93%E6%9E%9C.png)  
下图展示了分割结果与Groudtruth之间的异同，其中白色区域表示模型正确将目标划分为目标；黑色区域表示模型正确将背景划分为背景；红色区域表示模型错误将背景划分为目标；绿色区域表示模型错误将目标划分为背景。  
![评估](https://github.com/liao1995/DCNN-Image-Segmentation/blob/master/%E5%9B%BE4-6_%E5%88%86%E5%89%B2%E7%BB%93%E6%9E%9C%E8%AF%84%E4%BC%B0.png)  

***

### 与传统方法比较
![比较](https://github.com/liao1995/DCNN-Image-Segmentation/blob/master/%E5%9B%BE4-7_%E5%88%86%E5%89%B2%E7%BB%93%E6%9E%9C%E6%AF%94%E8%BE%83.png)  

***

### Demo
[Demo](https://github.com/liao1995/DCNN-Image-Segmentation/blob/master/Demo.mp4)  
