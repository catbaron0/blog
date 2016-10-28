#### 2.1.3 读写视频文件

OpenCV 提供了VideoCapture和VideoWriter类，支持多种视频文件格式。支持的格式根据系统有所区别，但是总是支持AVI格式。通过 read() 方法，VideoCapture 类会捕捉每一个新的帧，知道视频文件的结束。每一帧都是一张BGR格式的图像。相对的，一张图片也可以通过VideoWriter类的write()方法，写入到VideoWriter中的视频文件。我们通过一个例子，来看一下如何使用YUV编码从一个avi格式的文件中读出一些帧，然后写入到另一个AVI文件中。

~~~python
#这段代码有可能失败
import cv2

videoCapture = cv2.VideoCapture('MyInputVid.avi') # read video file
fps = videoCapture.get(cv2.cv.CV_CAP_PROP_FPS)  #get fps of input-video-file
size = (int(videoCapture.get(cv2.cv.CV_CAP_PROP_FRAME_WIDTH)), \
        int(videoCapture.get(cv2.cv.CV_CAP_PROP_FRAME_HEIGHT))) #get size of frame

videoWriter = cv2.VideoWriter(\
        'MyOutputVid.avi',\
        cv2.cv.CV_FOURCC('I', '4', '2', '0'),\
        fps,\
        size)

sucess,frame = videoCapture.read()
while sucess:
    videoCapture.write(frame)
    sucess, frame = videoCapture.read()
~~~



我们应该注意 VideoWriter 类的构造参数。首先需要制定要写入的视频文件名，之前存在的文件将会被重写。视频的编码方式同样需要指定，可选的编码格式因系统而异。包括如下：

* cv2.cv.CV_FOURCC('I','4','2','0')：未被压缩的YUV编码格式。这种编码适用范围很广，用来处理大文件的情况除外。文件扩展名应该为.avi。* cv2.cv.CV_FOURCC('P','I','M','1')：MPEG-1格式。文件扩展名应该为.avi。
* cv2.cv.CV_FOURCC('M','J','P','G')：motion-JPEG格式. 文件扩展名应该为.avi。* cv2.cv.CV_FOURCC('T','H','E','O')：Ogg-Vorbis格式。文件扩展名应该为.ogv。* cv2.cv.CV_FOURCC('F','L','V','1')：Flash格式. 文件扩展名应该为.flv。

此外，帧率和帧的尺寸也要指定。这里我们是从一个现有的视频文件拷贝，所以这些属性可以通过VideoCapture类的get()方法读取。#### 2.1.4 抓取摄像头

来自摄像头的帧流也可以用VideoCapture处理。不过，我们需要用摄像头的设备序号来代替视频文件，作为参数传递给VideoCapture类。这里我们用一个例子来展示，使用摄像头捕捉10秒的画面，并写入一个avi文件。

```python
#这段代码也可能会失败
import cv2

cameraCapture = cv2.VideoCapture(0)
fps = 30
size = (int(cameraCapture.get(cv2.cv.CV_CAP_PROP_FRAME_WIDTH)),\
        int(cameraCapture.get(cv2.cv.CV_CAP_PROP_FRAME_HEIGHT)))
videoWriter = cv2.VideoWriter(\
    'MyOutputCam.flv', cv2.cv.CV_FOURCC('F', 'L', 'V', '1'), fps, size)

success, frame = cameraCapture.read()
numFramesRemaining = 10 * fps -1
while success and numFramesRemaining > 0:
    videoWriter.write(frame)
    success, frame = cameraCapture.read()
    numFramesRemaining -= 1
```

>注： 上面两端代码我无法成功运行。似乎是Python-OpenCV在部分系统中的一个bug。详见：<https://github.com/ContinuumIO/anaconda-issues/issues/121> 。看起来似乎是对视频文件的读写有问题。只能读入小文件，写出时则总会失败。

遗憾的是，我们不能通过VideoCaputre类的get()方法获得一个准确的帧率，这个方法最后返回0. 为了生成一个合适的VideoWriter对象，我们要么就得假设一个帧率，就像代码里面写的这样，要么就要通过计时器（timer）计算帧率。后者更好，我们会在后面的章节讨论。

摄像头的个数当然也是不确定的，不幸的是，OpenCV并不提供任何方法来查询摄像头的个数。如果在构建VideoCapture对象的时候使用了非法的设备序号，VideoCapture对象将不能获得图像帧，它的read()方法将返回(false, none)。

当我们有一组摄像头，或者是一个多孔摄像头的时候（比如kinect的立体摄像装置），read()方法就不那么适用了。这时候我们要使用的时grab()和retrieve()方法。对于一组摄像头：

```python
success0 = cameraCapture0.grab()    success1 = cameraCapture1.grab()    if success0 and success1:        frame0 = cameraCapture0.retrieve()        frame1 = cameraCapture1.retrieve()
```

对于多孔摄像头，我们需要在retrieve()的参数中确定一个顺序：

~~~python
success = multiHeadCameraCapture.grab()    if success:        frame0 = multiHeadCameraCapture.retrieve(channel = 0)        frame1 = multiHeadCameraCapture.retrieve(channel = 1)
~~~

我们将在第五章详细讲多孔摄像头的处理细节。

#### 2.1.5 在窗口中显示摄像头内容

在OpenCV中，允许通过namedWindow()，imShow()，和destroyWindow()函数来创建，重绘或者销毁窗口。而且，在窗口中可以通过waitKey()函数来捕捉键盘输入，通过setMouseCallback()来捕捉鼠标输入。我们来看一个在窗口中展示摄像头内容的例子：

```python
import cv2

clicked = False
def onMouse(event, x, y, flags, param):
    global clicked
    if event == cv2.cv.CV_EVENT_LBUTONUP:
        clicked = True

cameraCapture = cv2.VideoCapture(0)
cv2.namedWindow('MyWindow')
cv2.setMouseCallback('MyWindow', onMouse)

print 'Showing camera feed. Click Window or press any key to stop.'
success, frame = cameraCapture.read()
while success and cv2.waitKey(1) == -1 and not clicked:
    cv2.imshow('MyWindow', frame)
    success, frame = cameraCapture.read()

cv2.destroyWindow('MyWindow')
```
waitKey()函数的参数是等待键盘输入的毫秒数。它的返回值可能是-1（意味着没有键盘输入）或者是按键的ASCII值，比如按下Esc键的时候会返回27。在<http://www.asciitable.com/>可以看到完整的ASCII列表。而且Python提供了标准的ord()函数来把字符转化为对应的ASCII值。比如ord('a')会返回97。

>**Tips：**在某些系统中，waitKey()函数返回的可能还有ASCII之外的编码值。（这是在Linux系统中，OpenCV使用GTK作为后端的GUI库的时候会出现的一个已知的bug。）在所有的系统中，我们都可以通过只读取返回值的最后一个字节来确保我们拿到的是正确的键值，就像这样：
~~~python
keycode = cv2.waitKey(1)    if keycode != -1:        keycode &= 0xFF
~~~

OpenCV的窗口和waitKey()函数是相互依赖的。只有在waitKey()被调用的时候OpenCV窗口才会更新；只有在OpenCV窗口获得焦点的时候waitKey()函数才会捕捉键盘。

正如之前代码所示，传递给setMouseCallback()函数的回调函数接收5个参数。回调函数的param参数，是通过setMouseCallback()第三个可选参数来设定的，默认是0。回调函数的event参数是下面的一种：

* cv2.cv.CV_EVENT_MOUSEMOVE: 鼠标移动* cv2.cv.CV_EVENT_LBUTTONDOWN: 按下左键* cv2.cv.CV_EVENT_RBUTTONDOWN: 按下右键* cv2.cv.CV_EVENT_MBUTTONDOWN: 按下滚轮* cv2.cv.CV_EVENT_LBUTTONUP: 弹起左键* cv2.cv.CV_EVENT_RBUTTONUP: 弹起右键* cv2.cv.CV_EVENT_MBUTTONUP: 弹起滚轮* cv2.cv.CV_EVENT_LBUTTONDBLCLK: 双击左键* cv2.cv.CV_EVENT_RBUTTONDBLCLK: 双击右键* cv2.cv.CV_EVENT_MBUTTONDBLCLK: 双击滚轮

鼠标回调函数的flags参数可能是下面标识位的逻辑组合：* cv2.cv.CV_EVENT_FLAG_LBUTTON: 点击左键* cv2.cv.CV_EVENT_FLAG_RBUTTON: 点击右键* cv2.cv.CV_EVENT_FLAG_MBUTTON: 点击滚轮* cv2.cv.CV_EVENT_FLAG_CTRLKEY: 按下Ctrl键* cv2.cv.CV_EVENT_FLAG_SHIFTKEY: 按下Shift键* cv2.cv.CV_EVENT_FLAG_ALTKEY: 按下Alt键不幸，OpenCV并不提供任何方法来处理窗口事件。比如我们不能通过点击窗口关闭按钮来停掉我们的应用。由于OpenCV对事件和GUI处理能力的局限性，很多开发者都会选择将它与其他框架结合使用。这一章后面的部分，我们会设计一个抽象的层来把OpenCV加入到其他的应用框架中去。