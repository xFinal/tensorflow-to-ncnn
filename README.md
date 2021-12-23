# Tensorflow to NCNN

这里以 https://github.com/GantMan/nsfw_model 为例介绍下怎么把tensorflow2模型转换为ncnn模型。[nsfw tf model下载](https://github.com/GantMan/nsfw_model/releases/download/1.2.0/mobilenet_v2_140_224.1.zip)

#### 大体上是tf to mlir to ncnn：

1. 准备好ncnn环境和tensorflow环境（我用的是ncnn-20210720和tf2.3）
2. 编译llvm（包含mlir），步骤看这里：https://github.com/Tencent/ncnn/wiki/build-mlir2ncnn （先不看编译mlir2ncnn部分）  
一定要注意：llvm要`checkout 74e6030bcbcc8e628f9a99a424342a0c656456f9`这个commit id，mlir2ncnn目前只能在这个llvm版本下正常工作
3. 编译ncnn tools里的mlir2ncn
	```
	cd tools/mlir
	mkdir build
	cd build
	cmake .. -D LLVM_PROJECT_INSTALL_DIR=<path/to/your/llvm_install>
	make
	```
4. 把模型目录mobilenet_v2_140_224放到与tf2_to_mlir.py同级，运行python文件，得到mlir模型nsfw.mlir    
注意：`model = tf.keras.models.load_model("./mobilenet_v2_140_224", custom_objects={'KerasLayer': hub.KerasLayer})`  
这里load的是SavedModel格式，如果你的模型是其它格式，要采取不同的load方式
5. 执行`./mlir2ncnn nsfw.mlir nsfw.param nsfw.bin`，得到ncnn模型
6. 最好optimize一下，`./ncnnoptimize nsfw.param nsfw.bin nsfw-opt.param nsfw-opt.bin 0`

#### 额外步骤：
针对本文的模型，第5步转换时会出现问题：`tf.Squeeze not supported yet!`  
这个squeeze操作在param文件152行，`tf.Squeeze       op_252                   1 1 299:12 300:12`，我们把它手动修改下：
1. 执行ncnn的extract，看squeeze的输入`299:12`，是1792 x 1 x 1
3. 所以把152行替换为：`Squeeze          op_252                   1 1 299:12 300:12 0=1 1=1 2=1792`  
注：squeeze后实际上是1792，不过和1792 x 1没区别

#### Reference:
https://zhuanlan.zhihu.com/p/152535430  


#### 心情好，weixin打赏一下
![webwxgetmsgimg ](https://user-images.githubusercontent.com/2231483/147183811-c8832374-adf4-4351-b898-b1ec75e2cf9d.jpeg)

