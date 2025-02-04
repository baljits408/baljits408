 value = metric.raw_value
                if metric.path in self.old_values:
                    value = value - self.old_values[metric.path]
                # provide an option for caller to manually set the value
                elif metric.value:
                    value = metric.value
                self.old_values[metric.path] = metric.raw_value

                if hasattr(statsd, 'StatsClient'):
