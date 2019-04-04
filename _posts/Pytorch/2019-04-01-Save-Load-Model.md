---
layout: post
title: Saving and Loading Models
category: Pytorch
tags: pytorch
description:
---

## 1. state_dict

### 1.1 torch.nn.Module.state_dict

```python
torch.nn.Module.state_dict(destination=None, prefix='', keep_vars=False)
```

返回一个包含模型状态信息的字典。包含参数（weighs and biases）和持续的缓冲值（如：观测值的平均值）。只有具有可更新参数的层才会被保存在模型的 state_dict 数据结构中。

**eg**

```python
module.state_dict().keys()
# ['bias', 'weight']

# visiualize model
model = torch.load('tensors.pth')
for param_tensor in m['state_dict']:
    print('{0:<50}{1:<40}{2:<10}{3:<10}'.format(param_tensor,
                                        str(model['state_dict'][param_tensor].size()),
                                        str(model['state_dict'][param_tensor].device),
                                        str(model['state_dict'][param_tensor].requires_grad)))
```

### 1.2 torch.optim.Optimizer.state_dict

```python
torch.optim.Optimizer.state_dict()
```

返回一个包含优化器状态信息的字典。包含两个词目：

- state：字典，保存当前优化器的状态信息。不同优化器内容不同。
- param_groups：字典，包含所有参数组（eg：超参数）

## 2. Core Function

### 2.1 torch.save

保存一个对象到一个硬盘文件上。

```python
torch.save(obj, f, pickle_module=<module 'pickle' from '.../pickle.py'>, pickle_protocol=2)
```

- obj：保存对象
- f：类文件对象 (必须实现写和刷新)或一个保存文件名的字符串
- pickle_module：用于 pickling 元数据和对象的模块
- pickle_protocol：指定 pickle protocal 可以覆盖默认参数

**eg**

```python
# Save to file
x = torch.tensor([0, 1, 2, 3, 4])
torch.save(x, 'tensor.pt')

# Save to io.BytesIO buffer
buffer = io.BytesIO()
torch.save(x, buffer)
```

### 2.2 torch.load

torch.load() 使用 Python 的 解压工具（unpickling）来反序列化 pickled object 到对应存储设备上。首先在 CPU 上对压缩对象进行反序列化并且移动到它们保存的存储设备上，如果失败了（如：由于系统中没有相应的存储设备），就会抛出一个异常。用户可以通过 register_package 进行扩展，使用自己定义的标记和反序列化方法。

```python
torch.load(f, map_location=None, pickle_module=<module 'pickle' from '.../pickle.py'>)
```

- f：类文件对象 (返回文件描述符)或一个保存文件名的字符串
- map_location：一个函数或字典规定如何映射存储设备
- pickle_module：用于 unpickling 元数据和对象的模块 (必须匹配序列化文件时的 pickle_module )

**eg:**

```python
torch.load('tensors.pt')

# Load all tensors onto the CPU
torch.load('tensors.pt', map_location=torch.device('cpu'))

# Load all tensors onto the CPU, using a function
torch.load('tensors.pt', map_location=lambda storage, loc: storage)

# Load all tensors onto GPU 1
torch.load('tensors.pt', map_location=lambda storage, loc: storage.cuda(1))

# Map tensors from GPU 1 to GPU 0
torch.load('tensors.pt', map_location={'cuda:1':'cuda:0'})

# Load tensor from io.BytesIO object
with open('tensor.pt') as f:
    buffer = io.BytesIO(f.read())
torch.load(buffer)
```

### 2.3 torch.nn.Module.load_state_dict

用来加载模型参数。将 state_dict 中的 parameters 和 buffers 复制到此 module 及其子节点中。

```python
torch.nn.Module.load_state_dict(state_dict, strict=True)
```

- state_dict(dict)：保存 parameters 和 persistent buffers 的字典
- strict(bool, optional)：state_dict 中的 key 是否和 model.state_dict() 返回的 key 一致。

## 3. Save & Load for inference

在加载模型进行推理时，必须使用 model.eval()，在运行推断之前将 dropout 和 batch 规范化层设置为评估模式。

1. 仅保存和加载模型参数（推荐）—— .pt/.pth

    ```python
    # save
    torch.save(model_object.state_dict(), 'params.pth')

    # load
    model_object.load_state_dict(torch.load('params.pth'))
    model_object.eval()
    ```

2. 保存和加载整个模型 —— .pkl

    以这种方式保存模型时将会用pickle模块把整个模型序列化保存，pickle没有保存模型本身, 而是保存了一个包含类的文件的路径。

    ```python
    # save
    torch.save(model_object, 'model.pkl')  

    # load
    model = torch.load('model.pkl')
    model.eval()
    ``` 

## 4. Save & Load a general for resume or inference

保存格式：.tar

```python
# save
# 先将需要保存的数据组织成字典的形式，然后用torch.save()保存

torch.save({
    'epoch': epoch, # 轮数
    'state_dict': model.state_dict(), # 模型参数
    'optimizer_state': optimizer.state_dict(), # 优化器参数
    'loss': loss,   # 损失函数
    ...
    },'checkpoint.tar')

# load
# 先初始化模型, 在利用 torch.load() 函数加载需要的数据
model = TheModelClass(*args, **kwargs)
optimizer = TheOptimizerClass(*args, **kwargs)

checkpoint = torch.load('checkpoint.tar')
model.load_state_dict(checkpoint['state_dict'])
optimizer.load_state_dict(checkpoint['optimizer_state'])
epoch = checkpoint['epoch']
loss = checkpoint['loss']

model.eval()
# - or -
model.train()
```

## 5. Save & Load multiple models in one file

保存格式：.tar

```python
# save
torch.save({
    "modelA_state_dict": modelA.state_dict(),
    "modelB_state_dict": modelB.state_dict(),
    "optimizerA_state_dict": optimizerA.state_dict(),
    "optimizerB_state_dict": optimizerB.state_dict(),
    ...
    }, 'checkpoint.tar')

# load
modelA = TheModelAClass(*args, **kwargs)
modelB = TheModelBClass(*args, **kwargs)
optimizerA = TheOptimizerAClass(*args, **kwargs)
optimizerB = TheOptimizerBClass(*args, **kwargs)

checkpoint = torch.load('checkpoint.tar')
modelA.load_state_dict(checkpoint["modelA_state_dict"])
modelB.load_state_dict(checkpoint["modelB_state_dict"])
optimizerA.load_state_dict(checkpoint["optimizerA_state_dict"])
optimizerB.load_state_dict(checkpoint["optimizerB_state_dict"])

modelA.eval()
modelB.eval()
# - or -
modelA.train()
modelB.train()
```

## 6. Save & Load model for transfer learning

```python
# save
torch.save(modelA.state_dict(), PATH)

# load
modelB = ThemodelBClass(*args, **kwargs)
modelB.load_state_dict(torch.load(PATH), strict=False)
```

## 7. Save & Load model across devices

### 7.1 Save on GPU, Load on CPU

```python
# save
torch.save(model.state_dict(), PATH)

# load
# 通过 torch.load() 函数的 map_location 参数来指定将模型的 state_dict 加载到哪个设备上
device = torch.device("cpu")
model = TheModelClass(*args, **kwargs)
model.load_state_dict(torch.load(PATH, map_location=device))
```

### 7.2 Save on GPU, Load on GPU

```python
# save
torch.save(model.state_dict(), PATH)

# load
device = torch.device("cuda:0")
model = TheModelClass(*args, **kwargs)
model.load_state_dict(torch.load(PATH))
model.to(device)

# 使用 .to 来将模型中的参数tensor转移到GPU设备上, 需要注意的是, 在进行训练或者预测之间, 还需要调用 tensor 的 .to() 方法来将 tensor 也转移到 GPU 设备上, 另外, 注意, mytensor.to(device) 实际上是在 GPU 中创建了 mytensor 的副本, 而并没有改变 mytensor 的值, 因此, 需要写成这样的形式来使得 mytensor 的值改变: my_tensor = my_tensor.to(torch.device("cuda"))
```

### 7.3 Save on CPU, Load on GPU

```python
# save
torch.save(model.state_dict(), PATH)

# load
device = torch.device("cuda")
model = TheModelClass(*args, **kwargs)
model.load_state_dict(torch.load(PATH, map_location="cuda:0"))
model.to(device)
# 由于模型是在CPU上存储的, 因此在模型加载时, 需要设置 torch.load() 函数的 map_location 参数为 cuda:0. 然后, 还需要调用 model 的 .to(device) 方法来将model的参数 tensor 全部转移到 GPU 上去, 另外别忘了将数据也要转移到 GPU 上去, my_tensor = my_tensor.to(torch.device("cuda")).
```

### 7.4 Saving torch.nn.DataParallel Models

```python
# save
torch.save(model.module.state_dict(), PATH)

# load
# Load to whatever device you want
```