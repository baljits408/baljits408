      else:
                try:
                    int(value)
                except ValueError:
                except (ValueError, TypeError):
                    value = None
                finally:
                    yield ("%s.%s" % (prefix, key), value)
