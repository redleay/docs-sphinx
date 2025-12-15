[toc]

# pth/pt文件

## pth内容

.pth文件通常包含两种内容：
1. 模型参数：训练过程中学习到的weight和bias等数值
2. 整个模型：除了参数外，还保存模型的结构信息，即模型的层、激活函数、损失函数等定义和配置。在加载模型时可直接从.pth文件中恢复模型结构，不需要重新定义模型结构

## pth保存和加载

1. 保存模型参数

```
torch.save(model, "model.pth")
model = torch.load("model.pth")
```

2. 仅保存模型参数

```
torch.save(model.state_dict(), "model.pth")

model = ResNet.ResNet()
model.load_state_dict(torch.load("model.pth"))
```

## pth转pt

使用torch.jit.script()将pth类型模型导出为pt类型时

```
serialized_model = torch.jit.script(netG)
serialized_model.save('model.pt')
```

输出以下报错：
```
torch.jit.frontend.NotSupportedError: Compiled functions can't take variable number of arguments or use keyword-only arguments with defaults:
```

原因：
jit不支持DataParallel

解决方法：
不使用nn.DataParallel()，
且将模型参数的key的module.前缀删除，可参考
[这篇文章](https://szukevin.site/2021/02/27/MODNet%E8%BD%AC%E6%88%90torchscript%E5%BD%A2%E5%BC%8F%E9%81%87%E5%88%B0%E7%9A%84%E5%9D%91/)

# 分布式训练

[新手手册：Pytorch分布式训练](https://mp.weixin.qq.com/s/G-uLl3HXzFJOW03nA7etig)  
[pytorch分布式训练](https://qiankunli.github.io/2021/10/30/pytorch_distributed.html)  
[Ring AllReduce简介](https://mp.weixin.qq.com/s/K8l7H2zCUr9sGzYehizFMA)  

## 数据并行

## 模型并行

# CUDA同步模式

设置CUDA同步执行，方便统计执行耗时：

```
# python
torch.cuda.synchronize()
start_time = time.time()
outputs = civilnet(img)
torch.cuda.synchronize()
print('gemfield model_time: ',time.time()-start_time)

# C++
#include <chrono>
#include <c10/cuda/CUDAStream.h>
#include <ATen/cuda/CUDAContext.h>
start = std::chrono::system_clock::now();
output = civilnet->forward(inputs).toTensor();
at::cuda::CUDAStream stream = at::cuda::getCurrentCUDAStream();
AT_CUDA_CHECK(cudaStreamSynchronize(stream));
forward_duration = std::chrono::system_clock::now() - start;
msg = gemfield_org::format(" time: %f",  forward_duration.count() );
std::cout<<"civilnet->forward(inputs).toTensor() "<<msg<<std::endl;
```

可以设置以下环境变量来近似设置CUDA同步执行模式：

```
export CUDA_LAUNCH_BLOCKING=1
```

# 常用函数

```
self.down = nn.Conv2d(nf, fm_nf, 1, 1, 0, bias = True)

p = torch.arange(0, fm_s_k, 1)
self.p = nn.Parameter(((p + 0.5)* self.pi / fm_s_k).view(1, 1, 1, fm_s_k, fm_s_k))

self.padding = nn.ZeroPad2d(1)


torch.sigmoid(out)
torch.nn.functional.softmax(out, dim=1)

out = out.permute(0, 2, 3, 1)
out = out.contiguous()
out = out.view(N, H * W, 2, K)

out = out[:, :, 0, :].view(N , H * W, K, 1, 1)
out = out.expand([-1, -1, -1, self.fm_s_k, self.fm_s_k])

p = p.expand([N, H * W, K, -1, -1])

kernel = torch.cos(hFrequency * p) * torch.cos(wFrequency * q)
kernel = torch.matmul(weight, kernel)

v = torch.nn.functional.unfold(x, kernel_size=sk, padding=int((sk - 1) / 2), stride=1)

z = z.squeeze(-1)

torch.stack((u, v), dim=2)
torch.nn.functional.grid_sample(model, uv_index, align_corners=True)
```

## meshgrid()

生成二维或三维的网格

```
x = torch.tensor([1, 2, 3])
y = torch.tensor([4, 5, 6, 7])
grid_x, grid_y = torch.meshgrid(x, y)

# 输出结果：
# grid_x
# tensor([[1, 1, 1, 1],
#         [2, 2, 2, 2],
#         [3, 3, 3, 3]])
# grid_y
# tensor([[4, 5, 6, 7],
#         [4, 5, 6, 7],
#         [4, 5, 6, 7]])
```

入情入理的网络与输入张量有如下关系：

```
grid_x[idx, :] = x[idx]
grid_y[:, idx] = y[idx]
```

## nn.functional.unfold()

手动实现(卷积)滑动窗口操作，也就是只有卷没有积

输入特征的维度为`[B, C, H, W]`，在这个输入特征的`H`和`W`维度上，根据设定的kernel、stride、dilation连续地取出特征。

输出三维张量`[B, C * kH * kW, L]`，其中`kH`和`kW`表示kernel size，`L`则是kernal在`H x W`空间上滑动的次数。

[torch.nn.functional.unfold的简单理解与用法](https://blog.csdn.net/qq_40714949/article/details/112836897)  
[torch.nn.functional.unfold函数使用详解](https://blog.csdn.net/qq_34914551/article/details/102940368)  

