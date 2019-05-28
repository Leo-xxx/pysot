## 挑战目标跟踪算法极限，商汤开源SiamRPN系列算法解读

[机器之心](javascript:void(0);) *3天前*

机器之心发布

**机器之心编辑部**

> 商汤科技智能视频团队首次开源其目标跟踪研究平台 PySOT。PySOT 包含了商汤科技 SiamRPN 系列算法，以及刚被 CVPR2019 收录为 Oral 的 SiamRPN++。此篇文章将解读目标跟踪最强算法 SiamRPN 系列。

**背景**

由于存在遮挡、光照变化、尺度变化等一些列问题，单目标跟踪的实际落地应用一直都存在较大的挑战。过去两年中，商汤智能视频团队在孪生网络上做了一系列工作，包括将检测引入跟踪后实现第一个高性能孪生网络跟踪算法的 SiamRPN（CVPR 18），更好地利用训练数据增强判别能力的 DaSiamRPN（ECCV 18），以及最新的解决跟踪无法利用到深网络问题的 SiamRPN++（CVPR 19）。其中 SiamRPN++ 在多个数据集上都完成了 10% 以上的超越，并且达到了 SOTA 水平，是当之无愧的目标跟踪最强算法。

项目地址：https://github.com/STVIR/pysot

![img](https://mmbiz.qpic.cn/mmbiz_gif/KmXPKA19gW9WUSVCvOYjNufwIuDFSR8H8qS3aHM9XibOljaYlNONI5l0wAOD7MZfPHShClj0vmiaGuJcXzndR5FA/640?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)

以上动图中，红色框是 SiamRPN++ 的跟踪效果，蓝色框是 ECCV 2018 上的 UPDT 的结果，可以看出 SiamRPN++ 的效果更佳，跟踪效果更稳定，框也更准。从这个图也可以看出跟踪的一些挑战：光照急剧变化，形状、大小变化等。

**SiamRPN (CVPR18 Spotlight)**

在 CVPR18 的论文中（SiamRPN），商汤智能视频团队发现孪生网络无法对跟踪目标的形状进行调节。之前的跟踪算法更多的将跟踪问题抽象成比对问题，但是跟踪问题其实和检测问题也非常类似，对目标的定位与对目标框的回归预测一样重要。
研究人员分析了以往跟踪算法的缺陷并对其进行改进：

1. 大多数的跟踪算法把跟踪考虑成定位问题，但它和检测问题也比较类似，对目标的定位和对目标边界框的回归预测一样重要。为此，SiamRPN 将跟踪问题抽象成单样本检测问题，即需要设计一个算法，使其能够通过第一帧的信息来初始化的一个局部检测器。为此，SiamRPN 结合了跟踪中的孪生网络和检测中的区域推荐网络：孪生网络实现对跟踪目标的适应，让算法可以利用被跟踪目标的信息，完成检测器的初始化；区域推荐网络可以让算法可以对目标位置进行更精准的预测。经过两者的结合，SiamRPN 可以进行端到端的训练。
2. 以往的滤波类的方法，没办法通过数据驱动的形式提升跟踪的性能。而 SiamRPN 可以端到端训练，所以更大规模的数据集 Youtube-BB 也被引入到了训练中，通过数据驱动的形式提升最终的性能。

![img](https://mmbiz.qpic.cn/mmbiz_png/KmXPKA19gW9WUSVCvOYjNufwIuDFSR8HvLqPBg55ygmIuibMdshrUiapgKsMFXicDaTfJibLVhibMTarcNzA4SVvzvg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

结合以上两点创新，在基线算法 SiamFC 的基础上，SiamRPN 实现了五个点以上的提升（OTB100，VOT15/16/17 数据集）；同时还达到了更快的速度（160fps）、也更好地实现了精度与速度的平衡。

**DaSiamRPN (ECCV18)**

SiamRPN 虽然取得了非常好的性能，但由于训练集问题，物体类别过少限制了跟踪的性能；同时，在之前的训练方式中，负样本只有背景信息，一定程度上也限制了网络的判别能力，网络只具备区分前景与不含语义的背景的能力。基于这两个问题，DaSiamRPN 设计了两种数据增强方式：

1. 孪生网络的训练只需要图像对，而并非完整的视频，所以检测图片也可以被扩展为训练数据。更准确的来说，通过对检测数据集进行数据增强，生成可用于训练的图片对。因此在 DaSiamRPN 中，COCO 和 ImageNet Det 也被引入了训练，极大地丰富了训练集中的类别信息。同时，数据量增大的本身也带来了性能上的提升。
2. 在孪生网络的训练过程中，通过构造有语意的负样本对来增强跟踪器的判别能力，即训练过程中不再让模板和搜索区域是相同目标；而是让网络学习判别能力，去寻找搜索区域中和模版更相似的物体，而并非一个简单的有语义的物体。

![img](https://mmbiz.qpic.cn/mmbiz_png/KmXPKA19gW9WUSVCvOYjNufwIuDFSR8HEibJlChia7YicyWicmIPjqbJicY53FQRqjr8dorqGlYpeYTfMic0f9FtgVFw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

经过上述的改进，网络的判别能力变得更强，检测分数也变得更有辨别力，这样就可以根据检测分数判断目标是否消失。基于此，DaSiamRPN 可以将短时跟踪拓展到长时跟踪，并且在 UAV20L 数据集上比之前最好的方法提高了 6 个点。在 ECCV18 的 VOT workshop 上面，DaSiamRPN 取得了实时比赛的冠军，**相比去年的冠军有了 80% 的提升**。

**SiamRPN++ (CVPR19 Oral)**

目前，孪生网络中的核心问题在于现有的孪生网络目标跟踪算法只能用比较浅的卷积网络（如 AlexNet），无法利用现代化网络为跟踪算法提升精度，而直接引入深网络甚至会使性能大幅衰减。

为了解决深网络这个 Siamese 跟踪器的痛点，商汤智能视频团队基于之前 ECCV2018 的工作（DaSiamRPN），通过分析孪生神经网络训练过程，发现孪生网络在使用现代化[深度神经网络](https://mp.weixin.qq.com/s?__biz=MzA3MzI4MjgzMw==&mid=2650762659&idx=2&sn=bdc392252946de1f0eef9a8603f1fa1c&chksm=871aa9ddb06d20cb04102e5e25125cabb01602a73aa9f0196534f8d5678edb24d92ab8d03b77&mpshare=1&scene=1&srcid=&key=45382ee80ea50780976dcaa7bc7d7831966d00f6352d40a0c0e5c6d82df16555b8788db9f58a6536abc56fe14e5cd1db4d5a74e9121ecb2e0c3959460b33bc0c588d6771d8791572c722f5e61d0dccd6&ascene=1&uin=MjMzNDA2ODYyNQ%3D%3D&devicetype=Windows+10&version=62060739&lang=zh_CN&pass_ticket=m3jApFpkl13zSqAL8fzqRdqtvCo2UY0oPXCRCEdhGMazpRCDzY4tZVxqhYzkaQxB)存在位置偏见问题，而这一问题是由于卷积的 padding 会破坏严格的平移不变性。然而深网络并不能去掉 padding，为了缓解这一问题，让深网络能够在跟踪提升性能，SiamRPN++ 中提出在训练过程中加入位置均衡的采样策略。通过修改采样策略来缓解网络在训练过程中的存在的位置偏见问题，让深网络能够发挥出应有的效果。

![img](https://mmbiz.qpic.cn/mmbiz_png/KmXPKA19gW9WUSVCvOYjNufwIuDFSR8HVrdeVlOgTg9SHD3TtNV2pJoICLY0mBPWauaEicqMttUWcWPviaSb5yag/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

通过加入这一采样策略，深层网络终于能够在跟踪任务中发挥作用，让跟踪的性能不再受制于网络的容量。同时，为了更好地发挥深层网络的性能，SiamRPN++ 中利用了多层融合。由于浅层特征具有更多的细节信息，而深层网络具有更多的语义信息，将多层融合起来以后，可以跟踪器兼顾细节和深层语义信息，从而进一步提升性能。

除此之外，研究人员还提出了新的连接部件，深度可分离相关层（Depthwise Correlation，后续简写为 DW）。相比于之前的升维相关层（UpChannel correlation，后续简写为 UP），DW 可以极大地简化参数量，平衡两支的参数量，同时让训练更加稳定，也能更好的收敛。

![img](https://mmbiz.qpic.cn/mmbiz_png/KmXPKA19gW9WUSVCvOYjNufwIuDFSR8H0uuWGZxetJkkuNxtSdGdr8WSCPDxgLerKMzhzN0CEicThgOgQS07skg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

为了验证以上提出的内容，研究人员做了详细的实验。在比较常用的 VOT 和 OTB 数据集上，SiamRPN++ 取得了 SOTA 的结果。在 VOT18 的长时跟踪，以及最近新出的一些大规模数据集上如 LaSOT，TrackingNet，SiamRPN++ 也都取得了 SOTA 的结果。

![img](https://mmbiz.qpic.cn/mmbiz_png/KmXPKA19gW9WUSVCvOYjNufwIuDFSR8HFka5JqvIP0rfgLN3a7EWInkD1PfMxIsLPMHpnI2xIypY3tOCOrmY7w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

目前相关代码现已上传至商汤科技开源目标跟踪研究平台 PySOT。PySOT 实现了目前 SOTA 的多个单目标跟踪算法，旨在提供高质量、高性能的视觉跟踪研究代码库，并将其灵活应用于新算法的实现和评估中。欢迎大家使用与交流！

**PySOT 开源项目**

- https://github.com/STVIR/pysot
- **SiamRPN**
- http://openaccess.thecvf.com/content_cvpr_2018/papers/Li_High_Performance_Visual_CVPR_2018_paper.pdf
- **DaSiamRPN** 
- http://openaccess.thecvf.com/content_ECCV_2018/papers/Zheng_Zhu_Distractor-aware_Siamese_Networks_ECCV_2018_paper.pdf 
- **SiamRPN++**
- https://arxiv.org/abs/1812.11703

*参考文献*

1. *Bo Li, Wei Wu, Qiang Wang, Fangyi Zhang, Junliang Xing, Junjie Yan, "SiamRPN++: Evolution of Siamese Visual Tracking with Very Deep Networks" (Oral) in Proceedings of the IEEE Conference on Computer Vision and Pattern Recognition (CVPR) 2019*
2. *Zheng Zhu, Qiang Wang, Bo Li, Wei Wu, Junjie Yan, "Distractor-aware Siamese Networks for Visual Object Tracking" European Conference on Computer Vision (ECCV) 2018*
3. *Bo Li, Junjie Yan, Wei Wu, Zheng Zhu, Xiaolin Hu, "High Performance Visual Tracking with Siamese Region Proposal Network" (Spotlight) in Proceedings of the IEEE Conference on Computer Vision and Pattern Recognition (CVPR) 2018*
4. *Luca Bertinetto, Jack Valmadre, João F. Henriques, Andrea Vedaldi, Philip H. S. Torr "Fully-Convolutional Siamese Networks for Object Tracking" in ECCV Workshop 2016*
5. *Goutam Bhat, Joakim Johnander, Martin Danelljan, Fahad Shahbaz Khan, Michael Felsberg."Unveiling the Power of Deep Tracking" European Conference on Computer Vision (ECCV) 2018![img](https://mmbiz.qpic.cn/mmbiz_png/KmXPKA19gW8Zfpicd40EribGuaFicDBCRH6IOu1Rnc4T3W3J1wE0j6kQ6GorRSgicib0fmNrj3yzlokup2jia9Z0YVeA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)*





**本文为机器之心发布，转载请联系本公众号获得授权。**

✄------------------------------------------------

**加入机器之心（全职记者 / 实习生）：hr@jiqizhixin.com**

**投稿或寻求报道：content@jiqizhixin.com**

**广告 & 商务合作：bd@jiqizhixin.com**









微信扫一扫
关注该公众号