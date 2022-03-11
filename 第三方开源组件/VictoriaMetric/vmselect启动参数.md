./vmselect-prod --help
vmselect-20201116-190952-tags-v1.47.0-cluster-0-ga00df790b
Usage of ./vmselect-prod:

#### 缓存文件的存储地址，如果不配置则不启用缓存

-cacheDataPath string
Path to directory for cache files. Cache isn't saved if empty

#### 如果同一个 metric 的两条数据时间差小于该值，则作为重复数据删除。如果为 0 则不启用去重；

-dedup.minScrapeInterval duration
Remove superflouos samples from time series if they are located closer to each other than this duration. This may be useful for reducing overhead when multiple identically configured Prometheus instances write data to the same VictoriaMetrics. Deduplication is disabled if the -dedup.minScrapeInterval is 0
-enableTCP6
Whether to enable IPv6 for listening and dialing. By default only IPv4 TCP is used
-envflag.enable
Whether to enable reading flags from environment variables additionally to command line. Command line flag values have priority over values from environment vars. Flags are read only from command line if this flag isn't set
-envflag.prefix string
Prefix for environment variables if -envflag.enable is set
-fs.disableMmap
Whether to use pread() instead of mmap() for reading data files. By default mmap() is used for 64-bit arches and pread() is used for 32-bit arches, since they cannot read data files bigger than 2^32 bytes in memory. mmap() is usually faster for reading small data chunks than pread()
-http.connTimeout duration
Incoming http connections are closed after the configured timeout. This may help spreading incoming load among a cluster of services behind load balancer. Note that the real timeout may be bigger by up to 10% as a protection from Thundering herd problem (default 2m0s)
-http.disableResponseCompression
Disable compression of HTTP responses for saving CPU resources. By default compression is enabled to save network bandwidth
-http.idleConnTimeout duration
Timeout for incoming idle http connections (default 1m0s)
-http.maxGracefulShutdownDuration duration
The maximum duration for graceful shutdown of HTTP server. Highly loaded server may require increased value for graceful shutdown (default 7s)
-http.pathPrefix string
An optional prefix to add to all the paths handled by http server. For example, if '-http.pathPrefix=/foo/bar' is set, then all the http requests will be handled on '/foo/bar/\*' paths. This may be useful for proxied requests. See https://www.robustperception.io/using-external-urls-and-proxies-with-prometheus
-http.shutdownDelay duration
Optional delay before http server shutdown. During this dealy the servier returns non-OK responses from /health page, so load balancers can route new requests to other servers
-httpListenAddr string
Address to listen for http connections (default ":8481")
-loggerDisableTimestamps
Whether to disable writing timestamps in logs
-loggerErrorsPerSecondLimit int
Per-second limit on the number of ERROR messages. If more than the given number of errors are emitted per second, then the remaining errors are suppressed. Zero value disables the rate limit (default 10)
-loggerFormat string
Format for logs. Possible values: default, json (default "default")
-loggerLevel string
Minimum level of errors to log. Possible values: INFO, WARN, ERROR, FATAL, PANIC (default "INFO")
-loggerOutput string
Output for the logs. Supported values: stderr, stdout (default "stderr")
-memory.allowedBytes value
Allowed size of system memory VictoriaMetrics caches may occupy. This option overrides -memory.allowedPercent if set to non-zero value. Too low value may increase cache miss rate, which usually results in higher CPU and disk IO usage. Too high value may evict too much data from OS page cache, which will result in higher disk IO usage
Supports the following optional suffixes for values: KB, MB, GB, KiB, MiB, GiB (default 0)
-memory.allowedPercent float
Allowed percent of system memory VictoriaMetrics caches may occupy. See also -memory.allowedBytes. Too low value may increase cache miss rate, which usually results in higher CPU and disk IO usage. Too high value may evict too much data from OS page cache, which will result in higher disk IO usage (default 60)

#### 当前时间和响应数据的最大差值，只查询原始数据，不适用缓存。如果发现数据源和 VictoriaMetrics 之间的时间同步存在问题，请增加此值。

-search.cacheTimestampOffset duration
The maximum duration since the current time for response data, which is always queried from the original raw data, without using the response cache. Increase this value if you see gaps in responses due to time synchronization issues between VictoriaMetrics and data sources (default 5m0s)

#### 如果部分-storageNode 实例无法执行查询时，是否拒绝部分响应。此项在保证一致性的时候牺牲了可用性

-search.denyPartialResponse
Whether to deny partial responses if a part of -storageNode instances fail to perform queries; this trades availability over consistency; see also -search.maxQueryDuration and -search.storageTimeout

#### 禁止返回数据缓存。此项在数据回填时候很有用

-search.disableCache
Whether to disable response caching. This may be useful during data backfilling

-search.latencyOffset duration
The time when data points become visible in query results after the collection. Too small value can result in incomplete last points for query results (default 30s)

#### 在日志中记录超过此时间的查询。0 则禁用慢查询日志

-search.logSlowQueryDuration duration
Log queries with execution time exceeding this value. Zero disables slow query logging (default 5s)

#### 查询搜索的并发数，不应该很大，因为单个搜索有可能让所有 cpu 饱和。

-search.maxConcurrentRequests int
The maximum number of concurrent search requests. It shouldn't be high, since a single request can saturate all the CPU cores. See also -search.maxQueueDuration (default 4)

#### /api/v1/export 导出接口的最大持续时间

-search.maxExportDuration duration
The maximum duration for /api/v1/export call (default 720h0m0s)

####

-search.maxLookback duration
Synonim to -search.lookback-delta from Prometheus. The value is dynamically detected from interval between time series datapoints if not set. It can be overridden on per-query basis via max_lookback arg. See also '-search.maxStalenessInterval' flag, which has the same meaining due to historical reasons

#### 一个请求中单个时间序列的最大点数

-search.maxPointsPerTimeseries int
The maximum points per a single timeseries returned from the search (default 30000)

#### 查询执行的最大持续时间

-search.maxQueryDuration duration
The maximum duration for query execution; see also -search.storageTimeout (default 30s)

#### 查询语句长度的最大字节数

-search.maxQueryLen value
The maximum search query length in bytes
Supports the following optional suffixes for values: KB, MB, GB, KiB, MiB, GiB (default 16384)

#### 达到 -search.maxConcurrentRequests 时，查询请求的最大等待时间

-search.maxQueueDuration duration
The maximum time the request waits for execution when -search.maxConcurrentRequests limit is reached; see also -search.maxQueryDuration (default 10s)

####

-search.maxStalenessInterval duration
The maximum interval for staleness calculations. By default it is automatically calculated from the median interval between samples. This flag could be useful for tuning Prometheus data model closer to Influx-style data model. See https://prometheus.io/docs/prometheus/latest/querying/basics/####staleness for details. See also '-search.maxLookback' flag, which has the same meaning due to historical reasons

####

-search.minStalenessInterval duration
The minimum interval for staleness calculations. This flag could be useful for removing gaps on graphs generated from time series with irregular intervals between samples. See also '-search.maxStalenessInterval'

#### 可选 authKey，用于通过/internal/resetRollupResultCache 接口来重置缓存

-search.resetCacheAuthKey string
Optional authKey for resetting rollup cache via /internal/resetRollupResultCache call

#### 在每个存储上的查询超时时间。该项允许某些 storage 节点查询缓慢时，返回部分查询数据

-search.storageTimeout duration
The timeout for per-storage query processing; this allows returning partial responses if certain -storageNode instances slowly process the query; see also -search.maxQueryDuration and -search.denyPartialResponse command-line flags

-search.treatDotsAsIsInRegexps
Whether to treat dots as is in regexp label filters used in queries. For example, foo{bar=~"a.b.c"} will be automatically converted to foo{bar=~"a\\.b\\.c"}, i.e. all the dots in regexp filters will be automatically escaped in order to match only dot char instead of matching any char. Dots in ".+", ".\*" and ".{n}" regexps aren't escaped. Such escaping can be useful when querying Graphite data

#### 查询节点列表。

-selectNode array
Addresses of vmselect nodes; usage: -selectNode=vmselect-host1:8481 -selectNode=vmselect-host2:8481
Supports array of values separated by comma or specified via multiple flags.

#### 存储节点列表

-storageNode array
Addresses of vmstorage nodes; usage: -storageNode=vmstorage-host1:8401 -storageNode=vmstorage-host2:8401
Supports array of values separated by comma or specified via multiple flags.
-tls
Whether to enable TLS (aka HTTPS) for incoming requests. -tlsCertFile and -tlsKeyFile must be set if -tls is set
-tlsCertFile string
Path to file with TLS certificate. Used only if -tls is set. Prefer ECDSA certs instead of RSA certs, since RSA certs are slow
-tlsKeyFile string
Path to file with TLS key. Used only if -tls is set
-version
Show VictoriaMetrics version
