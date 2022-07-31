# 8.1 컬렉션 팩토리

리스트 / 집합 / 맵

컬렉션을 생성할 때 단순하게 생성하기 위함

- 적은 요소를 포함하는 리스트 생성

    ```java
    List<String> friends = new ArrayList<>();
    friends.add("Raphael");
    friends.add("Olivia");
    friends.add("Thibaut");
    ```

    ```java
    List<String> friends2 = Arrays.asList("Raphael", "Olivia", "Thibaut");
    ```

  ⇒ 요소를 추가하려면 **UnsupportedOperationException** 발생

    ```java
    Set<String> friends3 
    		= new HashSet<>(Arrays.asList("Raphael", "Olivia", "Thibaut"));
    ```

    ```java
    Set<String> friends4 = Stream.of("Raphael", "Olivia", "Thibaut")
            .collect(Collectors.toSet());
    ```

  ⇒ set이라서 size문제는 없지만 불필요한 객체 할당이 많다



>💡 **컬렉션 리터럴**
>아래와 같은 문법이 가능 하다는 것 (파이썬, 그루비 등 몇몇 언어에서 지원)
>****`$fruits = [ "Apple", "Mango", "Guava" ];`


- `List.of`를 사용할 수 있지만 변경할 수 없는 리스트로 생성됨 → 변경할 수 없는 제약

>💡 **오버로딩 vs 가변인수**
>- 가변인수 버전은 추가 배열을 할당해서 리스트로 감싼다 → gc비용 지불
>- 가변인수로(파라미터로 배열을 받음) 하는게 아니라 정해진 다중 요소로 오버로딩
>cf. Set.of, Map.of


- `Set.of` : 중복요소로 생성 불가
- `Map.of` : 10개 이하의 키와 값 쌍으로 가진 맵 생성

    ```java
    Map<String, Integer> ageOfFriends
       = Map.of("Raphael", 30, "Olivia", 25, "Thibaut", 26);
    System.out.println(ageOfFriends);
    ```

- `Map.Entry` : 10개 보다 많은 맵

    ```java
    import static java.util.Map.entry;
    Map<String, Integer> ageOfFriends
           = Map.ofEntries(entry("Raphael", 30),
                           entry("Olivia", 25),
                           entry("Thibaut", 26));
    System.out.println(ageOfFriends);
    ```


# 8.2 리스트와 집합 처리

- removeIf : list나 set에서 프레디케이트를 만족하는 요소 제거
- replaceAll : list에서 요소 변경
- sort : 정렬

⇒ 호출한 컬렉션 자체를 변경 → 에러가 날 확률이 올라감

## 8.2.1 removeIf 메서드

- ConcurrentModificationException

    ```java
    for (Transaction transaction : transactions) {
       if(Character.isDigit(transaction.getReferenceCode().charAt(0))) {
           transactions.remove(transaction);
       }
    }
    ```

    ```java
    for (Iterator<Transaction> iterator = transactions.iterator();
         iterator.hasNext(); ) {
       Transaction transaction = iterator.next();
       if(Character.isDigit(transaction.getReferenceCode().charAt(0))) {
           transactions.remove(transaction);
       }
    }
    ```

  iterator와 collection 개별 객체가 컬렉션 관리

  ⇒ 객체의 remove()호출로 해결

    ```java
    for (Iterator<Transaction> iterator = transactions.iterator();
         iterator.hasNext(); ) {
       Transaction transaction = iterator.next();
       if(Character.isDigit(transaction.getReferenceCode().charAt(0))) {
           iterator.remove();
       }
    }
    ```


```java
transactions.removeIf(transaction ->
     Character.isDigit(transaction.getReferenceCode().charAt(0)));
```

## 8.2.2 replaceAll 메서드

- 요소 변경과 새 문자열 생성 그 사이 어딘가

  리스트의 각 요소를 새로운 요소로 변경..이라기보단 새 문자열 컬렉션 생성

    ```java
    referenceCodes.stream()
                  .map(code -> Character.toUpperCase(code.charAt(0)) +
         code.substring(1))
                  .collect(Collectors.toList())
                  .forEach(System.out::println);
    ```

  `ListIterator` (요소를 바꾸는 set())

    ```java
    for (ListIterator<String> iterator = referenceCodes.listIterator();
         iterator.hasNext(); ) {
       String code = iterator.next();
       iterator.set(Character.toUpperCase(code.charAt(0)) + code.substring(1));
    }
    ```


```java
referenceCodes.replaceAll(code -> Character.toUpperCase(code.charAt(0)) +
     code.substring(1));
```

# 8.3 맵 처리

디폴트 메서드는 13장에 To Be Continue...

- forEach 메서드
    - 맵의 항목 집합 반복

        ```java
        for(Map.Entry<String, Integer> entry: ageOfFriends.entrySet()) {
           String friend = entry.getKey();
           Integer age = entry.getValue();
           System.out.println(friend + " is " + age + " years old");
        }
        ```

    - BiConsumer 인수

        ```java
        ageOfFriends.forEach((friend, age) -> System.out.println(friend + " is " +
             age + " years old"));
        ```

- 정렬 메서드
    - Entry.comparingByValue
    - Entry.comparingByKey

        ```java
        Map<String, String> favouriteMovies
               = Map.ofEntries(entry("Raphael", "Star Wars"),
               entry("Cristina", "Matrix"),
               entry("Olivia",
               "James Bond"));
        
        favouriteMovies
          .entrySet()
          .stream()
          .sorted(Entry.comparingByKey())
          .forEachOrdered(System.out::println);
        ```



>💡 **HashMap 성능**
>기존 : 키로 생성한 해시코드로 접근할 수 있는 버켓에 저장
>→ 많은 키가 같은 해시코드로 반환되면 LinkedList로 버킷을 반환하여 성능 저하
>→ O(n)
>최근 : 버킷이 커지면 정렬 트리를 통해 동적으로 치환
>→ O(log(n))

- getOrDefault 메서드
    - NullPointerException 방지
    - 키 값 존재 여부에 따라 두 번째 인수 반환 여부 결정

    ```java
    Map<String, String> favouriteMovies
           = Map.ofEntries(entry("Raphael", "Star Wars"),
           entry("Olivia", "James Bond"));
    
    System.out.println(favouriteMovies.getOrDefault("Olivia", "Matrix")); // James Bond
    System.out.println(favouriteMovies.getOrDefault("Thibaut", "Matrix")); // Matrix
    ```

- 계산 패턴
    - `computeIfAbsent` : 키에 해당하는 값이 없으면, 새 값을 계산하여 맵에 추가
        - 예시) MessageDigest 인스턴스로 SHA-256 해시 계산

            ```java
            Map<String, byte[]> dataToHash = new HashMap<>();
            MessageDigest messageDigest = MessageDigest.getInstance("SHA-256");
            
            lines.forEach(line ->
               dataToHash.computeIfAbsent(line,
                                          this::calculateDigest));
            
            private byte[] calculateDigest(String key) {
               return messageDigest.digest(key.getBytes(StandardCharsets.UTF_8));
            }
            ```

        - 예시) 여러 값 저장하는 맵 처리

            ```java
            String friend = "Raphael";
            List<String> movies = friendsToMovies.get(friend);
            if(movies == null) {
               movies = new ArrayList<>();
               friendsToMovies.put(friend, movies);
            }
            movies.add("Star Wars");
            
            System.out.println(friendsToMovies);
            ```

            ```java
            friendsToMovies.computeIfAbsent("Raphael", name -> new ArrayList<>())
                          .add("Star Wars");
            ```


        값을 만드는 함수가 널을 반환하면 현재 매핑을 맵에서 제거
        
    - `computeIfPresent` : 키가 존재하면, 새 값을 계산하여 맵에 추가
    - `compute` : 제공된 키로 새 값을 계산하고 맵에 저장
- 삭제 패턴
    - [자바8] 키가 특정 값과 연관되어있을 때만 항목을 제거하는 오버로드 버전 메서드 제공

    ```java
    String key = "Raphael";
    String value = "Jack Reacher 2";
    if (favouriteMovies.containsKey(key) &&
         Objects.equals(favouriteMovies.get(key), value)) {
       favouriteMovies.remove(key);
       return true;
    }
    else {
       return false;
    }
    
    favouriteMovies.remove(key, value);
    ```

- 교체 패턴
    - `replaceAll` : BiFunction을 적용한 결과로 각 항목의 값 교체 (cf. List - `replaceAll`)
    - `Replace` : 키가 존재하면 맵의 값을 바꿈 (키가 특정 값으로 매핑되었을 때만 값을 교체하는 오버로드 버전도 있음)
- 합침
    - `putAll` : 두 개의 맵을 합침

        ```java
        Map<String, String> family = Map.ofEntries(
           entry("Teo", "Star Wars"), entry("Cristina", "James Bond"));
        Map<String, String> friends = Map.ofEntries(
           entry("Raphael", "Star Wars"));
        Map<String, String> everyone = new HashMap<>(family);
        everyone.putAll(friends);
        System.out.println(everyone);
        ```

    - `merge` : 중복 키가 있는 유연한 상황에서도 작동

        ```java
        Map<String, String> family = Map.ofEntries(
            entry("Teo", "Star Wars"), entry("Cristina", "James Bond"));
        Map<String, String> friends = Map.ofEntries(
            entry("Raphael", "Star Wars"), entry("Cristina", "Matrix"));
        
        Map<String, String> everyone = new HashMap<>(family);
        friends.forEach((k, v) ->
           everyone.merge(k, v, (movie1, movie2) -> movie1 + " & " + movie2));
        System.out.println(everyone);
        ```

      null처리도 가능

      지정된 키와 연관된 갑싱 없거나 값이 널이면 [merge]는 키를 널이 아닌 값과 연결한다. 아니면 [merge]는 연결된 값을 주어진 매핑 함수의 [결과] 값으로 대치하거나 결과가 널이면 [항목]을 제거한다.

        - 초기화 검사 구현

            ```java
            // 영화가 맵에 존재하는지 확인
            Map<String, Long> moviesToCount = new HashMap<>();
            String movieName = "JamesBond";
            long count = moviesToCount.get(movieName);
            if(count == null) {
               moviesToCount.put(movieName, 1);
            }
            else {
               moviesToCount.put(moviename, count + 1);
            }
            ```

            ```java
            moviesToCount.merge(movieName, 1L, (key, count) -> count + 1L);
            ```


# 8.4 개선된 ConcurrentHashMap

ConcurrentHashMap

- 동시성 친화적이며 최신 기술을 반영한 HashMap
- 내부 자료구조의 특정 부분만 잠궈 동시 추가, 갱신 작업을 혀용함
- 동기화 Hashtable에 비해 읽기 쓰기 연산 성능이 월등함
- 표준 HashMap은 비동기

## 8.4.1 리듀스와 검색

- `forEach` : 각 (키, 값)에 주어진 액션 실행
- `reduce` : 모든 (키, 값)을 제공된 리듀스 함수를 이용해 결과로 합침
- `search` : 널이 아닌 값을 반환할 때까지 (키, 값)에 함수 적용

|  | forEach | reduce | search |
| --- | --- | --- | --- |
| 키, 값으로 연산 | forEach | reduce | search |
| 키로 연산 | forEachKey | reduceKeys | searchKeys |
| 값으로 연산 | forEachValue | reduceValues | searchValues |
| Map, Entry 객체로 연산 | forEachEntry | reduceEntries | searchEntries |

병렬성 기준값(threshold) 지정으로 병렬성 조정 가능 (몇 개의 스레드로 연산을 하는지)

- reduceValues로 맵의 최댓값 찾기

    ```java
    ConcurrentHashMap<String, Long> map = new ConcurrentHashMap<>();
    long parallelismThreshold = 1;
    Optional<Integer> maxValue =
       Optional.ofNullable(map.reduceValues(parallelismThreshold, Long::max));
    ```


## 8.4.2 계수

- `mappingCount` : 맵의 매핑 개수 반환 (size말고 이거 써)

## 8.4.3 집합뷰

- `keySet` : 맵을 바꾸면 집합도 바뀌고 집합을 바꾸면 맵에도 영향
- `newKeySet` : ConcurrentHashMap으로 유지되는 집합 생성

# 8.5 마치며

- [자바9] 리스트, 집합, 맵 만드는 컬렉션 팩토리 지원
- 컬렉션 팩토리가 반환한 객세는 생성 후 변경 불가능
- List - removeIf, replaceAll, sort
- Set - removeIf
- Map - 많아..
- ConcurrentHashMap은 스레드 안전성을 제공함