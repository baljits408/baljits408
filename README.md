 def test_import(self):
        self.assertTrue(DiskSpaceCollector)

    def run_collection(self, statvfs_mock, os_major, os_minor):
        os_stat_mock = patch('os.stat')
        os_major_mock = patch('os.major', Mock(return_value=os_major))
        os_minor_mock = patch('os.minor', Mock(return_value=os_minor))
        os_path_isdir_mock = patch('os.path.isdir', Mock(return_value=False))
        open_mock = patch('__builtin__.open',
                          Mock(return_value=self.getFixture('proc_mounts')))
        os_statvfs_mock = patch('os.statvfs', Mock(return_value=statvfs_mock))
        os_stat_mock.start()
        os_major_mock.start()
        os_minor_mock.start()
        os_path_isdir_mock.start()
        open_mock.start()
        os_statvfs_mock.start()
        self.collector.collect()
        os_stat_mock.stop()
        os_major_mock.stop()
        os_minor_mock.stop()
        os_path_isdir_mock.stop()
        open_mock.stop()
        os_statvfs_mock.stop()
    @run_only_if_major_is_available
    @patch('os.access', Mock(return_value=True))
    def test_get_file_systems(self):
@@ -108,27 +131,7 @@ def test_should_work_with_real_data(self, publish_mock):
        statvfs_mock.f_flag = 4096
        statvfs_mock.f_namemax = 255

        os_stat_mock = patch('os.stat')
        os_major_mock = patch('os.major', Mock(return_value=9))
        os_minor_mock = patch('os.minor', Mock(return_value=0))
        os_path_isdir_mock = patch('os.path.isdir', Mock(return_value=False))
        open_mock = patch('__builtin__.open',
                          Mock(return_value=self.getFixture('proc_mounts')))
        os_statvfs_mock = patch('os.statvfs', Mock(return_value=statvfs_mock))
        os_stat_mock.start()
        os_major_mock.start()
        os_minor_mock.start()
        os_path_isdir_mock.start()
        open_mock.start()
        os_statvfs_mock.start()
        self.collector.collect()
        os_stat_mock.stop()
        os_major_mock.stop()
        os_minor_mock.stop()
        os_path_isdir_mock.stop()
        open_mock.stop()
        os_statvfs_mock.stop()
        self.run_collection(statvfs_mock, 9, 0)

        metrics = {
            'root.gigabyte_used': (284.525, 2),
@@ -152,7 +155,8 @@ def test_should_work_with_tmpfs(self, publish_mock):
            'interval': 10,
            'byte_unit': ['gigabyte'],
            'exclude_filters': [],
            'filesystems': 'tmpfs'
            'filesystems': 'tmpfs',
            'exclude_filters': '^/sys'
        })

        self.collector = DiskSpaceCollector(config, None)
@@ -168,27 +172,7 @@ def test_should_work_with_tmpfs(self, publish_mock):
        statvfs_mock.f_flag = 4096
        statvfs_mock.f_namemax = 255

        os_stat_mock = patch('os.stat')
        os_major_mock = patch('os.major', Mock(return_value=4))
        os_minor_mock = patch('os.minor', Mock(return_value=0))
        os_path_isdir_mock = patch('os.path.isdir', Mock(return_value=False))
        open_mock = patch('__builtin__.open',
                          Mock(return_value=self.getFixture('proc_mounts')))
        os_statvfs_mock = patch('os.statvfs', Mock(return_value=statvfs_mock))
        os_stat_mock.start()
        os_major_mock.start()
        os_minor_mock.start()
        os_path_isdir_mock.start()
        open_mock.start()
        os_statvfs_mock.start()
        self.collector.collect()
        os_stat_mock.stop()
        os_major_mock.stop()
        os_minor_mock.stop()
        os_path_isdir_mock.stop()
        open_mock.stop()
        os_statvfs_mock.stop()
        self.run_collection(statvfs_mock, 4, 0)

        metrics = {
            'tmp.gigabyte_used': (284.525, 2),
@@ -204,6 +188,47 @@ def test_should_work_with_tmpfs(self, publish_mock):
                           defaultpath=self.collector.config['path'])
        self.assertPublishedMany(publish_mock, metrics)

    @run_only_if_major_is_available
    @patch('os.access', Mock(return_value=True))
    @patch.object(Collector, 'publish')
    def test_should_work_in_system_directories(self, publish_mock):
        config = get_collector_config('DiskSpaceCollector', {
            'interval': 10,
            'byte_unit': ['gigabyte'],
            'exclude_filters': [],
            'filesystems': 'tmpfs',
            'exclude_filters': '^/tmp'
        })
        self.collector = DiskSpaceCollector(config, None)
        statvfs_mock = Mock()
        statvfs_mock.f_bsize = 4096
        statvfs_mock.f_frsize = 4096
        statvfs_mock.f_blocks = 360540255
        statvfs_mock.f_bfree = 285953527
        statvfs_mock.f_bavail = 267639130
        statvfs_mock.f_files = 91578368
        statvfs_mock.f_ffree = 91229495
        statvfs_mock.f_favail = 91229495
        statvfs_mock.f_flag = 4096
        statvfs_mock.f_namemax = 255
        self.run_collection(statvfs_mock, 4, 0)
        metrics = {
            '_sys_fs_cgroup.gigabyte_used': (284.525, 2),
            '_sys_fs_cgroup.gigabyte_free': (1090.826, 2),
            '_sys_fs_cgroup.gigabyte_avail': (1020.962, 2),
            '_sys_fs_cgroup.inodes_used': 348873,
            '_sys_fs_cgroup.inodes_free': 91229495,
            '_sys_fs_cgroup.inodes_avail': 91229495,
        }
        self.setDocExample(collector=self.collector.__class__.__name__,
                           metrics=metrics,
                           defaultpath=self.collector.config['path'])
        self.assertPublishedMany(publish_mock, metrics)
##########################################################################
if __name__ == "__main__":
    unittest.main()
0 commit comments
Comments
0
 (0)
Comment
You're not receiving notifications from this thread.
