# Native Contract 投票系统的设计

## 简介

该合约是一个投票系统，支持用 NEO 投票，可单选也可多选，投票是票数是自己地址中 NEO 的余额。

当投票结束后，有一个计票环节，执行且仅执行一次。

#### 功能包括：

[创建投票](#创建投票-create)

[编辑投票（可省略）](#编辑投票-edit)

[投票](#投票-vote)

[查看详情](#查看详情-getdetails)

[统计](#统计-getstatistic)

## 功能

### 创建投票 Create

```c#
Uint256 Create(Uint160 originator, string title, string description, object[] option, bool multiSelect, int voteDeadline, int statisticsDeadline)
```

#### 描述：

该方法的作用是让投票的发起者发起一次投票，需要填写此次投票的一些信息，最终返回调用该合约的 TxId 作为此次投票的唯一标识。

#### 参数：

originator 发起人，Uint160 类型的 ScriptHash

title 投票标题，string 类型

description 描述，string 类型

option 选项，object 数组

multiSelect 是否支持多选，bool 类型

voteDeadline 投票截止时间，截止时间后不能投票，此处为区块高度 int 类型

statisticsDeadline 统计票数的截止时间，要大于投票截止时间。投票的发起人要在该时间前调用 GetDetails 来进行统计票数的操作。要求投票者在投票截止时间和统计票数截止时间之间尽量不要转账，以免出现票数错误。此处为区块高度 int 类型

#### 返回值:

交易 ID，作为此次投票的唯一标识，Uint256 类型

### 编辑投票 Edit

```c#
Uint256 Edit(UInt256 id, string title, string description, object[] option, bool multiSelect, int voteDeadline, int statisticsDeadline, Uint160)
```

#### 描述：

该方法的作用是让投票的发起者修改投票信息，仅限刚创建好还没有其它人投票的时候。编辑投票需要填写之前创建好的投票的 ID，和信息。

#### 参数：

id，所编辑的投票

其它参与同 Create 方法

#### 返回值:

交易 ID，作为此次投票的唯一标识，Uint256 类型

### 投票 Vote

```c#
bool Vote(UInt256 id, Uint160 voter, object[] option)
```

#### 描述：

参与者用来投票的方法，填写投票的 ID，自己的地址以及选项，截止时间后不能投票。投票后，该地址里的 NEO 资产数量将作为票数。

#### 参数：

id 投票的 ID

voter 投票人

option 选项

#### 返回值：

投票是否成功，bool 类型

### 查看详情 GetDetails

```c#
RawDetails GetDetails(UInt256 id)
```

```c#
class RawDetails
{
    UInt256 id;
    UInt160 originator;
    string title;
    string description;
    object[] option;
    bool multiSelect;
    int deadline;
    RawData[] rawData; //原始数据
}
class RawData
{
    object[] option;
    UInt160 address;
}
```

#### 描述：

查询投票的信息。

获得投票的原始信息，返回的对象的 rawData 字段中包含每个地址投了哪个选项，任何时刻都可调用。

#### 参数：

id 投票的 ID

#### 返回值：

Deatils 类型的投票结果

### 统计 GetStatistic

```c#
Statistic GetStatistic(UInt256 id)
```

```c#
class Statistic
{
    UInt256 id;
    UInt160 originator;
    string title;
    string description;
    object[] option;
    bool multiSelect;
    int deadline;
    StatisticalData[] statisticalData; //统计数据
}
class StatisticalData
{
    object[] option;
    int voteCount;
}
```

#### 描述：

查询投票的统计信息。

获得统计信息，返回的对象的 statisticalData 字段中包含每个选项获得了多少票数。

该方法需要在 voteDeadline 之后，voteDeadline 之前调用一次，投票的发起人（或其它任何人）需要在这一段时间内进行调用，否则投票将失效。第一次会比较复杂，要计算每个地址的 NEO 余额。统计信息后将其存储，之后随时都可再次调用，再次调用时直接读取数据，不再进行复杂计算。

#### 参数：

id 投票的 ID

#### 返回值：

Statistic 类型的投票结果


