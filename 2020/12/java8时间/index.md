# Java8时间


## LocalDate 和 LocalTime

首先不附带任何与时区相关的信息。

```java
LocalDate.of(2020,1,1);
LocalDate.parse("2020-01-01")
LocalDate.now()

```

## Instant

以 Unix 元年(1970.1.1)开始所经历的秒数。设计初衷是为了便于机器使用。

```java
Instant.ofEpochSecond(3,1)//3是秒,1是納秒
Instant.now()

```

## Duration 或 Period

Duration 以秒和納秒为单位建模
Period 以以年月日为单位建模

## 时区

ZoneId 取代 TimeZone

Instant 和 LocalDateTime 可以通过 ZoneId 互相转换

ZoneOffset 是 ZoneId 一个子类，表示时区固定偏差。这种方式未考虑夏令时的影响，不推荐使用。

