# debanding

## v1

网络架构：UNet + FMNet 级联
数据构造：bitdepth退化为 8/7/6 bit、jpeg编码、

## v3

优化细节丢失问题

## 蓝点问题优化

原因： UNet底层（下采样倍数较大）的上采样模块pixel-shuffle引入了pattern

解决办法
- 将UNet的7层架构修改为3层，只下采样3次，避免pixel-shuffle引入上采样错误pattern
- 将FMNet从 UNet 后面移到 UNet 下底层，减少 FMNet 的输入分辨率和计算量，

