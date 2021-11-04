# Nginx日志统计说明
目前各个业务日志都已经收入click house，ES逐步放弃。<br>
原来配置的ES的图表需要切换到click house

## 一、语法说明
click house天生支持sql，所有的数据都用sql选取

总数：<br>
```
SELECT
    $timeSeries as t,
    count()
FROM $table

WHERE $timeFilter

GROUP BY t

ORDER BY t
```

饼图分组：
```
SELECT
    $timeSeries as t,
    api,
    count()
FROM $table

WHERE $timeFilter

GROUP BY t, api

ORDER BY t
```
饼图分组有个特例，就是响应时间。<br>
因为响应时间是几个时间段，ES统计的时候是通过列不等式获得的。这个不等的区间需要自己写。<br>
新版本把区间放到web页面上，可以自行配置。grafana面板最好用opentsdb的数据，这样就不用单独写区间范围，它可以跟着web页的配置自动变化

如果还有特殊需求，这里给个例子，照着改就行：
<font color='red'>注意备注，必须是反引号</font>
```
SELECT
    $timeSeries as t,
    count(if (toFloat32OrZero(requesttime)>=1 and toFloat32OrZero(requesttime) < 3, 1, null)) `1到3秒`,
    count(if (toFloat32OrZero(requesttime)>=3 and toFloat32OrZero(requesttime) < 5, 1, null)) `3到5秒`,
    count(if (toFloat32OrZero(requesttime)>=5, 1, null)) `5秒以上`,
    count(if (toFloat32OrZero(requesttime)>=0.1 and toFloat32OrZero(requesttime) < 0.3, 1, null)) `100毫秒300毫秒`,
    count(if (toFloat32OrZero(requesttime)>=0.3 and toFloat32OrZero(requesttime) < 0.5, 1, null)) `300毫秒500毫秒`,
    count(if (toFloat32OrZero(requesttime)>=0.5 and toFloat32OrZero(requesttime) < 1, 1, null)) `500毫秒到1秒`,
    count(if (toFloat32OrZero(requesttime)<0.1, 1, null)) `100毫秒以内`
FROM $table

WHERE $timeFilter

GROUP BY t

ORDER BY t
```

## 二、涉及范围
所有nginx相关的数据都需要改。

改的方向有两个，一个是走opentsdb，一个是走clickhouse。<br>
建议走opentsdb，这样后续再换数据源，也不用大改。clickhouse好处是数据相对实时，数据展示更灵活，有特殊需求的改click house。