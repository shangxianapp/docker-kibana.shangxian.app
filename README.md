# 服务日志监控可视化

## 服务和端口

- Elasticsearch
    - `kibana-elasticsearch` - 容器名称
    - `9200` - 端口
- Fluentd
    - `kibana-fluentd` - 容器名称
    - `8989` - Nginx Access 日志 UDP 端口
    - `8988` - Nginx Error 日志 UDP 端口
    - `24224` - Node.js 日志 UDP 端口
- Kibana
    - `kibana` - 容器名称
    - `5610` - 端口

## 域名

<https://kibana.shangxian.app>

## 目录

```
├── README.md
├── docker-compose.yml
├── pwd                                 - Nginx HTTP Basic Authentication
├── fluentd                             - Fluentd 编译
│   ├── Dockerfile
│   ├── conf
│   │   ├── node.conf                   - Node.js 日志配置
│   │   ├── nginx-access.conf           - Nginx access 日志收集配置
│   │   └── nginx-error.conf            - Nginx error 日志收集配置
│   ├── fluent.conf                     - Fluentd 配置
│   └── plugins
```

## Kibana 常用字段说明

- `env` - 标签，用来区分项目
- `env_source` - 标签来源，如：`node`、`nginx`
- `env_level` - 日志类型，如：`access`、`error`、`debug`、`info`
- `env_timestamp` - 日志源（Node.js、Nginx）发生时间，而`@timestamp` 是写入 ES 时的时间

## Nginx 日志接入

使用 `log_format` 配置一个 Fluentd 的日志格式，如：

```nginx
http {
    log_format Fluentd '{ "env_timestamp": "$time_iso8601", '
                        '"remote_addr": "$remote_addr", '
                        '"remote_user": "$remote_user", '
                        '"body_bytes_sent": "$body_bytes_sent", '
                        '"request_time": "$request_time", '
                        '"server_protocol": "$server_protocol", '
                        '"status": "$status", '
                        '"http_host": "$http_host", '
                        '"request_uri": "$request_uri", '
                        '"pid": "$pid", '
                        '"request_filename": "$request_filename", '
                        '"upstream_addr": "$upstream_addr",'
                        '"upstream_response_time": "$upstream_response_time",'
                        '"request_method": "$request_method", '
                        '"http_referrer": "$http_referer", '
                        '"http_x_forwarded_for": "$http_x_forwarded_for", '
                        '"nginx_cache_status": "$upstream_cache_status", '
                        '"http_user_agent": "$http_user_agent" }';
}
```

在使用时直接接入 Fluentd syslog 日志，如：

```
access_log syslog:server=fluentd:8989,tag=www Fluentd;
error_log syslog:server=fluentd:8988,tag=www error;
```

字段说明：

- `8989` - Fluentd 暴露的 `nginx.access` 收集端口
- `8988` - Fluentd 暴露的 `nginx.error` 收集端口
- `tag=www` - 使用 `tag` 透传一个日志中的 `env` 字段，用来做产品线/名称的筛选条件
- `fluentd:port` - Fluentd 服务容器名，在 Docker 内直连
- `error_log ... error` - 错误日志等级

## License

MIT