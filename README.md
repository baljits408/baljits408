          'percore':  'Collect metrics per cpu core or just total',
            'simple':   'only return aggregate CPU% metric',
            'normalize': 'for cpu totals, divide by the number of CPUs',
            'derivative': 'use derivative values for metrics',
        })
        return config_help

@@ -58,6 +59,7 @@ def get_default_config(self):
            'xenfix':   None,
            'simple':   'False',
            'normalize': 'False',
            'derivative': 'True',
        })
        return config

@@ -89,6 +91,10 @@ def cpu_delta_time(interval):
                post_check[i] -= pre_check[i]
            return post_check

        use_derivative = str_to_bool(self.config['derivative'])
        use_normalization = str_to_bool(self.config['normalize'])
        metrics = {}
        if os.access(self.PROC, os.R_OK):

            # If simple only return aggregate CPU% metric
@@ -143,26 +149,24 @@ def cpu_delta_time(interval):
            # Close File
            file.close()

            metrics = {'cpu_count': ncpus}
            metrics['cpu_count'] = ncpus

            for cpu in results.keys():
                stats = results[cpu]
                for s in stats.keys():
                    # Get Metric Name
                    metric_name = '.'.join([cpu, s])
                    # Get actual data
                    if ((str_to_bool(self.config['normalize']) and
                         cpu == 'total' and
                         ncpus > 0)):
                    div = 1
                    if use_normalization and cpu == 'total' and ncpus > 0:
                        div = ncpus
                    if use_derivative:
                        metrics[metric_name] = self.derivative(
                            metric_name,
                            long(stats[s]),
                            self.MAX_VALUES[s]) / ncpus
                            self.MAX_VALUES[s]) / div
                    else:
                        metrics[metric_name] = self.derivative(
                            metric_name,
                            long(stats[s]),
                            self.MAX_VALUES[s])
                        metrics[metric_name] = long(stats[s]) / div

            # Check for a bug in xen where the idle time is doubled for guest
            # See https://bugzilla.redhat.com/show_bug.cgi?id=624756
@@ -182,13 +186,6 @@ def cpu_delta_time(interval):
                else:
                    self.config['xenfix'] = False

            # Publish Metric Derivative
            for metric_name in metrics.keys():
                self.publish(metric_name,
                             metrics[metric_name],
                             precision=2)
            return True
        else:
            if not psutil:
                self.log.error('Unable to import psutil')
@@ -198,69 +195,70 @@ def cpu_delta_time(interval):
            cpu_time = psutil.cpu_times(True)
            cpu_count = len(cpu_time)
            total_time = psutil.cpu_times()
            for i in range(0, len(cpu_time)):
                metric_name = 'cpu' + str(i)
                self.publish(
                    metric_name + '.user',
                    self.derivative(metric_name + '.user',
                                    cpu_time[i].user,
                                    self.MAX_VALUES['user']),
                    precision=2)
                if hasattr(cpu_time[i], 'nice'):
                    self.publish(
                        metric_name + '.nice',
                        self.derivative(metric_name + '.nice',
                                        cpu_time[i].nice,
                                        self.MAX_VALUES['nice']),
                        precision=2)
                self.publish(
                    metric_name + '.system',
                    self.derivative(metric_name + '.system',
                                    cpu_time[i].system,
                                    self.MAX_VALUES['system']),
                    precision=2)
                self.publish(
                    metric_name + '.idle',
                    self.derivative(metric_name + '.idle',
                                    cpu_time[i].idle,
                                    self.MAX_VALUES['idle']),
                    precision=2)
            metric_name = 'total'
            self.publish(
                metric_name + '.user',
                self.derivative(metric_name + '.user',
                                total_time.user,
                                self.MAX_VALUES['user']) / cpu_count,
                precision=2)
            if hasattr(total_time, 'nice'):
                self.publish(
                    metric_name + '.nice',
                    self.derivative(metric_name + '.nice',
                                    total_time.nice,
                                    self.MAX_VALUES['nice']) / cpu_count,
                    precision=2)
            self.publish(
                metric_name + '.system',
                self.derivative(metric_name + '.system',
                                total_time.system,
                                self.MAX_VALUES['system']) / cpu_count,
                precision=2)
            self.publish(
                metric_name + '.idle',
                self.derivative(metric_name + '.idle',
                                total_time.idle,
                                self.MAX_VALUES['idle']) / cpu_count,
                precision=2)
            self.publish('cpu_count', psutil.cpu_count())
            return True
        return None
            metrics['cpu_count'] = cpu_count
            if str_to_bool(self.config['percore']):
                for i in range(0, len(cpu_time)):
                    cpu = 'cpu' + str(i)
                    if use_derivative:
                        metrics[cpu + '.user'] = self.derivative(
                            cpu + '.user',
                            long(cpu_time[i].user),
                            self.MAX_VALUES['user'])
                        metrics[cpu + '.system'] = self.derivative(
                            cpu + '.system',
                            long(cpu_time[i].system),
                            self.MAX_VALUES['system'])
                        metrics[cpu + '.idle'] = self.derivative(
                            cpu + '.idle',
                            long(cpu_time[i].idle),
                            self.MAX_VALUES['idle'])
                        if hasattr(cpu_time[i], 'nice'):
                            metrics[cpu + '.nice'] = self.derivative(
                                cpu + '.nice',
                                long(cpu_time[i].nice),
                                self.MAX_VALUES['nice'])
                    else:
                        metrics[cpu + '.user'] = long(cpu_time[i].user)
                        metrics[cpu + '.system'] = long(cpu_time[i].system)
                        metrics[cpu + '.idle'] = long(cpu_time[i].idle)
                        if hasattr(cpu_time[i], 'nice'):
                            metrics[cpu + '.nice'] = long(cpu_time[i].nice)
            div = 1
            if use_normalization and cpu_count > 0:
                div = cpu_count
            if use_derivative:
                metrics['total.user'] = self.derivative(
                    'total.user',
                    long(total_time.user),
                    self.MAX_VALUES['user']) / div
                metrics['total.system'] = self.derivative(
                    'total.system',
                    long(total_time.system),
                    self.MAX_VALUES['system']) / div
                metrics['total.idle'] = self.derivative(
                    'total.idle',
                    long(total_time.idle),
                    self.MAX_VALUES['idle']) / div
                if hasattr(total_time, 'nice'):
                    metrics['total.nice'] = self.derivative(
                        'total.nice',
                        long(total_time.nice),
                        self.MAX_VALUES['nice']) / div
            else:
                metrics['total.user'] = long(total_time.user) / div
                metrics['total.system'] = long(total_time.system) / div
                metrics['total.idle'] = long(total_time.idle) / div
                if hasattr(total_time, 'nice'):
                    metrics['total.nice'] = long(total_time.nice) / div
        # Publish Metric
        for metric_name in metrics.keys():
            self.publish(metric_name,
                         metrics[metric_name],
                         precision=2)
        return True
‎src/diamond/collector.py
+3
-2
Original file line number	Diff line number	Diff line change
@@ -5,6 +5,7 @@
"""

import os
import platform
import socket
import platform
import logging
@@ -91,14 +92,14 @@ def get_hostname(config, method=None):
        return hostname

    if method == 'uname_short':
        hostname = os.uname()[1].split('.')[0]
        hostname = platform.uname()[1].split('.')[0]
        get_hostname.cached_results[method] = hostname
        if hostname == '':
            raise DiamondException('Hostname is empty?!')
        return hostname

    if method == 'uname_rev':
        hostname = os.uname()[1].split('.')
        hostname = platform.uname()[1].split('.')
        hostname.reverse()
        hostname = '.'.join(hostname)
        get_hostname.cached_results[method] = hostname
