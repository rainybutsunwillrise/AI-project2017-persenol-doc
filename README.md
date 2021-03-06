#原理图示
![](http://upload-images.jianshu.io/upload_images/8926144-7aa7077b79022e37.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

原理很简单，利用了相似三角形计算距离，所以双目测距的主要任务在于前期摄像头的定标、双目图像点的特征匹配上。

#常用做法具体步骤

##1.双目定标和校正，获得摄像头的参数矩阵

摄像头定标一般都需要一个放在摄像头前的特制的标定参照物（棋盘纸），摄像头获取该物体的图像，并由此计算摄像头的内外参数。标定参照物上的每一个特征点相对于世界坐标系的位置在制作时应精确测定，世界坐标系可选为参照物的物体坐标系。在得到这些已知点在图像上的投影位置后，可计算出摄像头的内外参数。

![](http://upload-images.jianshu.io/upload_images/8926144-4cea4364c89c34a8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如上公式所示，摄像头由于光学透镜的特性使得成像存在着径向畸变，可由三个参数k1,k2,k3确定；由于装配方面的误差，传感器与光学镜头之间并非完全平行，因此成像存在切向畸变，可由两个参数p1,p2确定。
###具体操作：
cvStereoRectify 执行双目校正 
initUndistortRectifyMap 分别生成两个图像校正所需的像素映射矩阵 
cvremap 分别对两个图像进行校正
##2.立体匹配，获得视差图： 
###具体操作：
预处理： 图像归一化，减少亮度差别，增强纹理 
stereoBM生成视差图 
匹配过程： 滑动sad窗口，沿着水平线进行匹配搜索，由于校正后左右图片平行，左图的特征可以在右图对应行找到最佳匹配 
再过滤： 去除坏的匹配点 通过uniquenessratio 
输出视差图disparity：如果左右匹配点比较稠密，匹配点多，得到的图像与原图相似度比较大， 如果匹配点比较稀疏，得到的点与原图 相似度比较小
##3.得出测距：
根据提取的特征点上用上述双目测距的相似三角算法得出距离。

#小车中的实际应用

- SIFT特征提取算法对左右图像点提取特征
- knnMatch取k=2找到左右图片最佳匹配
- 再过滤去除坏的匹配点
- 对于剩下的点使用相似三角形计算公式得到图片各点景深标在图上
- 最终小车避障可根据其中少数点进行判断，或者取均值。
