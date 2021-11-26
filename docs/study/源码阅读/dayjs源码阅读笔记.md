## 前置知识

### 阅读资料

[前端时间国际化](https://mp.weixin.qq.com/s/Gw6UiovEvu76a-9zxL3BdA)

### 时间标准

#### GMT

格林尼治平均时间（Greenwich Mean Time，`GMT`）是指位于英国伦敦郊区的皇家格林尼治天文台当地的平太阳时，因为本初子午线被定义为通过那里的经线。

自 1924 年 2 月 5 日开始，格林尼治天文台负责每隔一小时向全世界发放调时信息。

格林尼治标准时间的正午是指当平太阳横穿格林尼治子午线时（也就是在格林尼治上空最高点时）的时间。由于地球每天的自转是有些不规则的，而且正在缓慢减速，因此格林尼治平时基于天文观测本身的缺陷，已经被原子钟报时的协调世界时（`UTC`）所取代。

#### UTC

协调世界时（英语：`Coordinated Universal Time`，法语：`Temps Universel Coordonné`，简称 `UTC`）是最主要的世界时间标准，其以原子时秒长为基础，在时刻上尽量接近于格林威治标准时间。

协调世界时是世界上调节时钟和时间的主要时间标准，它与 `0` 度经线的平太阳时相差不超过 `1` 秒，并不遵守夏令时。

现行的协调世界时根据国际电信联盟的建议《Standard-frequency and time-signal emissions》(ITU-R TF.460-6)所确定。`UTC` 基于国际原子时，并在必要时通过不规则的加入闰秒来抵消地球自转变慢的影响。

如果本地时间比 `UTC` 时间快，例如中国、蒙古、菲律宾、新加坡、马来西亚、澳大利亚西部的时间比 `UTC` 快 `8` 小时，就会写作 `UTC+8`，俗称`东八区`。相反，如果本地时间比 `UTC` 时间慢，例如夏威夷的时间比 `UTC` 时间慢 `10` 小时，就会写作 `UTC-10`，俗称`西十区`。

#### NTP

网络时间协议（英语：Network Time Protocol，缩写：NTP）是在数据网络潜伏时间可变的计算机系统之间通过分组交换进行时钟同步的一个网络协议，位于`OSI`模型的`应用层`。自1985年以来，NTP是目前仍在使用的最古老的互联网协议之一。NTP由特拉华大学的David L. Mills设计。

NTP意图将所有参与计算机的协调世界时（UTC）时间同步到几毫秒的误差内。它使用Marzullo算法的修改版来选择准确的时间服务器，其设计旨在`减轻可变网络延迟造成的影响`。NTP通常可以在公共互联网保持`几十毫秒`的误差，并且在理想的局域网环境中可以实现超过1毫秒的精度。不对称路由和拥塞控制可能导致100毫秒（或更高）的错误。

#### ISO

国际标准 ISO 8601，是国际标准化组织的日期和时间的`表示`方法，全称为《数据存储和交换形式·信息交换·日期和时间的表示方法》。目前是 2004 年 12 月 1 日发行的第三版“ISO8601:2004”

在 Javascript 中的 `Date.prototype.toISOString()` 中，返回的是 `YYYY-MM-DDTHH:mm:ss.sssZ`格式的字符串，时区总是 UTC（协调世界时），加一个后缀“Z”标识。

### Date.prototype上的输出时间方法

以时间戳1637736654012为例

| 方法                      | 格式                      | 输出                                             |
| ------------------------- | ------------------------- | ------------------------------------------------ |
| valueof                   | 时间戳(number)            | 1637736654012                                    |
| getTime                   | 时间戳(number)            | 1637736654012                                    |
| toString                  | 英语格式的本地时间字符串  | Wed Nov 24 2021 14:50:54 GMT+0800 (中国标准时间) |
| toUTCString               | 英语格式的 UTC 时间字符串 | Wed, 24 Nov 2021 06:50:54 GMT                    |
| toGMTString（标准已废弃） | 英语格式的 GMT 时间字符串 | Wed, 24 Nov 2021 06:50:54 GMT                    |
| toLocaleString            | 字符串因本地语言而不同    | 2021/11/24 下午2:50:54                           |
| toISOString               | ISO 格式的 UTC 时间字符串 | 2021-11-24T06:50:54.012Z                         |
| toJson                    | 与ISO相同                 | 2021-11-24T06:50:54.012Z                         |
| toDateString              | 只展示日期                | Wed Nov 24 2021                                  |



## 概述

官方概述：“`Day.js` 是 `Moment.js` 的 2kB 轻量化方案，拥有同样强大的 `API`”。优点是如下三个：

- 简易：`Day.js` 是一个轻量的处理时间和日期的 `JavaScript` 库，和 `Moment.js` 的 `API` 设计保持完全一样。
- 不可变：所有的 `API` 操作都将返回一个新的 `Dayjs` 实例。这种设计能避免 `bug` 产生，节约调试时间。
- 国际化：`Day.js` 对国际化支持良好。但除非手动加载，多国语言默认是不会被打包到工程里的。

我将其fork了一份，最后一次提交时间为2021/9/10,commit id为[b5a1391](https://github.com/time0verflow/dayjs/commit/b5a1391011b247d08863d291542db5937b23b427)

## 目录结构

```
dayjs
│  .editorconfig // 编辑器配置
│  .eslintrc.json // ESLint配置
│  .gitignore // git忽略配置
│  .npmignore // npm发布忽略配置
│  .travis.yml // 持续集成配置
│  babel.config.js // babel配置
│  CHANGELOG.md // 更新日志
│  CONTRIBUTING.md // 共建指南
│  karma.sauce.conf.js // karma测试配置
│  LICENSE // 许可声明
│  package.json
│  prettier.config.js // prettier 格式化配置
│  README.md
│
├─.github // github的一些配置
├─build // 构建打包
├─docs // 各语言的说明文档
├─src
│  │  constant.js // 常量
│  │  index.js // 主入口，定义Dayjs类
│  │  utils.js // 工具函数
│  │
│  ├─locale // 国际化
│  └─plugin // 插件
├─test // 测试
└─types // TypeScript
```

## locale文件夹

在这个文件夹里实现了很多种语言的国际化,`dayjs/locale/xxx.js`对应各国语言的模板和配置

以`zh-cn.js`为例

```javascript
// Chinese (China) [zh-cn]
import dayjs from 'dayjs'

const locale = {
  //对象名
  name: 'zh-cn',

  //月份和星期汉化表示的数组
  weekdays: '星期日_星期一_星期二_星期三_星期四_星期五_星期六'.split('_'),
  weekdaysShort: '周日_周一_周二_周三_周四_周五_周六'.split('_'),
  weekdaysMin: '日_一_二_三_四_五_六'.split('_'),
  months: '一月_二月_三月_四月_五月_六月_七月_八月_九月_十月_十一月_十二月'.split('_'),
  monthsShort: '1月_2月_3月_4月_5月_6月_7月_8月_9月_10月_11月_12月'.split('_'),

  /**
   * 返回例如2周，3日
   * @param {Number} number 
   * @param {String} period 
   * @returns 
   */
  ordinal: (number, period) => {
    switch (period) {
      case 'W':
        return `${number}周`
      default:
        return `${number}日`
    }
  },
  
  weekStart: 1,
  yearStart: 4,

  //格式化模板
  formats: {
    LT: 'HH:mm',
    LTS: 'HH:mm:ss',
    L: 'YYYY/MM/DD',
    LL: 'YYYY年M月D日',
    LLL: 'YYYY年M月D日Ah点mm分',
    LLLL: 'YYYY年M月D日ddddAh点mm分',
    l: 'YYYY/M/D',
    ll: 'YYYY年M月D日',
    lll: 'YYYY年M月D日 HH:mm',
    llll: 'YYYY年M月D日dddd HH:mm'
  },

  //相对时间的格式化模板
  relativeTime: {
    future: '%s内',
    past: '%s前',
    s: '几秒',
    m: '1 分钟',
    mm: '%d 分钟',
    h: '1 小时',
    hh: '%d 小时',
    d: '1 天',
    dd: '%d 天',
    M: '1 个月',
    MM: '%d 个月',
    y: '1 年',
    yy: '%d 年'
  },

  /**
   * 根据时、分返回相应的时间阶段
   * @param {Number} hour 
   * @param {Number} minute 
   * @returns 
   */
  meridiem: (hour, minute) => {
    const hm = (hour * 100) + minute
    if (hm < 600) {
      return '凌晨'
    } else if (hm < 900) {
      return '早上'
    } else if (hm < 1100) {
      return '上午'
    } else if (hm < 1300) {
      return '中午'
    } else if (hm < 1800) {
      return '下午'
    }
    return '晚上'
  }
}

//将locale对象加载到locale的LS中
dayjs.locale(locale, null, true)

export default locale

```

总结：可以看出这个模块内做了一些本地的格式化处理

## constant

这里面存放了一些常量和正则表达式

```javascript
//命名规则,X_A_Y值为单位从X=>Y所需除的数字
export const SECONDS_A_MINUTE = 60
export const SECONDS_A_HOUR = SECONDS_A_MINUTE * 60
export const SECONDS_A_DAY = SECONDS_A_HOUR * 24
export const SECONDS_A_WEEK = SECONDS_A_DAY * 7

export const MILLISECONDS_A_SECOND = 1e3
export const MILLISECONDS_A_MINUTE = SECONDS_A_MINUTE * MILLISECONDS_A_SECOND
export const MILLISECONDS_A_HOUR = SECONDS_A_HOUR * MILLISECONDS_A_SECOND
export const MILLISECONDS_A_DAY = SECONDS_A_DAY * MILLISECONDS_A_SECOND
export const MILLISECONDS_A_WEEK = SECONDS_A_WEEK * MILLISECONDS_A_SECOND

// English locales
export const MS = 'millisecond'
export const S = 'second'
export const MIN = 'minute'
export const H = 'hour'
export const D = 'day'
export const W = 'week'
export const M = 'month'
export const Q = 'quarter'
export const Y = 'year'
export const DATE = 'date'

//默认时间格式为ISO
export const FORMAT_DEFAULT = 'YYYY-MM-DDTHH:mm:ssZ'

//无效时间
export const INVALID_DATE_STRING = 'Invalid Date'

// regex
// 用于解析字符串格式的时间，便于生成Dayjs实例相关的Date对象
export const REGEX_PARSE = /^(\d{4})[-/]?(\d{1,2})?[-/]?(\d{0,2})[Tt\s]*(\d{1,2})?:?(\d{1,2})?:?(\d{1,2})?[.:]?(\d+)?$/
// 用于解析format参数,返回想要的时间格式
export const REGEX_FORMAT = /\[([^\]]+)]|Y{1,4}|M{1,4}|D{1,2}|d{1,4}|H{1,2}|h{1,2}|a|A|m{1,2}|s{1,2}|Z{1,2}|SSS/g
```

## utils

存放了一堆工具类的函数

```javascript
import * as C from './constant'

/**
 * @description 在string开头补充length-s.length个pad，猜测pad可能是单个字符
 * @param {String} string  被补充的字符串
 * @param {Number} length  被补充后达到的长度(前提是pad为单字符)
 * @param {String} pad     填充的内容
 * @returns {String} 补充后的字符串
 */
const padStart = (string, length, pad) => {
  const s = String(string)
  if (!s || s.length >= length) return string
  return `${Array((length + 1) - s.length).join(pad)}${string}`
}


/**
 * @description 返回实例的UTC偏移量(分钟),转化为[+|-]HH:mm格式
 * @param {Dayjs} instance Dayjs的实例
 * @returns {String} UTC的偏移量 [+|-]HH:mm格式
 */
const padZoneStr = (instance) => {
  const negMinutes = -instance.utcOffset()
  const minutes = Math.abs(negMinutes)
  const hourOffset = Math.floor(minutes / 60)
  const minuteOffset = minutes % 60
  return `${negMinutes <= 0 ? '+' : '-'}${padStart(hourOffset, 2, '0')}:${padStart(minuteOffset, 2, '0')}`
}

/**
 * @description 求两个实例间的月份差
 * @param {Dayjs} a Dayjs的实例
 * @param {Dayjs} b Dayjs的实例
 * @returns {Number} 两个实例实例的月份差
 */
const monthDiff = (a, b) => {
  // function from moment.js in order to keep the same result
  if (a.date() < b.date()) return -monthDiff(b, a)
  const wholeMonthDiff = ((b.year() - a.year()) * 12) + (b.month() - a.month())
  const anchor = a.clone().add(wholeMonthDiff, C.M)
  const c = b - anchor < 0
  const anchor2 = a.clone().add(wholeMonthDiff + (c ? -1 : 1), C.M)
  return +(-(wholeMonthDiff + ((b - anchor) / (c ? (anchor - anchor2) :
    (anchor2 - anchor)))) || 0)
}

//向0取整
const absFloor = n => (n < 0 ? Math.ceil(n) || 0 : Math.floor(n))

/**
 * @description 返回u对应的单位 如M=>month H=>hour W=>week等，若没找到匹配的，将u结尾的s删除作为单位
 * @param {String} u 
 * @returns 
 */
const prettyUnit = (u) => {
  const special = {
    M: C.M,
    y: C.Y,
    w: C.W,
    d: C.D,
    D: C.DATE,
    h: C.H,
    m: C.MIN,
    s: C.S,
    ms: C.MS,
    Q: C.Q
  }
  return special[u] || String(u || '').toLowerCase().replace(/s$/, '')
}

const isUndefined = s => s === undefined

export default {
  s: padStart,
  z: padZoneStr,
  m: monthDiff,
  a: absFloor,
  p: prettyUnit,
  u: isUndefined
}
```

## Dayjs类

### 基本结构

`src/index.js`为入口文件，代码的骨架为

1. 导入`constant`、`locale`、`utils`

   ```js
   import * as C from './constant'
   import en from './locale/en'
   import U from './utils'
   ```

2. 全局locale定义

   ```js
   let L = 'en' // global locale
   const Ls = {} // global loaded locale
   Ls[L] = en
   ```

3. 实例化Dayjs类的方法,这也是最终将会导出的

   ```js
   const dayjs = function (date, c) {}
   ```

4. 完善Utils工具类

   ```js
   const Utils = U // for plugin use
   Utils.l = parseLocale
   Utils.i = isDayjs
   Utils.w = wrapper
   ```

5. 定义Dayjs的类

   ```js
   // Dayjs 类
   class Dayjs {
     constructor(cfg) {}
     parse(cfg) {}
     init() {}
     $utils() {}
     isValid() {}
     isSame(that, units) {}
     isAfter(that, units) {}
     isBefore(that, units) {}
     $g(input, get, set) {}
     unix() {}
     valueOf() {}
     startOf(units, startOf) {}
     endOf(arg) {}
     $set(units, int) {}
     set(string, int) {}
     get(unit) {}
     add(number, units) {}
     subtract(number, string) {}
     format(formatStr) {}
     utcOffset() {}
     diff(input, units, float) {}
     daysInMonth() {}
     $locale() {}
     locale(preset, object) {}
     clone() {}
     toDate() {}
     toJSON() {}
     toISOString() {}
     toString() {}
   }
   ```

6. 丰富Dayjs的原型

   ```js
   const proto = Dayjs.prototype
   dayjs.prototype = proto;
   [
     ['$ms', C.MS],
     ['$s', C.S],
     ['$m', C.MIN],
     ['$H', C.H],
     ['$W', C.D],
     ['$M', C.M],
     ['$y', C.Y],
     ['$D', C.DATE]
   ].forEach((g) => {
     proto[g[1]] = function (input) {
       return this.$g(input, g[0], g[1])
     }
   })
   ```

7. 补充dayjs静态方法

```js
dayjs.extend = (plugin, option) => {
  if (!plugin.$i) { // install plugin only once
    plugin(option, Dayjs, dayjs)
    plugin.$i = true
  }
  return dayjs
}

dayjs.locale = parseLocale

dayjs.isDayjs = isDayjs

dayjs.unix = timestamp => (
  dayjs(timestamp * 1e3)
)

dayjs.en = Ls[L]
dayjs.Ls = Ls
dayjs.p = {}
```

### 小测试

```js
//console.log(dayjs())
$D: 24  //24日
$H: 20  //20点
$L: "en" //locale
$M: 10   //11月
$W: 3   //周三
$d: Wed Nov 24 2021 20:38:33 GMT+0800 (中国标准时间) {}
$m: 38   //38分
$ms: 573  //573毫秒
$s: 33   //33秒
$x: {}
$y: 2021  //2021年
    `[[Prototype]]: Object
    $g: ƒ (t2, e2, n2)
    $locale: ƒ ()
    $set: ƒ (t2, e2)
    $utils: ƒ ()
    add: ƒ (r2, h2)
    clone: ƒ ()
    date: ƒ (e2)
    day: ƒ (e2)
    daysInMonth: ƒ ()
    diff: ƒ (r2, d2, $2)
    endOf: ƒ (t2)
    format: ƒ (t2)
    get: ƒ (t2)
    hour: ƒ (e2)
    init: ƒ ()
    isAfter: ƒ (t2, e2)
    isBefore: ƒ (t2, e2)
    isSame: ƒ (t2, e2)
    isValid: ƒ ()
    locale: ƒ (t2, e2)
    millisecond: ƒ (e2)
    minute: ƒ (e2)
    month: ƒ (e2)
    parse: ƒ (t2)
    second: ƒ (e2)
    set: ƒ (t2, e2)
    startOf: ƒ (t2, e2)
    subtract: ƒ (t2, e2)
    toDate: ƒ ()
    toISOString: ƒ ()
    toJSON: ƒ ()
    toString: ƒ ()
    unix: ƒ ()
    utcOffset: ƒ ()
    valueOf: ƒ ()
    year: ƒ (e2)
    constructor: ƒ M2(t2)
	[[Prototype]]: Object`
```

### Dayjs和dayjs关系图

![image-20211124205613265](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a67f63b110cc42f69adb7799ef6e1d4c~tplv-k3u1fbpfcp-watermark.png)

### 补充工具函数

1. `parseLocale`

   ```js
   /**
    * @description 给全局对象Ls添加或修改一对键值对
    * @param {String|Object} preset 
    * @param {Object} object 
    * @param {Boolean} isLocal 判断是否为本地local
    * @returns {String} 返回键名
    */
   const parseLocale = (preset, object, isLocal) => {
     let l
     //不带参数时，返回当前使用的locale键
     if (!preset) return L
   
     //当preset为字符串时，将preset作为键，若第二个参数存在则作为值
     if (typeof preset === 'string') {
       if (Ls[preset]) {
         l = preset
       }
       if (object) {
         Ls[preset] = object
         l = preset
       }
       //当preset为对象时，提取对象的name作为键，对象作为值
     } else {
       const { name } = preset
       Ls[name] = preset
       l = name
     }
     //设置本地Local为false时修改全局Locale
     if (!isLocal && l) L = l
     //返回添加或修改的键名
     return l || (!isLocal && L)
   }
   ```

   这个对象被挂载到了dayjs的locale静态方法中

   `src/locale/zh-cn`中对这个方法的调用,此函数将返回`zh-cn`，并将`'zh-cn'-locale`添加到全局`LS`对象中

   ```js
   dayjs.locale(locale, null, true)
   ```

   Dayjs类的静态方法中同样有对该函数的调用,将键名存储在实例的$L字段中

   ```js
   this.$L = parseLocale(cfg.locale, null, true)
   ```

   

2. `isDayjs` ,判断对象是否为js实例

   ```js
   const isDayjs = d => d instanceof Dayjs
   ```

   

3. `dayjs`,实例化Dayjs，也是最终将会导出的函数

   ```js
   /**
    * @description 实例化Dayjs的方法，如果参数是Dayjs实例，直接返回
    * @param {Date|Dayjs} date 
    * @param {Object} c 
    * @returns {Dayjs} 返回一个Dayjs实例
    */
   const dayjs = function (date, c) {
     //如果第一个参数时Dayjs实例，直接克隆返回
     if (isDayjs(date)) {
       return date.clone()
     }
     const cfg = typeof c === 'object' ? c : {}
     cfg.date = date
     cfg.args = arguments
     // cfg:{date,args:arguments}
     return new Dayjs(cfg)
   }
   ```

   

4. `wrapper` 

```js
/**
 * @description 封装器，根据Date对象和Dayjs实例封装出一个Dayjs新实例
 * @param {*} date  Date实例或其他可以被parseDate解析成Date实例的，例如时间戳和ISO string等
 * @param {Dayjs} instance 
 * @returns {Dayjs}
 */
const wrapper = (date, instance) =>
  dayjs(date, {
    locale: instance.$L,
    utc: instance.$u,
    x: instance.$x,
    $offset: instance.$offset // todo: refactor; do not use this.$offset in you code
  })
```

5. `parsedata` 这是一个辅助函数，将传入Dayjs的配置项进行解析,解析出Date对象

   ```js
   /**
    * @description 从配置项中解析得到Date实例
    * @param {Object} cfg 被解析的配置项
    * @returns {Date} 解析得到的Date实例
    */
   const parseDate = (cfg) => {
     const { date, utc } = cfg
     //date为null时返回Invalid Date
     if (date === null) return new Date(NaN) // null is invalid
     //date为定义时默认为当前时刻
     if (Utils.u(date)) return new Date() // today
     //date是Date类时直接返回
     if (date instanceof Date) return new Date(date)
     //date为字符串且以Z结尾，即时间为ISO格式2021-11-24T06:50:54.012Z
     if (typeof date === 'string' && !/Z$/i.test(date)) {
       const d = date.match(C.REGEX_PARSE)
       if (d) {
         const m = d[2] - 1 || 0   //表示月份，需要-1，因Date月份范围为0-11
         const ms = (d[7] || '0').substring(0, 3)  //表示毫秒数，需要去掉Z
         if (utc) {
           return new Date(Date.UTC(d[1], m, d[3]
             || 1, d[4] || 0, d[5] || 0, d[6] || 0, ms))
         }
         return new Date(d[1], m, d[3]
             || 1, d[4] || 0, d[5] || 0, d[6] || 0, ms)
       }
     }
   
     return new Date(date) // everything else
   }
   ```

### Dayjs类内的方法解析

这是整个dayjs库的核心，方法可分为以下几类

+ 初始化： `parse`、`init`
+ 取赋值： `get`、`set`
+ 操作： `add`、`subtract`、 `startOf`、 `endOf`、 `utcOffset`
+ 显示： `format`、 `toDate`、 `toJSON`、 `toISOString`、 `toString`
+ 查询： `isValid`、 `isSame`、 `isAfter`、 `isBefore`、 `daysInMonth`、`diff`、 `unix`、 `valueOf`

#### 初始化

`parse`和`init`在初始化一个实例后会立即执行

```js
  constructor(cfg) {
    //$L存放当前locale
    this.$L = parseLocale(cfg.locale, null, true)
    this.parse(cfg) // for plugin
  }
   /**
   * @param {Object} cfg config配置对象 
   */
  parse(cfg) {
    //解析得到配置项中的date对象存放在$d中
    this.$d = parseDate(cfg)
    //$x存放{$timezone,$localeOffset}时区有关信息
    this.$x = cfg.x || {}
    this.init()
  }

  /**
   * @description 初始化内部字段
   */
  init() {
    const { $d } = this
    this.$y = $d.getFullYear()
    this.$M = $d.getMonth()
    this.$D = $d.getDate()
    this.$W = $d.getDay()
    this.$H = $d.getHours()
    this.$m = $d.getMinutes()
    this.$s = $d.getSeconds()
    this.$ms = $d.getMilliseconds()
  }
```

#### 取赋值

##### 用法

```js
dayjs().get('year')  dayjs().get('y')  dayjs('year') //2021

通用的 setter，两个参数分别是要更新的单位和数值，调用后会返回一个修改后的新实例。
dayjs().set('month', 3) // 四月
```

##### 源码分析

```js
  /**
   * @description 私有setter,不会修改原Dayjs对象，调用放回新的修改实例，修改主要是调用Date.prototype.setXXX接口
   * @param {String} units 单位
   * @param {Number} int 值
   * @returns 修改后的实例
   */
  $set(units, int) { // private set
    const unit = Utils.p(units)
    const utcPad = `set${this.$u ? 'UTC' : ''}`
    //unit=>name,将单位转化为相应的method
    const name = {
      [C.D]: `${utcPad}Date`,
      [C.DATE]: `${utcPad}Date`,
      [C.M]: `${utcPad}Month`,
      [C.Y]: `${utcPad}FullYear`,
      [C.H]: `${utcPad}Hours`,
      [C.MIN]: `${utcPad}Minutes`,
      [C.S]: `${utcPad}Seconds`,
      [C.MS]: `${utcPad}Milliseconds`
    }[unit]
    
    //如果单位是day(星期日、星期一)，则在当周内进行加减，一周从周日(0)到周六(6)
    const arg = unit === C.D ? this.$D + (int - this.$W) : int

    //把$d，也就是关联的Date对象设置为int
    if (unit === C.M || unit === C.Y) {
      // clone is for badMutable plugin
      const date = this.clone().set(C.DATE, 1)
      date.$d[name](arg)
      date.init()
      this.$d = date.set(C.DATE, Math.min(this.$D, date.daysInMonth())).$d
    } else if (name) this.$d[name](arg)

    this.init()
    return this
  }

  //先进行clone以便返回新实例
  set(string, int) {
    return this.clone().$set(string, int)
  }

  get(unit) {
    return this[Utils.p(unit)]()
  }
```



正常来说这几个方法是可以放在`class`里的，但因为比较相似，都是和时间相关的`getter/setter`方法，单独拿出来。

```js
const proto = Dayjs.prototype
dayjs.prototype = proto;
[
  ['$ms', C.MS],  //Dayjs.prototype.millisecond
  ['$s', C.S],
  ['$m', C.MIN],
  ['$H', C.H],
  ['$W', C.D],
  ['$M', C.M],
  ['$y', C.Y],
  ['$D', C.DATE]
].forEach((g) => {
  proto[g[1]] = function (input) {
    return this.$g(input, g[0], g[1])
  }
})
```

#### 操作

##### 用法

```js
//add  返回增加一定时间的复制的 Day.js 对象。 增加7天 不会改变原对象
dayjs().add(7, 'day')
//substract 返回减去一定时间的复制的 Day.js 对象
//startOf 返回复制的 Day.js 对象，并设置到一个时间的开始。
dayjs().startOf('year')  //$d是今年开始的Date,2021-1-1 00:00:00
//endof 返回复制的 Day.js 对象，并设置到一个时间的结束。
//utcOffset 获取 UTC 偏移量 (分钟)。 也可以传入分钟来得到一个更改 UTC 偏移量的新实例。
dayjs().utcOffset()  //480 东八区返回480分钟
// 以下两种写法是等效的
//如果输入在-16到16之间，会将您的输入理解为小时数而非分钟。
dayjs().utcOffset(8)  // 设置小时偏移量
dayjs().utcOffset(480)  // 设置分钟偏移量 (8 * 60)
```

##### 源码

```js
  /**
   * @description 根据单位和值，给当前关联的Date对象增加时间
   * @param {Number} number 增加的数值
   * @param {String} units 单位
   * @returns 
   */
  add(number, units) {
    number = Number(number) // eslint-disable-line no-param-reassign
    const unit = Utils.p(units)
    /**
     * @description 专门给week和date用来增加时间的工具函数
     * @param {Number} n  基础单位的天数，week是7 ，date是1 
     * @returns {Dayjs} 新的Dayjs实例
     */
    const instanceFactorySet = (n) => {
      const d = dayjs(this)

      return Utils.w(d.date(d.date() + Math.round(n * number)), this)
    }
    if (unit === C.M) {
      return this.set(C.M, this.$M + number)
    }
    if (unit === C.Y) {
      return this.set(C.Y, this.$y + number)
    }
    if (unit === C.D) {
      return instanceFactorySet(1)
    }
    //周
    if (unit === C.W) {
      return instanceFactorySet(7)
    }
    const step = {
      [C.MIN]: C.MILLISECONDS_A_MINUTE,
      [C.H]: C.MILLISECONDS_A_HOUR,
      [C.S]: C.MILLISECONDS_A_SECOND
    }[unit] || 1 // ms

    const nextTimeStamp = this.$d.getTime() + (number * step)
    return Utils.w(nextTimeStamp, this)
  }

  subtract(number, string) {
    return this.add(number * -1, string)
  }

	  /**
   * @description 根据单位实例设置到一个时间段的开始,
   * @param {String} units 单位 例如 "year"返回今年一月1日上午 00:00
   * @param {Boolean} startOf startOf标志 true:startOf, false:endOf
   * @returns {Dayjs} 返回Dayjs实例，cfg与原实例相同
   */
  startOf(units, startOf) { // startOf -> endOf
    //startOf未定义返回为true
    const isStartOf = !Utils.u(startOf) ? startOf : true
    //提取单位
    const unit = Utils.p(units)
    /**
     * @description 实例工厂函数，根据日，月(参数)和年份(实例)创建新的Dayjs实例
     * @param {Number} d 1-31
     * @param {Number} m 0-11
     * @returns {Dayjs} 返回新实例，如果isStartOf为false,返回当天的endOf
     */
    const instanceFactory = (d, m) => {
      const ins = Utils.w(this.$u ?
        Date.UTC(this.$y, m, d) : new Date(this.$y, m, d), this)
      return isStartOf ? ins : ins.endOf(C.D)
    }

    /**
     * @description 根据传入的method来返回Dayjs新实例，当单位为hour,minute,second,millisecond调用
     * @param {String} method 例如setUTCHours setSeconds等
     * @param {Number} slice 
     * @returns 
     */
    const instanceFactorySet = (method, slice) => {
      const argumentStart = [0, 0, 0, 0]
      const argumentEnd = [23, 59, 59, 999]
      return Utils.w(this.toDate()[method].apply( // eslint-disable-line prefer-spread
        this.toDate('s'),
        (isStartOf ? argumentStart : argumentEnd).slice(slice)
      ), this)
    }
    const { $W, $M, $D } = this
    const utcPad = `set${this.$u ? 'UTC' : ''}`
    switch (unit) {
      case C.Y:
        return isStartOf ? instanceFactory(1, 0) :
          instanceFactory(31, 11)
      case C.M:
        return isStartOf ? instanceFactory(1, $M) :
          instanceFactory(0, $M + 1)
      case C.W: {
        const weekStart = this.$locale().weekStart || 0
        const gap = ($W < weekStart ? $W + 7 : $W) - weekStart
        return instanceFactory(isStartOf ? $D - gap : $D + (6 - gap), $M)
      }
      case C.D:
      case C.DATE:
        return instanceFactorySet(`${utcPad}Hours`, 0)
      case C.H:
        return instanceFactorySet(`${utcPad}Minutes`, 1)
      case C.MIN:
        return instanceFactorySet(`${utcPad}Seconds`, 2)
      case C.S:
        return instanceFactorySet(`${utcPad}Milliseconds`, 3)
      default:
        return this.clone()
    }
  }

  endOf(arg) {
    return this.startOf(arg, false)
  }

  utcOffset() {
    // Because a bug at FF24, we're rounding the timezone offset around 15 minutes
    // https://github.com/moment/moment/pull/1871
    return -Math.round(this.$d.getTimezoneOffset() / 15) * 15
  }
```

#### 显示

##### 用法

```js
 //format 将字符放在方括号中，即可原样返回而不被格式化替换 (例如， [MM])。
dayjs.duration({
  seconds: 1,
  minutes: 2,
  hours: 3,
  days: 4,
  months: 6,
  years: 7
}).format('YYYY-MM-DDTHH:mm:ss') // 0007-06-04T03:02:01
```

`toDate` 、`toJSON`、 `toISOString`、 `toString`
 这四个方法都是原生Date对象里就有的方法

```js
  toDate() {
    return new Date(this.valueOf())
  }

  toJSON() {
    return this.isValid() ? this.toISOString() : null
  }

  toISOString() {
    // ie 8 return
    // new Dayjs(this.valueOf() + this.$d.getTimezoneOffset() * 60000)
    // .format('YYYY-MM-DDTHH:mm:ss.SSS[Z]')
    return this.$d.toISOString()
  }

  toString() {
    return this.$d.toUTCString()
  }
```

#### 查询

`isValid`、 `isSame`、 `isAfter`、 `isBefore`、 `daysInMonth`、`diff`、 `unix`、 `valueOf`

```js
  /**
   * @description 返回Date对象是否合规
   * @returns {Boolean}
   */
  isValid() {
    return !(this.$d.toString() === C.INVALID_DATE_STRING)
  }

  /**
   * @description 本实例是否和that提供的日期时间相同
   * @param {Dayjs|Sting} that 另一个Dayjs实例或时间字符串 
   * @param {String} units  单位
   * @returns 
   */
  isSame(that, units) {
    const other = dayjs(that)
    return this.startOf(units) <= other && other <= this.endOf(units)
  }

  /**
   * @description 给定单位下，本实例的时间是否晚于that提供的时间
   * @param {Dayjs|String} that 
   * @param {String} units 
   * @returns {Boolean}
   */
  isAfter(that, units) {
    return dayjs(that) < this.startOf(units)
  }

  /**
   * @description 给定单位下，本实例的时间是否早于that提供的时间
   * @param {Dayjs|String} that 
   * @param {String} units 
   * @returns {Boolean}
   */
  isBefore(that, units) {
    return this.endOf(units) < dayjs(that)
  }
  /**
   * @description 返回以秒为单位的时间戳
   * @returns {Number} 十位数字的时间戳
   */
  unix() {
    return Math.floor(this.valueOf() / 1000)
  }

  /**
   * @description 根据实例的Date对象返回以ms为单位的13位时间戳
   * @returns 
   */
  valueOf() {
    // timezone(hour) * 60 * 60 * 1000 => ms
    return this.$d.getTime()
  }
  /**
   * @description 查看一个月有多少天
   * @returns 当月的天数
   */
  daysInMonth() {
    return this.endOf(C.M).$D
  }
  /**
   * @description 返回指定单位下两个日期时间之间的差异
   * @param {String} input 输入的时间
   * @param {String} units 单位
   * @param {Boolean} float 是否需要取整
   * @returns {Number} 返回对应单位下的时间差
   */
  diff(input, units, float) {
    const unit = Utils.p(units)
    //input封装成实例
    const that = dayjs(input)
    //时区差距
    const zoneDelta = (that.utcOffset() - this.utcOffset()) * C.MILLISECONDS_A_MINUTE
    //毫秒差
    const diff = this - that
    //月份差
    let result = Utils.m(this, that)

    result = {
      [C.Y]: result / 12,
      [C.M]: result,
      [C.Q]: result / 3,
      [C.W]: (diff - zoneDelta) / C.MILLISECONDS_A_WEEK,
      [C.D]: (diff - zoneDelta) / C.MILLISECONDS_A_DAY,
      [C.H]: diff / C.MILLISECONDS_A_HOUR,
      [C.MIN]: diff / C.MILLISECONDS_A_MINUTE,
      [C.S]: diff / C.MILLISECONDS_A_SECOND
    }[unit] || diff // milliseconds

    return float ? result : Utils.a(result)
  }
```



### 静态属性

```js
/**
 * @description 挂载插件
 * @param {Function} plugin 插件,$i字段表示是否已经挂载 
 * @param {Object} option 插件选项
 * @returns {dayjs function} 返回dayjs函数对象
 */
dayjs.extend = (plugin, option) => {
  if (!plugin.$i) { // install plugin only once
    plugin(option, Dayjs, dayjs)
    plugin.$i = true
  }
  return dayjs
}

dayjs.locale = parseLocale

dayjs.isDayjs = isDayjs

/**
 * @description 解析传入的一个秒单位的Unix时间戳(10位数字),返回Dayjs实例
 * @param {Number} timestamp 10位数字的时间戳
 * @returns {Dayjs} 返回Dayjs实例
 */
dayjs.unix = timestamp => (
  dayjs(timestamp * 1e3)
)

dayjs.en = Ls[L]
dayjs.Ls = Ls
dayjs.p = {}
```

### 思考

## 插件

可以认为是对`dayjs`功能的扩充,在我所fork的版本里一共提供了`35`个插件，在`src/plugin`目录下,通过静态方法`days.extend`加载插件

```js
/**
 * @description 挂载插件
 * @param {Function} plugin 插件,$i字段表示是否已经挂载 
 * @param {Object} option 插件选项
 * @returns {dayjs function} 返回dayjs函数对象
 */
dayjs.extend = (plugin, option) => {
  if (!plugin.$i) { // install plugin only once
    plugin(option, Dayjs, dayjs)
    plugin.$i = true
  }
  return dayjs
}
```

`plugin`是一个函数，接收三个参数

+ `option`插件选项
+ `Dayjs`类
+ `dayjs`函数对象

插件会为`Dayjs`原型增加方法或者覆盖原本自带的方法

插件的大致写法为

```js
export default (option,Dayjs,dayjs)=>{
    const proto=Dayjs.prototype;
 	//pluginName   
    proto.pluginName=function(args){}
}
```

### 简单插件

1. **isLeapYear**

   判断是否为闰年

```js
export default (o, c) => {
  const proto = c.prototype
  //原型上挂载函数
  proto.isLeapYear = function () {
    return ((this.$y % 4 === 0) && (this.$y % 100 !== 0)) || (this.$y % 400 === 0)
  }
}
```

2. **isBetween**,**isSameOrAfter**,**isSameOrBefore**

```js
export default (o, c, d) => {
  /**
   * @description 返回一个boolean来展示一个时间是否介意二者之间
   * @param {String|Dayjs|Number} a 时间
   * @param {String|Dayjs|Number} b 时间
   * @param {String} u unit时间对象 
   * @param {String} i 区间闭开性() (] [] [)
   * @returns 
   */
  c.prototype.isBetween = function (a, b, u, i) {
    //实例化两端时间
    const dA = d(a)
    const dB = d(b)
    //判断两端开闭性
    i = i || '()'
    const dAi = i[0] === '('
    const dBi = i[1] === ')'
    //利用原型上的isAfter和isBefore实现isBetween判断
    return ((dAi ? this.isAfter(dA, u) : !this.isBefore(dA, u)) &&
           (dBi ? this.isBefore(dB, u) : !this.isAfter(dB, u)))
        || ((dAi ? this.isBefore(dA, u) : !this.isAfter(dA, u)) &&
           (dBi ? this.isAfter(dB, u) : !this.isBefore(dB, u)))
  }
}

