# Day 2
## New Collector Class
This further expands on the Steam API functionality.
Contains a `groupingBy` method similar to a GROUP BY clause in a SQL query.  
```
public Map<String, Long> getDVDsInEachGenre() {
	Map<String, Long> collect = DVDDao.getAllDVDs().collect(Collectors.groupingBy(DVD::getGenre, Collectors.counting()));
	return collect;
}
```
Contains neat summarizing methods (though, don't expect it to print it out pretty).
```
public DoubleSummaryStatistics getPricingStats() {
	return DVDDao.getAllDVDs().collect(Collectors.summarizingDouble(DVD::getPrice));
}
```

##Revamped Date/Time API
Thread safe classes, domain-driven design.  Manipulations can be chained (Fluent API).
LocalDate (date only), LocalTime (time only) - note these are immutable and when manipulations are made, new instances are created.  Example:
```
LocalDate ld = LocalDate.of(2015, Month.FEBRUARY, 29);
ld.plusDays(1);
System.out.println(ld) // will print 2015-02-29 not 2015-03-01
```

TemporalAdjuster allows for more complex date manipulations.

```
public static Appointment createFirstDayOfNextMonthAppointment(String description) {
	return new Appointment(description, LocalDateTime.of(LocalDate.now(), LocalTime.NOON).with(TemporalAdjusters.firstDayOfNextMonth()));
	}
```


[Go to Day 3 >>](/day3.md)