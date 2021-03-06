 # ctpn_new

### 2018.3.24 更新

这里是我们将要重构代码的地方，每个人的模块代码将存放在这里，在向仓库提交前请确保在根目录中的.gitignore文件中中添加了 **ctpn_tensorflow/***，使得git忽略你在原实现下的代码改动。然后将代码push，必要时PR。之后这份README请大家共同维护，在较大的更新时请在这里添加你所做的工作的描述，减少大家理解代码的时间。

之后将在这里上传基本的项目结构，大家有任何的建议都可以在群里说，Happy Hacking😄

### 2018.3.25 更新
新的项目结构
```
ctpn_new--|___ctpn 网络抽象层，以及网络运行时需要做的一些操作
          |___data_process 读取训练数据并处理成需要的形式，返回roidb对象
          |___input_layer 网络结果第一层，给核心网络提供每轮batch所需的数据
          |___network 核心网络 基类base_network和两个子类train_network和test_network
          |___lib 运行时所需要的某些cython扩展
          |___prepare 数据预处理脚本，结果直接输出到dataset
          |___run 训练数据和测试数据程序的入口
          |___configure.yml 全局配置 唯一配置
```
**所有程序的运行都在ctpn_new目录下运行，书写时请注意包和模块的路径问题**
#### 数据处理格式说明
要将数据整理成的格式如下，存放在dataset/for_train下
```
---| Imageset 保存图片文件
   ----|xxxxxx.jpg xxxxxx为图片名(不带扩展名)
   | Imageinfo 保存每张图片对应的txt文本
   ----|xxxxxx.txt xxxxxx为图片名(不带扩展名) ,每一行为一个文本框，格式为xmin,ymin,xmax,ymax,width,height,channel
   ----|..........
   | train_set.txt 保存所有训练样本对应的文件名 每行为一张图片的信息
```
说明：Imageinfo下的每个txt格式变更，原来~~xmin,ymin,xmax,ymax,width,height,channel~~弃用，新的格式为**xmin,ymin,xmax,ymax**,
train_set.txt格式变为 **xxxx.jpg,scale,width,height,channel, scale是缩放比例**
> 说明：Imageinfo下的每个txt格式变更，原来~~xmin,ymin,xmax,ymax,width,height,channel~~弃用，新的格式为**xmin,ymin,xmax,ymax**,
> train_set.txt格式变为 **xxxx.jpg,width,height,channel,scale** scale为缩放比例


**将原始数据放在dataset/ICPR_text_train下，文件夹分别为image和text, 两个文件夹的数据必须对应一致。在ctpn_new目录下运行预处理脚本, 处理后的数据将存在dataset/for_train下**
## 重要提示 请大家在书写代码之前确认.gitignore中已经加入了如下的语句,并做一次提交, 然后在开始写代码：

```
ctpn_tensorflow/*
ctpn_new/dataset/*
```
### 2018.3.26 更新
dataset为数据和缓存文件目录,为提高代码共享的效率，添加gitignore
之后被忽略，但应保持其结构一致，避免不必要的麻烦，dataset项目目录结构如下
```
dataset---|___checkpoints tensorflow断点存储目录
          |___ICPR_text_train 原始数据集
                 |___text 
                 |___image  text和image文件夹中的数据必须一一对应
          |___for_train 训练数据集
                 |___Imageset
                 |___Imageinfo
                 |___train_set.txt
          |___for_test 测试数据集 暂时没有设计
          |___pretrain
                 |____VGG16.npy预训练模型参数
          |___output 网络运行中间的输出目录
          |___log 日志输出目录
          |___cache 存储

```
**为增加程序的鲁棒性，所欲的路径在书写代码都要判断路径是否存在，若不存在则创建**

数据处理部分，建议做一个类imagedatabese（名字无所谓），类里面有一个关键属性，就是roidb。roidb是一个列表，其长度等于图片的张数，列表的每个元素都是一个字典，一个字典包含列一张图片的全部信息，字典先包含如下键和值：
- image_name：图片的名字
- image_scale：图片的缩放比例
- image_width：图片的宽
- image_height：图片的高
- gt_box：一个N×4的矩阵，矩阵的每一行表示一个文本框的坐标，x1,y1,x2,y2，共N个文本框

不需要**gt_ishard,dont_care_area**这两个键
源代码里面有一个类RoiDataLayer，他的输入参数之一是imagedatabase类的的imdb属性。该类的核心功能是根据imdb生成可以输入到网络的blobs，其核心函数是源代码中的get_next_minibach()，该函数的功能是返回一个blobs，并把指针下移一位，图片都不需要翻转。

### 2018.3.27更新
数据预处理、roidb模块、网络模块初步完成，下一步可以开始进行测试。开始着手分析具体每张图片中文本框的形状分布。

### 2018.4.1更新
注意，为保持一致，im_info里面装着的是（高，宽，缩放比）。任何情况下，高在宽的前面，先是高的信息，再是宽的信息！！
缩放比定义为：修改后的尺寸 / 原图尺寸

### 2018.4.9 更新
测试数据集的目录机构
```
data 
  |_______img 预测框的图片，每次运行测试时候要清空
  |_______predict 预测的文本框 每张图片对应一个txt文本，每行为一个文本框 格式如下
  |_______gt 真值文本框 每张图片对应一个txt文本，每行为一个真值框

```

## 按照比赛的要求，需要提交的文本格式如下,xxxxxx.jpg 对应一个 xxxxxx.txt
## 所给的训练样本为四个角点的顺时针或逆时针方向排列的，起点不固定，我们统一处理成以左上角为起点，顺时针旋转
## 最后生成的预测框，以如下的为标准每行文本按照如下的格式，即左上 右上 右下 左下的格式提交，即顺时针方向

```
 x1 y1            x2 y2 



 x4 y4            x3 y3

 x1,y1,x2,y2,x3,y3,x4,y4
 ......

```
