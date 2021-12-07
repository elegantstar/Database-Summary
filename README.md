# Index

## 보조 기억 장치

- `데이터베이스`는 `File`들의 집합으로 저장되며, 각 File은 일반적으로 동일한 유형의 `Record`들의 모임으로 이루어진다. 각 Record는 연관된 Filed들의 모임이며, 동일한 개수 및 일정한 크기를 갖는 Field들로 이루어진 `고정 길이 레코드`, 각 레코드마다 서로 다른 Field의 개수를 갖거나 각 Field의 길이가 가변적인 `가변 길이 레코드`로 구분된다.
- File들은 일반적으로 `Disk`와 같은 `보조 기억 장치`에 저장된다. 원하는 데이터를 검색하기 위해서는, DBMS는 디스크 상의 데이터베이스로부터 원하는 데이터를 포함하고 있는 `Block`을 읽어서 `주기억 장치`로 가져와야 한다.
- 보조 기억 장치에서 주기억 장치로 이동하는 데이터의 단위는 `블록(주기억 장치에서는 페이지)`이다. 블록의 크기는 운영체제에 따라 다르지만, 전형적인 크기는 4,096byte이다. 각 File은 고정된 크기의 블록들로 나뉘어 저장된다.
- 디스크에서 임의의 블록을 읽어오거나 기록하는데 걸리는 시간은 `Seek Time(탐구 시간)`, `Rotational Delay(회전 지연 시간)`, `Transfer Time(전송 시간)`의 합이다.

  - **탐구 시간(Seek Time)** : 디스크 헤드가 원하는 실린더 위에 놓일 때까지 걸리는 시간
  - **회전 지연 시간(Rotational Delay)** : 원하는 블록이 디스크 헤드 밑에 올 때까지 걸리는 시간
  - **전송 시간(Transfer Time)** : 블록을 주기억 장치로 전송하는데 걸리는 시간

- 디스크 접근에 소요되는 시간을 줄이기 위해서는 `평균 회전 지연 시간`을 줄이고, `블록 전송 횟수`를 줄이는 것이 관건이다.
- 블록의 크기는 일반적으로 레코드 크기보다 훨씬 크기 때문에 많은 레코드들이 한 블록에 저장된다. 그러나 레코드 길이가 블록 크기를 초과하는 경우, 한 레코드를 두 개 이상의 블록에 걸쳐서 저장하는데 이러한 레코드를 `Spanned Record(신장된 레코드)`라고 한다.
- `Fill Factor(채우기 인수)`는 각 블록의 레코드를 채우는 공간의 비율을 말하는데, Fill Factor에 따라 블록에 레코드를 가득 채우지 않고 빈 공간을 남겨 두면 추가적인 레코드 삽입에 대응할 수 있다.

<br>

## Heap File(비순서 파일) vs Sequential File(순차 파일)

### Heap File

`Heap File(비순서 파일)`은 가장 단순한 파일 조직으로, **레코드들이 삽입된 순서대로 파일에 저장**된다. 일반적으로 새로 삽입되는 레코드는 파일의 가장 끝에 삽입되며, 레코드를 삭제하는 경우 삭제된 레코드가 차지하던 공간을 재사용하지 않는다. 따라서 삽입은 쉬우나 레코드들의 순서는 존재하지 않기 때문에 원하는 레코드를 찾기 위해서는 모든 레코드를 순차적으로 접근해야 한다.  
또한, 시간이 지날수록 삭제된 레코드들이 차지했던 공간이 재사용되지 않아 파일의 크기가 증가하게 된다. 데이터 조회 시, 이러한 빈 공간도 탐색하기 때문에 조회 시간이 길어지게 된다. 따라서 Heap File의 성능 유지를 위해서는 주기적으로 재조직할 필요가 있다.

Heap File은 릴레이션에 데이터를 한꺼번에 적재할 때(Bulk Loading), 릴레이션의 블록 수가 적을 때, 모든 튜플들이 검색 위주로 사용될 때 주로 사용된다. 정리하면, **Heap File은 레코드 삽입이 쉽고, Query에서 레코드 순서에 관계 없이 전체 레코드를 조회하는 경우에 효율적**이다. 그러나 **특정 레코드를 검색하거나 범위 검색을 하는 등 많은 경우에 모든 레코드들에 접근해야만 하는 비효율적인 구조**를 가지고 있다.

### Sequential File

`Sequential File(순차 파일)`은 **레코드들이 하나 이상의 필드 값에 따라 순서대로 저장된 파일**이다. 레코드들이 일반적으로 레코드의 `Search Key(탐색 키)` 값의 순서에 따라 저장된다. 탐색키는 **순차 파일을 정렬하는데 사용되는 필드**를 의미하며, 당연하게도 레코드들에 순차 접근하는 경우에 적합하다. 이런 탐색키 값은 오름차순으로 정렬되어 있기 때문에 특정 레코드 탐색 시 `Binary Search(이진 탐색)`을 활용할 수 있어 순차 탐색 보다 탐색 시간을 줄일 수 있다.  
순차 파일에서 삽입 연산을 진행할 때에는 레코드의 순서를 고려해야 하기 때문에 시간이 많이 걸릴 수 있다. 삽입할 위치가 비어 있는 경우 삽입을 진행할 수 있지만, 빈 공간이 없는 경우 삽입할 레코드를 `Overflow 블록`에 넣거나 다음 블록으로 이동하는 등의 작업을 수행한다. 삭제의 경우 비순서 파일과 마찬가지로 삭제된 레코드의 공간을 빈 공간으로 남겨두기 때문에 주기적인 재조직이 필요하다.

Primary Index가 순차 파일에 정의되지 않는 한 순차 파일은 거의 사용되지 않는다. **Sequential File은 Search Key(탐색 키)를 기반으로 탐색하는 경우에 효율적**이다.

<hr>

## 데이터베이스에서의 Index

`Index`는 의미 그대로 서적의 가장 뒤쪽에 수록되어 특정 주제가 실린 페이지를 쉽게 찾도록 도와주는 색인과 같은 기능을 한다. 따라서 데이터베이스에서의 Index는 어떤 일정한 순서에 따라 데이터가 저장되어 있는 주소를 기록하고 있는 색인으로, 각 레코드는 서적의 특정 내용, 레코드가 저장된 주소값은 해당 주제가 실린 페이지로 비유할 수 있다.  
DBMS가 데이터베이스 릴레이션 내의 모든 데이터를 탐색하는 것은 많은 시간이 소요되지만, **<탐색 키, 레코드에 대한 포인터>로써 key-value 형식의 index를 활용한다면 특정 레코드들을 빠르게 찾을 수 있다**. 따라서 **index는 임의 접근이 필요한 경우에 효율적**이다. 여기서 탐색 키는 index가 정의된 field를 말한다.

**DBMS의 index는 항상 정렬된 상태를 유지하기 때문에 특정 데이터를 탐색하는 것은 빠르지만, 새로운 레코드를 추가하거나 삭제 및 수정을 하는 경우에는 Query 실행 속도가 느려진다. 즉, DBMS에서 index는 데이터 갱신(삽입, 삭제, 수정) 성능을 희생시켜 탐색 선응을 대폭 향상시키는 기능**이다. 단, index가 탐색에 효과적이라고 하여 모든 attribute에 index를 생성하는 것은 데이터 갱신 성능을 저해하고, 불필요한 저장 공간을 사용하게 되는 것이기에 index를 생성할 attribute를 선정하는 것도 중요하다.

### Index 저장 알고리즘, B+-Tree Index Algorithm

**DBMS에서 index를 구현하고 있는 알고리즘은 `B+-Tree` 알고리즘**이다. B+-Tree index는 `Column`의 값을 변형하지 않고, 원래 값을 이용하여 indexing 한다. 원래 값을 변형하지 않는다고 했지만, 실제로는 값의 앞 부분만 잘라서 관리한다. B-Tree는 `Balanced Tree`로, `Root Node`, `Branch Node`, `Leaf Node`가 존재하며, 이 중 최하위에 존재하는 Leaf Node에는 실제 데이터 레코드를 검색할 수 있도록 `Primary key`가 저장되어 있다.

### Hash Index Algorithm

**`Hash Index Algorithm`은 Column의 값으로 `Hash value`를 계산해서 indexing 하는 알고리즘으로 매우 빠른 검색을 지원**한다. 그러나 원래 값을 변형하여 indexing 하므로, 특정 문자로 시작하는 값으로 검색 하는 등 전방 일치와 같이 값의 일부만으로 검색하고자 하는 경우 Hash index를 사용할 수 없다. Hash index는 `Hash Table`의 특성을 닮아 있기에 Hash value의 범위에 따라 성능이 달라진다. Hash value의 범위가 넓은 경우는 `bucket`의 크기가 커져서 공간의 낭비가 커지고, Hash value의 범위가 좁은 경우에는 `Hash Collision` 빈도가 증가하기 때문에 Hash index의 메리트가 사라지는 단점을 갖는다. 이러한 Hash index는 메모리 기반의 데이터베이스에서 주로 사용된다.

### Index 관리에 B+-Tree를 사용하는 이유

데이터에 접근하는 `Time Complexity`만 놓고 생각해 본다면 Time Complexity가 O(1)인 Hash Table이 더 효율적인 것처럼 보인다. 그러나 실제로 데이터를 조회하는 SELECT Query 조건에는 부등호(< >) 연산도 포함된다. Hash Table이 등호(=) 연산에서 위력을 발휘할지라도, 부등호 연산을 수행해야 할 경우에는 Hash Function에 의해 값이 변형되므로 연산 자체가 불가능해진다. 따라서 동등 연산에 특화된 Hash Table은 데이터베이스의 자료구조로 적합하지 않다. 이러한 이유로 데이터베이스의 indexing에는 기존 값을 변형하지 않는 B+-Tree를 사용한다.

<br>

## Primary Index vs Secondary Index

### Primary Index

**`Primary Index(기본 인덱스)`는 `Search key`가 데이터 파일의 `Primary key`인 index**이다. 레코드들은 Primary key의 값에 따라 Clustering 된다. 여기서 Clustering이란 단어 뜻 그대로 관련있는 것들을 모아둔다는 의미로, 레코드들을 Primary key의 value에 따라 묶어서 저장한다는 것이다. 이는 주로 관련 있는 유사한 데이터들을 함께 조회하는 경우가 많다는 점에서 착안된 것인데, 이런 유사한 데이터들은 디스크상에 물리적으로 인접한 곳에 저장된다. 즉, Primary index는 Primary key에 따라 정렬된 데이터 파일에 대해 정의되며, `MySQL`에서는 Primary Index를 갖는 데이터 파일은 Primary key 값이 증가하는 순서로 정렬되어 저장된다.  
**Primary index는 `Sparse index(희소 인덱스)`로 유지할 수 있는데, Sparse index는 일부 key 값에 대해서만 index에 entry를 유지하는 index를 말한다.** 일반적으로 각 블록마다 한 개의 Search key 값이 인덱스 엔트리에 포함되며, 각 인덱스 엔트리는 블록 내 첫 번째 레코드의 key 값(`Block Anchor`)을 갖는다. 이런 구조로 인해 데이터 블록당 레코드 수와 인덱스 블록 당 엔트리 수를 비교한다면, 인덱스 블록 당 엔트리 수가 훨씬 많게 된다.

### Clustering Index

**`Clustering Index`는 Search key 값에 따라 정렬된 데이터 파일에 대해 정의되며, index의 순서와 디스크 상의 파일의 저장 순서가 동일할 때 이를 Clustering index라고 표현한다.** 즉, Clustering index는 Primary key에 대해서만 적용된다. **Clustering index는 파일의 저장 순서와 index 순서가 동일하기 때문에 `Range Query(범위 질의)`에 효과적**이다. Clustering index에서는 인접한 Search key 값을 갖는 레코드들이 디스크에서 가깝게 저장되어 있으므로, 범위의 시작 값에 해당하는 index를 먼저 찾고 범위에 속하는 인덱스 엔트리들을 따라가면서 레코드를 검색할 수 있다. 이런 특성 덕분에 Range Query를 할 때 디스크에서 읽어오는 블록 수가 최소화된다.  
그러나 Clustering Index는 Primary key에 대해서 적용되는 것이기에 한 Relation(Table) 당 한 개만 생성할 수 있으며, Primary key에 수정이 발생하는 경우 레코드의 물리적 저장 위치 또한 함께 변경되어야 하는 리스크가 동반된다. 이러한 이유 때문에 Primary key를 선정할 때에는 신중하게 선정해야 한다.

### Secondary Index

**`Secondary Index(보조 인덱스)`는 Search key 값에 따라 정렬되지 않은 데이터 파일에 대해 정의되는 index**이다. 하나의 파일을 정렬하는데 두 개의 attribute에 대해 동시에 정렬하는 것은 불가능하기 때문에, **Primary key가 아닌 attribute에 대해 정의된 index가 Secondary index**가 된다. Secondary index도 역시 Primary index처럼 레코드를 빠르게 검색할 수 있도록 하는 기능이다. 다만, Secondary index는 `Dense index(밀집 인덱스)`이기 때문에 같은 수의 레코드들을 접근할 때 Primary index를 이용하는 경우보다 디스크 접근 횟수가 증가할 수 있고, 순차 접근을 할 경우에 비효율적이다. 여기서 **Dense index는 각 레코드마다 한 개의 인덱스 엔트리를 갖는 인덱스**를 말한다.  
Secondary index는 Primary index처럼 자주 사용되지는 않으나 레코드 검색에 용이한 attribute에 만들어 사용함으로써, index를 사용하지 않는 경우와 비교했을 때 검색 성능을 대폭 향상시킬 수 있다. 예를 들어, 어떤 신용카드 회사에서 주로 신용카드번호를 사용하여 고객 레코드를 검색한다면 신용카드번호에 대해 Primary index를 생성한다. 그러나 신용카드번호를 기억하지 못하거나 분실하여 확인이 어려운 경우에는 주민등록번호를 사용하여 고객 레코드를 빠르게 찾을 수 있어야 한다. 이때 주민등록번호에 Secondary index가 존재하지 않는다면 조회하는 데에만 수십 분이 걸릴 수 있기에 실무에서 사용할 수가 없게 된다.

<br>

## Sparse Index vs Dense Index

**`Sparse Index`는 인덱스 엔트리의 포인터가 파일의 레코드를 직접 가리키는 것이 아니라, 해당 레코드가 포함된 블록 내 첫 번째 레코드를 가리키는 인덱스**이다. 즉, 각 데이터 블록마다 한 개의 엔트리를 갖는다. Sparse index의 장점은 레코드의 개수보다 인덱스의 크기가 작기 때문에 인덱스를 메모리에 올리는 시간과 디스크 접근 횟수를 줄일 수 있다는 것이다. Sparse index는 다단계 인덱스에서 2단계 이상의 인덱스에서는 필수이며, 단일 단계에서는 선택사항이다. 대부분의 경우에는 1단계에서도 Sparse index를 사용하는 편이 좋다.

**`Dense Index`는 인덱스 엔트리의 포인터가 파일의 레코드를 직접 가리키는 인덱스**이다. 즉, 각 레코드마다 한 개의 엔트리를 갖기 때문에 인덱스 엔트리 수와 파일의 레코드 수가 동일하다. 일반적으로 Sparse index를 사용하면 대부분의 갱신과 쿼리에 대해 더 효율적이지만, Dense index만이 갖는 장점도 있다. 예를 드어, Query에서 index가 정의된 attribute만 검색하는 경우(데이터의 개수를 세는 `COUNT`, 데이터 존재 유무를 확인하는 `EXIST` 등)에는 데이터 파일에 접근할 필요 없이 인덱스만 접근하여 Query를 수행할 수 있으므로 Dense index가 Sparse index보다 더 효율적이다.

## Clustering Index vs Secondary Index

Clustering Index는 Sparse Index일 경우가 많으며 Range Query 등에 효율적이다. Clustering index가 불리한 경우는 Relation의 중간에 Tuple이 삽입되어 Overflow를 야기하고, 이로 인해 Clustering의 장점을 잃게 되는 경우이다. Clustering index를 정의할 때는 Fill Factor에 낮은 값을 지정하여 추가로 삽입되는 레코드들에 대비하는 것이 바람직하다. Secondary index는 Dense index이므로 일부 Query에 대해서는 파일에 접근할 필요 없이 처리할 수 있다.

## 다단계 인덱스

**`다단계 인덱스`는 인덱스에 인덱스를 생성한 것을 의미**한다. 단일 단계 인덱스 자체를 인덱스가 정의된 필드의 값에 따라 정렬된 파일로 보고, 그에 따라 인덱스를 생성하는 것이다. 인덱스 자체의 크기가 클 경우에는 인덱스를 탐색하는 시간도 오래 걸릴 수 있기 때문에, 인덱스 엔트리 탐색 시간을 줄이기 위한 방안으로 다단계 인덱스를 도입하였다. 위에서 언급했듯 1단계 인덱스는 Sparse index 또는 Dense index 모두 가능하지만, 2단계 이상의 인덱스는 Sparse index만 가능하다.

이러한 **다단계 인덱스는 가장 상위 단계의 모든 인덱스 엔트리들이 한 블록에 들어갈 수 있을 때까지 반복**하여 만든다. 이렇게 만들어진 다단계 인덱스의 가장 상위 단계 인덱스는 `Master Index`라고 부른다. Master Index는 한 블록으로 이루어지기 때문에 주기억 장치에 상주할 수 있다는 장점이 있다.  
대부분의 다단계 인덱스는 `B+-Tree`로 구현되어 있다. B+-Tree의 `Inner Node`는 다수의 `Child Node`를 갖고 각 Node는 한 개의 디스크 블록을 차지하는데, 일반적으로 한 블록에 자식 노드들에 대한 포인터를 수백 개 저장할 수 있다. B+-Tree는 새롭게 추가될 인덱스 엔트리에 대응하기 위해 각 인덱스 블록에 예비 공간을 남겨 둔다. 다단계 인덱스는 각 단계의 인덱스가 오름차순으로 유지되어야 하기 때문에 인덱스 엔트리에 갱신이 발생할 경우 단일 단계 인덱스의 경우보다 복잡한 처리 과정을 거쳐야 한다. 그럼에도 불구하고 **대부분의 데이터베이스에서는 검색 비율이 갱신 비율보다 월등히 높기 때문에 모든 DBMS에서는 인덱스를 다단계 인덱스로 유지한다.**

## Composite Index

**한 릴레이션에 속하는 두 개 이상의 attribute들의 조합(`Composite Attribute`)에 대해 하나의 인덱스를 정의할 수 있는데, 이러한 인덱스를 `Composite Index(복합 인덱스)`라고 한다.** Composite index를 정의할 때 attribute의 수는 3개 이하를 사용하는 편이 좋다. 인덱스가 정의된 composite attribute에 포함된 attribute의 수가 늘어날수록 이 인덱스를 활용하는 탐색 조건이 복잡해지고, 인덱스 엔트리의 길이가 늘어나기 때문에 탐색 성능이 저하되기 때문이다.

**복합 인덱스를 정의할 때는 attribute의 순서가 중요**하다. 정의된 순서에 따라서 attribute의 필드가 정렬 되기 때문이다.

> 만약 product라는 릴레이션의 (brand, price) attribute에 대해 복합 인덱스를 생성한다고 가정해 보자. 그러면 brand 이름 순으로 먼저 정렬이 되고, 각 brand 마다 가격 별로 상품들이 정렬이 된다. 따라서 brand를 기준으로 search하거나 brand와 price의 값을 모두 이용하여 search하는 경우는 composite index의 효과를 볼 수가 있다. 그러나 두 번째 attribute인 price만을 이용하여 search 하는 경우에는 index의 효과를 볼 수가 없게 된다. 전체 상품이 price를 기준으로 정렬되어 있는 것이 아니기 때문이다. 단, WHERE절의 첫 번째 조건으로 첫 번째 attribute에 대해서 LIKE, 부등호, IN을 사용하는 경우에, 두 번째 조건에서는 index 활용을 하지 못 할 수도 있다. 예를 들어, brand에 대해 Range Search를 하게 되면 price에 대해서는 정렬되지 않기 때문에 index 활용을 하지 못 하는 것이다.

이러한 이유로 실무에서 SELECT Query 사용 방식에 따라 index 설계에 지대한 영향을 미친다. 따라서 단순히 이론적으로 index를 어떻게 생성하는지 아는 것에 그치지 않고, 실제 서비스 상에서 어떤 query를 자주 사용하고 어떤 방식으로 활용될 것인지를 파악하여 index 설계에 반영하는 것이 매우 중요하다.

<br>

## Index 선정 및 데이터베이스 튜닝

### Index 성능 및 고려 사항

INDEX는 SELECT query의 성능을 월등히 향상시키는 중요한 요소이다. 그렇다면 검색 성능 향상을 위해 모든 attribute에 index를 생성한다면 어떻게 될까?  
_기대와는 달리, index를 과도하게 생성한다면 과유불급이라는 말처럼 외려 성능에 좋지 않은 영향을 주게 된다._

index 생성이 수반하는 문제는 다음과 같다.

**첫째, cost의 증가가 필연적이다.**  
index는 검색 속도를 향상시키지만 **index를 저장하기 위한 공간이 추가로 필요**하며, **INSERT, DELETE, UPDATE 연산 시 별도의 과정이 동반되기에 연산 속도를 저하**시킨다. `INSERT Query`에서는 INDEX에 대한 데이터를 추가해야 하므로 그만큼의 성능 손실이 발생하며, `DELETE Query`의 경우 인덱스 엔트리를 삭제하지 않고 '사용하지 않음'을 표시하게 된다. 즉, INDEX record 수는 삭제 연산 후에도 그대로 유지된다. 이러한 작업이 반복되다 보면, 삭제되지 않은 index record들 때문에 실제 데이터 수에 비해 인덱스 엔트리 수가 더 많아지기 때문에 index가 제 기능을 할 수 없게 될 수도 있다. `UPDATE`의 경우에는 INSERT와 DELETE 연산의 문제를 모두 수반한다.

**둘째, attribute를 이루는 데이터의 형식에 따라 index의 성능이 크게 좌우된다.**  
이 점은 더욱 중요하다. cost 문제를 크게 야기하지 않는 경우라도 index가 제 기능을 할 수 없는 경우가 있기 때문이다. index 사용이 효율적/비효율적인 데이터의 형식이 존재한다는 것이다. index는 결국 탐색 범위를 좁혀 탐색 속도를 향상하는 것이 목적이므로, **index를 사용하여 탐색의 범위를 효과적으로 좁힐 수 있는 attribute에 대해 index를 생성해야 한다. 즉, index는 파일의 record를 충분히 분해할 수 있어야 한다.**

> name, age, gender 세 가지 필드를 갖는 테이블에서 인덱스를 생성한다고 가정해 보자. name의 경우에는 셀 수 없을 정도로 많은 경우의 수가 존재하고, age는 INT 타입으로 정의될 것이며, gender의 경우 일반적으로 male, female 두 가지 경우만 존재할 것이다. 이런 상황에서는 name에 대한 index만 정의하는 것이 효율적이다.  
> age 또는 gender에 대한 index를 생성하는 것은 왜 비효율적일까? index 사용 시 탐색의 범위를 크게 좁힐 수 없기 때문이다. gender attribute에 대한 index를 이용하여 조회하는 상황을 생각해 보자. 이 경우, index를 사용한다고 해도 줄일 수 있는 값의 range는 50%이다. 만약 row의 수가 10000인 테이블에 대해 2000개 단위로 gender index 블록을 생성하는 상황이라면, 일반적인 경우 한 번에 인덱스를 읽어 오지 못 해 추가적인 디스크 I/O가 발생할 수 밖에 없을 것이다. age의 경우에도 실질적으로 존재할 수 있는 경우의 수는 100도 채 되지 않으며, 특정 연령대를 타겟팅하는 서비스의 경우 그 경우의 수는 더욱 줄어들 것이다.
> 따라서 이와 같은 attribute에 대해 index를 생성해야 하는 경우에는, **field의 cardinality가 높은 것부터 낮은 순으로 composite index를 생성하는 것이 효율적**이다.

### Index 선정 지침

1. primary key는 Index를 정의할 좋은 후보가 되기 때문에 대부분의 DBMS는 primary key로 명시한 attribute에 대해 자동적으로 index를 생성한다.
2. foreign key 역시 index 정의에 중요한 후보이기에, 어떤 DBMS에서는 foreign key로 지정한 attribute에 대해 자동적으로 index를 생성하기도 한다.
3. 한 attribute에 들어 있는 상이한 값들의 개수가 전체 record 수와 비슷하고(high cardinality), 그 attribute가 동등 조건에 사용된다면 index를 생성하는 것이 좋다.
4. 갱신이 빈번한 attribute에는 index를 정의하지 않는 것이 좋으며, 갱신이 빈번한 relation에 많은 index는 피해야 한다.
5. 대량의 데이터를 삽입할 때는 모든 index를 제거하고, 데이터 삽입이 끝난 후에 index를 다시 생성하는 것이 좋다.
6. 정렬 속도 향상을 위해서 ORDER BY절에 자주 사용되는 attribute에는 index를 생성하는 것이 좋고, 그룹화 속도 향상을 위해서 GROUP BY절에 자주 사용되는 attribute도 index를 생성하는 것이 좋다.

### Query Tunning을 위한 추가 지침

1. `DISTINCT`절 사용을 최소화한다.
2. `GROUP BY`절, `HAVING`절 사용을 최소화한다.

<br>
<hr>

# Normalization(정규화)

`Normalization(정규화)`은 테이블을 무손실 분해하여 데이터의 중복 및 삽입, 삭제, 갱신 등의 `Anomaly(이상 현상)`을 제거하는 작업이다.

## Anomaly(이상 현상)

Anomaly 상황에 대한 예시 스키마: {brand_id, product_id, category}

### 1. Insertion Anomaly(삽입 이상)

예시 테이블의 Primary key는 {brand_id, product_id}이다. 만약 신규 입점한 브랜드가 상품 등록을 하기 전이라면 어떤 product_id도 가질 수 없다. 그러나 Primary key에는 NULL을 삽입할 수 없으므로, 이 테이블에는 등록한 상품이 없는 브랜드에 대한 정보는 삽입할 수가 없다.  
이렇듯 **불필요한 정보를 함께 저장하지 않고는 어떤 정보를 저장하는 것이 불가능한 이상 현상을 삽입 이상(Insertion Anomaly)이라고 한다.**

### 2. Deletion Anomaly(삭제 이상)

어떤 브랜드가 품목 리뉴얼을 위해 판매 중인 상품을 모두 내리는 경우, 브랜드의 정보(브랜드 번호, 브랜드 카테고리)까지 전부 삭제되는 문제가 발생한다.  
이렇듯 **유용한 정보를 함께 삭제하지 않고는 어떤 정보를 삭제하는 것이 불가능한 이상 현상을 삭제 이상(Deletion Anomaly)이라고 한다.**

### 3. Modification Anomaly(수정 이상)

만약 어떤 브랜드의 카테고리가 변경되는 경우, 해당 브랜드의 모든 category를 전부 변경해야만 한다. 만약 이 과정에서 변경 누락이 발생하여 기존 category를 유지하는 tuple이 생긴다면 데이터의 불일치가 발생한다.  
이렇듯 **반복된 데이터 중에 일부만 수정하여 데이터 불일치가 발생하는 이상 현상을 수정 이상 또는 갱신 이상(Modification Anomaly)이라고 한다.**

<br>

## Functional Dependency(함수적 종속성)

`Functional Dependency(FD, 함수적 종속성)`은 관계형 데이터 모델에서 가장 중요한 제약 조건 중 하나로, 정규화 이론의 핵심이다. 함수적 종속성은 attribute들의 의미로부터 결정되며, 릴레이션의 특정 인스턴스가 아닌 릴레이션 스키마에 대한 제약 조건이다. 따라서 함수적 종속성 제약 조건은 릴레이션의 모든 인스턴스들이 만족해야 한다.

### Determinant(결정자)

**`Determinant(결정자)`는 주어진 릴레이션에서 다른 attribute(또는 attribute set)를 고유하게 결정하는 하나 이상의 attribute를 의미**한다. 예를 들어, {brand_id, brand_name, phone, product_id, product_name} 스키마가 있을 때, brand_id는 brand_name, phone의 determinant이다. 또한, product_id는 product_name의 determinant이다. 결정자는 [ A → B ] 와 같이 표기하고, "A가 B를 결정한다(또는 A는 B의 결정자이다)"라고 표현한다.  
릴레이션 R에서 attribute A가 B의 결정자이면 임의의 두 tuple에서 A의 값이 같으면 B의 값도 같아야 한다. A와 B는 composite attribute일 수 있다.

### Functional Dependency(함수적 종속성)

만약 **attribute A가 attribute B의 determinant이면 B가 A에 `함수적으로 종속`한다**고 말한다. 달리 표현하면, 주어진 릴레이션 R에서 attribute B가 attribute A에 함수적으로 종속하는 필요 충분 조건은 각 A 값에 대해 반드시 한 개의 B 값이 대응된다는 것이다.

### Full Functional Dependency(FFD, 완전 함수적 종속성)

**주어진 릴레이션 R에서 attribute B가 attribute A에 함수적으로 종속하면서, attribute A의 어떠한 진부분 집합에도 함수적으로 종속하지 않으면 attribute B가 attribute A에 `완전하게 함수적으로 종속`한다**고 말한다. 여기서 attribute A는 composite attribute이다.

### Transitive Functional Dependency(이행적 함수적 종속성)

한 릴레이션의 attribute A, B, C가 주어졌을 때, **attribute C가 `이행적`으로 A에 종속하는(A → C) 필요 충분 조건은 [ A → B ∧ B → C ]가 성립하는 것**이다. A가 릴레이션의 Primary key라면 정의에 따라 A → B와 A → C가 성립한다. 만약 C가 A 외에 B에도 함수적으로 종속한다면 C는 A에 직접 함수적으로 종속하면서 B를 거쳐서 A에 이행적으로도 종속한다. 이행적 종속성이 존재하는 릴레이션에는 key가 아닌 attribute가 적어도 두 개 이상 있어야 한다.

<br>

## Decomposition(릴레이션 분해)

**`Decomposition(릴레이션 분해)`는 하나의 릴레이션을 두 개 이상의 릴레이션으로 나누는 것**이다. 릴레이션을 분해하면 중복이 감소되고 이상 현상이 줄어드는 장점이 있다. 정규화의 목적은 데이터의 중복을 제거하고, 이상 현상을 제거하는 것이기에 정규화 과정에서 릴레이션의 분해는 필수적인 작업이 된다.  
그러나 릴레이션을 분해하는 것은 이런 장점 외에도 몇 가지 문제들을 야기할 수 있다. 첫째, 릴레이션을 분해하면 JOIN이 필요 없는 query가 JOIN이 필요한 query로 바뀔 수 있다. 둘째, 분해된 릴레이션들을 사용하여 원래 릴레이션을 재구성하지 못할 수도 있다.  
만약 릴레이션을 분해한 후, **분해된 릴레이션들을 JOIN하여 기존의 릴레이션에 들어 있는 정보를 완전하게 얻을 수 있다면 이러한 분해를 `Lossless Decomposition(무손실 분해)`라고 한다.** 여기서 **`정보의 손실`이란 분해 후 생성된 릴레이션들을 JOIN한 결과에 들어 있는 정보가 원래의 릴레이션에 들어 있는 정보보다 적거나 많은 것을 모두 포함**하는 개념이다.

<br>

## Normal Form(NF, 정규형)

`Normal Form(정규형)`에는 제1정규형(1NF), 제2정규형(2NF), 제3정규형(3NF), BCNF, 제4정규형(4NF), 제5정규형(5NF) 등이 있으나, 일반적으로 DB를 설계할 때 3NF 또는 BCNF까지만 고려한다. 이 중 2NF부터 BCNF까지의 정규형은 함수적 종속성 이론에 기반을 둔다.

### 제1정규형(1NF)

**모든 속성의 도메인이 `원자 값(atomic value, 더 이상 분해할 수 없는 값)`만으로 구성되어 있다**면 `제1정규형(1NF)`을 만족한다.

### 제2정규형(2NF)

**제1정규형을 만족하면서, 어떤 `candidate key`에도 속하지 않는 모든 attribute들이 primary key에 완전하게 함수적으로 종속한다**면 `제2정규형(2NF)`을 만족한다. 즉, **2NF는 릴레이션 내에 존재하는 모든 `부분 함수적 종속`을 제거하여 `완전 함수적 종속`으로 만드는 단계**이다.

### 제3정규형(3NF)

**제2정규형을 만족하면서, key가 아닌 모든 attribute가 primary key에 이행적으로 종속하지 않는다**면 `제3정규형(3NF)`을 만족한다. 즉, **3NF는 `이행적 함수 종속`을 제거하는 단계**이다.

### BCNF(Boyce-Codd Normal Form)

**제3정규형을 만족하면서, `모든 결정자(Determinant)`가 `후보키(candidate key)`라면 `BCNF`를 만족**한다.

<br>

## Denormalization(비정규화, 반정규화, 역정규화)

정규화는 단계가 진행될수록 중복이 감소하고 이상 현상도 줄어드는 장점이 있어 데이터베이스 설계의 중요한 요소이지만, 높은 수준의 정규형을 만족하는 것이 항상 최선인 것은 결코 아니다. 정규형의 단계가 한 단계 높아질수록 하나의 릴레이션이 적어도 두 개 이상의 릴레이션으로 분해되기 때문에 정규화 이전과 같은 정보를 얻기 위해서는 많은 JOIN 연산이 필요하게 된다. 즉, 높은 정규형을 만족할수록 릴레이션 스키마의 성능은 나빠질 수 있다.  
**`Denormalization`은 JOIN 연산이 빈번하게 발생하여 응답 시간이 느려지는 경우, query 수행 속도 향상을 위해 이미 분해된 두 개 이상의 릴레이션들을 하나의 릴레이션으로 합치는 작업**이다. 즉, 낮은 정규형으로 되돌아가는 것이다. 조회 성능이 중요한 작업의 경우는 이렇게 비정규화를 통해 성능상 요구사항을 만족하는 전략을 취할 수 있다.
