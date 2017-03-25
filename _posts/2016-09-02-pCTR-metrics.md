---
layout: post
title:  "点击率预估的metrics"
date:   2016-09-02
categories: jekyll update
---

__Related Paper__:[Predictive Model Performance: Offline and Online Evaluations](http://chbrown.github.io/kdd-2013-usb/kdd/p1294.pdf)

__Data__: 1) Bings search data, July,Aug. 2012; 2) Microsoft contextual ads.

__Online Metrics__: 每页搜索结果的广告点击数目 Click yield (CY)

__Offline Metrics__: AUC, MAE

## Drawbacks
具体数据例证可参见论文配图、表，此处仅作总结

#### AUC

假设，对于每个广告位，当前model已经预测到正确的CTR值，则

1. 当“维持model的CTR pred不变，而增加低点击率的neg ads数目“时
   - 应当：model效果变差（真实CTR下降，差距变大）
   - 事实：AUC反而变好（更多容易被排序正确的低ads）

2. 当“维持model的CTR pred不变，而测试数据中低点击率ads的分布变化“时
   - 应当：model的效果基本一致（我们更关注data前端分类）
   - 事实：AUC大幅下降（排序错误的可能性上升）

3. 当“维持model的CTR pred不变，而等比例增加全部neg ads数目“时
   - 应当：model的效果降低（真实CTR下降，差距变大）
   - 事实：AUC保持不变（不影响排序正确概率）

#### RIG/PE

__RIG__(Relative Info Gain), __PE__(Predictive Error), 只能描述整体值，对局部(low CTR 和 high CTR)的效果不能捕捉，而我们更关心高CTR值广告排序的正确性。

#### MAE/MSE

可能出现当预测的pCTR＝CTR时，error并不为最小值的情况，尤其是当数据不均衡的时候。

## Contribution

提出新的metric：Simulated Metric，保证线下线上performance尽量一致
