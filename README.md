next_window = math.floor(time.time() / interval) * interval

    if str_to_bool(self.config['stagger_collection']):
    if str_to_bool(collector.config['stagger_collection']):
        stagger_offset = random.uniform(0, interval - 1)
