# [详细实践参考exercize.md](https://github.com/snowflowersnowflake/rust_exercize/blob/main/exercise.md)
# 开发环境指引
1. IDE可选择Clion试用版，安装Rust插件，rust编译环境使用官方命令行安装

# Rust题目1
## 背景
马斯克发布Tweet后，BTC现货币价经常有波动，对于量化交易来说，这是一种风险，也是一种机会
## 需求
1. 监控马斯克的Twitter账号有新的tweet发布时发送报警事件；
2. 报警事件发出后，每30秒输出一次BTC现货币价和发布时的涨跌幅；
3. 【可选】上述报警事件和涨跌幅如果用web页面实时展示出来，Web开发可参考使用actix_web开发框架+tera模板引擎；
4. 【可选】发送报警事件时，给出是对BTC现价是利空消息还是利多消息的概率估计

# Rust题目2
## 需求描述
对于给定的kline的离线数据(本空间中文件v3_kline_2021_06_23.tar.gz)，请使用rust写回测，1分钟涨1%之后买入，第二分钟平掉，预期利润是多少？

```text
// 文件说明
v3_kline_2021_06_23tar.gz文件解压后得到的文件为6月23日0点到9点按每分钟一聚合的K线数据，文件是文本文件使用lzma压缩算法压缩而成，具体数据格式见下
// 数据格式说明
dump纳秒时间戳 '\t' shmId '\t' 交易所 '\t' preCoin '\t' postCoin '\t' 交易所返回的k线时间  '\t' 开 '\t' 高 '\t' 低 '\t' 收 '\t' 量 '\t' 其他
```
