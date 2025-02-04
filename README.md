def test_should_open_proc_net_snmp(self, publish_mock, open_mock):
        IPCollector.PROC = ['/proc/net/snmp']
        open_mock.return_value = StringIO('')
        self.collector.collect()
        self.collector.collect_ipv4()
        open_mock.assert_called_once_with('/proc/net/snmp')

        open_mock.reset_mock()
        IPCollector.PROC6 = ['/proc/net/snmp6']
        open_mock.return_value = StringIO('')
        self.collector.collect_ipv6()
        open_mock.assert_called_once_with('/proc/net/snmp6')
    @patch('os.access', Mock(return_value=True))
    @patch('__builtin__.open')
    @patch('diamond.collector.Collector.publish')
    def test_should_work_with_synthetic_data(self, publish_mock, open_mock):
        IPCollector.PROC = ['/proc/net/snmp']
        self.setUp(['A', 'C'])
        open_mock.return_value = StringIO('''
        IPCollector.PROC6 = ['/proc/net/snmp6']
        self.setUp(['A', 'C', 'A6', 'C6'])
        open_mock.side_effect = [StringIO('''
Ip: A B C
Ip: 0 0 0
'''.strip())
'''.strip()), StringIO('''
A6    0
B6    0
C6    0
'''.strip())]

        self.collector.collect()

        open_mock.return_value = StringIO('''
        open_mock.side_effect = [StringIO('''
Ip: A B C
Ip: 0 1 2
'''.strip())
'''.strip()), StringIO('''
A6    0
B6    1
C6    2
'''.strip())]

        publish_mock.call_args_list = []

        self.collector.collect()

        self.assertEqual(len(publish_mock.call_args_list), 2)
        self.assertEqual(len(publish_mock.call_args_list), 4)

        metrics = {
            'A': 0,
            'C': 2,
            'A6': 0,
            'C6': 2
        }

        self.assertPublishedMany(publish_mock, metrics)

    @patch('diamond.collector.Collector.publish')
    def test_should_work_with_real_data(self, publish_mock):
        self.setUp(['InDiscards', 'InReceives', 'OutDiscards', 'OutRequests'])
        self.setUp(['InDiscards', 'InReceives', 'OutDiscards', 'OutRequests',
                    'Ip6InDiscards', 'Ip6InReceives', 'Ip6OutDiscards',
                    'Ip6OutRequests'])

        IPCollector.PROC = [self.getFixturePath('proc_net_snmp_1')]
        IPCollector.PROC6 = [self.getFixturePath('proc_net_snmp6_1')]
        self.collector.collect()
        self.assertPublishedMany(publish_mock, {})

        IPCollector.PROC = [self.getFixturePath('proc_net_snmp_2')]
        IPCollector.PROC6 = [self.getFixturePath('proc_net_snmp6_2')]
        self.collector.collect()

        metrics = {
            'InDiscards': 0,
            'InReceives': 2,
            'OutDiscards': 0,
            'OutRequests': 1,
            'Ip6InReceives': 2,
            'Ip6InDiscards': 0,
            'Ip6OutDiscards': 4,
            'Ip6OutRequests': 1
        }

        self.assertPublishedMany(publish_mock, metrics)
@@ -121,12 +146,16 @@ def test_should_work_with_all_data(self, publish_mock):
        IPCollector.PROC = [
            self.getFixturePath('proc_net_snmp_1'),
        ]
        IPCollector.PROC6 = [self.getFixturePath('proc_net_snmp6_1')]
        self.collector.collect()
        self.assertPublishedMany(publish_mock, {})

        IPCollector.PROC = [
            self.getFixturePath('proc_net_snmp_2'),
        ]
        IPCollector.PROC6 = [self.getFixturePath('proc_net_snmp6_1')]
        self.collector.collect()

        self.setDocExample(collector=self.collector.__class__.__name__,
