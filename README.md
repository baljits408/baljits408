global NR_account_idle_time;
global CPUT_account_idle_time;

global NR_account_process_tick;

global NR_account_idle_ticks;
global TICK_account_idle_ticks;

global NR_do_stolen_accounting;

global NR_xen_timer_interrupt;

probe kernel.function("account_idle_time")
{
  ++NR_account_idle_time;
  CPUT_account_idle_time += $cputime;
}

probe kernel.function("account_process_tick")
{
  ++NR_account_process_tick;
}

probe kernel.function("account_idle_ticks")
{
  ++NR_account_idle_ticks;
  TICK_account_idle_ticks += $ticks;
}

probe kernel.function("do_stolen_accounting")
{
  ++NR_do_stolen_accounting;
}

probe kernel.function("xen_timer_interrupt")
{
  ++NR_xen_timer_interrupt;
}

probe timer.sec(10)
{
  NR_account_idle_time /= 10;
  CPUT_account_idle_time /= 10;
  NR_account_process_tick /= 10;
  NR_account_idle_ticks /= 10;
  TICK_account_idle_ticks /= 10;
  NR_do_stolen_accounting /= 10;
  NR_xen_timer_interrupt /= 10;
  
  printf("account_idle_time():         calls = %d, cputime sum = %d\n", NR_account_idle_time, CPUT_account_idle_time);
  printf("  account_process_tick():    calls = %d\n", NR_account_process_tick);
  printf("  account_idle_ticks():      calls = %d, ticks sum = %d\n", NR_account_idle_ticks, TICK_account_idle_ticks);
  printf("    do_stolen_accounting():  calls = %d\n", NR_do_stolen_accounting);
  printf("      xen_timer_interrupt(): calls = %d\n", NR_xen_timer_interrupt);
  exit();
}
