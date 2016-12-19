---
layout: post
title:  标签倾斜修正方法记要
categories: [machine-learning,prml]
---


使用分类模型时，大多人会遇到一个常见的问题：标签倾斜。比如用分类器去判断x光片中的癌症，这是一个二元分类问题，由于癌症的比例是非常小的，比如0.001。那么，将这些样本放到大多数分类模型中训练，模型的表现会非常相似，将所有数据都预测为**没有癌症**，因为这样也可以得到**99.999%**的准确率。

此时可用样本重采样的方法平衡标签比例。仍以上面的例子，保留所有癌症的数据，一般称之为正样本；然后在没有癌症的数据中进行采样，得到与癌症样本数量差不多的样本（几倍与正样本也可以），称为负样本。使用此数据进行训练，得到的模型才会根据不同数据预测为癌症或非癌症。但是，由于引入的人工重采样，会使得预测的准确率较高，但是召回率非常低。可能的原因是采样过程中，舍弃了绝大部分样本，导致模型获取的信息不足。经验数据表明，训练数据保持正样本数量不变，逐渐增加负样本数量，会使得模型的准确率会逐渐**下降**，召回率会逐渐**上升**。 

最近在读PRML，其中1.5.4节中介绍了一种**标签倾斜修正**的方法，没有尝试过，先记录相关过程，后面找机会进行验证。

首先，你的模型必须是一个软分类器，即预测值为0到1之间的概率。假设输入向量x，预测标签$C_k$，那么可以用条件概率表示，即计算$p(C_k\|x)$的概率。根据贝叶斯理论，条件概率可以如下变化

$$
	p(C_k|x) = \frac{p(x|C_k)p(C_k)}{p(x)}
$$

上面是没有做重采样时，得到概率。当做重采样时，只是改变了先验概率，即将$p(C_k)$变为$p'(C_k)$。所以，修正的方法就是将计算出的概率除以$p'(C_k)$，并乘以原始先验概率$p(C_k)$。同时，需要对最终结果做出正规化，得到最终的修正结果。

仍借助上面癌症例子。假设癌症的先验概率为$a$,那么没有癌症的概率为$1-a$。我们对样本进行了重采样，将癌症的先验概率修改为b，没有癌症的为$1-b$。令$C_1$表示癌症，$C_0$表示没有癌症，使用一个二元软分类器（如逻辑回归）训练了一个模型$h$,对于样本x，可计算如下结果

$$
	p(C_1|x) = h(x)=p \\
	p(C_0|x) = 1-h(x)=1-p
$$

首先，调整模型比例

$$
	\frac{pa}{b}, \frac{(1-p)(1-a)}{1-b}
$$

然后，正规化

$$
	p_M(C_1|x) = \frac{\frac{pa}{b}}{\frac{pa}{b}+\frac{(1-p)(1-a)}{1-b}}
			   = \frac{pa-pab}{pa+b-ab-pb} \\
	p_M(C_0|x) = \frac{\frac{(1-p)(1-a)}{1-b}}{\frac{pa}{b}+\frac{(1-p)(1-a)}{1-b}} 
			   = \frac{b-ab-pb+pab}{pa+b-ab-pb} \\
$$


下面是PRML中相关章节摘录(67页)，

	Compensating for class priors. Consider our medical X-ray problem again, 
	and suppose that we have collected a large number of X-ray images from the general 
	population for use as training data in order to build an automated screening
	system. Because cancer is rare amongst the general population, we might find
	that, say, only 1 in every 1,000 examples corresponds to the presence of cancer.
	If we used such a data set to train an adaptive model, we could run into
	severe difficulties due to the small proportion of the cancer class. For instance,
	a classifier that assigned every point to the normal class would already achieve
	99.9% accuracy and it would be difficult to avoid this trivial solution. Also,
	even a large data set will contain very few examples of X-ray images corresponding
	to cancer, and so the learning algorithm will not be exposed to a
	broad range of examples of such images and hence is not likely to generalize
	well. A balanced data set in which we have selected equal numbers of examples
	from each of the classes would allow us to find a more accurate model.
	However, we then have to compensate for the effects of our modifications to
	the training data. Suppose we have used such a modified data set and found
	models for the posterior probabilities. From Bayes’ theorem (1.82), we see that
	the posterior probabilities are proportional to the prior probabilities, which we
	can interpret as the fractions of points in each class. We can therefore simply
	take the posterior probabilities obtained from our artificially balanced data set
	and first divide by the class fractions in that data set and then multiply by the
	class fractions in the population to which we wish to apply the model. Finally,
	we need to normalize to ensure that the new posterior probabilities sum to one.
	Note that this procedure cannot be applied if we have learned a discriminant
	function directly instead of determining posterior probabilities.