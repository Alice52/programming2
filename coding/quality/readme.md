[toc]

## 代码质量

1. **可维护性**: 修改老代码 & 添加新代码

   - 能够快速地修改或者添加代码
   - 扩展性和可读性有的时候是相冲突的
   - 如何做: 可读 & 简洁 & 扩展
   - `与语言无关 | 与行业无关 | 不等于 bug 数`

2. **可读性**: 别人阅读不会有太多疑问

   - 与性能维护相悖
   - 命名 + 注释 + 功能{函数} + 调用 + 编码规范 + 设计模块 + 模块划分 + 封装多态继承抽象{耦合}

3. **`可扩展性`**: 扩展点

   - [OCP]不修改或少修改, 新加实现
   - 功能扩展点
   - 接口 & 抽象多态继承 & 耦合 & 分层 & 模块

4. 灵活性

   - 添加新功能有扩展点 & 修改时不耦合
   - 实现功能有复用沉淀
   - 接口抽象满足各种场景

5. 简洁性: 代码简洁 & 逻辑清晰

   - _思从深而行从简_

6. **可复用性**

   - 继承 & 多态
   - 职责单一
   - 解耦内聚 & 模块
   - 设计[如适配器模式等]

7. [日志问题](https://www.yuque.com/u22083403/vglar3/ag8at662xs8sao0q)

8. 可测试性

   - 写单元测试容易
   - 单元测试不能依赖于环境: h2 内存数据库

   ```java
   // 参数列表过于复杂 + []含义不明
   private List<String> before(List<String[]> sourceInfos, Map<String, List<String>> resultMap, int pageSize, User user) {
      Map<String, Integer> sourceNumMap = new HashMap<>(sourceInfos.size());
      sourceInfos.stream().forEach(s -> sourceNumMap.put(s[0], Integer.parseInt(s[1]) * pageSize / 100));
      List<String> resultList = new ArrayList<>();
      resultMap.forEach((s, strings) -> resultList.addAll(strings.stream().limit(sourceNumMap.get(s)).collect(
         Collectors.toList())));
      // 弥补条数, 防止数据量不足
      if (resultList.size() < pageSize) {
         compensate(resultList, pageSize, user.getAliuid());
      }
      return resultList;
   }

   private List<String> after(List<SourceInfo> sourceInfos, int pageSize, String aliuid) {
      // 获取结果集
      List<String> resultList = sourceInfos.stream()
         .flatMap(sourceInfo -> {
               int needNum = (int)(sourceInfo.getSourceRatio() * pageSize / 100);
               return listSource(sourceInfo.getSourceChannel(), needNum, aliuid).stream();
         }).collect(Collectors.toList());
      // 补偿数据
      compensate(resultList, pageSize, aliuid());
      return resultList;
   }
   ```

## 代码质量细节

1. 目录模块划分是否清晰, 代码结构是否满足 OOP & "高内聚、松耦合"
2. 遵循设计原则和思想: SOLID、DRY、KISS、YAGNI、LOD
3. 设计模式使用得当
4. 代码评判

   ![avatar](/static/image/dp/design-guideline.png)
