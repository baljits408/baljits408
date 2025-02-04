--------|---------|-------------|-----
byte_unit | byte | Default numeric output(s) | str
enabled | False | Enable collecting these metrics | bool
host | localhost | Hostname | str
host | localhost | Hostname, using http scheme by default. For https pass e.g. "https://localhost" | str
measure_collector_time | False | Collect the collector run time in ms | bool
metrics_blacklist | None | Regex to match metrics to block. Mutually exclusive with metrics_whitelist | NoneType
metrics_whitelist | None | Regex to match metrics to transmit. Mutually exclusive with metrics_blacklist | NoneType
