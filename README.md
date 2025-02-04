@@ -0,0 +1,57 @@
<!--This file was generated from the python source
Please edit the source to make changes
-->
EtcdCollector
=====
Collects metrics from an Etcd instance.
#### Example Configuration
```
    host = localhost
    port = 2379
```
#### Options
Setting | Default | Description | Type
--------|---------|-------------|-----
byte_unit | byte | Default numeric output(s) | str
ca_file |  | Only applies when use_tls=true. Path to CA certificate file to use for server identity verification | str
enabled | False | Enable collecting these metrics | bool
host | localhost | Hostname | str
measure_collector_time | False | Collect the collector run time in ms | bool
metrics_blacklist | None | Regex to match metrics to block. Mutually exclusive with metrics_whitelist | NoneType
metrics_whitelist | None | Regex to match metrics to transmit. Mutually exclusive with metrics_blacklist | NoneType
port | 2379 | Port (default is 2379) | int
timeout | 5 | Timeout per HTTP(s) call | int
use_tls | False | Use TLS/SSL or just unsecure (default is unsecure) | bool
#### Example Output
```
servers.hostname.etcd.self.is_leader 1
servers.hostname.etcd.self.recvAppendRequestCnt 5870
servers.hostname.etcd.self.sendAppendRequestCnt 2097127
servers.hostname.etcd.self.sendBandwidthRate 901.090846975
servers.hostname.etcd.self.sendPkgRate 11.7635880806
servers.hostname.etcd.store.compareAndDeleteFail 0
servers.hostname.etcd.store.compareAndDeleteSuccess 2047
servers.hostname.etcd.store.compareAndSwapFail 355
servers.hostname.etcd.store.compareAndSwapSuccess 9156
servers.hostname.etcd.store.createFail 2508
servers.hostname.etcd.store.createSuccess 6468
servers.hostname.etcd.store.deleteFail 2138
servers.hostname.etcd.store.deleteSuccess 2468
servers.hostname.etcd.store.expireCount 0
servers.hostname.etcd.store.getsFail 922428
servers.hostname.etcd.store.getsSuccess 1685131
servers.hostname.etcd.store.setsFail 123
servers.hostname.etcd.store.setsSuccess 733
servers.hostname.etcd.store.updateFail 0
servers.hostname.etcd.store.updateSuccess 4576
servers.hostname.etcd.store.watchers 51
```
‎src/collectors/etcdstat/etcdstat.py
+109
Original file line number	Diff line number	Diff line change
@@ -0,0 +1,109 @@
# coding=utf-8
"""
Collects metrics from an Etcd instance.
#### Example Configuration
```
    host = localhost
    port = 2379
```
"""
import diamond.collector
import json
import urllib2
METRICS_KEYS = ['sendPkgRate',
                'recvPkgRate',
                'sendAppendRequestCnt',
                'recvAppendRequestCnt',
                'sendBandwidthRate',
                'recvBandwidthRate']
class EtcdCollector(diamond.collector.Collector):
    def get_default_config_help(self):
        config_help = super(EtcdCollector,
                            self).get_default_config_help()
        config_help.update({
            'host': 'Hostname',
            'port': 'Port (default is 2379)',
            'timeout': 'Timeout per HTTP(s) call',
            'use_tls': 'Use TLS/SSL or just unsecure (default is unsecure)',
            'ca_file': 'Only applies when use_tls=true. Path to CA certificate'
                       ' file to use for server identity verification',
        })
        return config_help
    def get_default_config(self):
        config = super(EtcdCollector, self).get_default_config()
        config.update({
            'host': 'localhost',
            'port': 2379,
            'path': 'etcd',
            'timeout': 5,
            'use_tls': False,
            'ca_file': '',
        })
        return config
    def __init__(self, *args, **kwargs):
        super(EtcdCollector, self).__init__(*args, **kwargs)
    def collect(self):
        self.collect_self_metrics()
        self.collect_store_metrics()
    def collect_self_metrics(self):
        metrics = self.get_self_metrics()
        if 'state' in metrics and metrics['state'] == "StateLeader":
            self.publish("self.is_leader", 1)
        else:
            self.publish("self.is_leader", 0)
        for k in METRICS_KEYS:
            if k not in metrics:
                continue
            v = metrics[k]
            key = self.clean_up(k)
            self.publish("self.%s" % key, v)
    def collect_store_metrics(self):
        metrics = self.get_store_metrics()
        for k, v in metrics.iteritems():
            key = self.clean_up(k)
            self.publish("store.%s" % key, v)
    def get_self_metrics(self):
        return self.get_metrics("self")
    def get_store_metrics(self):
        return self.get_metrics("store")
    def get_metrics(self, category):
        try:
            opts = {
                'timeout': int(self.config['timeout']),
            }
            if self.config['use_tls']:
                protocol = "https"
                opts['cafile'] = self.config['ca_file']
            else:
                protocol = "http"
            url = "%s://%s:%s/v2/stats/%s" % (protocol, self.config['host'],
                                              self.config['port'], category)
            return json.load(urllib2.urlopen(url, **opts))
        except (urllib2.HTTPError, ValueError), err:
            self.log.error('Unable to read JSON response: %s' % err)
            return {}
    def clean_up(self, text):
        return text.replace('/', '.')
‎src/collectors/etcdstat/test/fixtures/follower-self-metrics.json
+15
Original file line number	Diff line number	Diff line change
@@ -0,0 +1,15 @@
{
  "name": "my-other-etcd-server",
  "id": "65c9529291ea35ac",
  "state": "StateFollower",
  "startTime": "2016-12-16T10:58:39.85475587+01:00",
  "leaderInfo": {
    "leader": "640331e25aeba433",
    "uptime": "2h47m56.735839145s",
    "startTime": "2016-12-16T10:58:40.201314756+01:00"
  },
  "recvAppendRequestCnt": 79367,
  "recvPkgRate": 6.557436727874493,
  "recvBandwidthRate": 527.021189819273,
  "sendAppendRequestCnt": 0
}
‎src/collectors/etcdstat/test/fixtures/leader-self-metrics.json
+15
Original file line number	Diff line number	Diff line change
@@ -0,0 +1,15 @@
{
  "name": "my-etcd-server",
  "id": "fc89b7d53ea5a0e2",
  "state": "StateLeader",
  "startTime": "2016-11-22T13:35:23.343775657+01:00",
  "leaderInfo": {
    "leader": "fc89b7d53ea5a0e2",
    "uptime": "19h6m9.317124825s",
    "startTime": "2016-11-22T13:49:49.748041155+01:00"
  },
  "recvAppendRequestCnt": 5870,
  "sendAppendRequestCnt": 2097127,
  "sendPkgRate": 11.763588080610418,
  "sendBandwidthRate": 901.0908469747579
}
‎src/collectors/etcdstat/test/fixtures/store-metrics.json
+18
Original file line number	Diff line number	Diff line change
@@ -0,0 +1,18 @@
{
  "getsSuccess": 1685131,
  "getsFail": 922428,
  "setsSuccess": 733,
  "setsFail": 123,
  "deleteSuccess": 2468,
  "deleteFail": 2138,
  "updateSuccess": 4576,
  "updateFail": 0,
  "createSuccess": 6468,
  "createFail": 2508,
  "compareAndSwapSuccess": 9156,
  "compareAndSwapFail": 355,
  "compareAndDeleteSuccess": 2047,
  "compareAndDeleteFail": 0,
  "expireCount": 0,
  "watchers": 51
}
‎src/collectors/etcdstat/test/fixtures/store-metrics2.json
+18
Original file line number	Diff line number	Diff line change
@@ -0,0 +1,18 @@
{
  "getsSuccess": 396632,
  "getsFail": 255837,
  "setsSuccess": 98571,
  "setsFail": 12,
  "deleteSuccess": 6,
  "deleteFail": 6,
  "updateSuccess": 2,
  "updateFail": 0,
  "createSuccess": 1294,
  "createFail": 0,
  "compareAndSwapSuccess": 4839,
  "compareAndSwapFail": 136,
  "compareAndDeleteSuccess": 1239,
  "compareAndDeleteFail": 0,
  "expireCount": 0,
  "watchers": 0
}
‎src/collectors/etcdstat/test/test_etcdstat.py
+132
Original file line number	Diff line number	Diff line change
@@ -0,0 +1,132 @@
#!/usr/bin/python
# coding=utf-8
##########################################################################
from diamond.collector import Collector
from test import CollectorTestCase
from test import get_collector_config
from test import unittest
from mock import patch
from mock import Mock
from etcdstat import EtcdCollector
try:
    import simplejson as json
except ImportError:
    import json
##########################################################################
class TestEtcdCollector(CollectorTestCase):
    def setUp(self):
        config = get_collector_config('EtcdCollector', {
            'interval': 10
        })
        self.collector = EtcdCollector(config, None)
    def test_import(self):
        self.assertTrue(EtcdCollector)
    @patch.object(Collector, 'publish')
    def test_should_work_with_real_follower_data(self, publish_mock):
        patch1_collector = patch.object(
            EtcdCollector,
            'get_self_metrics',
            Mock(return_value=json.loads(
                 self.getFixture('follower-self-metrics.json').getvalue())))
        patch2_collector = patch.object(
            EtcdCollector,
            'get_store_metrics',
            Mock(return_value=json.loads(
                 self.getFixture('store-metrics2.json').getvalue())))
        patch1_collector.start()
        patch2_collector.start()
        self.collector.collect()
        patch2_collector.stop()
        patch1_collector.stop()
        metrics = {
            'self.is_leader': 0,
            'self.sendAppendRequestCnt': 0,
            'self.recvAppendRequestCnt': 79367,
            'self.recvPkgRate': 6.557436727874493,
            'self.recvBandwidthRate': 527.021189819273,
            'store.compareAndDeleteFail': 0,
            'store.watchers': 0,
            'store.setsFail': 12,
            'store.createSuccess': 1294,
            'store.compareAndSwapFail': 136,
            'store.compareAndSwapSuccess': 4839,
            'store.deleteSuccess': 6,
            'store.updateSuccess': 2,
            'store.createFail': 0,
            'store.getsSuccess': 396632,
            'store.expireCount': 0,
            'store.deleteFail': 6,
            'store.updateFail': 0,
            'store.getsFail': 255837,
            'store.compareAndDeleteSuccess': 1239,
            'store.setsSuccess': 98571,
        }
        self.assertPublishedMany(publish_mock, metrics)
    @patch.object(Collector, 'publish')
    def test_should_work_with_real_leader_data(self, publish_mock):
        patch1_collector = patch.object(
            EtcdCollector,
            'get_self_metrics',
            Mock(return_value=json.loads(
                 self.getFixture('leader-self-metrics.json').getvalue())))
        patch2_collector = patch.object(
            EtcdCollector,
            'get_store_metrics',
            Mock(return_value=json.loads(
                 self.getFixture('store-metrics.json').getvalue())))
        patch1_collector.start()
        patch2_collector.start()
        self.collector.collect()
        patch2_collector.stop()
        patch1_collector.stop()
        metrics = {
            'self.is_leader': 1,
            'self.sendAppendRequestCnt': 2097127,
            'self.recvAppendRequestCnt': 5870,
            'self.sendPkgRate': 11.763588080610418,
            'self.sendBandwidthRate': 901.0908469747579,
            'store.compareAndDeleteFail': 0,
            'store.watchers': 51,
            'store.setsFail': 123,
            'store.createSuccess': 6468,
            'store.compareAndSwapFail': 355,
            'store.compareAndSwapSuccess': 9156,
            'store.deleteSuccess': 2468,
            'store.updateSuccess': 4576,
            'store.createFail': 2508,
            'store.getsSuccess': 1685131,
            'store.expireCount': 0,
            'store.deleteFail': 2138,
            'store.updateFail': 0,
            'store.getsFail': 922428,
            'store.compareAndDeleteSuccess': 2047,
            'store.setsSuccess': 733,
        }
        self.assertPublishedMany(publish_mock, metrics)
        self.setDocExample(collector=self.collector.__class__.__name__,
                           metrics=metrics,
                           defaultpath=self.collector.config['path'])
##########################################################################
if __name__ == "__main__":
    unittest.main()
