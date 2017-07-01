# OpenCV.js

This is a JavaScript binding that exposes OpenCV library to the web. This project is made possible by support of Intel corporation. Currently, this is based on OpenCV 3.1.0.

### How to Build

You can build two different versions of OpenCV.js: the **asm.js** version, or the **WebAssembly** one (experimental). If you want to build the later, you will need the "incoming" version of Emscripten.
Since the "incoming" version of Emscripten can build both, *only the installation of Emscripten-incoming will be detailed here*, for simplicity.
If you absolutely want the "stable" one, just replace "incoming" with "master" everywhere it appears in the following command lines.

1. Get the source code

  ```
  git clone https://github.com/ucisysarch/opencvjs.git
  cd opencvjs
  git clone https://github.com/opencv/opencv
  cd opencv
  git checkout 3.1.0
  ```

2. Install emscripten. You can obtain emscripten by using [Emscripten SDK](https://kripken.github.io/emscripten-site/docs/getting_started/downloads.html).

  ```
  ./emsdk update
  ./emsdk install sdk-incoming-64bit --shallow
  ./emsdk activate sdk-incoming-64bit
  source ./emsdk_env.sh
  ```

3. Patch Emscripten & Rebuild.

  ```
  patch -p1 < PATH/TO/patch_emscripten.diff -d emscripten/incoming
  ./emsdk install sdk-incoming-64bit --shallow
  ```

4. Optionally, if you want to reduce the size of the built library, use *filter-bindings.py* to keep only the code that your JS files are actually using (if you want to cancel that and build the full library again, just remove the generated file *bindings2.cpp*).

    ```
    python filter-bindings.py /path/to/files/*.js
    ```

5. Compile OpenCV and generate bindings by executing make.py script.

    a. WebAssembly version (experimental):
      ```
        python make.py --wasm
      ```

    b. asm.js version:
      ```
        python make.py
      ```



### Tests
Test suite contains several tests and examples demonstrating how the API can be used. Run the tests by launching test/tests.html file usig a browser.

The file `tests/minimal-example.html` aims to be a minimal working example. It demonstrates the use of OpenCV JS, converting between ImageData and cv.Mat objects, and the use of the window.Module to initiate the runtime.

### Exported OpenCV Subset
Classes and functions that are intended for binding generators (i.e. come with wrapping macros such as CV_EXPORTS_W and CV_WRAP) are exposed. Hence, supported OpenCV subset is comparable to OpenCV for Python. Also, enums with exception of anonymous enums are also exported.

Currently, the following modules are supported. You can modify the make script to exclude certain modules.

1. Core
2. Image processing
3. Photo
4. Shape
5. Video
6. Object detection
7. Features framework
8. Image codecs

### At a glance
The following example demonstrates how to apply a gaussian blur filter on an image. Note that everything is wrapped in a JavaScript module ('cv').

```Javascript
  // Gaussian Blur
  var mat1 = cv.Mat.ones(7, 7, cv.CV_8UC1),
      mat2 = new cv.Mat();

  cv.GaussianBlur(mat1, mat2, [3, 3], 0, 0, cv.BORDER_DEFAULT);

  mat1.delete();
  mat2.delete();
```
Next example shows how to calculate image keypoints and their descriptors using ORB (Oriented Brief) method.
```Javascript
  var numFeatures = 900,
	    scaleFactor = 1.2,
	    numLevels = 8,
	    edgeThreshold = 31,
		  firstLevel =0,
		  WTA_K= 2,
		  scoreType = 0, //ORB::HARRIS_SCORE
		  patchSize = 31,
		  fastThreshold=20,
		  keyPoints = new cv.KeyPointVector(),
		  descriptors = new cv.Mat();

	var orb = new cv.ORB(numFeatures, scaleFactor, numLevels, edgeThreshold, firstLevel,
									     WTA_K, scoreType, patchSize, fastThreshold);

  // image and mask are of type cv.Mat
	orb.detect(image, keyPoints, mask);
	orb.compute(image, keyPoints, descriptors);

	keyPoints.delete();
	descriptors.delete();
	orb.delete();
```

Functions work on cv::Mat and various vectors. The following vectors are registered and can be used.

```cpp
  register_vector<int>("IntVector");
  register_vector<unsigned char>("UCharVector"););
  register_vector<float>("FloatVector");
  register_vector<std::vector<Point>>("PointVectorVector");
  register_vector<cv::Point>("PointVector");
  register_vector<cv::Mat>("MatVector");
  register_vector<cv::KeyPoint>("KeyPointVector");
  register_vector<cv::Rect>("RectVector");
  register_vector<cv::Point2f>("Point2fVector");
```
### Memory management
All the allocated objects should be freed manually by calling delete() method. To avoid manual memory management for basic types, the following data types are exported as JavaScript value arrays.

```
cv.Size
cv.Point
```

### File System Access
If your OpenCV application needs to access a file, for instance a dataset or a previoulsy trained classifier, you can modify the make script and attach the files by using emscripten "--preload-file" flag.


### Limitations
1. MatExpr is not exported.
2. No support for default parameters yet.
2. Constructor overloading are implemented by number of paramteres and not their types. Hence, only following Mat constructors are exported.

```cpp
  cv::Mat()
  cv::Mat(const std::vector<unsigned char>& data)
  cv::Mat(Size size, int type)
  cv::Mat(int rows, int cols, int type)
  cv::Mat(Size size, int type, void* data, size_t step)
```
