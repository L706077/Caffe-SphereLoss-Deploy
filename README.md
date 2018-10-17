# Caffe-SphereLoss-Deploy
---

- [reference](https://blog.csdn.net/cuixing001/article/details/79207109)

---

### Step 1.

***git clone or download frome [sphereloss](https://github.com/wy1iu/sphereface)*** ：
(1).
```C++
在 sphere-master/tools/caffe-shpereface目錄下：
到 caffe-shpereface/src/caffe/include/caffe/layers 目錄下
  margin_inner_product_layer.cpp、
  margin_inner_product_layer.cu
拷貝到 $ YOUR_CAFFE_ROOT_PATH\src\caffe\layers 目錄裡
```
<br/>

(2).
```C++
在"caffe-sphereface/include/caffe/layers " 目錄下
margin_inner_product_layer.hpp
拷貝到 $ YOUR_CAFFE_ROOT_PATH\include\caffe\layers 目錄裡
```
<br/>

### Step 2.
***caffe.proto改寫：***

(1).
```C++
到$ YOUR_CAFFE_ROOT_PATH\src\caffe\proto 目錄裡打開"caffe.proto"
在 message LayerParameter {
...
}
裡面加入：
MarginInnerProdcuctParameter margin_inner_product_parameter = 151
```
<br/>

(2).
```C++
到$ YOUR_CAFFE_ROOT_PATH\src\caffe\proto 目錄裡打開"caffe.proto"
在最底下加入：
message MarginInnerProductParameter {
  optional uint32 num_output = 1; // The number of outputs for the layer
  enum MarginType {
    SINGLE = 0;
    DOUBLE = 1;
    TRIPLE = 2;
    QUADRUPLE = 3;
  }
  optional MarginType type = 2 [default = SINGLE]; 
  optional FillerParameter weight_filler = 3; // The filler for the weight

  // The first axis to be lumped into a single inner product computation;
  // all preceding axes are retained in the output.
  // May be negative to index from the end (e.g., -1 for the last axis).
  optional int32 axis = 4 [default = 1];
  optional float base = 5 [default = 1];
  optional float gamma = 6 [default = 0];
  optional float power = 7 [default = 1];
  optional int32 iteration = 8 [default = 0];
  optional float lambda_min = 9 [default = 0];
}
```
<br/>

### Step 3.
***重新編譯caffe.pb.cc caffe.pb.h:***
```C++
$ cd YOUR_CAFFE_ROOT_PATH/src/caffe/proto
$ protoc --cpp_out=/YOUR_CAFFE_ROOT_PATH/src/caffe/proto caffe.proto
$ cp /YOUR_CAFFE_ROOT_PATH/src/proto/caffe.pb.h /YOUR_CAFFE_ROOT_PATH/include/caffe/proto/caffe.ph.h
```

### Step 4.
***重新編譯caffe:***
```C++
$ cd  YOUR_CAFFE_ROOT_PATH
$ mkdir build
$ make 
$ make runtest
```

### Step 5.
***編寫train.prototxt在最下方加入：***
```C++
############### A-Softmax Loss ##############
layer {
  name: "fc6"
  type: "MarginInnerProduct"
  bottom: "fc5"
  bottom: "label"
  top: "fc6"
  top: "lambda"
  param {
    lr_mult: 1
    decay_mult: 1
  }
  margin_inner_product_param {
    num_output: 10572
    type: QUADRUPLE
    weight_filler {
      type: "xavier"
    }
    base: 1000
    gamma: 0.12
    power: 1
    lambda_min: 5
    iteration: 0
  }
}
layer {
  name: "softmax_loss"
  type: "SoftmaxWithLoss"
  bottom: "fc6"
  bottom: "label"
  top: "softmax_loss"
}
```







