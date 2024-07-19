多字段排序：

``` java
private static List<Item> sortV2(List<Item> items) { return items.stream() .sorted(Comparator .comparing(Item::getA, Comparator.reverseOrder()) .thenComparing(Item::getB, Comparator.reverseOrder()) .thenComparing(Item::getC, Comparator.reverseOrder())) .collect(Collectors.toList()); }
```
