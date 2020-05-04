# 使用Mathematica绘制股票相关曲线

Mathematica是个好东西。虽然早有耳闻功能强大，但是没想到强大到这种地步。

原本因为课程原因，研究了一下如何在Mathematica里绘制股票数据。不过现在**停止**，只完成了一小部分。剩下的……想折腾的话，按照已经有的代码，能容易改出来其他功能。

## 数据来源

首先要说的是，Mathematica中**自行提供**了非权威的金融数据。这对我们很方便。

使用命令`FinancialData`,就可以获取大量需要的数据。具体可以查看文档。不过问题是，不知何种原因，该函数对深沪股票支持很差，所以在实际使用时，并不能直接使用该函数，比较可惜……

我们需要实现自己的数据处理。

经过一番寻找后，我发现了一个基于Python的证券数据平台`baostock`。经过简单的封装后，Mathematica就可以使用Python从平台上拉取数据了。

这个例子只能拉取日K。

```mathematica

BeginPackage["StockLink`"]

StockLink::usage="StockLink";
CloseLink::usage="CloseLink";

DailyKLine::usage="Get KLine data";

CandlestickData::usage="adjust KLine data to fit Candlestick Chart";
TradingChartData::usage="adjust KLine data to fit Trading Chart";

Begin["`Private`"]

StockLink[]:=Module[{conn},
conn=StartExternalSession["Python"];
ExternalEvaluate[conn,"
import baostock as bs
lg=bs.login()
"];
conn
];

CloseLink[conn_]:=Module[{},
ExternalEvaluate[conn,"
bs.logout()
"];
DeleteObject[conn];
];

MMAListToPythonList[list_]:=StringReplace[ToString[list,InputForm],{"\" "->"'","{"->"[","}"->"]"}];
DateObjectToStr[date_]:=DateString[date,"ISODate"];

DailyKLine[conn_,code_,startDate_,endDate_]:=Module[{},
raw=ExternalEvaluate[conn,
StringTemplate["
rs = bs.query_history_k_data_plus('``',
    'date,open,high,low,close,preclose,volume,amount,adjustflag,turn,tradestatus,pctChg,isST',
    start_date='``', end_date='``',
    frequency='d', adjustflag='3')
data_list = {}
while (rs.error_code == '0') & rs.next():
    temp=rs.get_row_data()
    data_list[temp[0]]=temp[1:];
data_list"][code,DateObjectToStr[startDate],DateObjectToStr[endDate]]];
ToExpression[#]& /@ raw
];

CandlestickData[raw_]:=Table[{DateObject[key],raw[key][[1;;4]]},{key,Keys[raw]}];
TradingChartData[raw_]:=Table[{DateObject[key],raw[key][[1;;5]]},{key,Keys[raw]}];

End[]

EndPackage[]
```

## 图表绘制

在实际使用时，首先需要导入该库，并且初始化链接后，灵活组合各部分功能即可。

```mathematica
<< (NotebookDirectory[] <> "StockLink.wl");
conn = StockLink[];

(:例如获取某个日期区间内的数据并转化，绘制交互图标:)
DailyKLine[conn, "ss.000001", Today, Today] // TradingChartData // InteractiveTradingChart
```

输入日期时，可以直接按`ctrl+=`，在框中描述时间，比较方便。

## 可能遇到的问题

如果python配置不对的话，可能会出现Mathematica找不到python。问题的解决方法在官方文档上非常清晰。

不过我不折腾股票，就这样了。
