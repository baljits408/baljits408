except ImportError:
    setproctitle = None

from diamond.collector import str_to_bool
from diamond.utils.signals import signal_to_exception
from diamond.utils.signals import SIGALRMException
from diamond.utils.signals import SIGHUPException
@@ -35,18 +36,24 @@ def collector_process(collector, metric_queue, log):

    interval = float(collector.config['interval'])

    log.debug('Starting')
    log.debug('Interval: %s seconds', interval)
    # Validate the interval
    if interval <= 0:
        log.critical('interval of %s is not valid!', interval)
        log.critical('Interval of %s is not valid!', interval)
        sys.exit(1)

    log.debug('Starting %s', collector.name)
    log.debug('Interval: %s seconds', interval)
    # Initially set stagger_offset to 0. If stagger_collection is enabled in
    # the config, this value will soon be overridden.
    stagger_offset = 0
    # Start the next execution at the next window plus some stagger delay to
    # avoid having all collectors running at the same time
    next_window = math.floor(time.time() / interval) * interval
    stagger_offset = random.uniform(0, interval - 1)
    if str_to_bool(self.config['stagger_collection']):
        stagger_offset = random.uniform(0, interval - 1)

    # Allocate time till the end of the window for the collector to run. With a
    # minimum of 1 second
@@ -95,7 +102,7 @@ def collector_process(collector, metric_queue, log):
            # us and end up with half a loaded config
            signal.alarm(0)

            log.info('Reloading config reload due to HUP')
            log.info('Reloading config due to SIGHUP')
            collector.load_config()
            log.info('Config reloaded')
