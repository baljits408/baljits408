import diamond.collector
import json
import urllib2
from urlparse import urlparse


class MesosCollector(diamond.collector.Collector):
@@ -27,7 +28,8 @@ def get_default_config_help(self):
        config_help = super(MesosCollector,
                            self).get_default_config_help()
        config_help.update({
            'host': 'Hostname',
            'host': 'Hostname, using http scheme by default. For https pass '
                    'e.g. "https://localhost"',
            'port': 'Port (default is 5050; please set to 5051 for mesos-slave)'
        })
        return config_help
@@ -51,12 +53,17 @@ def collect(self):
            key = self.clean_up(k)
            self.publish(key, v)

    def _get_url(self):
        parsed = urlparse(self.config['host'])
        scheme = parsed.scheme or 'http'
        host = parsed.hostname or self.config['host']
        return "%s://%s:%s/%s" % (
            scheme, host, self.config['port'], self.METRICS_PATH)
    def get_metrics(self):
        try:
            url = "http://%s:%s/%s" % (self.config['host'],
                                       self.config['port'],
                                       self.METRICS_PATH)
        url = self._get_url()

        try:
            return json.load(urllib2.urlopen(url))
        except (urllib2.HTTPError, ValueError), err:
            self.log.error('Unable to read JSON response: %s' % err)
