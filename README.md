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

<br>
<hr>

# Transaction

**`Transaction`은 하나의 작업을 수행하기 위해 필요한 데이터베이스 연산들을 묶어 놓은 것으로, 데이터베이스에서 논리적인 작업의 단위**를 말한다. 시스템 관점에서는 데이터에 접근하거나 데이터베이스의 상태를 변경하는 프로그램의 단위로도 볼 수 있다.  
DBMS는 동시에 여러 사용자의 요청을 처리하게 되는데, 데이터베이스는 다수의 사용자가 DB에 접근하여 연산을 수행하더라도 항상 모순이 없는 정확한 데이터를 유지해야 한다. 또한 데이터베이스에 장애가 발생하더라도 빠른 시간 내에 기존 상태로 복구할 수 있어야 한다. DBMS는 DB가 항상 정확하고 일관된 상태를 유지할 수 있도록 관리하는데, Transaction 관리를 통해서 DB의 `회복`과 `동시성 제어`가 가능하기 때문에 결과적으로 DB가 일관된 상태를 유지할 수 있게 된다.

## Transaction의 특성

Transaction은 데이터베이스 시스템에서 매우 중요한 개념으로, 아래 네 가지 특성을 만족해야 한다. 각 특성의 앞 글자를 따서 `ACID 특성`이라고도 한다.

### Atomicity(원자성)

**`원자성(Atomicity)`은 한 트랜잭션 내의 모든 연산들이 완전히 수행되거나 전혀 수행되지 않음(all or nothing)을 의미**한다. 즉 트랜잭션 중간에 문제가 발생하는 경우 트랜잭션 내의 어떠한 작업 내용도 수행되어서는 안되며(nothing), 아무런 문제 없이 작업이 완료되었을 때에만 트랜잭션의 모든 작업 결과가 데이터베이스에 반영(all)되어야 한다.

### Consistency(일관성)

**한 트랜잭션을 정확하게 수행하고 나면 데이터베이스가 하나의 일관된 상태에서 다른 일관된 상태로 변하는데 이를 `일관성(Consistency)`이라고 한다.** 즉, 일관된 상태의 데이터베이스는 트랜잭션이 완료된 후에도 일관성을 유지할 수 있어야 한다. (테이블의 스키마가 변경되는 등의 문제가 발생해서는 안 된다)

### Isolation(고립성)

**`고립성(Isolation)`이란, 한 트랜잭션이 데이터를 갱신하는 동안 이 트랜잭션이 완료되기 전에는 갱신 중인 데이터를 다른 트랜잭션들이 접근하지 못하도록 하는 것**을 말한다. 고립성은 다수의 트랜잭션이 동시에 수행되는 것을 다루며, 각각의 트랜잭션은 독립적으로 수행되어야 한다는 것을 의미한다.

### Durability(지속성)

**`지속성(Durability)`은 한 트랜잭션이 완료되면 이 트랜잭션이 갱신한 데이터는 시스템 고장이 발생하더라도 손실되지 않는**다는 특징이다. 즉, 완료된 트랜잭션의 결과는 영구적으로 데이터베이스에 저장되어야 한다는 것이다.

<br>

## Concurrency Control(동시성 제어)

다수의 사용자가 동일한 데이터에 접근하여 트랜잭션을 수행하는 경우, 변경 중인 데이터를 다른 트랜잭션이 사용하게 되면 데이터의 일관성이 훼손될 수 있다. **`동시성 제어(Concurrency Control)` 기법은 다수의 사용자들의 트랜잭션들을 동시에 수행하는 환경에서 부정확한 결과를 생성할 수 있는, 트랜잭션들 간의 간섭이 생기지 않도록 제어하는 것**을 말한다.

각 트랜잭션은 데이터베이스의 일관성을 유지하므로 여러 트랜잭션들의 집합을 한 번에 한 트랜잭션씩 차례대로 수행하는 `직렬 스케줄(Serial Schedule)`에서는 데이터베이스의 일관성이 유지된다. 여러 트랜잭션들을 동시에 수행하는 `비직렬 스케줄(Non-Serial Schedule)`의 결과가 어떤 직렬 스케줄의 수행 결과와 동등하다면 `직렬 가능(Serializable)`하다고 표현한다.

만약 DBMS가 동시성 제어를 하지 않고 다수의 트랜잭션을 동시에 수행한다면 `Lost Update(갱신 손실)`, `Dirty Read`, `Unrepeatable Read` 등의 문제가 발생할 수 있다.

#### Lost Update(갱신 손실)

**`Lost update(갱신 손실)`은 수행 중인 트랜잭션이 갱신한 내용을 다른 트랜잭션이 덮어 씀으로써 갱신이 무효가 되는 것**을 말한다.

#### Dirty Data

**`Dirty Data`는 완료되지 않은 트랜잭션이 갱신한 데이터**이다. `Dirty Read`는 한 트랜잭션이 Dirty Data를 읽어들이는 것을 말한다.

#### Unrepeatable Read

**`Unrepeatable Read`는 한 트랜잭션이 동일한 데이터를 두 번 읽을 때 서로 다른 값을 읽는 것**을 말한다.

<br>

### 데이터베이스 연산

사용자 프로그램에서는 데이터베이스로부터 검색한 데잍터에 대해 여러 가지 연산을 수행할 수 있지만, DBMS는 `읽기(read)`와 `쓰기(write)` 연산에만 관심을 갖는다.

#### Input(X) / Output(X)

데이터베이스 항목을 포함하고 있는 디스크 블록을 주기억 장치와 디스크 간에 이동하는 연산은 `Input(X)`와 `Output(X)`이다. Input(X) 연산은 데이터베이스 항목 X를 포함하고 있는 블록을 주기억 장치의 `버퍼(Buffer)`로 읽어들인다. Output(X) 연산은 데이터베이스 항목 X를 포함하고 있는 블록을 디스크에 기록한다.

#### read_item(X) / write_item(X)

주기억 장치의 버퍼와 응용 프로그램 간에 데이터베이스 항목을 이동하는 연산은 `read_item(X)`와 `write_item(X)`이다. read_item(X) 연산은 주기억 장치 버퍼에서 데이터베이스 항목 X의 값을 프로그램 변수 X로 복사한다. write_item(X) 연산은 프로그램 변수 X의 값을 주기억 장치 내의 데이터베이스 항목 X에 기록한다. read_item(X)와 write_item(X)에 Input(X) 연산이 모두 필요하다.

<br>

## Locking

`Locking`은 동시에 수행되는 트랜잭션들의 동시성을 제어하기 위해 가장 널리 사용되는 기법으로, **한 트랜잭션에서 사용 중인 데이터베이스 내 데이터 항목에 대해 다른 트랜잭션의 접근을 제한하는 것**을 말한다. 일반적으로 데이터베이스 내의 모든 데이터 항목마다 Lock이 존재하며, 각 트랜잭션이 데이터 항목에 접근할 때마다 요청한 Lock에 관한 정보는 `Lock Table` 등에 유지된다.

트랜잭션에서 갱신을 목적으로 데이터 항목에 접근할 때는 `독점 로크(X-lock, eXclusive lock)`를 요청한다. 반면에 트랜잭션에서 읽기만 할 목적으로 데이터 항목에 접근할 때는 `공유 로크(S-lock, Shared lock)`을 요청한다. 한 트랜잭션에서 어떤 데이터에 Shared Lock을 걸어 놓은 경우에, 또 다른 트랜잭션이 그 데이터 항목을 읽으려고 Shared Lock을 요청한다면 이를 함께 허용해도 무방하다. 그러나 Shared Lock이 걸려 있는 데이터 항목에 대해 X-Lock을 요청하거나, X-Lock이 걸려 있는 데이터 항목에 대해 X-Lock이나 Shared Lock을 요청하는 경우에는 이를 허용해서는 안 된다. Lock은 트랜잭션이 데이터 항목에 대한 접근을 끝낸 후에 `해제(Unlock)`한다.

### Lock 양립성 행렬

|                    |  현재 lock ->  | Shared Lock | eXclusive Lock | No Locking |
| :----------------: | :------------: | :---------: | :------------: | :--------: |
| **요청 중인 Lock** |  Shared Lock   |    허용     |      대기      |    허용    |
| **요청 중인 Lock** | eXclusive Lock |    대기     |      대기      |    허용    |

<br>

### Two-Phase Locking Protocol(2PL, 2단계 라킹 프로토콜)

**`Two-Phase Locking Protocol`은 트랜잭션이 진행되는 동안 Lock과 Unlock 연산을 `Growing Phase(확장 단계)`와 `Shrinking Phase(축소 단계)`로 구분하여 수행하는 기법**을 말한다.

#### Growing Phase(확장 단계)

`Growing Phase`에서는 트랜잭션이 데이터 항목에 대해 새로운 Lock을 요청할 수 있지만 Unlock 연산은 수행할 수 없다. 즉, **Lock 연산만 수행이 가능**하다.

#### Shrinking Phase(축소 단계)

`Shrinking Phase`에서는 Unlock 연산은 수행할 수 있지만 새로운 Lock을 요청할 수는 없다. 즉, **Unlock 연산만 수행 가능**하다. Shrinking Phase에서는 Unlock을 조금씩 수행할 수도 있고, 한꺼번에 수행할 수도 있는데 일반적으로 한꺼번에 Lock을 해제하는 방식이 사용된다.

#### Lock Point

**`Lock Point`는 한 트랜잭션에서 필요로 하는 모든 Lock을 걸어놓은 시점**을 말한다. 따라서 Lock Point를 전후로 하여 Growing Phase와 Shrinking Phase로 나뉜다.

<br>

이러한 특성 때문에 2PL을 따른다면 트랜잭션이 진행되는 도중 단 한 번의 Unlock이라도 수행하게 되면 Shrinking Phase로 들어가고, Lock 연산을 수행할 수 없다. 이렇게 Lock과 Unlock 연산을 분리하여 진행하는 이유는 `Consistency`가 위배되는 상황을 방지하기 위해서이다. Loking을 사용하더라도 동시성 제어 문제가 완벽하게 해결되지는 않기 때문에 2PL 기법으로 동시성 제어를 하여 DB의 Consistency를 보장한다. 아래는 2PL을 따르지 않고 두 개의 프로토콜을 진행했을 때 일관성이 위배되는 상황에 대한 예시이다.

> T1과 T2는 각각 하나의 트랜지션이다. T1에서는 데이터 항목 A와 B에 각각 3을 더한다. 트랜잭션이 수행되기 전에 A = B를 만족하고 있었다면 수행 후에도 A = B를 만족해야 한다. T2는 데이터 항목 A와 B에 각각 3을 곱한다. 마찬가지로 트랜잭션 수행 전에 A = B를 만족하고 있었다면 수행 후에도 A = B를 만족해야 한다. 따라서 T1과 T2는 어느 것을 먼저 수행하든 최종 결과는 A = B를 만족해야 한다. 그러나 아래 스케줄에 따라 두 트랜잭션을 수행하면 동시성 제어를 위해 Locking을 사용했더라도 A ≠ B가 된다. 그 이유는 Unlock을 너무 일찍 수행했기 때문이다.

| T1                                                                                                                                              | T2                                                                                                                                                                                                                                                                                                      |
| ----------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| <span style="color:red">X-lock(A);</span> <br> read_item(A); <br> A = A + 3; <br> write_item(A); <br> <span style="color:red">unlock(A);</span> |                                                                                                                                                                                                                                                                                                         |
|                                                                                                                                                 | <span style="color:red">X-lock(A);</span> <br> read\*item(A); <br> A = A \* 3; <br> write_item(A); <br> <span style="color:red">unlock(A);</span> <br> <span style="color:red">X-lock(B);</span> <br> read_item(B); <br> B = B \* 3; <br> write_item(B); <br> <span style="color:red">unlock(B);</span> |
| <span style="color:red">X-lock(B);</span> <br> read_item(B); <br> B = B+3; <br> write_item(B); <br> <span style="color:red">unlock(B)</span>;   |                                                                                                                                                                                                                                                                                                         |

<br>

### Deadlock(교착 상태)

**`Deadlock(교착 상태)`은 두 개 이상의 트랜잭션들이 서로 상대방이 보유하고 있는 Lock을 요청하면서 기다리고 있는 상태**를 말한다. 이런 경우 각 트랜잭션은 2PL의 Growing Phase에 있으므로 Unlock을 수행할 수 없어 서로 필요로 하는 데이터 항목에 대한 Lock을 점유한 채로 무한 대기 상태에 빠지게 된다. 일반적인 DBMS는 Deadlock을 독자적으로 검출하여 보고한다. 아래는 Deadlock에 대한 간단한 상황 예시이다.

> T1과 T2는 데이터 항목 A와 B에 대해 각각 Lock을 요청해 가는 과정에서 Deadlock이 발생하는 경우를 생각해 볼 수 있다. (2PL의 예시에서 사용했던 상황을 대입해도 무방)
> ① T1이 A에 대해 X-Lock을 요청하여 허가 받음.
> ② T2가 B에 대해 X-Lock을 요청하여 허가 받음.
> ③ T1이 B에 대해 S-Lock이나 X-Lock을 요청하면 T2에서 Unlock을 수행할 때까지 대기하게 됨.
> ④ T2가 A에 대해 S-Lock이나 X-Lock을 요청하면 T1에서 Unlock을 수행할 때까지 대기하게 됨.

#### Deadlock 빈도를 낮추는 방법

- Transaction을 자주 commit 한다.
- 정해진 순서로 테이블에 접근한다. 위의 예시에서 T1은 A → B 순으로 접근했고, T2는 B → A 순으로 접근하여 문제가 발생하였으므로, 트랜잭션들이 데이터 항목에 접근할 때 동일한 순서로 접근하도록 한다.
- Shared Lock 사용을 피한다.
- 다수의 트랜잭션이 동일한 테이블 내의 여러 tuple들을 갱신하는 경우 Deadlock이 발생하기 쉽다. 이런 경우 테이블 단위의 Lock을 요청하여 작업을 직렬화 하면 동시성은 떨어지지만 Deadlock은 피할 수 있다. (`Multiple Granularity`)

<br>

### Multiple Granularity(다중 Lock 단위)

트랜잭션들이 많은 tuple에 접근하는 경우에 tuple 단위로만 Lock을 걸면 Lock Table에서 Lock 충돌을 검사하고, Lock 정보를 기록하는 시간이 오래 걸린다. 따라서 트랜잭션이 접근하는 tupel 수에 따라 Lock 하는 데이터 항목의 단위를 구분하는 것이 필요하다.  
**한 트랜잭션에서 Lock 할 수 있는 데이터 항목이 두 가지 이상 있으면 이를 `Multiple Granularity(다중 Lock 단위)`라고 한다.** 데이터베이스에서 Lock 할 수 있는 단위로는 `데이터베이스`, `릴레이션`, `디스크 블록`, `튜플` 등이 있다. 일반적으로 DBMS는 각 트랜잭션에서 접근하는 tuple 수에 따라 자동적으로 Lock 단위를 조정한다. Lock 단위는 작을수록 Locking에 따른 오버헤드가 증가하지만, 동시성의 정도는 증가한다.

### Phantom Problem

**한 트랜잭션에서 같은 query를 두 번 수행할 때, 그 사이에 다른 트랜잭션에서 새로운 데이터를 삽입(insert)하는 경우 동일한 query임에도 상이한 결과가 나타나게 되는데 이러한 현상을 `Phantom Problem` 또는 `Phantom Read`라고 한다.** 각각의 트랜잭션이 tuple 단위로 Lock을 걸어 연산을 수행하는 경우, 새로 추가되는 tuple에 대해서는 Lock이 걸려있지 않기 때문에 이러한 현상이 발생하게 되는 것이다.

<br>

## Recovery(회복)

어떤 트랜잭션을 수행하는 도중에 시스템이 다운된다면 트랜잭션 내 연산의 결과가 일부만 데이터베이스에 반영될 수 있다. 이럴 경우 이 트랜잭션의 수행 결과를 취소하여 Atomicity를 보장해야 한다. 또한, 트랜잭션이 종료된 직후에 시스템이 다운되는 경우에는 그 수행 결과를 주기억 장치로부터 데이터베이스에 기록하지 못할 수도 있다. 이런 경우 Durability를 보장하기 위해서는 추후 그 수행 결과를 완전하게 데이터베이스에 반영할 수 있어야 한다.  
**DBMS의 `Recovery Module`은 트랜잭션 수행 과정에서 비정상적으로 중단되는 상황이 발생했을 때 트랜잭션의 Atomicity와 Durability를 보장하는 기능**을 말한다.

트랜잭션이 버퍼에는 갱신 사항을 반영했으나 버퍼의 내용을 디스크에 기록하기 전에 고장이 발생했다면 트랜잭션이 완료 명령을 수행한 경우와 그렇지 못한 경우 두 가지 상황으로 나눌 수 있다. 트랜잭션이 완료 명령을 수행했다면 DBMS의 회복 모듈은 이 트랜잭션의 갱신 사항을 `REDO`하여 트랜잭션의 갱신이 `Durability`을 갖도록 해야 한다. 그러나 완료 명령을 수행하지 못했다면 `Atomicity`을 보장하기 위해서 이 트랜잭션이 데이터베이스에 반영했을 가능성이 있는 갱신 사항을 `UNDO`해야 한다.

DBMS는 데이터베이스의 회복을 위해 `Backup`, `Logging`, `Checkpoint` 등을 수행한다. `Backup`은 데이터베이스를 주기적으로 자기 테이프 등에 복사하는 것이다. `Logging`은 현재 수행 중인 트랜잭션의 상태와 데이터베이스의 갱신 사항을 기록하며, `Checkpoint`는 시스템이 붕괴된 후 재기동되었을 때 재수행하거나 취소해야 하는 트랜잭션들의 수를 줄여준다.

<br>

### Log를 사용한 즉시 갱신

거의 모든 데이터베이스는 `Log`를 기반으로 한 즉시 갱신 방법을 사용한다. DBMS는 고장 상황 발생 시, 트랜잭션의 Atomicity와 Durability를 보장하기 위해 Log라는 추가적인 정보를 유지한다. **`Log`는 데이터 갱신 내용을 시간에 따라 기록한 것**이다. 따라서 Log를 이용하면 고장 상황 이후 DB에서 완료된 트랜잭션의 수행 결과를 취소하거나, 철회된 트랜잭션의 수행 결과를 DB에 반영할 수 있다.

Log는 DB와의 동시 손상을 피하기 위해 일반적으로 전용 디스크에 저장되며, DB 항목에 영향을 미치는 모든 트랜잭션의 연산들에 대해서 `Log record`를 기록한다. 각 Log record는 `Log Sequence Number(LSN, 로그 순서 번호)`로 식별된다.

각 Log record가 어떤 트랜잭션에 속한 것인지 식별하기 위해서 각 Log record마다 트랜잭션 ID를 포함시킨다. 트랜잭션이 생성될 때마다 고유한 번호가 부여되는데 이를 트랜잭션 ID라고 한다. 동일한 트랜잭션에 속하는 Log record들은 Linked List로 유지되며, Log와 관련된 모든 작업은 사용자에게 투명하게 DBMS에서 이루어진다.

#### 흔히 사용되는 로그 레코드 유형

- [Trans-ID, start] : **한 트랜잭션이 생설될 때 기록**되는 로그 레코드
- [Trans-ID, X, old_value, new_value] : **주어진 Trans-ID를 갖는 트랜잭션이 데이터 항목 X를 이전 값(old_value)에서 새 값(new_value)으로 수정했음**을 나타내는 로그 레코드. 이전 값은 트랜잭션을 UNDO할 때 사용하고, 새 값은 트랜잭션을 REDO할 때 사용한다.
- [Trans-ID, commit] : **주어진 Trans-ID를 갖는 트랜잭션이 데이터베이스에 대한 갱신을 모두 성공적으로 완료하였음**을 나타내는 로그 레코드. 어떤 트랜잭션에 대해 디스크 로그에 이런 로그 레코드가 기록되어 있으면 그 트랜잭션의 갱신 사항은 데이터베이스에 영구적으로 반영될 수 있다.
- [Trans-ID, abort] : **주어진 Trans-ID를 갖는 트랜잭션이 철회되었음**을 나타내는 로그 레코드. 어떤 트랜잭션에 대해 디스크 로그에 이런 로그 레코드가 기록되어 있으면 그 트랜잭션의 갱신 사항을 데이터베이스에서 취소해야 한다.

한 트랜잭션의 DB 갱신 연산이 모두 끝나고 DB 갱신 사항이 로그에 기록되었을 때 그 트랜잭션이 `Commit point(완료점)`에 도달한다고 말한다. DBMS의 Recovery Module은 로그를 검사하여 로그에 [Trans-ID, start] 로그 레코드와 [Trans-ID, commit] 로그 레코드가 모두 존재하는 트랜잭션들은 재수행(REDO)한다. 반면에 [Trans-ID, start] 로그 레코드는 존재하나 [Trans-ID, commit] 로그 레코드는 존재하지 않는 트랜잭션들은 모두 취소(UNDO)한다. 트랜잭션의 재수행은 로그가 기록되는 방향으로 진행되고, 트랜잭션의 취소는 로그를 역방향으로 따라가면서 진행된다.

<br>

### Write-Ahead Logging(WAL, 로그 먼저 쓰기)

트랜잭션이 DB를 갱신하면 주기억 장치의 DB 버퍼에 갱신 사항을 기록하고, 로그 버퍼에는 이에 대응되는 로그 레코드를 기록한다. 그리고 이 두 버퍼를 모두 디스크에 기록해야 하는데 이 둘을 동시에 기록할 수는 없기 때문에 로그 버퍼를 먼저 기록한다. 이처럼 **데이터베이스 버퍼보다 로그 버퍼를 먼저 디스크에 기록하는 것을 `Write-Ahead Logging`이라고 한다.** 어떤 트랜잭션을 취소하려면 그 트랜잭션이 갱신한 데이터베이스 항목의 이전 값을 알아야 한다. 그런데 DB 갱신 사항을 먼저 기록하고 이후 로그 레코드가 디스크에 기록되기 전에 시스템이 다운되었다면, 로그 레코드가 없어 이전 값을 알 수 없으므로 트랜잭션 취소가 불가능해진다. 따라서 데이터베이스 버퍼보다 로그 버퍼를 디스크에 먼저 기록해야 한다.

### Checkpoint

DBMS가 로그를 사용하더라도 어떤 트랜잭션의 갱신 사항이 주기억 장치 버퍼로부터 디스크에 기록되었는가를 구분할 수는 없다. 즉, 추가 정보 없이 로그 검사만으로는 주기억 장치의 버퍼가 디스크에 기록되었는가를 식별할 수 없다. 따라서 DBMS는 회복 시 재수행할 트랜잭션의 수를 줄이기 위해서 주기적으로 `Checkpoint`를 수행한다. **Checkpoint 시점에는 주기억 장치의 버퍼 내용이 디스크에 강제로 기록되므로, Checkpoint를 수행하면 디스크 상에서 로그와 DB의 내용이 일치하게 된다. 이를 식별하기 위해 Checkpoint 작업이 끝나면 로그에 [checkpoint] 로그 레코드를 기록한다.**

따라서 시스템 다운 후 재기동되었을 때 DBMS Recovery module은 [checkpoint] 로그 레코드를 찾는다. [checkpoint] 이전에 시작되었더라도 완료되지 않은 트랜잭션들은 모두 취소해야 하지만, [checkpoint] 로그 레코드 이전에 완료된 트랜잭션들은 재수행할 필요가 없다. [checkpoint] 이후에 완료된 모든 트랜잭션들은 재수행한다. Checkpoint는 적절한 시간 간격을 두고 수행된다.

#### Checkpoint 시 수행되는 작업

1. 수행 중인 트랜잭션을 일시적으로 중지. (이 작업은 회복 알고리즘에 따라 필요하지 않을 수 있음)
2. 주기억 장치의 로그 버퍼를 디스크에 강제로 출력.
3. 주기억 장치의 데이터베이스 버퍼를 디스크에 강제로 출력.
4. [checkpoint] 로그 레코드를 로그 버퍼에 기록한 후 디스크에 강제로 출력. Checkpoint 시점에 수행 중이던 트랜잭션들의 ID도 [checkpoint] 로그 레코드에 함께 기록.
5. 일시적으로 중지된 트랜잭션 수행 재개.

<br>

### Isolation Level(고립 수준)

2PL을 엄격하게 적용할 때 생성되는 Serializable한 스케줄은 한 트랜잭션씩 차례대로 수행한 결과와 동등하지만, 동시성은 떨어지기 때문에 성능 저하를 유발할 수 있다. `Isolation Level`은 한 트랜잭션이 다른 트랜잭션과 고립되어야 하는 정도를 나타낸다. Isolation level이 낮으면 동시성은 높아지지만 데이터의 정확성은 떨어지고, 반대로 Isolation level이 높으면 데이터의 정확성은 높아지지만 동시성이 저하된다. 그러므로 프로그램의 성격에 따라 허용 가능한 Isolation level(DB의 정확성)을 선택함으로써 성능을 향상시킬 수 있다. DBMS가 사용하는 Locking 동작은 Isolation level에 따라 달라진다.

**1. READ UNCOMMITTED (Level 0)**

트랜잭션 내의 query들이 S-Lock을 걸지 않고 데이터를 읽는 가장 낮은 isolation level. 다른 트랜잭션의 갱신 내용이 Commit이든 Rollback이든 관계 없이 보여지기 때문에 Dirty Read를 유발한다. 갱신하려는 데이터에 대해서는 X-Lock을 걸고, 트랜잭션이 끝날 때까지 보유한다.

```SQL
    SET TRANSACTION READ WRITE
        ISOLATION LEVEL READ UNCOMMITTED;
```

**2. READ COMMITTED (Level 1)**

트랜잭션 내 query들이 읽으려는 데이터에 대해 S-Lock을 걸고, 읽기가 끝나마자마 Unlock한다. 따라서 동일한 데이터를 다시 읽을 경우, 이전에 읽은 값과 다른 값을 읽는 경우가 발생할 수 있다. (`Repeatable Read`에서 Consistency가 보장되지 않음) READ COMMITTED에서는 Commit이 수행된 데이터에만 접근이 가능하고 Commit이 되지 않은 데이터에는 UNDO 영역에 백업된 데이터를 참조하기 때문에 Dirty Read는 발생하지 않는다. 갱신하려는 데이터에 대해서는 X-Lock을 걸고 트랜잭션이 끝날 때까지 보유한다. READ COMMITTED는 RDB에서 가장 많이 사용하는 `Defaul Isolation Level`이다.

```SQL
    SET TRANSACTION READ WRITE
        ISOLATION LEVEL READ COMMITTED;
```

**3. REPEATABLE READ (Level 2)**

트랜잭션 내 query에서 검색하는 모든 데이터들에 대해 S-Lock을 걸고 트랜잭션이 끝날 때까지 보유한다. 한 트랜잭션 내에서 동일한 query를 두 번 이상 수행할 때에는 매번 같은 값을 포함한 결과를 검색하게 된다. 갱신하려는 데이터에 대해서는 X-Lock을 걸고 트랜잭션이 끝날 때까지 보유한다.

```SQL
    SET TRANSACTION READ WRITE
        ISOLATION LEVEL REPEATABLE READ;
```

**4. SERIALIZABLE (Level 3)**

query에서 검색되는 tuple들 뿐만 아니라 인덱스에 대해서도 S-Lock을 걸고 트랜잭션이 끝날 때까지 보유하는 가장 높은 isolation level. 인덱스에 S-lock을 걸어두기 때문에 Phantom problem도 방지한다. 갱신하려는 데이터에 대해서는 X-Lock을 걸고 트랜잭션이 끝날 때까지 보유한다. 일반적으로 데이터베이스에서는 사용하지 않는다.

```SQL
    SET TRANSACTION READ WRITE
        ISOLATION LEVEL SERIALIZABLE;
```

#### 고립 수준에 따른 동시성 문제

| Isolation Level  | Dirty Read | Unrepeatable Read | Phantom Read |
| :--------------: | :--------: | :---------------: | :----------: |
| READ UNCOMMITTED |     Y      |         Y         |      Y       |
|  READ COMMITTED  |     N      |         Y         |      Y       |
| REPEATABLE READ  |     N      |         N         |      Y       |
|   SERIALIZABLE   |     N      |         N         |      N       |

<br>
<hr>
