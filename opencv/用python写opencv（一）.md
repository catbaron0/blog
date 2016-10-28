#用Python写OpenCV（一）

 **前言**
 
 *最近刚到岛国，研究室的方向是图像处理，虽然还没有固定的课题，但是openCV肯定是少不了的。但是C++实在是爱不起来，好在OpenCV支持python。于是决定参照《OpenCV Computer Vision With Python》来学习，顺道汉化一下。我也是边学边写，错误在所难免。观者勿怪。*
 
 *第一章是安装和配置，这里就不记了。需要的基本组件有*
 
 * *python （2.7版本）*
 * *numpy 这是python的一个库，这里主要用来强化python的数组，把数组当做矩阵来计算*
 * *OpenCV 这个就不说了吧*
 
 *看到这篇文章的这些环境应该都不是问题，不多说，下面从第二章开始*
 
 
## 第二章 处理文件、摄像头和GUI

这一章会介绍OpenCV中I/O相关的功能，同时也会讨论项目的概念，并会开始一个面向对象设计的项目，在随后的章节中会逐渐完善这个项目。

通过开始学习I/O功能和设计模式，我们会像做三明治一样构建我们的项目：从外至内。先是外层，就像面包片，然后才是填充物，或者是算法之类的。我们之所以用这样的方式，是因为计算机视觉是要对外处理的——它要深入理解计算机外部的世界——而我们要做的是，在随后的章节中用一个统一的借口来把算法应用到所有的外部信息。

>所有本章的完整代码都可以在坐着的网站上下载到:<http://nummist.com/opencv/3923_02.zip>.

### 2.1 基本的I/O脚本

所有的CV应用都需要读入一张图片。其中大多数还会生成一张图片作为输出。一个交互式的CV应用可能还需要把摄像头作为输入源，并使用一个窗口作为输出端。然而，还有其他的输入源或者输出端，包括图片文件，视频文件和数位序列。比如数位序列可以通过网络连接来接受/发送，或者在通过算法来生成。我们来分别看一下这些方式。
#### 2.1.1 读/写图片文件
OpenCV提供的imread()和imwrite()函数支持各种静态图片的文件格式。支持的文件格式随操作系统有所不同，但应该总是支持BMP格式。一般来说，也应该支持PNG，JPEG，和TIFF格式。图片可以从一种格式的文件中读取，然后存储为另一种格式。比如说，让我们来吧一张图片从PNG转化为JPEG：

```import cv2image = cv2.imread('MyPic.png')cv2.imwrite('MyPic.jpg', image)```
>Tips：我们使用的大部分OpenCV功能都来自cv2模块。你可以浏览一下其他基于cv或者cv2.cv的OpenCV手册，但那是已经废弃的模块。当我们需要用到还没有包含到cv2中的内容时，我们会使用cv2.cv模块。imread()函数默认返回一个BGR图片，就算读入的时灰度图像也是一样。BGM(blue-green-red)与RGB(red-green-blue)图像一样，只是通道顺序互逆。
当然我们也可以指定imread()的读入模式为CV_LOAD_IMAGE_COLOR (BGR模式)，CV_LOAD_IMAGE_GRAYSCALE (灰度模式)，或者是CV_LOAD_IMAGE_UNCHANGED (根据读入文件的色彩空间决定是BGR模式还是灰度模式)。举个例子，我们来用灰镀膜是加在一张PNG图片（这样会失去图片信息），然后把它保存为一张只有灰度的PNG图片：
```import cv2grayImage = cv2.imread('MyPic.png', cv2.CV_LOAD_IMAGE_GRAYSCALE)cv2.imwrite('MyPicGray.png', grayImage)
```
不管是什么模式，imread()函数都会丢掉alpha通道（透明度）。imwrite()函数需要指定图片是BGR格式或者灰度格式，并且每个通道都需要有一定数位。举例来说，BMP格式每个通道需要8bit，PNG格式则允许每个通道有8bit或者16bit。#### 2.1.2 图片和raw byte互相转换

理论上来说，一个字节是一个0~255之间的整数。在实时图像普及的今天，一个像素通常会用每个通道上的一个字节表示，虽然其它表示方法也行的通。

一个OpenCV图像是一个二维或三维numpy.array数组。一个8bit的灰度图像是一个包含字节信息的二维数组，一个24bit的BGR图像是一个三维数组，同样也包含字节信息。我们可以通过类似于image[0, 0]或者image[0, 0, 0]的表达式来访问每个字节信息。其中第一个值是像素的y坐标，或者说“行”，0的意思是最上面。第二个值是像素的x坐标，或者说“列”，0的意思是在最左边。第三个值（如果有的话）表示的是色彩通道。

举个例子，在8bit的灰度图像中，如果在最左上的位置有一个白色的像素，那么image[0, 0]就是255。对一个24bit的BGR图像来说，如果在最左上的位置有一个蓝色的像素，那么image[0, 0]就是[255, 0, 0]。

>Tips：类似 image[0, 0] 或者 image[0, 0] = 128 的表达式只是一个选择，我们可以使用类似 image.item((0, 0)) 或者 image.setitem((0, 0), 128)的表达式。后者对于单个像素的操作更加高效。然而，我们在随后的章节中会看到，我们通常都需要在大规模的图层上操作，而不是对单个像素操作。

既然图片在每个图层都有8bit数据，那么我们就可以把它转化为Python中标准的bytearray。这是一个一维数组：

```
byteArray = bytearray(image)
```
同样的，既然bytearray是字节信息的有序排列，那么我们就可以把它们重组为一个numpy.array数组类型，也就是一张图片：

```grayImage = numpy.array(grayByteArray).reshape(height, width)bgrImage = numpy.array(bgrByteArray).reshape(height, width, 3)
```
下面我们来看一个稍微复杂点的例子。我们来把一个随机的bytearray转换为一张灰度图像和BGR图像。```   import cv2   import numpy   import os   # Make an array of 120,000 random bytes.   randomByteArray = bytearray(os.urandom(120000))   flatNumpyArray = numpy.array(randomByteArray)   # Convert the array to make a 400x300 grayscale image.   grayImage = flatNumpyArray.reshape(300, 400)   cv2.imwrite('RandomGray.png', grayImage)   # Convert the array to make a 400x100 color image.   bgrImage = flatNumpyArray.reshape(100, 400, 3)   cv2.imwrite('RandomColor.png', bgrImage)
```
执行这个脚本之后，我们会在脚本的当前目录下得到一对随机生成的图片：RandomGray.png和RandomColor.png。

>在这里我们使用了Python标准的os.urandom()函数来得到随机数组，然后把它转换为numpy数组。需要注意的时，这里也可以直接生成一个随机的numpy数组（而且会更高效）。做法是numpy.random.randint(0, 256, 120000).reshape(300, 400)。我们这里使用os.urandom()的原因只是为了示范如何从raw bytes进行转换。


