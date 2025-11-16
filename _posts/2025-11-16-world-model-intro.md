---
layout: post
title:  LLM之后可能的另外一个新热门领域-World Model
categories: [LLM,World Model]
---

最近，图灵将得主Yann LeCun被爆出将要[离开Meta](https://www.nasdaq.com/articles/metas-chief-ai-scientist-yann-lecun-depart-and-launch-ai-start-focused-world-models)，并组建一个初创公司从事World Model相关的工作。作为Meta前首席科学家，多次公开表示LLM并不是通往AGI的道路，这种观点明显和Meta CEO 扎克伯格的主张背道而驰。小扎在LLM上投入了很多，但自家的开源LLM llama最近的更新也表现拉跨，首席科学家也在高调“被刺”他，那小扎就高薪雇一个28岁的年轻人总管Meta AI，并且要求LeCun向其汇报。所以，LeCun离职去研究World Model也就不足为奇。小扎有自己的商业目标，LeCun有自己的学术追求，无所谓对与错，正所谓道不同，不相为盟，好聚好散也挺好。这里其实有个职场潜规则：你要想在当前的职位继续发展，就需要支持你的领导的决策，至少不能公开反对，即使你心怀不满。强如LeCun，违背了这个原则仍然要走人。

所以，World Model成功的引起的笔者的好奇心，到底是一个什么技术，使得图灵奖得主宁愿辞去Meta首席科学家的职位，也要去探索这个新的方向。经过一番搜寻和整理，下面是World Model发展的大致时间线：

* 2014	VAE	引入 latent representation
* 2015	Video Prediction Models	世界模型的萌芽
* 2018	World Models	“世界模型”概念成形
* 2019	PlaNet	RSSM 开创现代世界模型结构
* 2020	Dreamer	imagination-based learning
* 2021	DreamerV2	离散 latent
* 2022	DreamerV3	多领域 SOTA
* 2023	Diffusion World Models	强视觉生成能力
* 2024	Genie	像素级可交互世界生成
* 2024	JEPAG	世界模型框架标准化
* 2025	LM as World Model	世界模型进入多模态 AGI 方向

这里面，2018年的这篇工作[World Model](https://arxiv.org/abs/1803.10122)定义了世界模型的概念，截至目前位置该文章被引用1,700+。该文章提出一种V+M+C的分层架构来模拟世界，以及训练智能体，使得智能体可以在“梦境”中进行训练，而不是需要直接和环境进行交互。这里的“梦境”就是世界模型的一种实现。其中V是对可视化世界的编码，利用VAE将现实世界编码到隐空间。M是基于MDN-RNN（序列预测模型）来学习世界的变化规律，这个世界是在隐空间里面。M和V训练的数据，都是通过rollout随时生成的，无监督训练，成本较低。最终V和M训练好后，用来训练智能体C，由于这篇文章是专注世界模型，所以智能体C的结构相对简单，但是足以说明问题。这篇文章提出了新的范式：潜在空间预测，并解除控制器训练对环境的依赖。所以被视为该领域的里程碑。

技术博客断更好久了，近期会续上，记录自己对生活和工作的思考。











