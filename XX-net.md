netsh interface Teredo show state

netsh int ipv6 set teredo enterpriseclient default default default
第1个 default：服务器名称，或 default 或域名或 IP
第2个 default：客户端刷新间隔，或 default 或自定义数字（秒）
第3个 default：客户端端口，或 default 或自定义数字（4-5位数字？）
此3个参数在命令中的位置固定不变，可以按需从命令末尾开始少1-3个参数，或参见运行 netsh int ipv6 set teredo 后的命令提示。或仅Win10可用 natawareclient 替换 enterpriseclient