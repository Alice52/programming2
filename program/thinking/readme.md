## 思想

1. 遇到困难的问题

   - 回归现实: 从人的角度思考怎么实现
   - 减小排查范围, 确定出问题的地方

2. 12306 这样的技术不能解决的问题

   - 有些事需要业务上的妥协

3. 不要为了极小概率的事情而增加系统的复杂度

   - 8-2 原则

4. 将长链路转换为短链路 可以提高并发

   - 异步延迟满足
   - 加中间状态

5. 延迟初始化开销很大的资源

   - 真正用到它的时候再执行

6. 鸵鸟算法: 不去管它

## 项目

1. 池化
2. 拆分: 小而快
3. 高并发-分治

   - 缓存: 空间换时间 || 一致性换性能
   - 异步: mq + 延迟满足
   - 无锁化

---

## reference

1. [progamming](https://github.com/Alice52/Alice52/issues/131)
2. [docs](https://github.com/Alice52/Alice52/issues/123)
3. [training](https://github.com/private-repoes/interview/issues/2)
