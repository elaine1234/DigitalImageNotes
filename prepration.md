>  The hardest part about learning OpenCV is installing the damn thing!

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

   ```
    brew install cmake pkg-config
    brew install jpeg libpng libtiff openexr
    brew install eigen tbb
   ```

   这边进行的非常顺利。继续往下走~

6. 安装opencv

   ``` 
   brew install opencv3
   ```

   想当然这种玄学的科技不是这样就能安装下来的~会遇到这个错误：`Error: opencv3: Does not support building both Python 2 and 3 wrappers`。但是我们并不需要homebrew去check安装的python环境，而且homebrew可以在不同的python版本环境中编译两次opencv，让两个版本的python都可以使用。可以试一下用以下命令去安装

   ```
   brew install opencv3 --with-contrib --with-python3 --without-python
   ```

   如果还是不行，最简单粗暴的办法是酱紫。运行命令行`brew edit opencv3`。在配置代码中找下面这一段

   ``` 
   if build.with?("python3") && build.with?("python")
     # Opencv3 Does not support building both Python 2 and 3 versions
     odie "opencv3: Does not support building both Python 2 and 3 wrappers"
   end
   ```

   然后把这一段全注释掉，再跑上面的命令。

   如果这时候，你还是像我一样倒霉，遇到了这个问题：`Error: No such file or directory - /private/tmp/opencv3-20170712-32425-22jrb3/3rdparty/ippicv/downloader.cmake`。这是由于*3rdparty/ippicv/downloader.cmake*已经从远程仓库中被移除了，homebrew没有及时更新到。莫要惊慌，把`--head`去掉再来一次。

7. 在上述步骤安装完成以后，先等等，别急着运行，否则是找不到模块的。。安装完成的log中有这样一段话：

   ```
   This formula is keg-only, which means it was not symlinked into /usr/local,
   because opencv3 and opencv install many of the same files.

   If you need to have this software first in your PATH run:
     echo 'export PATH="/usr/local/opt/opencv3/bin:$PATH"' >> ~/.bash_profile

   For compilers to find this software you may need to set:
       LDFLAGS:  -L/usr/local/opt/opencv3/lib
       CPPFLAGS: -I/usr/local/opt/opencv3/include
   For pkg-config to find this software you may need to set:
       PKG_CONFIG_PATH: /usr/local/opt/opencv3/lib/pkgconfig


   If you need Python to find bindings for this keg-only formula, run:
     echo /usr/local/opt/opencv3/lib/python2.7/site-packages >> /usr/local/lib/python2.7/site-packages/opencv3.pth
   ```

   所以还需要在命令行中执行`echo /usr/local/opt/opencv3/lib/python2.7/site-packages >> /usr/local/lib/python2.7/site-packages/opencv3.pth`。Done & Done with python 2.7！这时候在命令行运行`python`， `import cv2`可以看到已经能正常import了。

8. 然鹅，如果你用pthon3，发现还是不能正常引用这个包的。接下来就来填python3.5+的坑吧。记得在第6步的时候我们在安装opencv的时候添加了`--with-python3`，这个确实有用，我们的python3.5+版本下已经有了*cv2.so*。但是！如果你用`ls -l /usr/local/opt/opencv3/lib/python3.5/site-packages/`会发现它不叫*cv2.so*，而是有一串后缀的。。所以解决方法也很简单。

   ```
   cd /usr/local/opt/opencv3/lib/python3.6/site-packages/
   mv cv2.cpython-36m-darwin.so cv2.so
   ```

   然后继续上面的命令`$ echo /usr/local/opt/opencv3/lib/python3.6/site-packages >> /usr/local/lib/python3.6/site-packages/opencv3.pth`就完工了。

   ​

   PS： 如果你之前安装过macports，用homebrew安装opencv的时候还会有问题。要`sudo mv /opt/local ~/macports`，不然环境变量之类的设置会有些问题。