# 卫星影像的AI分类与识别 
这是Datafountain2017的竞赛，题目是卫星影像的AI分类与识别。

## 1 赛题简介
【赛题十】佳格数据-基于深度学习的遥感影像地表覆盖、地表利用分类

赛题背景： 遥感影像解译，作为数字图像分析的一个重要组成部分，长期以来被广泛应用于国土、测绘、国防、城市、农业、防灾减灾等各个领域。随着机器学习技术的发展，如地表覆盖分类等基于遥感影像的数字图像分析技术也得到了一定程度的发展。但是长期以来，基于遥感影像的应用仍停留在目视解译的阶段，自动化的程度较低。一个重要的原因即遥感影像的机器学习分析方法效率不高，还不足以支撑现有的应用。

本赛题目标为在基于一定量的目视解译样本基础上，通过各类图像处理、机器学习算法，提取影像中各类地物的光谱或形状等特别的特征。计算其统计信息，同时用这些种子类别对模型进行训练，随后用训练好的模型去对其他待分数据进行分类。使每个像元按不同的规则将其划分到和其最相似的样本类，以此完成对整个图像的分类。
本次大赛我们将提供在2015年某地区的高分辨率遥感影像，包括基于该遥感影像目视解译出来的地表覆盖样本数据（图片），参赛队伍需要对其进行数据挖掘和必要的机器学习训练。模型建立完毕后，我们会提供采用该高分辨率遥感影像数据来做评测，检测您的算法是否能准确的识别出地表覆盖物。

任务描述：
参赛者需要对我们提供的2015年的高分辨率遥感数据以及相应的样本数据进行预处理、特征提取、训练等各个维度进行数据挖掘和特征创建，并自行创建训练数据中的负样本，进行合适的机器学习训练。在我们提供2015年的数据中，通过您的算法或模型准确的对该地区的地表覆盖物进行分类。分类结果以准确率（overall accuracy）作为主要评估标准。 
## 2 提供数据
数据集：

 2015年某地区的高分辨率遥感影像，包括基于该遥感影像目视解译出来的地表覆盖样本数据（图片）。

数据介绍：

本次比赛提供的样本，为中国南方某地区的高分辨率遥感影像，影像的空间分辨率为亚米级，光谱为可见光波段（R，G，B），已去除坐标信息；训练影像为train1.png、train2.png和train3.png；对应的标注数据为train1_labels_8bits.png、train2_labels_8bits.png和train3_labels_8bits.png。

比赛采用的样本为通过目视解译并手动勾画的多边形，为了降低比赛难度，本次提供的样本一共简化为五类：植被（标记1）、建筑（标记2）、水体（标记3）、道路（标记4）以及其他(标记0)。其中，耕地、林地、草地均归为植被类。影像收集的时间跨度从4月到8月，地表的变化较大。部分耕地和林地处于收获或砍伐后的状态，均被划为植被类。注意，其他类（标记为0）的分类精度不算入最后精度评判标准，最后结果只统计植被、道路、建筑、水体四类的分类精度。

## 3 数据分析

拿到数据的的一刻，发现这个赛题并不是其描述的“分类与识别”，其实这是一个图像分割任务。
初赛数据：
>* 训练数据：两张图像，以及对应的两张Mask。两张图像大小为：7939*7969
>* 测试数据：三张图像，图像大小为7939*7969

复赛数据：
>* 训练数据：三张图像，以及对应的三张Mask。三张图像大小分别为：5112*5564，2470*4011，6116*3357
>* 测试数据：三张图像，图像大小分别为：5112*5564，2470*4011，6116*3357

以下是对数据的分析：
>* 首先这并不是一个“大数据”问题啊，以往我接触过的图像集都是500张以上的规模，这种规模在使用预训练的情况下，准确率还是挺好的。但只有三张图像，而且不能使用其他数据集作为补充，就有点坑了。小数据，如何提取有效的特征？
>* 初赛数据集，有一个大问题：训练集和测试集图像并不属于一类。
训练集图像是城市/城镇等建筑物密集的图像，但属于植被，水体的像素，可以说是非常非常少了。
而测试集是农田等植被、水体大片覆盖的图像，属于建筑的像素，非常少。
如果直接把训练集中图像的每个像素同等对待，那么训练出来的模型肯定会倾向于预测结果是建筑物。
选择什么样的像素集加入训练集，如何对像素集进行处理，对分割结果起到很重要的作用。
![chu_xun_1.png]()
初赛训练图像1
![]()
![]()
初赛测试图像1和2
>* 训练数据集还有一个问题：Mask标的很随意！把一条河流标成一条线，把荒地标成建筑物啥的，都是大片大片存在的。这标错的部分可以算作噪声？还是网络需要学会这种“随意”，因为这样可以模仿人类标注？

## 4 解决方法
>* 针对小数据问题，变为大数据的一种方式是切割图像。将图像采用某种方式（有重叠依次切割/无重叠依次切割/手动选择区域切割/等），切割成某种大小（300*300/500*500/800*800/1000*1000）的像素块。当然，选择哪种方式好，并不能用交叉验证的思想解决。而因为竞赛提供的是光学图像，其实我可以用在ImageNet上训练后的预训练模型，这可以大大减小训练收敛时间，因为其底层特征是一样的。
>* 初赛时，无差别对待训练图像中的每个像素，会导致模型学到的分布和测试图像的分布不相同。这时候，可以选择手动干预训练集，以手动的方式，从训练图像中选择本来就占图像一小部分的像素。我在这样做后，发现选取的像素还是不够，所以只能每张图象复制多次加入训练集。
>* 针对Mask标注的问题，猜想平台用于测试的数据应该也是和给出的标注一样，所以决定不把这种错误标注作为噪声。

## 5 选择模型
在做这个竞赛之前，我研究的方向一直是目标检测。虽然目标检测也是一种视觉任务，但是和分割还是不同的。于是只能一顿搜索，了解一下state-of-the-art的分割方法都是什么。
首先了解的一个模型FCN，这个是神经网络用于语义分割的开山之作啊。然后是SegNet，SegNet比FCN更晚一点出现。然后是U-Net，这个跟SegNet的网络结构很像，但是有点区别。最后接触了Deeplab，据说是state-of-the-art的成果。实际我选择了SegNet、u-Net、Deeplab做对比，提交结果一对比，发现Deeplab效果最好。

## 6 训练模型
>* 寻找SegNet、U-Net、Deeplab的prototxt原型文件
>* 修改caffe，使其可用于训练这些原型文件
>* 调整prototxt文件，使其适配于当前版本caffe
>* 根据数据集生成train_val.txt
>* 开始训练
>* 训练结束后，就可以用模型啦
## 7 测试
>* 把图像分割为500*500有重叠的块。我在这里只使用500*500图像块的中心100*100个像素，作为最终的Mask。所以，在分割图像时一定要注意，要能使所有图像块中心100*100像素的组合，可以覆盖原始图像。
>* 生成test.txt。开始测试，Segnet和unet在测试后就能直接得到相应图像块的Mask，这些Mask就能组合成最终的Mask。但是，Deeplab还要多一步条件随机场，然后才能出来最终预测结果。
## 6 调整参数
主要调整了以下几个参数：
>* 最大迭代次数
>* learning rate
>* batch size
最大迭代次数，主要看验证集上的准确率，开始下降可能就是过拟合了。
learning rate主要影响迭代收敛速度，步长不能太大，会影响loss下降，步长也不能太小，会影响收敛速度。
batch size 不用说了，GPU内存不够是一定要调整的。但是size不能为1，因为这样是不收敛的。





