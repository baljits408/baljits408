        return config

    def collect(self):
        for pattern in self.config['metrics']:
        metrics = self.config['metrics']
        if not isinstance(metrics, list):
            metrics = [str(metrics)]
        for pattern in metrics:
            for filename in glob.glob(pattern):
                self.collect_from(filename)
