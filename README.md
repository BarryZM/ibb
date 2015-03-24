1. 所有周期`period`长度单位为月, 0表示趸交或一次性领取
2. 所有期间`term`, `age`单位是年, 如果为term=0, 表示终身
3. 所有价格`price`是实际价格的1000倍
4. 所有比率`rate`为千分数
5. 所有ID数字部分长度为8
6. 每个产品的每个子表里的ID的数字编号, 都必须从1开始
7. 性别`gender`: `M` - 男, `F` - 女 

#INSURANCE PRODCUT 保险产品
##INSURANCE_PRODUCT (insurance_product.csv)

###数据定义
field           | type        | null | default  | comment
--------------- | ----------- | ---- | -------- | -------
product_id      | CHAR(10)    | N    |          | ID
name            | VARCHAR(50) | N    |          | 产品名称
type            | ENUM        | N    |          | 产品类型
growth_withdraw | BOOL        | N    | false(0) | 是否允许递增领取
rate_calc_type  | ENUM        | N    |          | 费率计算方式
amount_base     | INT         | Y    |          | 基本保额/保费
min_age         | INT         | N    | 0        | 最小投保年龄
max_age         | INT         | N    | 100      | 最大投保年龄
min_premium     | INT         | Y    |          | 最低保费 
max_premium     | INT         | Y    |          | 最高保费
min_amount      | INT         | Y    |          | 最低保额
max_amount      | INT         | Y    |          | 最高保额
promotion       | BOOL        | N    | false(0) | 是否是开门红产品


###说明
1. type: 
    * TLF(Term Life), 定期人寿
    * WLF(Whole Life), 终身人寿
    * IVT(Investment), 投资连接
    * LVD(Live and Death), 两全
    * SIL(Serious Illness), 重大疾病
    * MED(Medicare), 医疗
    * HOS(Hospital), 住院津贴
    * ADT(Accident), 意外
    * PSN(Pension), 年金
    
2. rate_calc_type:
    * A(base on Amount), 指定保额计算保费
    * P(base on Premium), 指定保费计算保额
    
###示例

| product_id | name                         | type | growth_withdraw | rate_calc_type | amount_base | min_age | max_age | min_premium | min_premium | min_amount | max_amount | promotion
| ---------- | ---------------------------- | ---- | --------------- | -------------- | ----------- | ------- | ------- | ----------- | ----------- | ---------- | ---------- | ---------
| I006815014 | 太平卓越优享终身年金保险分红型    | PSN  | 0               | A              | 1000000     | 18      | 60      | 1000000     | 1000000000  | 100000000  | 200000000  | 0

##INSURANCE_COVERAGE 保险责任表 (注意: 可能需要重构以满足所有险种的需要) (insurance_coverage.csv)
###数据定义
field            | type         | Null | Comment
---------------- | ------------ | ---- | ------------------------------
coverage_id      | CHAR(11)     | N    | 保险责任编号
product_id       | CHAR(10)     | N    | 产品ID
name             | VARCHAR(50)  | N    | 保险责任名称, 如生存金
description      | VARCHAR(500) | N    | 保险责任详情

###示例
coverage_id      | product_id   | name  | description
---------------- | ------------ | ----- | ------------------------
CVG00000001      | I001712002   | 养老金 | XXXXXXXXXXXXXXXXXXXXXXX
CVG00000002      | I001712002   | 祝寿金 | XXXXXXXXXXXXXXXXXXXXXXX

##INSURANCE_PAYMENT_MODE 交费方式表 (insurance_payment_mode.csv)
###数据定义
field           | type        | null | default | comment
--------------- | ----------- | ---- | ------- | -------
mode_id         | CHAR(11)    | N    |         | ID
product_id      | CHAR(10)    | N    |         | 保险产品ID
gender          | ENUM        | N    |         | 性别
period          | INT         | N    |         | 交费周期
term_type       | ENUM        | N    |         | 交费期间类型
term            | INT         | N    |         | 交费期间


###说明
1. term: 字段含义由term_type决定
2. term_type: 
    * A(Age), 表示交至多少岁
    * Y(Year), 表示交多少年
    * L(Lifetime), 表示终身交费, 此时term必须为0
    * O(Once), 表示趸交, 此时term和period都为0
3. mode_id并不是唯一主键, 当其他字段都相同, 仅仅period字段不同时, 意味着这两种交费方式仅仅是交费周期不同, 此时请使用同样mode_id, 即两种交费方式并不会影响费率
    
###示例
mode_id      | product_id   | gender | period   | term | term_type | 说明
------------ | ------------ | ------ | -------- | ---- | --------- | --------------------------------------------------------------------
PAY00000001  | I006815013   | F      | 1        | 10   | Y         | 女性, 每月交, 交10年, 选择此交费方式时, 最小投保年龄0岁, 最大投保年龄53岁
PAY00000001  | I006815014   | M      | 0        | 0    | O         | 男性, 趸交, 选择此交费方式时, 最小投保年龄2岁, 最大投保年龄54岁
PAY00000002  | I006815014   | M      | 6        | 50   | A         | 男性, 每半年交, 交至50岁, 选择此交费方式时, 最小投保年龄1岁, 最大投保年龄55岁
PAY00000003  | I006815014   | M      | 6        | 0    | L         | 男性, 每半年交, 交至终身, 选择此交费方式时, 最小投保年龄0岁, 最大投保年龄57岁

##INSURANCE_RATE_RATIO 费率比例表(insurance_rate_ratio.csv)
###数据定义
field           | type        | null | default | comment
--------------- | ----------- | ---- | ------- | -------
product_id      | CHAR(10)    | N    |         | 保险产品ID
period          | INT         | N    |         | 周期
type            | ENUM        | N    |         | 类型
ratio           | INT         | N    |         | 比例

###说明
1. 默认的交费周期和领取年金周期是年, 当按月, 按多年交费或领取时, 默认费率乘以相应比例即可
2. type: 
    * P(Payment): 交费
    * W(Withdraw): 领取

###示例
product_id   | period | type | ratio | 说明 
------------ | ------ | ---- | ----- | ---------------------
I006815014   | 6      | P    | 501   | 按半年交费时, 保费为, 按年交费的保费乘以50.1%
I006815014   | 1      | P    | 84    | 按月交费时, 保费为, 按年交费的保费乘以8.4%
I006815014   | 3      | W    | 251   | 按每三个月领取时, 领取金额为, 按年领取的金额乘以25.1%

##INSURANCE_TERM 保险期间表 (insurance_term.csv)
###数据定义
field           | type        | null | default | comment
--------------- | ----------- | ---- | ------- | -------
term_id         | CHAR(12)    | N    |         | ID
product_id      | CHAR(10)    | N    |         | 保险产品ID
gender          | ENUM        | N    |         | 性别
term_type       | ENUM        | N    |         | 保险期间类型
term            | INT         | N    |         | 保险期间

###说明
1. term: 字段含义由term_type决定
2. term_type: 
    * A(Age), 表示保至多少岁
    * Y(Year), 表示保多少年
    * L(Lifetime), 表示保至终身, 此时term必须为0

###示例 
term_id      | product_id | gender | term_type | term | 说明
------------ | ---------- | ------ | --------- | ---- | ------------
TERM00000001 | I9099101   | F      | A         | 55   | 女性, 保至55岁
TERM00000002 | I9099102   | M      | Y         | 10   | 男性, 保10年
TERM00000003 | I9099102   | M      | L         | 0    | 男性, 保至终身

##INSURNACE_WITHDRAW_MODE 领取方式表 (insurance_withdraw_mode.csv)
###数据定义
field            | type         | null | default  | comment
---------------- | ------------ | ---- | -------- | ------
mode_id          | CHAR(16)     | N    |          | ID
product_id       | CHAR(10)     | N    |          | 保险产品ID
coverage_id      | CHAR(11)     | N    |          | 保险责任ID
group_num        | INT          | N    |          | 分组编号
gender           | ENUM         | N    |          | 性别
period           | INT          | N    |          | 领取周期
start_type       | ENUM         | N    |          | 开始领取时间类型
start            | INT          | N    |          | 开始领取时间
end_type         | ENUM         | Y    |          | 结束领取时间类型
end              | INT          | Y    |          | 结束领取时间
value_type       | ENUM         | N    |          | 领取金计算方式
value            | INT          | N    |          | 领取金数额
value_extra_info | VARCHAR(200) | Y    |          | 领取金计算信息
effect_rate      | BOOL         | N    | false(0) | 是否影响费率率
refer_mode_id    | CHAR(16)     | Y    |          | 关联领取方式ID

###说明
1. group_num: group_num和coverage_id相同的项目, 只能择一选取
2. start_type:
    * A(Age): 从哪一岁开始
    * Y(Year): 从哪年开始
3. start字段含义由start_type决定
4. end_type:
    * A(Age): 到哪岁开始
    * Y(Year): 到哪年开始
    * L(Lifetime): 终身领取
5. end字段含义由end_type决定, 如果end_type=L, 字段值必须为0
6. period, 如果为0, 表示一次性领取, 此时end, end_type字段为空
7. value_type: 
    * DIR(Direct Amount): 直接给出金额
    * PRM(Premium): 每期保费
    * TPM(Total Premium): 总保费
    * AMT(Insurance Amount): 保额
    * EXP(Expression): 使用公式计算
    * TXT(Text): 使用文本说明
8. value字段含义由value_type决定, value_type=PRM, TPM, AMT是, value表示千分比
9. refer_mode_id字段存储关联领取方式, 如果领取A由领取B决定, 即选择领取B时, 领取A自动选定, 则领取A的refer_mode_id存储领取B的id

###示例

withdraw_id | product_id | coverage_id   | group_num | gender | period | start_type | start | end_type | end | value_type | value    |value_extra_info      | effect_rate | refer_mode_id
----------- | ---------- | ------------- | --------- | ------ | ------ | ---------- | ----- | -------- | --- | ---------- | -------- |--------------------- | ----------- | -------------
WITHDRAW001 | I9099101   | CVG00000001   | 1         | F      | 12     | Y          | 1     | A        | 59  | PRM        | 90       |另加分红              | 0           |
WITHDRAW002 | I9099101   | CVG00000001   | 2         | F      | 0      | A          | 40    |          | 0   | DIR        | 10000000 |                      | 0           |
----------- | ---------- | ------------- | --------- | ------ | ------ | ---------- | ----- | -------- | --- | ---------- | -------- |--------------------- | ----------- | -------------
WITHDRAW003 | I9099101   | CVG00000002   | 1         | F      | 0      | A          | 40    |          | 0   | DIR        | 10000000 |                      | 1           |
WITHDRAW004 | I9099101   | CVG00000002   | 1         | F      | 0      | A          | 50    |          | 0   | DIR        | 10000000 |                      | 1           |
WITHDRAW005 | I9099101   | CVG00000002   | 1         | F      | 0      | A          | 60    |          | 0   | DIR        | 10000000 |                      | 1           |
WITHDRAW003 | I9099101   | CVG00000002   | 2         | F      | 0      | A          | 40    |          | 0   | DIR        | 10000000 |                      | 1           |
WITHDRAW004 | I9099101   | CVG00000002   | 2         | F      | 0      | A          | 50    |          | 0   | DIR        | 10000000 |                      | 1           |
WITHDRAW005 | I9099101   | CVG00000002   | 2         | F      | 0      | A          | 60    |          | 0   | DIR        | 10000000 |                      | 1           |
----------- | ---------- | ------------- | --------- | ------ | ------ | ---------- | ----- | -------- | --- | ---------- | -------- |--------------------- | ----------- | -------------
WITHDRAW006 | I9099101   | CVG00000003   | 1         | F      | 12     | A          | 55    | L        | 0   | TPM        | 100      |每年递增10%           | 0           |
WITHDRAW007 | I9099101   | CVG00000003   | 2         | F      | 12     | A          | 60    | A        | 80  | AMT        | 100      |每年递增10%           | 0           |
WITHDRAW008 | I9099101   | CVG00000003   | 3         | F      | 0      | A          | 40    |          | 0   | DIR        | 10000000 |                      | 0           | WITHDRAW003 
WITHDRAW009 | I9099101   | CVG00000003   | 3         | F      | 0      | A          | 50    |          | 0   | DIR        | 10000000 |                      | 0           | WITHDRAW004
WITHDRAW010 | I9099101   | CVG00000003   | 3         | F      | 0      | A          | 60    |          | 0   | DIR        | 10000000 |                      | 0           | WITHDRAW005
WITHDRAW011 | I9099101   | CVG00000004   | 4         | F      | 0      | A          | 60    |          | 0   | EXP        |          |5% * TPM + 50% * AMT  | 0           |
WITHDRAW012 | I9099101   | CVG00000005   | 4         | F      | 0      | A          | 60    |          | 0   | TXT        |          |XXXXXXXXXXXXXXXXXXXX  | 0           |

**说明**  

* 第1条记录: 按保险责任1, 一年后开始到59岁止, 每年领取保费的9%, 另加分红
* 第2, 3, 4条记录: 按保险责任2, 可选择在40, 50, 或60岁时一次性领取10000元(10000000 / 1000), 该项选择会影响费率
* 第5, 6条记录: 按保险责任3, 从55岁开始到终身, 每年领取总保费的10%, 每年递增10%; 从60岁开始到80岁在止, 每年领取基本保额的10%, 每年递增10%;
* 第7条记录: 按保险责任4, 60岁时, 一次性领取总保费的5%加基本保额的50%
* 第8条记录: 按保险责任5, XXXXXXXXXXXXXXXXXXX (纯文本说明)

##INSURANCE_RATE 保险费率表 (insurance_rate.csv)
###数据定义
field            | type         | null | default | comment
---------------- | ------------ | ---- | ------- | ------------------------------
rate_id          | CHAR(12)     | N    | ID
product_id       | CHAR(10)     | N    | 产品ID
gender           | ENUM         | N    | 性别
age              | INT          | N    | 年龄
term_id          | CHAR(12)     | N    | 保险期间ID
payment_id       | CHAR(11)     | N    | 交费方式ID
premium          | INT          | N    | 保费
amount           | INT          | N    | 保额
factor_1         | VARCHAR(20)  | Y    | 影响因素1
factor_2         | VARCHAR(20)  | Y    | 影响因素2
factor_3         | VARCHAR(20)  | Y    | 影响因素3
factor_4         | VARCHAR(20)  | Y    | 影响因素4

###说明
1. term_id: 关联INSURANCE_TERM表ID
2. payment_id: 关联INSURANCE_PAYMENT表ID 
3. facotr_1 ~ factor_4: 预留四个字段, 用来存储影响费率的其他因素, 如年金类保险中的领取方式. 每类保险产品对这四个字段的使用方式是一致的

###示例
**下表中,factor_1表示领取方式, factor_2表示平准性还是递增性产品**

rate_id      | product_id  | gender  | age  | term_id         | payment_id  | premium    | amount   | factor_1   | factor_2 | factor_3 | factor_4 |
------------ | ----------- | ------- | ---- | --------------- | ----------- | ---------- | -------- | ---------- | -------- | -------- | -------- |
RATE00000001 | I001712002  | F       | 21   | TERM00000001    | PAY002      | 123456     | 1000000  | WITHRAW001 | S        |          |          |
RATE00000001 | I001712002  | M       | 22   | TERM00000001    | PAY002      | 123457     | 1000000  | WITHRAW002 | A        |          |          |

