---
layout: post
title:  Automatic Differentiation Behind Scene
categories: [deep-learning]
---



Automatic differentiation is one of the foundational functions for deep learning libraries, such as Tensorflow, PyTorch and Theano. As deep learning is becoming the main solution for machine-learning problems, it's necessary to know how it works behind scene.  I like to ask interviewee,  and especially the one who declares to be good at deep learning in the resume, about automatic differentiation to screen qualified candidates during the interview.

There are two other methods, numerical and symbolic differentiation, that can also be used to calculate the differentiation. Before introducing automatic differentiation, it's better to cover them at first, so that we can understand the distinction among them clearly.   



## Numerical Differentiation

Numerical differentiation in its basic form follows the definition of gradient, given that


$$
f^\prime(x) = \frac{df}{dx} = \lim_{\Delta x \rightarrow 0}\frac{f(x+\Delta x)-f(x)}{\Delta x}
$$


The key problem with numerical differentiation is the computational cost, which grows with the number of the parameters in the loss function. In deep learning models, there are always millions of parameters to be estimated, so it's impossible to only one $\Delta x$. However, numerical differentiation can be used to check if gradients are being computed correctly.



## Symbolic Differentiation

Symbolic differentiation is what we learn in calculus. We use the defined rules to calculate the differentiation, such as 


$$
\frac{d }{dx} x^n = nx^{n-1}
$$


and 


$$
\frac{d}{dx} \left(f(x) + g(x)\right) = \frac{d}{dx}f(x) + \frac{d}{dx}g(x)
$$


Now, we can successively apply the rules to calculate the gradient of $ f(x) = 2x^3+3x^2$. The key problem for symbolic differentiation is that it requires a close forme mathematical expression for the loss function. If your loss functions involves for-loop, or if-else clause , it cannot work. Another inherent problem is that it, in some cases, can lead to an explosion of symbolic terms, which make it impossible to calculate the gradient.



## Automatic Differentiation

Automatic differentiation mainly relays on symbolic differentiation, but it remove the drawbacks of symbolic differentiation. The key intuitions behind automatic differentiation are:

* Use the chain rule to construct the DAG computational graph, which is at the basis of symbolic differentiation.
* Instead of storing the intermediate results, it evaluating them directly, which avoids expression swell and can handles control flow statements (contain if-else, for-loop, etc).

Automatic Differentiation is the decomposition of differentials provided by the chain rule. For the simple composition


$$
y = f(g(h(x))) = f(g(h(w_0))) = f(g(w_1))=f(w_2)=w_3
$$


We can have followings


$$
w_0 = x \\
w_1 = h(w_0) \\
w_2 = g(w_1) \\
w_3 = f(w_2)
$$


Then, the chain rule gives


$$
\frac{dy}{dx} = \frac{dy}{dw2}\frac{dw_2}{dw_1}\frac{dw_1}{dw_0} = \frac{df(w_2)}{dw_2}\frac{dg(w_1)}{dw_1}\frac{h(w_0)}{dw_0}
$$


Usually, there are two modes to calculate the gradients, **forward modes** and **reverse modes**.

1. Forward mode traverses the chain rules from inside to outside: $\frac{dw_i}{dx} = \frac{dw_i}{dw_{i-1}}\frac{dw_{i-1}}{dx}$ 
2. Reverse mode traverses the chain rules from outside to inside: $\frac{dy}{dw_i}=\frac{dy}{dw_{i+1}}\frac{dw_{i+1}}{dw_i}$  



Theoretically, the two modes arrive at the same result, however the computational costs are significantly different. Let's take a simple function $f(x_1, x_2)=(x_1^2+x_2^2)^{\frac{1}{2}}$ for example. Its computational graph is following,

![](\_posts\deep-learning\img\simple-dag.png)



In forward mode, we should traverse the computational graph n times, which n is the number of parameters in the loss function. In this example, we traverses the computational graph 2 times as following. Imaging we have millions of parameters, it costs too much. The following example illustrates the process.

![](\_posts\deep-learning\img\foward-mode.png)

We use $\dot{v}_i = \dot{v}_{i-1}\frac{\partial  v_i}{\partial v_{i-1}}$ to compute at each node. 

![](\_posts\deep-learning\img\foward-mode-x1.png)It computes the gradient for $x_1$

![](\_posts\deep-learning\img\foward-mode-x2.png)

It computes the gradient for $x_2$.

In reverse mode, it only traverses the graph 2 times, no matter how many parameters in the loss function. Firstly, evaluate the value in each node forwardly. Then, compute the gradients from the output note reversely.  Compared with forward mode, the main advantage is that reverse mode reuses results across parameters, which dramatically reduces the unnecessary computations. The following example illustrates the process.

![](\_posts\deep-learning\img\reverse-mode.png)

We use $\bar{v}_i = \bar{v}_{i+1}\frac{\partial v_{i+1}}{\partial v_i}$ to compute at each node.

![](\_posts\deep-learning\img\reverse-mode-forward-pass.png)

It traverses forwardly.

![](\_posts\deep-learning\img\reverse-mode-bardward-pass.png)

It traverse backwardly.



OK, that is all, hoping it could be helpful. The following is the reference:

* Deep Learning with Python : A Hands-on Introduction. Chapter 9 Automatic Differentiation
* [Automatic Differentiation on Wiki](https://en.wikipedia.org/wiki/Automatic_differentiation)





