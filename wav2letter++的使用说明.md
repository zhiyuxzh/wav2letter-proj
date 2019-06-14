# wav2letter++ 使用记录及说明
```
作者:徐泓洋
时间:2019.2.13
```

## 一.目录

- 安装及配置
- 运行设置及操作
- 数据情况说明


## 二.安装及配置

- 安装

步骤参见[教程](https://github.com/facebookresearch/wav2letter/blob/master/docs/installation.md)

- docker

1: CPU版

```
$ docker pull wav2letter/wav2letter:cpu-latest
```
[问题](https://github.com/facebookresearch/wav2letter/issues/187)中描述的错误表示cpu版需要[MKL-DNN](https://github.com/intel/mkl-dnn/) 按照链接中的说明build 后即可解决;

2: GPU版

```
$ docker pull wav2letter/wav2letter:cuda-latest
$ nvidia-docker run -it wav2letter/wav2letter:cuda-latest /bin/bash
```
cuda 9.2
cudnn 

问题:docker: Error response from daemon: create nvidia_driver_410.93: error looking up volume plugin nvidia-docker: plugin "nvidia-docker" not found.

解决: sudo service nvidia-docker start

3: dockerfile 

参见[教程](https://github.com/facebookresearch/wav2letter/blob/master/docs/installation.md),但是经过尝试,有问题,暂时不可用.

## 三.运行设置及操作

- Tutorials on miniLibrispeech dataSet

#### 3.1、配置 train.cfg

配置文件的地址: ./wav2letter/tutorials/1-librispeech_clean/train.cfg

```
--datadir=/data/w2l/                                     # 所有数据所在的根目录
--tokensdir=/data/w2l
--rundir=/data/w2l
--archdir=./wav2letter/tutorials/1-librispeech_clean/    # 网络配置文件地址
--train=data/train-clean-100                             # 训练的数据集地址
--valid=data/dev-clean                                   # 验证的数据集地址
--input=flac                                             # 输入的音频格式
--arch=network.arch                                      # 网络配置文件名称
--tokens=data/tokens.txt                                 # 音素
--criterion=ctc                                          # 
--lr=0.1                                                 #
--maxgradnorm=1.0
--replabel=2
--surround=|                                             #
--onorm=target
--sqnorm=true
--mfsc=true                                              #
--filterbanks=40                                         # MFCC特征数
--nthread=4                                              # GPU线程数
--batchsize=4                                            #
--runname=librispeech_clean_trainlogs                    # log及模型文件存储地址
--iter=100
```

#### 3.2、训练声学模型

```
执行指令:

./wav2letter/build/Train train --flagsfile ./wav2letter/tutorials/1-librispeech_clean/train.cfg  --logtostderr=1 --reportiters=1000

其中: --flagsfile  "File specifying gflags"
      --logtostderr=1 ""
      --reportiters=10 "number of iterations after which we will run val and save model,if 0 we only do this at end of epoch"
```

train只是其中一种训练方式,还有另外两种continue和fork,具体参见[教程](https://github.com/facebookresearch/wav2letter/blob/master/docs/train.md)

```
继续训练的指令

./wav2letter/build/Train continue /data/CNSpeech/alldata_trainlogs/ --flagsfile ./wav2letter/tutorials/AllData/train.cfg --logtostderr=1 --reportiters=1000
```

- 文件结构:以英文数据集 LibriSpeech 为例
```
	# ├── data
	# │   ├── dev-clean [10812 entries exceeds filelimit, not opening dir]
	# │   ├── tokens.txt
	# │   ├── test-clean [10480 entries exceeds filelimit, not opening dir]
	# │   └── train-clean-100 [114156 entries exceeds filelimit, not opening dir]
	# ├── LibriSpeech
	# │   ├── BOOKS.TXT
	# │   ├── CHAPTERS.TXT
	# │   ├── dev-clean [40 entries exceeds filelimit, not opening dir]
	# │   ├── LICENSE.TXT
	# │   ├── README.TXT
	# │   ├── SPEAKERS.TXT
	# │   ├── test-clean [40 entries exceeds filelimit, not opening dir]
	# │   └── train-clean-100 [251 entries exceeds filelimit, not opening dir]
	# └── lm
	#     ├── 3-gram.arpa
	#     └── lexicon.txt
	#
	# 9 directories, 8 files
```
- 训练产出四个模型文件

```
001_config                          - config used for training

001_model_librispeech#dev-clean.bin - saved model for best validation error rate. We'll use this for decoding.

001_model_last.bin                  - model at the end of last epoch

001_perf                            - perf, loss, LER metrics for each epoch
```
## 四.数据情况说明

- 数据文件包括四部分
```
1: 音频 -- .wav/.flac 等声音文件本体;
2: 内容 -- .wrd 存储音频对应的文本内容,英语为单词,中文为拼音,字与字之间空格隔开;
3: 分割 -- .tkn 以最小单元对文本内容进行分割,空格用 '|' 替换, 最小单元之间用空格隔开;
4: 文件 -- .id  文件名称,如  这种形式;
```
- 拼音
```
1: 音频 -- 000000000.wav
2: 内容 -- 000000000.wrd      er dui lou shi cheng jiao yi zhi zuo yong zui da de xian gou
3: 分割 -- 000000000.tkn      e r | d u i | l o u | s h i | c h e n g | j i a o | y i | z h i | z u o | y o n g | z u i | d a | d e | x i a n | g o u 
4: 文件 -- 000000000.id       file_id	0
```
- 汉字
```
1: 音频 -- 000000000.wav
2: 内容 -- 000000000.wrd     实现 品牌效应 的 最大化
3: 分割 -- 000000000.tkn     实 现 | 品 牌 效 应 | 的 | 最 大 化
4: 文件 -- 000000000.id      file_id	0
```
- 注意:三个数据集的音频文件以及对应的语料文件的命名必须按照 000000000.wav, 000000000wrd, 000000000.tkn, 0000000000.id 的格式, 从0开始递增,共9位数字; 中文的文本内容需要进行分词处理

## 五.测试

- 测试指令

```
$ ./wav2letter/build/Test -tokens data/tokens.txt -lexicon /data/THCH/lm/lexicon.txt -am /data/THCH/thch_trainlogs/001_model_data#dev.bin -datadir /data/THCH/ -test data/test -emission_dir /data/THCH/ -maxload -1 -show
```

返回:total WER: 13.7002%, total LER: 7.57315%, time: 92.6822s
********
- test.cfg
```
--datadir=/data/wav2letter++/THCH/
--tokensdir=/data/wav2letter++/THCH/
--rundir=/data/wav2letter++/THCH/
-tokens=data/tokens.txt
-lexicon=lm/lexicon.txt
-am=thch_trainlogs/001_model_data#dev.bin
-test=data/test
-emission_dir=/data/wav2letter++/THCH/
-maxload=-1
-show
```
## 六.解码

#### 6.1 语言模型

语言模型的生成用到kenlm工具,docker 里面已经集成了,可以直接用

```
$ bin/lmplz -o 3 -S 40% --text /data/cn_text.txt  --arpa cn_text.arpa

	# -o n-gram
	# --text  语料文件
	# --arpa 保存的语言模型文件名称
	# -S 当出现报错：Cannot allocate memory for 22159151056 bytes in malloc 时使用
```
生成二进制的语言模型
```
$ ./kenlm/build/bin/build_binary /data/THCH/lm/3-gram.arpa 3-gram.mmap
```
#### 6.2词典lexicon.txt

由语言模型生成词典,在prepare_lm.py 文件里进行实现.

```
$ python3 prepare_lm.py
```

#### 6.3解码方式

1:Using acoustic model

```
$ ./wav2letter/build/Decoder decoder --flagsfile ./wav2letter/tutorials/THCH/decode.cfg --logtostderr=1 --reportiters=1000    # '-am <path/to/acoustic_model.bin>'
```

2:Using emission set

```
$ ./wav2letter/build/Decoder decoder --flagsfile ./wav2letter/tutorials/THCH/decode.cfg --logtostderr=1 --reportiters=1000    # '-emission_dir <path/to/emission_dir/>'
```

####6.4解码参数记录

```
--maxload=-1
--beamsize=50
--beamscore=10

(669 samples) in 3934.55s (actual decoding time 2.53s/sample) -- WER: 61.3356, LER: 47.5658
```
```
--maxload=20
--beamsize=1000
--beamscore=10

(20 samples) in 1380.56s (actual decoding time 3.42s/sample) -- WER: 61.4493, LER: 49.0276
```
```
--maxload=20
--beamsize=20
--beamscore=10
(20 samples) in 1403.4s  (actual decoding time 2.32s/sample) -- WER: 62.3188, LER: 49.9488
```

## 七.数据预处理

#### 7.1、数据生成

```
w2l_thch_data_processing.ipynb —— 处理清华数据集，生成 train，test，dev

Data_process_4_dataset.py         —— 同时处理四个数据集，四个数据集合并在一起，生成 train，test，dev
```

#### 7.2、tokens
```
直接根据语料生成，不够的手动补全即可
```
## 八.实验实现

#### 8.1、实验环境
- 服务器:s2(全量数据400h)
- docker-GPU: e73 

- 服务器：s4(小数据集150h)
- docker-GPU: daef

#### 8.2、训练过程中的问题记录:

- 1、loss:inf 
解决办法:降低lr值 [方案](https://github.com/facebookresearch/wav2letter/issues/154)
- 2、decode :core dump or killed
解决办法：参见[方案](https://github.com/facebookresearch/wav2letter/issues/213)

