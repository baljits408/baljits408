    return config

    def collect(self):
        if ((not os.access(self.config['bin'], os.X_OK) or
             (str_to_bool(self.config['use_sudo']) and
              not os.access(self.config['sudo_cmd'], os.X_OK)))):
        # Cannot access binary and not using sudo
        if (not os.access(self.config['bin'], os.X_OK) and
            not str_to_bool(self.config['use_sudo'])):
            return
        # Using sudo but cannot access it
        if (str_to_bool(self.config['use_sudo']) and
            not os.access(self.config['sudo_cmd'], os.X_OK)):
            return

        command = [self.config['bin'],
