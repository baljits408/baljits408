global NR_account_steal_time;
global CPUT_account_steal_time;

global NR_timer_interrupt;

probe kernel.function("account_steal_time")
{
  ++NR_account_steal_time;
  CPUT_account_steal_time[$p] += $steal;
}

probe kernel.function("timer_interrupt")
{
  ++NR_timer_interrupt;
}

probe timer.sec(10)
{
  printf("account_steal_time(): calls = %d\n", NR_account_steal_time / 10);
  foreach([p] in CPUT_account_steal_time) {
    printf("CPUT_account_steal_time[%x] = %d\n", p, CPUT_account_steal_time[p] / 10);
  }
  printf("  timer_interrupt():  calls = %d\n", NR_timer_interrupt / 10);
  exit();
}
