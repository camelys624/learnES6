# 应答器设置规则

## 一般规则

1. 应答器组内相邻应答器的距离为 5±0.5m
2. 发送线路参数的应答器组，车站正线组内应答器组距绝缘节距离不小于 30m。应答器组距绝缘节距离从靠近绝缘节的应答器计算。
3. CTCS-2 和 CTCS-3 级列控系统的应答器组内应答器数量不宜超过3个，发送线路参数的应答器组由两个及以上应答器构成。
4. 设置在车站的应答器组中的有源应答器应靠近信号机侧。

## 命名规则

1. 每个应答器 (组) 命名应以 B 开头，后加公里标或信号机名称，其中公里标参照区间通过信号机命名规则执行，即应答器名称以该应答器 (组) 所在位置坐标公里数和百米数组成，对于 km 后的单位采用四舍五入的方式计算,下行编号奇数，上行编号偶数。
2. 应答器名称应区分应答器组内的位置，分别在应答器名称后加 "-1","-2" 等表示组内第一个应答器和第二个应答器信息。

## 编号规则

1. 应答器编号应接 "大区号-分区号-车站号-应答器单元编号-组内编号" 格式填写。
2. 单元编号规则
  - 应答器单元编号以列车运行正方向或用途为参照，按正线贯通，从小到大的原则进行编号，下行为编号奇数 ，上行为偶数。
  - 单元编号由三位十进制表示，编号范围为 1-255。
3. 大区编号由三位十进制表示，编号范围为 1-127。
4. 分区号由一位十进制表示，编号范围为 1-7。
5. 车站编号规则
  - 车站编号由两位十进制表示，编号范围为 1-60，一个分区的车站数量一般不超过 50 个进行分配。
  - 接分区内车站的下行方向顺次进行车站编号。

## 里程规则

1. 里程应填写应答器安装的实际线路运营里程 (格式为 KXXX + XXX)，精确到米，以靠近的信号机里程为参照点。

## 类型

1. 空心三角形 "△" 表示无源应答器。
2. 实心三角形 "▲" 表示有源应答器。

## 用途

1. CTCS-0 站应答器组 [c2-c0] 设置。
2. CTCS-0 车站向 CTCS-2 区域方向出战口 (含反向) 上下行各设置两个有源应答器，向列车发送线路数据和临时限速信息。

## UML 数据建模属性

属性描述：

| 属性       | 中文名 | 数据类型   |
| ---------- | ------ | ------ |
| B-Name     | 名称   | String |
| B-Num      | 编号   | String |
| B-Location | 里程   | String |
| B-Type     | 类型   | String |


## 安装依赖包步骤

1. 第一步，win + R 打开运行
2. 第二步，输入 cmd 回车
3. 第三步，输入 `pip instll xxx` 'xxx' 就是需要安装的包名
