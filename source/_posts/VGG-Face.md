---
title: 基于VGG-Face的人脸识别测试
date: 2017-03-03 08:46:47
reward: true
categories: "caffe"
tags:
	- caffe
	- 人脸识别
---
VGG Face Descriptor 是牛津大学VGG小组的工作，现在已经开源训练好的网络结构和模型参数，本文将基于此模型在caffe上使用自己的人脸数据微调，并进行特征提取与精确度验证。   
数据传送门：[CASIA WebFace](http://www.cbsr.ia.ac.cn/english/CASIA-WebFace-Database.html)  
模型传送门：[http://www.robots.ox.ac.uk/~vgg/software/vgg_face/](http://www.robots.ox.ac.uk/~vgg/software/vgg_face/)
<!--more-->  
#### 模型准备  
1.从上面的网址中下载VGG-Face已经训练好的模型和网络结构文件，根据deploy.proto文件来修改得到train_val.prototxt文件，主要有：修改数据层的输入和最后全连接层以及损失层等，注意fc8层名称的修改，具体如下：  
``` python
name: "VGG_FACE_16_layers"
layer {
  name: "data"
  type: "Data"
  top: "data"
  top: "label"
  include {
    phase: TRAIN
  }
  transform_param {
    mirror: true
    crop_size: 224
#    mean_file: "data/ilsvrc12/imagenet_mean.binaryproto"
  }
  data_param {
    source: "vggface/webface_train_lmdb"
    batch_size: 32
    backend: LMDB
  }
}
layer {
  name: "data"
  type: "Data"
  top: "data"
  top: "label"
  include {
    phase: TEST
  }
  transform_param {
    mirror: false
    crop_size: 224
#    mean_file: "data/ilsvrc12/imagenet_mean.binaryproto"
  }
  data_param {
    source: "vggface/webface_val_lmdb"
    batch_size: 32
    backend: LMDB
  }
}  
......
layer {
  bottom: "fc7"
  top: "fc8_s"
  name: "fc8_s"
  type: "InnerProduct"
  param {
    lr_mult: 10
    decay_mult: 1
  }
  param {
    lr_mult: 20
    decay_mult: 0
  }
  inner_product_param {
    num_output: 2031
    weight_filler {  
      type: "gaussian"
      std: 0.01  
    }  
    bias_filler {  
      type: "constant"  
      value: 0.1
    }  
  }
}
layer {
  name: "accuracy"
  type: "Accuracy"
  bottom: "fc8_s"
  bottom: "label"
  top: "accuracy"
  include {
    phase: TEST
  }
}
layer {
  name: "loss"
  type: "SoftmaxWithLoss"
  bottom: "fc8_s"
  bottom: "label"
  top: "loss"
}
```   

#### 模型训练  
本次训练，我采用的是webface中人脸数量在50张以上的个人类别，总共有两千多个。按照caffe的工具转换成lmdb格式即可开始训练。   
#### 模型效果测试
模型训练结束之后，在lfw上验证实验效果。lfw数据对使用官方提供的txt文件，不过我觉得格式不太好，就自己用脚本进行了些许修改：  
pairs.txt  

``` python
Abel_Pacheco	1	4
Akhmed_Zakayev	1	3
Akhmed_Zakayev	2	3
Amber_Tamblyn	1	2
Anders_Fogh_Rasmussen	1	3
Anders_Fogh_Rasmussen	1	4
```  

修改后，将一对图片的路径放在一起，属于同一个人为1，否则为0  
``` python
Abel_Pacheco/Abel_Pacheco_0001.jpg  Abel_Pacheco/Abel_Pacheco_0004.jpg	1
Akhmed_Zakayev/Akhmed_Zakayev_0001.jpg	Akhmed_Zakayev/Akhmed_Zakayev_0003.jpg	1
Akhmed_Zakayev/Akhmed_Zakayev_0002.jpg	Akhmed_Zakayev/Akhmed_Zakayev_0003.jpg	1
Amber_Tamblyn/Amber_Tamblyn_0001.jpg	Amber_Tamblyn/Amber_Tamblyn_0002.jpg	1
```  

脚本如下：   
``` python
#!/usr/bin/python  
#-*- coding: utf-8 -*-  

#Created on Thur Mar 2 10:17:38 2017
#Goal:parse pairs.txt of LFW database to label.txt for face #recognition test
#@author: wujiyang

import sys

def get_all_images(filename):
	file = open(filename)
	lines = file.readlines()
	list = []
	for line in lines:
		line_split = line.strip('\n').split('\t')
		if(len(line_split)) == 3:
   			line_split[-1] = line_split[-1].zfill(4)
   			line_split[-2] = line_split[-2].zfill(4)
		if(len(line_split)) == 4:
   			line_split[-1] = line_split[-1].zfill(4)
   			line_split[-3] = line_split[-3].zfill(4)
		list.append(line_split)
	file.close()
	return list

def save2labelfile(list):
	file = open('label.txt', 'w')
	labellines = []
	for i in range(len(list)):
		if len(list[i]) == 3:
			labelline = list[i][0] + '/' + list[i][0] + '_' + list[i][1] + '.jpg' + '\t' + 'original' + '/' + list[i][0] + '/' + list[i][0] + '_' +list[i][2] + '.jpg' + '\t' + '1\n'
			labellines.append(labelline)
		elif len(list[i]) == 4:
			labelline = list[i][0] + '/' + list[i][0] + '_' + list[i][1] + '.jpg' + '\t' + 'original' + '/' + list[i][2] + '/' + list[i][2] + '_' + list[i][3] + '.jpg' + '\t' + '0\n'
			labellines.append(labelline)
	file.writelines(labellines)
	file.close()


'''''
使用方法：执行脚本  python pair2label.py pairs.txt
'''
if __name__ == "__main__":
	if len(sys.argv) != 2:
		print "Format Error! Usuage: python %s pairs.txt" %(sys.argv[0])
		sys.exit()
	list = get_all_images("pairs.txt")
	save2labelfile(list)
	print "Done!"
```   
#### 特征提取与验证
此脚本采用caffe的python接口进行特征提取与验证。
```python
# -*- coding: utf-8 -*-
"""
Created on Mon Apr 20 16:55:55 2015

@author: wujiyang
@brief：在lfw数据库上验证训练好了的网络
"""
import math
import sklearn
import numpy as np
import matplotlib.pyplot as plt
import skimage
caffe_root = '/home/wujiyang/caffe/'  
import sys
sys.path.insert(0, caffe_root + 'python')
import caffe

import sklearn.metrics.pairwise as pw


#模型初始化相关操作
def initilize():
    print 'model initilizing...'
    deployPrototxt = "./vgg-face-deploy.prototxt"
    modelFile = "./vgg-face.caffemodel"
    caffe.set_mode_gpu()
    caffe.set_device(0)
    net = caffe.Net(deployPrototxt, modelFile,caffe.TEST)
    return net

def read_imagelist(labelfile):
    '''
    @brief：从列表文件中，读取图像数据到矩阵文件中
    @param： labelfile 图像列表文件
    @return ：4D 的矩阵
    '''
    file = open(labelfile)
    lines = file.readlines()
    test_num=len(lines)
    file.close()
    x = np.empty((test_num,3,224,224))
    y = np.empty((test_num,3,224,224))
    labels = []
    i = 0
    for line in lines:
        path = line.strip('\n').split('\t')
        #read left image
        filename = path[0]
        img = skimage.io.imread(filename,as_grey=False)
        image = skimage.transform.resize(img,(224,224))*255
        if image.ndim < 3:
            print 'gray:'+filename
            x[i,0,:,:]=image[:,:]
            x[i,1,:,:]=image[:,:]
            x[i,2,:,:]=image[:,:]
        else:
            x[i,0,:,:]=image[:,:,0]
            x[i,1,:,:]=image[:,:,1]
            x[i,2,:,:]=image[:,:,2]
        #read right image
        filename = path[1]
        img = skimage.io.imread(filename,as_grey=False)
        image = skimage.transform.resize(img,(224,224))*255
        if image.ndim < 3:
            print 'gray:'+filename
            y[i,0,:,:]=image[:,:]
            y[i,1,:,:]=image[:,:]
            y[i,2,:,:]=image[:,:]
        else:
            y[i,0,:,:]=image[:,:,0]
            y[i,1,:,:]=image[:,:,1]
            y[i,2,:,:]=image[:,:,2]
        #read label
        labels.append(int(path[2]))

        i=i+1
    return x, y, labels

def extractFeature(leftdata,rightdata):
    #提取左半部分的特征
    test_num=np.shape(leftdata)[0]
    #data 是输入层的名字
    out = net.forward_all(data = leftdata)                                                                    
    feature1 = np.float64(out['fc7'])
    featureleft=np.reshape(feature1,(test_num,4096))
    #np.savetxt('feature1.txt', feature1, delimiter=',')

    #提取右半部分的特征
    out = net.forward_all(data = rightdata)                                                                                   
    feature2 = np.float64(out['fc7'])
    featureright=np.reshape(feature2,(test_num,4096))
    #np.savetxt('feature2.txt', feature2, delimiter=',')

    return featureleft, featureright

def calculate_accuracy(distance,labels,num):    
    '''
    #计算识别率,
    选取阈值，计算识别率
    '''    
    accuracy = {}
    predict = np.empty((num,))
    threshold = 0.1
    while threshold <= 0.9 :
        for i in range(num):
            if distance[i] >= threshold:
            	 predict[i] = 0
            else:
            	 predict[i] = 1
        predict_right = 0.0
        for i in range(num):
        	if predict[i] == labels[i]:
        	   predict_right += 1.0
        current_accuracy = (predict_right / num)
        accuracy[str(threshold)] = current_accuracy
        threshold = threshold + 0.001
    #将字典按照value排序
    temp = sorted(accuracy.items(), key = lambda d:d[1], reverse = True)
    highestAccuracy = temp[0][1]
    thres = temp[0][0]
    return highestAccuracy, thres



if __name__=='__main__':
    #模型初始化
    net = initilize()
    print 'network input :' ,net.inputs  
    print 'network output： ', net.outputs
    #读取图像数据
    leftdata,rightdata,labels = read_imagelist('tmp.txt')
    #计算特征
    featureleft, featureright = extractFeature(leftdata, rightdata)

    #计算每个特征之间的距离 cosine距离
    test_num = len(labels)
    mt = pw.pairwise_distances(featureleft, featureright, metric='cosine')
    distance = np.empty((test_num,))
    for i in range(test_num):
          distance[i] = mt[i][i]
    print 'Distance before normalization:\n', distance
    print 'Distance max:', np.max(distance), ' Distance min:', np.min(distance), '\n'
    # 距离需要归一化到0--1,与标签0-1匹配
    distance_norm = np.empty((test_num,))
    for i in range(test_num):
            distance_norm[i] = (distance[i]-np.min(distance))/(np.max(distance)-np.min(distance))
    print 'Distance after normalization:\n', distance_norm
    print 'Distance_norm max:', np.max(distance_norm), ' Distance_norm min:', np.min(distance_norm), '\n'

    #根据label和distance_norm计算精确度
    highestAccuracy, threshold = calculate_accuracy(distance_norm,labels,len(labels))
    print ("the highest accuracy is : %.4f, and the corresponding threshold is %s \n"%(highestAccuracy, threshold))
```  
