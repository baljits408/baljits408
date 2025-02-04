import re
import os
import sys
from ceph import CephCollector
sys.path.insert(0, os.path.join(os.path.dirname(os.path.dirname(__file__)),
                                'ceph'))
from ceph import CephCollector

patternchk = re.compile(r'\bclient io .*')
numberchk = re.compile(r'\d+')
unitchk = re.compile(r'[a-zA-Z]{1,2}')

# This is external to the CephCollector so it can be tested
# separately.
def to_bytes(value, unit):
    fval = float(value)
    unit = str(unit.lower()).strip()
    if unit == "b":
        return fval
    unit_list = {'kb': 1, 'mb': 2, 'gb': 3, 'tb': 4, 'pb': 5, 'eb': 6}
    for i in range(unit_list[unit]):
        fval = fval * 1000
    return fval
def process_ceph_status(output):
    res = patternchk.search(output)
    if not res:
@@ -26,21 +40,28 @@ def process_ceph_status(output):
    if not ceph_stats:
        return {}
    ret = {}
    rd = wr = iops = None
    rd = wr = iops = runit = wunit = None
    rd = numberchk.search(ceph_stats)
    if rd is not None:
        ret['rd'] = rd.group()
        runit = unitchk.search(ceph_stats, rd.end())
        if runit is None:
            self.log.exception('Could not get read units')
            return {}
        ret['rd'] = repr(to_bytes(rd.group(), runit.group()))
        wr = numberchk.search(ceph_stats, rd.end())
        if wr is not None:
            ret['wr'] = wr.group()
            wunit = unitchk.search(ceph_stats, wr.end())
            if runit is None:
                self.log.exception('Could not get read units')
                return {}
            ret['wr'] = repr(to_bytes(wr.group(), wunit.group()))
            iops = numberchk.search(ceph_stats, wr.end())
            if iops is not None:
                ret['iops'] = iops.group()
    return ret


class CephStatsCollector(CephCollector):
    def _get_stats(self):
        """
        Get ceph stats
@@ -52,7 +73,6 @@ def _get_stats(self):
                'Could not get stats: %s' % err)
            self.log.exception('Could not get stats')
            return {}
        return process_ceph_status(output)

    def collect(self):
@@ -61,5 +81,4 @@ def collect(self):
        """
        stats = self._get_stats()
        self._publish_stats('cephstats', stats)
        return
‎src/collectors/cephstats/test/test_ceph.py
+1
-1
Original file line number	Diff line number	Diff line change
@@ -22,7 +22,7 @@ def test_sample_data(self):
        Get ceph information from sample data
        """
        f = open('sample.txt')
        ret = {'rd': '8643', 'wr': '4821', 'iops': '481'}
        ret = {'rd': '8643000.0', 'wr': '4821000.0', 'iops': '481'}
        self.assertEqual(process_ceph_status(f.read()), ret)
        f.close()

