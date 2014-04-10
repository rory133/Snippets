### Main points
* Stateless beans, singletons, and mdbs can be registered by the timer service, but stateful beans can’t and
shouldn’t use the scheduling api.
* By default, each @Schedule annotation corresponds to a single persistent timer, but you can
also define nonpersistent timers.

#### TimerService API
* createTimer. Creates a timer based on dates, intervals, or durations. These methods do not use calendar-based expressions.
* createSingleActionTimer. Creates a single-action timer that expires at a given point in time or after a specified
duration. The container removes the timer after the timeout callback method has been successfully invoked.
*createIntervalTimer. Creates an interval timer whose first expiration occurs at a given point in time and whose subsequent expirations occur after specified intervals.
*createCalendarTimer. Creates a timer using a calendar-based expression with the ScheduleExpression helper class.
*getAllTimers. Returns the list of available timers (interface javax.ejb.Timer).

#### Examples.
1. Listing shows the StatisticsEJB bean that defines several methods. statisticsItemsSold() creates a timer
that will call the method every first day of the month at 5:30 a.m. The generateReport() method creates two timers
(using @Schedules): one every day at 2 a.m. and another one every Wednesday at 2 p.m. refreshCache() creates a
nonpersistent timer that will refresh the cache every ten minutes.
```
@Stateless
public class StatisticsEJB {
  @Schedule(dayOfMonth = "1", hour = "5", minute = "30")
  public void statisticsItemsSold() {
  // ...
  }

  @Schedules({
      @Schedule(hour = "2"),
      @Schedule(hour = "14", dayOfWeek = "Wed")
  })
  public void generateReport() {
    // ...
  }
  
  @Schedule(minute = "*/10", hour = "*", persistent = false)
  public void refreshCache() {
    // ...
  }
}
```
2. When CustomerEJB creates a new customer in the system (createCustomer() method), it creates
a calendar timer based on the date of birth of the customer. Thus, every year the container will trigger a bean to create
and send a birthday e-mail to the customer. To do that, the stateless bean first needs to inject a reference of the timer
service (using @Resource). The createCustomer() persists the customer in the database and uses the day and the
month of her birth to create a ScheduleExpression. A calendar timer is created and given this ScheduleExpression
and the customer object using a TimerConfig. new TimerConfig(customer, true) configures a persistent timer
(as indicated by the true parameter) that passes the customer object. Once you create the timer, the container will invoke the @Timeout method (sendBirthdayEmail()) every year,
which passes the Timer object. Because the timer had been serialized with the customer object, the method can
access it by calling the getInfo()method. Note that a bean can have at most one @Timeout method for handling
programmatic timers.
```
@Stateless
public class CustomerEJB {
  @Resource
  TimerService timerService;
  
  @PersistenceContext(unitName = "chapter08PU")
  private EntityManager em;
  
  public void createCustomer(Customer customer) {
    em.persist(customer);
    ScheduleExpression birthDay = new ScheduleExpression().dayOfMonth(customer.getBirthDay()).month(customer.getBirthMonth());
    timerService.createCalendarTimer(birthDay, new TimerConfig(customer, true));
  }

  @Timeout
  public void sendBirthdayEmail(Timer timer) {
  Customer customer = (Customer) timer.getInfo();
  // ...
  }
}
```
#### Expression examples

| Every weekday at 6:55 Paris time | minute="55", hour="6", dayOfWeek="Mon-Fri", timezone="Europe/Paris" |
| Every minute of every hour of every day | minute="*", hour="*" |
| Every Monday and Friday at 30 seconds past noon | second="30", hour="12", dayOfWeek="Mon, Fri" |
| Every five minutes within the hour | minute="*/5", hour="*" | 
| The last Monday of December at 3 p.m. | hour="15", dayOfMonth="Last Mon", month="Dec" |
| Three days before the last day of each month at 1 p.m. | hour="13", dayOfMonth="-3" | 
| Every other hour within the day starting at noon on the second Tuesday of every month | hour="12/2", dayOfMonth="2nd Tue"|
| Every 10 seconds within the minute, starting at second 30 | second = "30/10" |
