# Average Quantization: High-Quality 0.x-bit Activation to Reduce Memory Usage for Training

This repository is the official implementation of https://openreview.net/forum?id=YgC62m4CY3r.

+ The proposed learning rule store auxiliary activations instead of actual input activations during forward propagation.
+ The proposed learning rule reduces training memory requirements without training speed reduction while achieving high performance close to backpropagation.

## Abstract

As the size of deep learning models increases, their performance improves according to scaling laws. However, training such large models requires a significant amount of memory and hence cannot be realized without a considerable number of expensive GPUs. Recently proposed activation compression techniques alleviate memory requirements by compressing activations with different compression rates based on their sensitivity to minimize accuracy degradation. However, when we deeply compress activations with the target precision close to 1 bit, a large portion of activations would be represented in 1 bit regardless of their sensitivity since 0.x-bit precision is not supported in those methods, severely degrading training accuracy. To address this issue, we propose to represent a group of activations with a single approximate value, effectively producing 0.x-bit activations. We prove that the boundary function for the gradient variance between the actual and the approximate activation is minimized when the activations are approximated using their average values. Based on this observation, we propose Average Quantization which provides high-quality 0.x-bit activations by replacing a group of activations with a single average value and then performing quantization. By assigning 0.x-bit precision to activations with low sensitivity, we can allocate more bits to activations with higher sensitivity, which can result in a better trade-off between accuracy and memory saving. In experiments, the proposed Average Quantization successfully trains various models with a high compression rate of up to 22.6$\times$, which translates to a significantly higher compression rate for similar training performance compared to prior methods.
![gitfig](https://user-images.githubusercontent.com/114454500/192741100-52b870ac-21e0-40de-bac6-2a9c11bfae1d.png)

## Install

+ Requirements
```bash
conda env create -f aal.yaml
conda activate aal
```

+ Buld AAL:
```bash
git clone
cd setpup/aal
pip install -v -e .
```

We also slightly modified code of ActNN and Mesa to apply it with aal simultaneously.

+ Buld ActNN for AAL
```bash
pip install -v -e .
```
+ Buld Mesa for AAL
```bash
cd setpup/mesa
python setup.py develop
```

## Usage 

+ Implementing ARA
```python
from aal.aal import Conv2d_ARA, Distribute_ARA

# define convolution layer wich uses ARA
self.conv1 = nn.Conv2d(64,64,3,1,1)
self.conv2 = Conv2d_ARA(64,64,3,1,1)
self.conv3 = Conv2d_ARA(64,64,3,1,1)
# define Distribute_ARA which is layer for implementing ARA
self.dsa = Distribute_ARA()

# define auxiliary residual activation for updating Conv2d_ARA
ARA = x.clone()
# doing backpropagation for conv1
x = self.conv1(x)
# adding auxiliary activation to output activation (residual connection)
# and propagating to Conv2d_ARA
x += ARA
x, ARA = self.conv2(x, ARA)
X += ARA
x, ARA = self.conv3(x, ARA)
# Distribute ARA makes self.conv2 and self.conv3 updates weight with ARA, not x!
x, ARA = self.dsa(x, ARA)
```

+ Implementing ASA
```python
from aal.aal import Linear_ASA

# define linear layer for ASA
self.linear = Linear_asa(256,256)

# propagating to Linear_ASA layer
# it would perform add 1-bit auxiliary sign activation during forward propagation
# and store this 1-bit auxiliary sign activation.
# during backprop, it would update weights by 1-bit auxiliary sign activation
x = self.linear(x)
```

## Example

[ResNet](https://github.com/asdfasgqergadsad/Auxiliary_Activation_Learning/tree/main/experiments/ResNet)

[Transformer](https://github.com/asdfasgqergadsad/Auxiliary_Activation_Learning/tree/main/experiments/Transformer)

[BERT_L](https://github.com/asdfasgqergadsad/Auxiliary_Activation_Learning/tree/main/experiments/BERT_L)

[ViT_L](https://github.com/asdfasgqergadsad/Auxiliary_Activation_Learning/tree/main/experiments/ViT_L)

[MLP-Mixer_L](https://github.com/asdfasgqergadsad/Auxiliary_Activation_Learning/tree/main/experiments/MLP-Mixer_L)


## Results

![result1](https://user-images.githubusercontent.com/114454500/192739402-7dc05b90-f19a-482a-995c-ce66a3949abd.png)
  
![result2](https://user-images.githubusercontent.com/114454500/192739585-98379113-d735-47e9-b72a-84e92028b3b3.png)

 
## Acknowledgments
  
  In this repository, code of [ActNN](https://github.com/ucbrise/actnn) and [Mesa](https://github.com/ziplab/Mesa) are modified to apply with our AAL.
  Thanks the authors for open-source code.
  
 ## Lisense

> All content in this repository is licensed under the MIT license. 
