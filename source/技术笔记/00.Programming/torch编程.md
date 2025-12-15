# Torch编程

[toc]

## pth/pt文件

### pth内容

.pth文件通常包含两种内容：
1. 模型参数：训练过程中学习到的weight和bias等数值
2. 整个模型：除了参数外，还保存模型的结构信息，即模型的层、激活函数、损失函数等定义和配置。在加载模型时可直接从.pth文件中恢复模型结构，不需要重新定义模型结构

### pth保存和加载

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

### pth转pt

使用torch.jit.script()将pth类型模型导出为pt类型时

```
serialized_model = torch.jit.script(netG)
serialized_model.save('model.pt')
```

输出以下报错：
```
torch.jit.frontend.NotSupportedError: Compiled functions can't take variable number of arguments or use keyword-only arguments with defaults:
```

原因：jit不支持DataParallel，解决方法为不使用nn.DataParallel()，且将模型参数的key的module.前缀删除，可参考[这篇文章](https://szukevin.site/2021/02/27/MODNet%E8%BD%AC%E6%88%90torchscript%E5%BD%A2%E5%BC%8F%E9%81%87%E5%88%B0%E7%9A%84%E5%9D%91/)

## CUDA同步模式

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

## pth/pt文件
