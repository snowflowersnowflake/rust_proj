
#  rust练习记录



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


## day1 


- [x] 命令行安装rust
- [x] 第一个rust程序（helloworld）
- [x] clion安装和配置
- [x] cargo
- [x] toml和rust插件
- [x] 构建rust project，调试第一个rust程序
- [x] make_assum.rs (参考目录to learn rust) 
- [x] rand.rs
- [x] cmp.rs 

```
// 2 | use rand::Rng;
// |     ^^^^ use of undeclared crate or module `rand`
// https://crates.io/crates/rand
// Add this to your Cargo.toml:
// [dependencies]
// rand = "0.8.0"


// error: process didn't exit successfully: `C:\Users\y00\.cargo\bin\rustc.exe -vV` (exit code: 1)
// --- stderr
// error: the 'rustc.exe' binary, normally provided by the 'rustc' component, is not applicable to the 'stable-x86_64-pc-windows-gnu' toolchain
// Process finished with exit code 101
// Theoretically this could fix lack of the rustc component:
// rustup component add rustc
// rustup default stable
```

## day2 

- [x] 获取最新动态内容
- [x] 定时监控最近一条动态是否发生变化

## day3
- [x] 动态更新时触发预警事件分支
- [x] 发送request请求获取实时币价
- [x] rust多线程基础
- [x] 用新的预警线程执行定时事件
- [x] 30s输出一次币价涨跌幅
- [x] 格式化输出

```
限于时间，描述可选需求思路：

核心思想是把所有Twitter中的词作为坐标，在一条Twitter上，对应词的词频作为在该坐标上的值，即可将一篇Twitter按词频转换成词向量。每条推特文本展开成向量如【词1，词2...】和两类文本的向量计算相似度。相似度可以采用距离公式、余弦等等。

对于该问题较通用的优化是TF-IDF取代简单的词频统计。这样每个利多和利空分为两类文本统计词集，元素a[i][j]表示j词在i类文本中的tf-idf权重，然后就可以计算推特文本与两类文本的余弦相似。

如果想进一步优化就要考虑语义相似度，可以引用诸如bert之类的nlp深度学习模型进行解决。
```
制作中间过程和程序运行结果可以参考[过程记录1](https://github.com/snowflowersnowflake/rust_exercize/blob/main/%E8%BF%87%E7%A8%8B%E8%AE%B0%E5%BD%951.pdf)

主函数文件： 
```rust
#[macro_use]
extern crate serde_derive;
extern crate reqwest;
extern crate serde;
extern crate serde_json;
extern crate toml;

use bindings::{
    Windows::Foundation::Uri,
    Windows::Web::Syndication::SyndicationClient,
};
use std::{thread, time};
use std::thread::{sleep, spawn};
use std::time::Duration;
mod config;

#[derive(Serialize, Deserialize, Debug)]
struct Crypto {
    ticker: Ticker,
    success: bool,
    error: String
}

#[derive(Serialize, Deserialize, Debug)]
struct Ticker {
    base: String,
    target: String,
    price: String,
    volume: String,
    change: String
}

fn main() -> windows::Result<()> {
    let mut latest_twitter = String::new();
    loop{
        let ten_sec = time::Duration::from_secs(10);
        // let now = time::Instant::now();
        let uri = Uri::CreateUri("https://rsshub.app/twitter/user/elonmusk")?;
        let client = SyndicationClient::new()?; // SyndicationClient用于订阅
        let feed = client.RetrieveFeedAsync(uri)?.get()?; // 检索源
        for item in feed.Items()? { // 简单遍历title
            let tmp_twitter = item.Title()?.Text()?.to_string();
            println!("{}", tmp_twitter);
            if tmp_twitter == latest_twitter {
                println!("no new twitter");
            } else {
                println!("not same, get the BTC price !! ");
                let interruptus = spawn(interruptus);
            }
            // println!("last {}",latest_twitter);
            latest_twitter = tmp_twitter;
            break;
        }
        println!("waiting");
        thread::sleep(ten_sec); // sleep 10s
        // return Ok(());
    }
}

/** 用于发送请求获取币价的函数 **/
fn create_request_url(crypto_iso: String, fiat_iso: String) -> String {
    return format!("https://api.cryptonator.com/api/ticker/{}-{}", &crypto_iso, &fiat_iso);
}
fn make_request(req_url: String) -> Result<Crypto, reqwest::Error> {
    return reqwest::get(&req_url)?.json()
}
fn convert_price(crypto_price: String) -> f64 {
    return crypto_price.parse().unwrap();
}
fn format_price(price: String) -> String {
    return format!("{:.*}", 2, convert_price(price));
}
fn format_target_currency(target: String) -> String {
    return format!("{}", target);
}

fn handler(e: reqwest::Error) {
    if e.is_http() {
        match e.url() {
            None => println!("No Url given"),
            Some(url) => println!("Problem making request to: {}", url),
        }
    }
    // Inspect the internal error and output it
    if e.is_serialization() {
        let serde_error = match e.get_ref() {
            None => return,
            Some(err) => err,
        };
        println!("problem parsing information {}", serde_error);
    }
    if e.is_redirect() {
        println!("server redirecting too many times or making loop");
    }
}

/** 多线程 预警后用新线程执行循环币价推送任务 **/
fn interruptus() {
    // 存储第一次的价格，用于计算涨跌幅
    let mut first_price:f64 = 0 as f64;
    let mut current_price:f64 = 0 as f64;
    let mul = 100.0;
    let config = config::get_config();
    let resp: Result<Crypto, reqwest::Error> = make_request(create_request_url(config.crypto_iso, config.fiat_iso));
    match resp {
        Err(e) => handler(e), // 请求不成功时的handler arm
        Ok(resp) => {
            first_price = format_price(resp.ticker.price).parse::<f64>().unwrap();
            println!("第一次发布时币价{} {}", first_price, format_target_currency(resp.ticker.target));
        }
    }
    // 每30s执行一次计算
    loop {
        println!("30s Interruptus");
        sleep(Duration::from_secs(30));
        let config = config::get_config();
        let resp: Result<Crypto, reqwest::Error> = make_request(create_request_url(config.crypto_iso, config.fiat_iso));
        match resp {
            Err(e) => handler(e), // 请求不成功时的handler arm
            Ok(resp) => {
                current_price = format_price(resp.ticker.price).parse::<f64>().unwrap();
                println!("当前币价{} {}", current_price, format_target_currency(resp.ticker.target));
                println!("当前涨跌幅{:.*}%", 2, (( current_price - first_price) / first_price ) * mul );
            }
        }
    }
}

```

## day4

- [x] 通过rust读取单个文件的一行
- [x] 逐行阅读整份文件
- [x] 对单行进行split，并将数据存入Vector
- [x] 构建二维Vector，对于满足条件的行数据进行存储，目前只对huobi的btc/usdt执行该策略
- [x] 花了许多时间解决了一些阴魂不散的生命周期报错

## day5
- [x] 批量解压所有的文件整理到同一目录
- [x] walkdir遍历目录，filtermap+filetype过滤出文件名
- [x] 清洗出所有文件中huobi-btc/usdt数据 （580条）作为事件队列存入容器
- [x] 从容器中按时间戳顺序逐行读取，并配置基础回测参数
- [x] 处理纳秒时间和交易逻辑中1分钟关系
- [x] 核心交易逻辑与交易逻辑验证
- [x] btc/usdt较波动平稳，无法开仓，综合多次实验考虑，选择huobi doge/usdt进行回测
- [x] 完成收益计算

[过程记录2](https://github.com/snowflowersnowflake/rust_exercize/blob/main/%E8%BF%87%E7%A8%8B%E8%AE%B0%E5%BD%952.pdf)

```rust
use std::io::prelude::*;
use std::fs::File;
use std::io::BufReader;
use walkdir::WalkDir;
const FIRST_BALANCE: f64 = 1000000.0;
fn main() {
    let mut vv:Vec<Vec<String>> = Vec::new();
    for entry in WalkDir::new("./src/v3_kline_2021_06_23").into_iter().filter_map(|e| e.ok()) {
        // let metadata = fs::metadata(entry.path());
        let file_type = entry.file_type(); // 过滤获取的path
        if file_type.is_file() == false {
            continue;
        }
        // println!("{}", entry.path().display());
        let file = File::open(entry.path()).unwrap();
        let fin = BufReader::new(file);
        // data process，将需要的数据存入事件容器组vv
        for line in fin.lines(){
            let mut v:Vec<String> = Vec::new();
            let line = line.unwrap();
            let split_line = line.split("\t");
            for x in split_line {
                v.push(x.to_string());
            }
            // 过滤需要的币对
            if ( v[2] == "huobi" ) && ( v[3] == "DOGE") && ( v[4] == "USDT") {
            //     println!("{}", entry.path().display());
                vv.push(v);
            }
            // break;
        }
    }
    //1分钟涨1%之后买入，第二分钟平掉，预期利润是多少？
    // config: 成交率默认100% 资产以USDT计算 参考close价格下单 滑点（负）费率0.2% 符合交易逻辑则全仓买入
    let mut balance = FIRST_BALANCE;
    let mut stock:f64 = 0.0; // 取决于上面设置的标的
    //用于交易逻辑的变量
    let mut last_time:u64 = 0;
    let mut current_time:u64 = 0;
    let mut close:f64 = 0.0;
    let mut last_close:f64 = 0.0;
    let mut time_interval:u64 = 0;
    let mut accumulated_time_interval:u64 = 0;
    let mut rise:f64 = 0.0;
    let mut limit_threshold:f64 = 0.0;
    let mut open_flag:bool = false;
    // 遍历容器，一行记录视为一个事件进行回测
    for (index, value) in vv.iter().enumerate() {
        println!("{:?} is at index {}", value, index);
        close = value[9].parse::<f64>().unwrap();
        // 以纳秒计算超过59.5秒为准，并将涨跌幅和时间差值归一化成分钟涨跌幅。时间间隔超过2分钟的视为异常数据，不参与开仓
        current_time = value[0].parse::<u64>().unwrap();
        time_interval = current_time - last_time;
        if time_interval < 120000000000 {
            accumulated_time_interval += time_interval;
        }else { last_close = 0.0;accumulated_time_interval = 0; }
        if accumulated_time_interval > 59500000000 && open_flag == false{
            println!("判断一次交易逻辑");
            //核心逻辑在这里
            rise = (close - last_close) / close; //实际涨跌幅
            limit_threshold = ( (accumulated_time_interval as f64 / 60000000000.0 as f64 ) as f64 ) * 0.01; // 归一化后的买入阈值 1% 539
            if rise > limit_threshold{
                println!("买入开仓, 价格:{}, 一分钟涨幅:{}", close, rise);
                make_order(close, false, &mut balance, &mut stock);
                open_flag = true;
            }
            else {
                println!("不满足开仓条件");
            }
            accumulated_time_interval = 0; // 重置计时器
            last_close = 0.0;
        }
        if accumulated_time_interval > 59500000000 && open_flag == true{
            println!("卖出平仓");
            make_order(close, true, &mut balance, &mut stock);
            open_flag = false;
        }
        // println!("{}",time_interval);
        last_time = current_time;
        if last_close == 0.0 {
            last_close = close;
        }
    }
    println!("绝对收益:{}, 收益率:{}%",balance - FIRST_BALANCE, ( balance - FIRST_BALANCE ) / FIRST_BALANCE * 100.0 )
}
fn make_order(price: f64, if_sell: bool, balance: & mut f64, stock: & mut f64){
    // config: 成交率默认100% 资产以USDT计算 参考close价格下单 滑点（负）费率0.02% 符合交易逻辑则全仓买入
    let fee = 0.0002;
    let spread = 0.0002;
    if if_sell == false {
        *stock = ( *balance / ( price * (1.0 + spread)) ) * (1.0 - fee);
        *balance = 0.0;
        println!("下单成功！ 当前资产{} USD，当前收益{} ", *stock * price, *stock * price - FIRST_BALANCE );
    }
    else {
        *balance = *stock * ( price * (1.0 - spread) * (1.0 - fee) );
        *stock = 0.0;
        println!("当前资产{} USD，当前收益{}",*balance, *balance - FIRST_BALANCE);
    }
}
```
开平仓记录（以纳秒计算超过59.5秒间隔后进行交易逻辑判断，超过1%的涨幅视为满足开仓条件，可以配置任意品种回测）：
```
["1624377600005255406", "20003", "huobi", "DOGE", "USDT", "1624377540000", "0.192097", "0.193247", "0.191698", "0.193192", "190985.88230456", "system_ts=1624377599933;host=jp706"] is at index 0
["1624377600006110620", "20003", "huobi", "DOGE", "USDT", "1624377540000", "0.192097", "0.193247", "0.191698", "0.193193", "191178.79517471", "system_ts=1624377600003;host=jp706"] is at index 1
["1624377660012851050", "20003", "huobi", "DOGE", "USDT", "1624377600000", "0.193118", "0.195004", "0.191253", "0.194481", "1610773.2924725204", "system_ts=1624377659924;host=jp706"] is at index 2
判断一次交易逻辑
不满足开仓条件
["1624377660034773011", "20003", "huobi", "DOGE", "USDT", "1624377600000", "0.193118", "0.195004", "0.191253", "0.194484", "1611070.9074770403", "system_ts=1624377660012;host=jp706"] is at index 3
["1624377720003277654", "20003", "huobi", "DOGE", "USDT", "1624377660000", "0.194526", "0.198", "0.194456", "0.197243", "2476298.4056878462", "system_ts=1624377719922;host=jp706"] is at index 4
判断一次交易逻辑
买入开仓, 价格:0.197243, 一分钟涨幅:0.014003031793270303
下单成功！ 当前资产999600.0799840033 USD，当前收益-399.9200159966713 
["1624377720003892484", "20003", "huobi", "DOGE", "USDT", "1624377660000", "0.194526", "0.198", "0.194456", "0.197354", "2476876.905320586", "system_ts=1624377720003;host=jp706"] is at index 5
["1624377780006278472", "20003", "huobi", "DOGE", "USDT", "1624377720000", "0.197284", "0.199999", "0.195311", "0.199649", "1752273.7062022432", "system_ts=1624377779947;host=jp706"] is at index 6
卖出平仓
当前资产1011388.6763481848 USD，当前收益11388.676348184817
["1624377780006793701", "20003", "huobi", "DOGE", "USDT", "1624377720000", "0.197284", "0.199999", "0.195311", "0.199761", "1752935.9435165233", "system_ts=1624377780006;host=jp706"] is at index 7
判断一次交易逻辑
买入开仓, 价格:0.199761, 一分钟涨幅:0.012605063050345125
下单成功！ 当前资产1010984.2017725607 USD，当前收益10984.201772560715 
["1624377840036648322", "20003", "huobi", "DOGE", "USDT", "1624377780000", "0.199648", "0.200941", "0.198662", "0.198938", "2095173.0170526959", "system_ts=1624377839907;host=jp706"] is at index 8
卖出平仓
当前资产1006416.3370583036 USD，当前收益6416.337058303645
["1624377840093087857", "20003", "huobi", "DOGE", "USDT", "1624377780000", "0.199648", "0.200941", "0.198662", "0.198914", "2097527.319413866", "system_ts=1624377840036;host=jp706"] is at index 9
判断一次交易逻辑
不满足开仓条件
...
绝对收益:3959.1581025613705, 收益率:0.3959158102561371%
```
