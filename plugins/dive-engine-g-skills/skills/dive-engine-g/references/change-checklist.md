# Change Checklist

## 改代码前

- 需求属于哪类链路
- 入口文件在哪里
- `business + rewardType` 是什么
- 是否命中注册中心
- 是否会影响 MQ / callback / DB / Redis / Apollo

## 改代码时

- 先复用同类实现
- 实现与注册一起改
- 若是券或现金链路，检查 sender 文件里的业务集合
- 若新增异步消息，同步改 `config/ddmq.yaml`

## 改代码后

- Go 文件已 `gofmt`
- 注册点无遗漏
- `business + rewardType` 的散点集合无遗漏
- 最终说明里标注未做编译/运行验证
