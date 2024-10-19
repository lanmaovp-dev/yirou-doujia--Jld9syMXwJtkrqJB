

> 尽管文本到图像的扩散模型已被证明在图像合成方面达到了最先进的结果，但它们尚未证明在下游应用中的有效性。先前的研究提出了在有限的真实数据访问下为图像分类器训练生成数据的方法。然而，这些方法在生成内部分布图像或描绘细粒度特征方面存在困难，从而阻碍了在合成数据集上训练的分类模型的泛化能力。论文提出了`DataDream`，一个合成分类数据集的框架，在少量目标类别示例的指导下，更真实地表示真实数据分布。
> 
> 
> `DataDream`在生成训练数据之前，对图像生成模型的`LoRA`权重进行微调，使用少量真实图像。然后，使用合成数据微调`CLIP`的`LoRA`权重，以在各种数据集上改善下游图像分类性能，超越先前的方法。
> 
> 
> 通过大量实验展示了`DataDream`的有效性，在`10`个数据集中的`7`个数据集上，以少量示例数据超越了最先进的分类准确率，同时在其他`3`个数据集上也表现竞争力。此外，论文提供了关于多种因素的影响的见解，例如真实图像和生成图像的数量以及微调计算对模型性能的影响。
> 
> 
> 来源：晓飞的算法工程笔记 公众号，转载请注明出处


**论文: DataDream: Few\-shot Guided Dataset Generation**


![](https://developer.qcloudimg.com/http-save/6496381/a414d420f6a2bc49a7c76a99c824dbc8.png)


* **论文地址：[https://arxiv.org/abs/2407\.10910](https://github.com)**
* **论文代码：[https://github.com/ExplainableML/DataDream](https://github.com):[樱花宇宙官网](https://yzygzn.com)**


# Introduction




---


文本到图像生成模型的出现，例如稳定扩散（`Stable Diffusion`），不仅能够创建照片真实感的合成图像，还为增强下游任务提供了机会。一个潜在的应用是在合成数据上训练或微调特定任务的模型。这在真实数据获取有限的领域尤其有用，因为生成模型提供了一种经济高效的方式来生成大量训练数据。论文研究了合成训练数据在低样本设置下对图像分类任务的影响，即当每个类别只有少量图像可用，但收集整个数据集的成本将是难以承受的。


![](https://developer.qcloudimg.com/http-save/6496381/b41bb137c6a050295b4799f119450d88.png)


之前的研究主要集中在使用给定数据集的类名称来指导数据生成过程。具体来说，他们使用文本到图像扩散模型生成图像，将类名称作为条件输入。为了更好地引导模型生成目标对象的准确描绘，他们将每个类的文本描述纳入提示中，这些描述来自语言模型或人工标注的类描述。尽管这些方法直观，但导致一些生成的图像缺乏所关注的对象。例如，来自`ImageNet`数据集的类名称“`clothes iron`”的真实图像显示的是用于熨烫衣物的电器，而`FakeIt`生成的图像大多描绘的是金属熨斗或由其制成的任意物体（见图`1`，左侧）。这种情况发生在生成模型误解类名称的模糊性或稀有类别时。现实图像与合成图像之间的这种不一致限制了生成图像在图像分类中的信息价值，并阻碍了性能的提升。


为了弥合真实图像与合成图像之间的差距，真实图像可以更好地为生成模型提供有关真实数据分布特征的信息。例如，正在同时开发的`DISEF`方法在生成合成数据集时，从部分带噪声的真实图像开始，将少量样本作为条件输入到预训练的扩散模型中。它还使用预训练的图像描述模型来多样化文本到图像的提示。虽然这种方法改善了真实数据和合成数据分布的对齐，但有时未能捕捉到细粒度特征。例如，尽管航空数据集中“`DHC-3-800`”类名称的真实图像在机翼前包含一个螺旋桨，但`DISEF`生成的合成图像缺乏这个细节（见图`1`，右侧）。准确表示类区分特征对分类任务来说可能至关重要，尤其是在细粒度数据集中。


为此，论文提出了一种新方法`DataDream`，旨在利用少量真实数据来适应生成模型。受到个性化生成建模方法的启发，这些方法通过少量描绘相同对象的真实图像对生成模型进行微调，该方法侧重于将生成模型对齐到一个具有多类和每类多样化对象的目标数据集。这与之前的少量样本数据集生成方法不同，后者并未探索微调生成模型的可能性。


具体来说，通过两种方式基于`LoRA`来调整`Stable Diffusion`： DataDreamclsDataDreamcls ，为每个类训练`LoRA`，以及 DataDreamdsetDataDreamdset ，为所有类训练一个`LoRA`。论文是首个提出使用少量样本数据来适应生成模型以生成合成训练数据的方法，而不是利用已冻结的预训练生成模型。在训练之后，使用相同的提示生成图像，该提示用于微调`DataDream`，生成的图像描绘了所关注的对象（例如衣物熨斗）或细粒度特征（例如`DHC-3-800`飞机的螺旋桨），如图`1`的最后一行所示。


通过大量实验验证了`DataDream`的有效性，只使用合成数据时，在所有数据集中达到了最先进的水平，并且在同时使用真实少量样本和合成数据进行训练时，在`10`个数据集中有`7`个获得了最佳性能。为了理解该方法的有效性，论文分析了真实数据与合成数据之间的对齐情况，揭示了该方法在与真实数据分布的对齐方面优于基线方法。最后，通过增加合成数据点和真实样本的数量，探讨了该方法的可扩展性，显示了更大数据集的潜在好处。


总之，论文的贡献如下：


1. 引入了`DataDream`，一种新颖的少量样本方法，该方法改进了`Stable Diffusion`，以生成更好的同类分布图像，从而用于下游训练。在`10`个数据集中，`DataDream`在`7`个上超过了最先进的少量样本分类表现，其余`3`个数据集的表现则相当。
2. 强调仅使用合成数据报告结果的重要性。证明当仅使用合成数据训练分类器时，论文的方法能够取得更优的性能，在某些情况下甚至超过了仅使用真实少量样本图像训练的分类器，这表明论文的方法生成的图像能够从少量真实数据中提取出更具洞察力的信息。
3. 通过分析合成数据与真实数据之间的分布对齐情况来研究论文方法的有效性。在少量样本的指导下，该方法生成的合成数据与真实数据的对齐效果最佳。


# Methodology




---


## Preliminaries


* ### Latent diffusion model


论文的方法基于`Stable Diffusion`实现，这是一种概率生成模型，通过文本提示学习生成真实的图像。给定数据 (x,c)∈D(x,c)∈D ，其中 xx 是一幅图像， cc 是描述 xx 的标题，该模型通过逐渐去噪潜在空间中的高斯噪声来学习条件分布 p(x\|c)p(x\|c) 。给定一个预训练的编码器 EE ，它将图像 xx 编码为潜在变量 zz ，即 z\=E(x)z\=E(x) ，目标函数定义为：


minθE(x,c)∼D,ϵ∼N(0,1),t\[‖ϵ−ϵθ(zt,τ(c),t)‖22],其中 t 是时间步， zt 是距离潜在变量 z t 步的潜在带噪声数据， τ 是文本编码器， ϵθ 是潜在扩散模型。直观上，参数 θ 被训练用于去噪给定文本提示 c 作为条件信息的潜在 zt 。在推理阶段，一个随机噪声向量 zT 通过潜在扩散模型进行了 T 次传递，并与标题 c 一起，得到去噪后的潜在变量 z0 。随后，将 z0 输入到一个预训练的解码器 D 中，以生成图像 x′\=D(z0) ，用于文本到图像的生成。


* ### Low\-rank adaptation


低秩适配方法（`LoRA`）是一种微调方法，用于以参数高效的方式将大型预训练模型调整到下游任务。给定预训练模型权重 θ∈Rd×k ，`LoRA`引入一个新的参数 δ∈Rd×k ，该参数被分解为两个矩阵， δ\=BA ，其中 B∈Rd×r ， A∈Rr×k ，且具有较小的`LoRA`秩 r ，即 r≪min(d,k) 。`LoRA`权重添加到模型权重中以获得微调后的权重，即 θ(ft)\=θ\+δ ，以适应下游任务。在训练过程中， θ 保持固定，而仅更新 δ 。


## DataDream method


论文的目标是通过利用扩散模型生成的合成图像来提高分类性能，至关重要的是将合成图像的分布与真实图像的分布对齐。通过将扩散模型调整为少量真实图像的数据集来实现这种对齐。


![](https://developer.qcloudimg.com/http-save/6496381/c675edf9c06598855deb87b0ab8ce784.png)


假设可以访问一个少量样本的数据集 Dfs\={(xi,yi)}KNi\=1 ，其中 xi 是一张图像， yi∈{1,2,⋯,N} 是它的标签， K 是每个类别的样本数量， N 是类别的数量。为了匹配真实数据的分布，使用少量样本的数据集 Dfs 进行微调。具体来说，在扩散模型的文本编码器和 U\-net 中引入 LoRA 权重，在这里选择有效地调整注意力层的参数。对于每个注意力层，考虑查询、键、值和输出投影矩阵 Wq, Wk, Wv, Wo，在每个矩阵中，线性投影被替换为


hl,⋆\=W⋆hl−1\+B⋆A⋆hl−1其中 h 表示投影的输入/输出激活，最终得到每个注意力层 l 的可训练`LoRA`权重 δ(l)\={A⋆,B⋆\|∀⋆∈{q,k,v,o}} 。为了简化符号，省略偏置权重。所有其他模型参数（包括 W⋆ ）保持不变，而 δ 权重则通过梯度下降进行优化。


为了从预训练的扩散模型`checkpoint`开始训练，权重矩阵 B⋆ 被初始化为零，而 A⋆ 则随机初始化。因此，组合的微调权重 B⋆A⋆ 最初为零，并逐步学习对原始预训练权重的修改。在测试时，`LoRA`权重可以通过更新权重 W(ft)⋆\=W⋆\+B⋆A⋆ 集成到模型中，使得推理时间与预训练模型相同。与`DreamBooth`相比，不微调所有网络权重，也不添加保留损失，因为其正则化会阻碍与真实图像的强对齐。


进一步考虑两种设置：`1`) DataDreamdset ，在该设置中，在整个数据集 Dfs 上训练扩散模型的`LoRA`权重，`2`) DataDreamcls ，在该设置中，为数据集中的每个类别初始化 N 组`LoRA`权重 {δn\|n\=1,⋯,N} ，每组权重针对子集 Dfsn\={(x,y)\|(x,y)∈Dfs,y\=n} 进行训练。


在 DataDreamdset 设置中，原始模型参数 θ 保持不变，仅对`LoRA`权重进行训练，目标函数为


minδLD\=minδE(x,y)∼Dfs,ϵ∼N(0,1),t\[\|\|ϵ−ϵθ,δ(zt,τδ(C(y)),t)\|\|22].在 DataDreamcls 设置中， Dfsn 和 δn 分别替代 Dfs 和 δ 。由于使用的是文本到图像的扩散模型，通过函数 C 定义文本条件，该函数将标签 y （即类名）映射到使用标准模板 "`a photo of a`\[`CLS`]" 的提示。该提示会通过文本编码器传递，并在扩散模型的解码步骤中使用。


这两种设置各有不同的优势。在 DataDreamdset 中，类之间的`LoRA`权重共享允许在整个数据集内进行关于共性特征的知识转移。这对于那些在各类别中共享粗粒度特征的细粒度数据集是有益的。另一方面， DataDreamcls 为学习每个类别的细节分配了更多的权重，这使得生成模型能够更好地与每个类别的数据分布对齐。


在将扩散模型适应于少样本数据集后，使用调整后的模型在相同的文本提示条件下为每个类别生成`500`张图像，该文本提示与`DataDream`使用的相同，从而形成一个合成数据集 Dsynth 。在仅使用合成图像或合成与真实少样本图像的组合 Dfs 上训练分类器。


对于分类器的训练，调整了一个`CLIP`模型，类似于之前在少样本分类中的工作。为`CLIP ViT-B`/`16`模型的图像编码器和文本编码器添加了`LoRA`适配器。在同时使用合成图像和真实图像进行训练时，使用来自真实数据和合成数据的损失的加权平均。


LC\=λE(x,y)∼DfsCE(f(x),y)\+(1−λ)E(x,y)∼DsynthCE(f(x),y),其中 λ 是分配给来自真实数据的损失的权重，函数 CE 是交叉熵损失。


* ### Implementation details


基于`Stable Diffusion`版本`2.1`实现了`DataDream`，计算基于三个随机种子。对于每个种子，从每个数据集的训练样本中随机抽样少量图像。在所有数据集上训练`200`个周期，批量大小为`8`，唯一的例外是 DataDreamdset 在`ImageNet`上训练`100`个周期。因此， DataDreamdset 和 DataDreamcls 有相同的训练计算量，即每 N 个 DataDreamcls 适配器权重（每类一个）执行 S/N 次更新步骤，其中 S 是整个数据集的 DataDreamdset 的总步骤数。


使用`AdamW`作为优化器，学习率为 1e−4 ，并采用余弦退火调度器。对`DataDream`中所有适配权重使用`LoRA`级别 r\=16 。对于`DataDream`的合成图像生成，使用`50`次步骤和指导尺度`2.0`。如果未提及，则每类生成`500`张图像。对于分类器，使用`CLIP ViT-B`/`16`作为基础模型，并在`CLIP`的图像编码器和文本编码器上应用`LoRA`进行微调，级别为`16`。将分配给真实损失项的权重设置为 λ\=0\.8 。


# Experiments




---


![](https://developer.qcloudimg.com/http-save/6496381/15d242f7dd9918d0bc5113dcc275771a.png)


![](https://developer.qcloudimg.com/http-save/6496381/349e020ae96c6ae8704db03b4a3b838c.png)


 
 
 



> 如果本文对你有帮助，麻烦点个赞或在看呗～
> 更多内容请关注 微信公众号【晓飞的算法工程笔记】


![work-life balance.](https://upload-images.jianshu.io/upload_images/20428708-7156c0e4a2f49bd6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
