# 从零到一理解卷积神经网络

## 以Pytorch为例

### 1. 准备数据

Tensor：多维数组，存储数据

DataLoader：加载数据的加载器，可以进行shuffle

### 2. 定义神经网络

nn.Module：基类，由此派生模型类，定义网络层结构，以及前向传播函数

Modules:卷积层，全连接，激活函数

Containers：nn.Sequential

Loss Functions：nn.MSELoss，nn.CrossEntropyLoss

Functional Interface：nn.functional(F)

```python
class SimpleNet(nn.Module):
	def __init__(self):
        super(SimpleNet,self).__init__()
        self.fc1=nn.Linear(10,20)
        self.relu=nn.ReLU()
        self.fc2=nn.Linear(20,1)
    def forward(self,x):#guidebook:x->fc1->relu->fc2->output
		x=self.fc1(x)
        x=self.relu(x)
        x=self.fc2(x)
        return x
```

### 3. 定义损失函数以及优化器

#### 3.1 损失函数

```python
loss_fn=nn.MSELoss()
```

#### 3.2 优化器

参考梯度下降

```python
optimizer=torch.optim.Adam(model.parameters(),lr=1e-3,betas(0.9,0.999))
```

loss.backward()->计算更新梯度grad

optimizer.step()->利用梯度进行更新 param=param-lr*grad

### 4. 训练网络

前向传播->计算损失->反向传播->更新参数

epoch:一个epoch中全部的训练数据集都被看过一遍

可将一个epoch分成多个batch进行训练

预测->算损失->反向->更新

```python
pred=model(x)
loss=loss_fn(pred,y)
optimizer.zero_grad()
loss.backward()
optimizer.step()
```

### Ps. 详情结果可看zeroToOne.ipynb

