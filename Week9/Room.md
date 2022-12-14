# Room
## 개요
안드로이드 앱은 SQLite 데이터베이스를 사용하여 복잡하거나 양이 많은 자료를 체계적으로 관리할 수 있다. 하지만 프레임워크에서 제공하는 API만 사용하여 개발하면 아래와 같은 문제가 발생한다.

1.	SQL 쿼리문을 수행하는 코드를 작성하는 경우, 쿼리문 중간에 데이터를 끼워넣는 과정에서 오타 등의 실수가 발생해도 컴파일 시점에서는 발견할 수 없고 쿼리문을 실제로 실행해야만 정상적으로 동작하는지 확인할 수 있다. 따라서 오류를 발견하고 수정하기까지 시간이 많이 걸릴 수 있으며, 운이 안 좋으면 애플리케이션이 출시되기 전까지 발견하지 못하기도 한다.
2.	SQL 쿼리로 얻은 결과를 코드에서 쉽게 사용할 수 있도록 객체 형태로 변환하기 위해 작성해야 하는 코드는 양이 많고 복잡하다.
3.	메인 스레드에서 별다른 제약 없이 데이터베이스의 자료를 수정하거나 저장된 자료를 불러올 수 있다. 이 때문에, 메인 스레드에서 데이터베이스 관련 작업을 오랫동안 수행하면 UI 업데이트가 지연되어 사용자 경험에 좋지 않은 영향을 주며, 최악의 경우 ANR(Application Not Responding) 상태에 빠질 수 있다.   

이러한 문제들을 해결하기 위해, 안드로이드 아키텍처 컴포넌트를 룸(Room) 라이브러리를 제공한다.

## 소개
Room 라이브러리는 안드로이드 앱에서 SQLite 데이터베이스를 쉽고 편리하게 사용할 수 있도록 하는 기능을 제공한다. 특히, 데이터베이스 내에서 다루는 자료를 객체 형태로 변환하는 수고를 하지 않아도 된다.   
안드로이드 프레임워크에서 제공하는 데이터베이스 관련 API는 단순히 SQLite 데이터베이스를 조작할 수 있는 인터페이스만 제공한다. 하지만 Room 라이브러리는 데이터베이스를 더 체계적으로 사용할 수 있도록 관련 기능을 Room 데이터베이스(Room Database), 데이터 접근 객체(Data Access Object), 엔티티(Entity) 총 세 개의 구성요소로 나누어 제공한다.

![image](https://user-images.githubusercontent.com/50148363/202990416-2dbd1236-b11f-408f-abb5-060879b38355.png)

## Room Database
Room Database는 데이터베이스를 생성하거나 버전을 관리하는 등 실제 데이터베이스 파일과 밀접한 작업을 담당한다. 또한, 어노테이션(@)을 통해 데이터베이스 파일을 사용할 데이터 접근 객체를 정의하여 데이터 접근 객체와 데이터베이스 파일을 연결하는 역할도 수행한다. 필요에 따라 여러 개의 Room Database를 정의할 수 있으므로, 각각 다른 데이터베이스 파일에 연결된 데이터 접근 객체들을 선언하는 것도 가능하다.   
Room Database를 정의하는 클래스는 반드시 RoomDatabase를 상속한 추상 클래스여야 하며, 클래스의 멤버 함수 형태로 데이터베이스와 연결할 데이터 접근 객체를 선언한다. 또한, @Database 어노테이션을 사용하여 데이터베이스에서 사용할 엔티티와 데이터베이스의 버전을 지정한다.

* Room Database 클래스를 정의한 예시
``` kotlin
// RoomDatabase를 상속하는 추상 클래스로 룸 데이터베이스 클래스를 선언한다.
// @Database 어노테이션으로 룸 데이터베이스의 속성을 지정한다.
// entities에 데이터베이스에서 사용할 엔티티의 클래스를 배열 형태로 넣어주며, 
// version에 데이터베이스의 버전을 넣어준다.
@Database(entities = [Memo::class], version = 1)
abstract class AppDataBase: RoomDatabase() {
    // 메모 정보에 접근하는 MemoDao 데이터 접근 객체를
    // Room Database인 AppDataBase와 연결한다.
    abstract fun memoDao(): MemoDao
}
```

데이터를 실제로 다루는 역할을 하는 Data Access Object(데이터 접근 객체)를 얻으려면 Room Database의 인스턴스가 필요하다.   
Room Database는 추상 클래스로 정의되므로, Room Database 구현체의 인스턴스를 얻으려면 Room.databaseBuilder() 함수를 사용해야 한다.

* Room.databaseBuilder() 함수를 사용하여 AppDataBase의 인스턴스를 생성하는 예시
``` kotlin
// AppDataBase의 인스턴스를 생성한다.
// 컨텍스트와 생성할 Room Database의 클래스, 그리고 생성될 데이터베이스 파일의 이름을 지정한다.
appDataBaseInstance = Room.databaseBuilder(
    appInstance.applicationContext,
    AppDataBase::class.java, "exampleApp.db"
)
    .fallbackToDestructiveMigration() // DB version 달라졌을 경우 데이터베이스 초기화
    .allowMainThreadQueries() // 메인 스레드에서 접근 허용
    .build()    
```

Room Database의 인스턴스를 한번 생성한 후에는 다른 곳에서도 계속 사용할 수 있도록 생성한 인스턴스를 계속 유지하는 것이 좋다. 그러므로 싱글톤 패턴 혹은 유사한 방법을 사용하여 앱 내에서 인스턴스를 공유하도록 구현하는 것이 좋다.
> Room.inMemoryDatabaseBuilder()를 사용하면 파일 형태로 저장되는 데이터베이스 대신 메모리에 데이터베이스를 저장하는 Room Database 인스턴스를 생성할 수 있다. 여기에 저장되는 데이터베이스는 애플리케이션 프로세스가 종료되는 즉시 사라진다.

## Data Access Object
Data Access Object(데이터 접근 객체)는 데이터베이스를 통해 수행할 작업을 정의한 클래스이다.   
데이터 삽입, 수정, 삭제 작업이나 저장된 데이터를 불러오는 작업 등을 함수 형태로 정의하며, 앱의 비즈니스 로직에 맞는 형태로 작업을 정의할 수 있다.
Data Access Object는 인터페이스나 추상 클래스로 정의할 수 있으며, 반드시 @Dao 어노테이션을 붙여 주어야 한다.   
@Query 어노테이션을 사용하면 Data Access Object 내에 정의된 함수를 호출했을 때 수행할 SQL 쿼리문을 작성할 수 있다. @Query 어노테이션을 사용한 함수는 어노테이션에 작성된 SQL 쿼리문에 함수의 매개변수를 결합할 수 있다. 검색 결과를 반환하는 쿼리문은 @Query 어노테이션을 사용한 함수의 반환 타입에 맞게 결과가 변환되어 출력된다.   
@Query 어노테이션 내에서 사용할 수 있는 SQL문은 INSERT, UPDATE, DELETE로 제한된다.   
데이터를 조작하는 일부 SQL문은 @Query 어노테이션 대신 특화된 어노테이션을 사용할 수 있다. @Insert, @Update, @Delete 어노테이션을 사용할 수 있다.

* @Query 어노테이션과 특화된 어노테이션을 사용하여 Data Access Object에 작업을 정의한 예시
``` kotlin
@Dao
interface MemoDao {
    // Memo 테이블의 모든 데이터를 반환한다.
    @Query("SELECT * FROM Memo")
    suspend fun getAllMemo(): List<Memo>

    @Insert // Long을 return하면 해당 memo의 id를 알 수 있다.
    suspend fun insertMemo(memo: Memo): Long

    @Delete
    suspend fun deleteMemo(memo: Memo)

    // Memo 테이블의 id와 일치하는 id를 가진 데이터를 삭제한다.
    @Query("DELETE FROM Memo Where id = :id")
    suspend fun deleteMemoByID(id: Long)

    // Memo 테이블에서 id 필드의 값이 'id'인 모든 레코드의 memo 값을 memo 변경한다.
    @Query("UPDATE Memo SET memo = :memo WHERE id = :id")
    suspend fun modifyMemo(id: Long, memo: String)
}
```

## Entity
Entity(엔티티)는 데이터베이스에 저장할 데이터의 형식을 정의하며, Entity가 하나의 테이블을 구성한다. 이 때문에 룸 데이터베이스를 정의할 때 해당 데이터베이스에서 사용하는 Entity를 @Database 어노테이션 내에 반드시 지정해 주어야 한다. 룸 데이터베이스에서 지정하지 않은 Entity를 사용하면 컴파일 에러가 발생한다.   
Entity는 @Entity 어노테이션을 사용하여 정의한다. 데이터베이스에 저장할 정보는 필드로 표현하며, public 수준의 가시성을 갖거나 Getter/Setter를 사용하여 필드에 접근할 수 있어야 한다. 코틀린은 필드와 Getter/Setter 대신 프로퍼티를 사용하여 데이터 베이스에 저장할 정보를 정의할 수 있다.   
Entity는 최소한 하나의 주요키(Primary Key)를 지정해야 한다. @PrimaryKey 어노테이션을 사용하면 Entity에서 사용할 주요 키를 지정할 수 있다. 만약 여러 필드를 주요 키로 사용하고 싶다면 @Entity 어노테이션에서 주요 키로 사용할 필드의 이름을 지정할 수 있다. 클래스에 포함된 필드 중 데이터베이스에 저장하고 싶지 않은 필드가 있을 경우 @Ignore 어노테이션을 필드에 추가하면 된다.

* Entity를 정의한 클래스의 예시
``` kotlin
// 메모장 정보를 표현하는 Entity를 정의한다.
// Entity 이름과 동일한 Memo 테이블이 생성된다.
@Entity
class Memo (
    // id를 주요 키로 사용한다.
    // id를 자동으로 지정
    @PrimaryKey(autoGenerate = true) var id: Long,
    var memo: String,
    var editMode: Boolean
)
```
각 Entity별로 생성되는 테이블 이름과 테이블 내의 열(column) 이름이 Entity 클래스 및 필드 이름과 동일하다. @Entity 어노테이션 내에서 tableName 속성을 사용하면 생성되는 테이블 이름을 변경할 수 있고, @ColumnInfo 어노테이션의 name 속성을 사용하면 각 필드의 데이터를 저장할 열 이름을 지정할 수 있다.

### 예시 프로젝트
[RoomMemo Project](https://github.com/Lsh15/RoomMemo)

### 참고
https://developer.android.com/training/data-storage/room?hl=ko   

