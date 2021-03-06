# Scipy 以及其它

```py
# 来源：NumPy Biginner's Guide 2e ch10
```

## 保存和加载 mat 文件

```py
import numpy as np
from scipy import io

a = np.arange(7)

# 将 a 作为 array 保存到 mat 文件中
io.savemat("a.mat", {"array": a})

'''
matlab/octave:
octave-3.4.0:7> load a.mat
octave-3.4.0:8> a

octave-3.4.0:8> array
array =

  0
  1
  2
  3
  4
  5
  6
'''
```

## 分析随机值

```py
from scipy import stats
import matplotlib.pyplot as plt

# 生成 900 个随机数，满足正态分布
generated = stats.norm.rvs(size=900)

# 获取均值和标准差
print “Mean”, “Std”, stats.norm.fit(generated)
# Mean Std (0.0071293257063200707, 0.95537708218972528)

# 偏斜测试，第二个返回值是数据服从正态分布的概率
print “Skewtest”, “pvalue”, stats.skewtest(generated)
# Skewtest pvalue (-0.62120640688766893, 0.5344638245033837)

# 峰值测试，第二个返回值是数据服从正态分布的概率
print “Kurtosistest”, “pvalue”, stats.kurtosistest(generated)
# Kurtosistest pvalue (1.3065381019536981, 0.19136963054975586)

# 正态性测试，第二个返回值是数据服从正态分布的概率
print “Normaltest”, “pvalue”, stats.normaltest(generated)
# Normaltest pvalue (2.09293921181506, 0.35117535059841687)

# 打印第 95 个百分位数
print “95 percentile”, stats.scoreatpercentile(generated, 95)
# 95 percentile 1.54048860252

# 打印第一个百分位数
print “Percentile at 1”, stats.percentileofscore(generated, 1)
# Percentile at 1 85.5555555556

# 绘制图像
plt.hist(generated)
plt.show()
```

## 比较股票收益

```py
from matplotlib.finance import quotes_historical_yahoo
from datetime import date
import numpy as np
from scipy import stats
from statsmodels.stats.stattools import jarque_bera
import matplotlib.pyplot as plt

# 根据股票代码获取一年内的收盘价
def get_close(symbol):
   today = date.today()
   start = (today.year - 1, today.month, today.day)

   quotes = quotes_historical_yahoo(symbol, start, today)
   quotes = np.array(quotes)

   return quotes.T[4]

# 获取 SPY 和 DIA 的对数收益
spy =  np.diff(np.log(get_close(“SPY”)))
dia =  np.diff(np.log(get_close(“DIA”)))

# 检查两个样本是否均值相同，第二个返回值是概率
print “Means comparison”, stats.ttest_ind(spy, dia)
# Means comparison (-0.017995865641886155, 0.98564930169871368)

# K-S 检验告诉我们两个样本是否服从相同分布，第二个返回值是概率
print “Kolmogorov smirnov test”, stats.ks_2samp(spy, dia)

# 对两个收益的差值调用 Jarque-Bera 检验
print “Jarque Bera test”, jarque_bera(spy - dia)[1]

# 绘制两个股票收益，以及收益差值的直方图
plt.hist(spy, histtype=”step”, lw=1, label=”SPY”)
plt.hist(dia, histtype=”step”, lw=2, label=”DIA”)
plt.hist(spy - dia, histtype=”step”, lw=3, label=”Delta”)
plt.legend()
plt.show()
```

## 检测股票趋势

```py
from matplotlib.finance import quotes_historical_yahoo
from datetime import date
import numpy as np
from scipy import signal
import matplotlib.pyplot as plt
from matplotlib.dates import DateFormatter
from matplotlib.dates import DayLocator
from matplotlib.dates import MonthLocator

# 获取 QQQ 一年的收盘价
today = date.today()
start = (today.year - 1, today.month, today.day)

quotes = quotes_historical_yahoo(“QQQ”, start, today)
quotes = np.array(quotes)

dates = quotes.T[0]
qqq = quotes.T[4]

# signal.detrend 移除信号的线性趋势
# 返回值为 y - linearfit(x)
y = signal.detrend(qqq)

# 创建月和日的 Locator
alldays = DayLocator()              
months = MonthLocator()
# 创建日期的 Formatter
month_formatter = DateFormatter(“%b %Y”)

fig = plt.figure()
ax = fig.add_subplot(111)

# 绘制收益的图像，和拟合直线的图像
plt.plot(dates, qqq, ‘o’, dates, qqq - y, ‘-’)
# 设置 Locator 和 Formatter
ax.xaxis.set_minor_locator(alldays)
ax.xaxis.set_major_locator(months)
ax.xaxis.set_major_formatter(month_formatter)
# 将 x 轴标签格式化为日期
fig.autofmt_xdate()
plt.show()
```