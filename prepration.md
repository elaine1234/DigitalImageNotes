>  在mac上安装openCV的辛酸血泪史

## 傻瓜式的尝试：brew安装大法

``` 
# add opencv
brew tap homebrew/science

# install opencv
brew install opencv

# 安装一些必要的python库
pip install numpy
pip install matplotlib
```

接下来用跑一段代码测试一下是否安装成功

``` python
import cv2
import numpy as np
from matplotlib import pyplot as plt

img = cv2.imread('road.png', 0)
plt.imshow(img, cmap='gray', interpolation='bicubic')
plt.xticks([]), plt.yticks([]) # to hide tick values on X and Y axis
plt.show()
```

总结：果然图方便是不行的，直接使用brew安装的OpenCV有bug，但是作者已经在最新的版本中修复了：[issue](https://github.com/Itseez/opencv/pull/3006)。

所以要重新安装openCV了， 先卸载掉现有的。

```
brew uninstall opencv
```

## 第二次尝试

第二次尝试循规蹈矩的按照[这个教程](http://www.pyimagesearch.com/2016/12/05/macos-install-opencv-3-and-python-3-5/)一步一步走。因为我本身就有xCode和python环境，所以直接从第四步开始了

4. 安装python虚拟环境。

   ``` 
   pip install virtualenv virtualenvwrapper
   ```

   教程中说安装这个后会存在这样一个路径*/usr/local/bin/virtualenvwrapper.sh*。事实是并没有，卸载后重新安装也没有。。如果有的话跟着下面的步骤配置环境变量后应该就可以分出单独的python环境了。但是我弄了很久也没找到这个文件不存在的原因到底是啥。反正隔离环境不是特别必要，就放弃了进行下一步。后续如果有新发现再update。

5. 安装open的前置依赖