    setproctitle = None

for path in [
    os.path.join('opt', 'diamond', 'lib'),
    os.path.join('/usr', 'share', 'pyshared'),
    os.path.join('/opt', 'diamond', 'lib'),
    os.path.abspath(os.path.join(os.path.dirname(__file__), '..', 'src'))
]:
    if os.path.exists(os.path.join(path, 'diamond', '__init__.py')):
