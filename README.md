'/proc/net/snmp',
    ]

    PROC6 = [
        '/proc/net/snmp6'
    ]
    GAUGES = [
        'Forwarding',
        'DefaultTTL',
    ]

    DEFAULT_METRICS = [
        'InAddrErrors', 'InDelivers', 'InDiscards', 'InHdrErrors',
        'InReceives', 'InUnknownProtos', 'OutDiscards', 'OutNoRoutes',
        'OutRequests',
        'Ip6InAddrErrors', 'Ip6InDelivers', 'Ip6InDiscards',
        'Ip6InHdrErrors', 'Ip6InReceives', 'Ip6InUnknownProtos',
        'Ip6OutDiscards', 'Ip6OutNoRoutes', 'Ip6OutRequests'
    ]
    def process_config(self):
        super(IPCollector, self).process_config()
        if self.config['allowed_names'] is None:
@@ -56,30 +69,38 @@ def get_default_config(self):
        config = super(IPCollector, self).get_default_config()
        config.update({
            'path': 'ip',
            'allowed_names': 'InAddrErrors, InDelivers, InDiscards, ' +
            'InHdrErrors, InReceives, InUnknownProtos, OutDiscards, ' +
            'OutNoRoutes, OutRequests'
            'allowed_names': ','.join(self.DEFAULT_METRICS)
        })
        return config

    def open_file(self, filepath):
        if not os.access(filepath, os.R_OK):
            self.log.error('Permission to access %s denied', filepath)
            return
        fp = open(filepath)
        if not fp:
            self.log.error('Failed to open %s', filepath)
            return
        return fp
    def collect(self):
        self.collect_ipv4()
        self.collect_ipv6()
    def collect_ipv4(self):
        metrics = {}

        for filepath in self.PROC:
            if not os.access(filepath, os.R_OK):
                self.log.error('Permission to access %s denied', filepath)
            file = self.open_file(filepath)
            if not file:
                continue

            header = ''
            data = ''

            # Seek the file for the lines which start with Ip
            file = open(filepath)
            if not file:
                self.log.error('Failed to open %s', filepath)
                continue
            while True:
                line = file.readline()

@@ -118,3 +139,25 @@ def collect(self):
                self.publish_gauge(metric_name, value, 0)
            else:
                self.publish_counter(metric_name, value, 0)
    def collect_ipv6(self):
        metrics = {}
        for filepath in self.PROC6:
            fp = self.open_file(filepath)
            for line in fp.readlines():
                key, value = line.split()
                metrics[key] = long(value)
            fp.close()
        for metric, value in metrics.iteritems():
            if (len(self.config['allowed_names']) > 0
                    and metric not in self.config['allowed_names']):
                continue
            # Publish the metric
            if metric in self.GAUGES:
                self.publish_gauge(metric, value, 0)
            else:
                self.publish_counter(metric, value, 0)
