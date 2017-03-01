Jackson 라이브러리 성능 팁
====
다음 두 링크를 참고하여 작성. 대부분 잭슨측의 주장(?)을 반영. 검증하지 않았으니, 참고만..
- http://wiki.fasterxml.com/JacksonBestPracticesPerformance
- http://k11i.biz/blog/2016/10/01/high-performance-jackson/

0. 결론
----
- 가급적 스트림 API를 쓰자.
- ObjectMapper, JsonFactory 객체는 가급적 한개만.

벤치마크
https://github.com/komiya-atsushi/java-playground/tree/master/jackson-performance

방법|초당처리 수
---|---
ObjectMapper 매번생성|2.596
ObjectMapper 재사용|32.533
ObjectReader 사용|33.306
ObjectReader#readValues() 사용|43.641
Afterburner 사용|49.764
Streaming API사용|77.969

1. 객체 재사용
----
ObjectMapper, JsonFactory 등의 객체는 생성하는데 시간이 오래걸린다.
내부적으로 한번 변환했던 객체모양(xxx.class)을 캐싱하고, threadsafe하므로
한번 만들어 놓은 인스턴스로 돌려막기(?)하는 것이 좋다.

하지만, ObjectReader/ObjectWriter는 생성비용이 그닥 크지 않기 때문에, 매번 만들어서 써도 별 차이 없다.

2. 쓰고나면 정리
----
JsonParser/JsonGenerator 같은 스트림 API들은 안쪽에 버퍼를 가지고 있어서, 다 쓰고 나면 close로 잘 처분해야 한다.

3. 입력 소스는 최소한의 프로세스로
----
잭슨은 쌩(raw)데이터 처리에 최적화 되어 있고, 변환이 필요한 경우의 버퍼링 처리를 잘 하고 있다.
뭐 해줄려고 하지말고, 그냥 쌩데이터를 넘겨라.

이하, 잭슨측 주장
- UTF-8 인코딩/디코딩의 경우, 기본 JDK보다 빠르다
- 버퍼링, 스트림 닫는거 신경쓰지마라, 알아서 다 해준다.
  - 입력이 InputStream 으로 들어온다면, InputStreamReader 같은거 쓰지말고 그냥 InputStream을 넘겨라
  - 잭슨은 적절한 인코딩을 알아서 찾는다. 신경쓰지 말고 그냥 InputStream 넘겨라.
  - JsonParser나 JsonGenerator가 닫힐 때, URL이나 File에서 온 InputStream 같은건 알아서 닫아준다. 그냥 넘겨라.

4. 설정변경은 꼭! 필요할때만
----
기본 설정이 꽤 최적화가 잘 되어있는 편이라, 꼭 필요해서가 아닌 상태로 건드리면 변경한 옵션 동작으로 성능이 떨어질 수 있다.

5. 재처리가 필요한경우, 파싱을 다시 하지 않도록
----
잭슨의 트리모델을 이해해야 할 듯 한데..뭔 소린지 잘 모르겠다.
http://wiki.fasterxml.com/JacksonTreeModel
나중에 다시 보자.

6. 정적타이핑
----
정적 타입은 직렬화(serialization, toJson)에 유리하다. final 클래스같은 아이들은 일부 타입 체크를 하지 않아도 되기 때문에 성능향상이 있다(10-20%정도). 오로지 이것때문에!는 좀 오바지만, 알아두면 나쁠거 없다고..

7. ObjectMapper보다는, ObjectReader/ObjectWriter를 쓰라
----
2.1 이후의 잭슨은 serializer/deserializer를 재사용 하므로, 조금 더 효율적으로 움직인다.

8. 순차로 들어오는 입력에는 ObjectReader.readValues를 써라
----
기본적으로 array나 list로 잘 던져주지만 http://www.baeldung.com/jackson-collection-array
같은 모양의 객체가 반복되는 경우, ObjectMapper.readValue()보다 ObjectMapper.readValues()가 좀 더 효율적이다.

그 외 참고링크
------
- http://www.baeldung.com/jackson-bidirectional-relationships-and-infinite-recursion
- https://www.stubbornjava.com/posts/practical-jackson-objectmapper-configuration
