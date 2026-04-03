# Java Stream API Interview Coding Questions

This guide contains 30 frequently asked Java Stream API coding questions with concise solution snippets.

## 1) Find all even numbers from a list
```java
List<Integer> out = nums.stream().filter(n -> n % 2 == 0).toList();
```

## 2) Remove duplicates and sort
```java
List<Integer> out = nums.stream().distinct().sorted().toList();
```

## 3) Find second highest number
```java
Integer out = nums.stream()
    .distinct()
    .sorted(Comparator.reverseOrder())
    .skip(1)
    .findFirst()
    .orElse(null);
```

## 4) Find top 3 maximum numbers
```java
List<Integer> out = nums.stream()
    .sorted(Comparator.reverseOrder())
    .limit(3)
    .toList();
```

## 5) Count occurrences of each element
```java
Map<String, Long> out = list.stream()
    .collect(Collectors.groupingBy(Function.identity(), Collectors.counting()));
```

## 6) Find duplicate elements
```java
Set<Integer> seen = new HashSet<>();
Set<Integer> out = nums.stream()
    .filter(n -> !seen.add(n))
    .collect(Collectors.toSet());
```

## 7) Partition numbers into even and odd
```java
Map<Boolean, List<Integer>> out = nums.stream()
    .collect(Collectors.partitioningBy(n -> n % 2 == 0));
```

## 8) Convert strings to uppercase and sort by length
```java
List<String> out = names.stream()
    .map(String::toUpperCase)
    .sorted(Comparator.comparingInt(String::length))
    .toList();
```

## 9) First non-repeated character in a string
```java
Character out = s.chars().mapToObj(c -> (char) c)
    .collect(Collectors.groupingBy(
        Function.identity(),
        LinkedHashMap::new,
        Collectors.counting()))
    .entrySet().stream()
    .filter(e -> e.getValue() == 1)
    .map(Map.Entry::getKey)
    .findFirst()
    .orElse(null);
```

## 10) First repeated character in a string
```java
Set<Character> seen = new HashSet<>();
Character out = s.chars().mapToObj(c -> (char) c)
    .filter(ch -> !seen.add(ch))
    .findFirst()
    .orElse(null);
```

## 11) Count vowels in a string
```java
long out = s.toLowerCase().chars()
    .filter(c -> "aeiou".indexOf(c) >= 0)
    .count();
```

## 12) Reverse each word in a sentence
```java
String out = Arrays.stream(sentence.split(" "))
    .map(w -> new StringBuilder(w).reverse().toString())
    .collect(Collectors.joining(" "));
```

## 13) Flatten `List<List<Integer>>`
```java
List<Integer> out = listOfLists.stream().flatMap(List::stream).toList();
```

## 14) Merge two lists and remove duplicates
```java
List<Integer> out = Stream.concat(l1.stream(), l2.stream())
    .distinct()
    .toList();
```

## 15) Create comma-separated string from names
```java
String out = names.stream().collect(Collectors.joining(","));
```

## 16) Find employee with highest salary
```java
Employee out = employees.stream()
    .max(Comparator.comparingDouble(Employee::getSalary))
    .orElse(null);
```

## 17) Group employees by department
```java
Map<String, List<Employee>> out = employees.stream()
    .collect(Collectors.groupingBy(Employee::getDepartment));
```

## 18) Department-wise average salary
```java
Map<String, Double> out = employees.stream().collect(Collectors.groupingBy(
    Employee::getDepartment,
    Collectors.averagingDouble(Employee::getSalary)));
```

## 19) Employees with salary greater than X sorted descending
```java
List<Employee> out = employees.stream()
    .filter(e -> e.getSalary() > x)
    .sorted(Comparator.comparingDouble(Employee::getSalary).reversed())
    .toList();
```

## 20) Names of employees in IT department
```java
List<String> out = employees.stream()
    .filter(e -> "IT".equals(e.getDepartment()))
    .map(Employee::getName)
    .toList();
```

## 21) Convert `List<Employee>` to `Map<id, name>`
```java
Map<Integer, String> out = employees.stream()
    .collect(Collectors.toMap(Employee::getId, Employee::getName));
```

## 22) Handle duplicate keys while converting to map
```java
Map<Integer, Employee> out = employees.stream().collect(Collectors.toMap(
    Employee::getId,
    Function.identity(),
    (oldVal, newVal) -> newVal));
```

## 23) Department with maximum employees
```java
String out = employees.stream()
    .collect(Collectors.groupingBy(Employee::getDepartment, Collectors.counting()))
    .entrySet().stream()
    .max(Map.Entry.comparingByValue())
    .map(Map.Entry::getKey)
    .orElse(null);
```

## 24) Departments with average salary > 80000
```java
List<String> out = employees.stream()
    .collect(Collectors.groupingBy(Employee::getDepartment, Collectors.averagingDouble(Employee::getSalary)))
    .entrySet().stream()
    .filter(e -> e.getValue() > 80000)
    .map(Map.Entry::getKey)
    .toList();
```

## 25) Sort map by value ascending
```java
LinkedHashMap<String, Integer> out = map.entrySet().stream()
    .sorted(Map.Entry.comparingByValue())
    .collect(Collectors.toMap(
        Map.Entry::getKey,
        Map.Entry::getValue,
        (a, b) -> a,
        LinkedHashMap::new));
```

## 26) Find common elements between two lists
```java
Set<Integer> s2 = new HashSet<>(l2);
List<Integer> out = l1.stream().filter(s2::contains).distinct().toList();
```

## 27) Check all positive / any negative
```java
boolean allPositive = nums.stream().allMatch(n -> n > 0);
boolean anyNegative = nums.stream().anyMatch(n -> n < 0);
```

## 28) Sum of squares of odd numbers
```java
int out = nums.stream()
    .filter(n -> n % 2 != 0)
    .mapToInt(n -> n * n)
    .sum();
```

## 29) Find longest string
```java
String out = names.stream()
    .max(Comparator.comparingInt(String::length))
    .orElse("");
```

## 30) Total amount spent per user from transactions
```java
Map<String, Double> out = transactions.stream().collect(Collectors.groupingBy(
    Transaction::getUser,
    Collectors.summingDouble(Transaction::getAmount)));
```

---

## Push this to a new GitHub repository

```bash
# from this project folder
git init
git add STREAM_API_INTERVIEW_QUESTIONS.md
git commit -m "Add Java Stream API interview coding questions"
git branch -M main
git remote add origin https://github.com/<your-username>/<new-repo-name>.git
git push -u origin main
```

If this directory is already a git repository, skip `git init` and just add/commit/push.
