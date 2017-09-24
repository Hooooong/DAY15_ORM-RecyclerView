Android Programing
----------------------------------------------------
### 2017.09.22 9일차

#### 예제
____________________________________________________

- ORM 를 사용한 메모장 만들기 (준비중)

#### 공부정리
____________________________________________________

##### __ORM__

- ORM 이란?

  > 객체 관계 매핑(Object-relational mapping; ORM)은 데이터베이스와 객체 지향 프로그래밍 언어 간의 호환되지 않는 데이터를 변환하는 프로그래밍 기법이다. 객체 관계 매핑 라고도 부른다. 객체 지향 언어에서 사용할 수 있는 "가상" 객체 데이터베이스를 구축하는 방법이다.

  - ORM 이란 __O__ bject- __R__ elational __M__ appging 의 약자로 객체형 데이터 와 관계형 데이터 사이에 개념적으로 일치하지 않는 부분을 해결하기 위하여 이 둘 사이의 데이터를 매핑(Mapping)하는 것이다.

  - 객체형 데이터와 관계형 데이터의 각 속성들을 매핑할 경우 관계형 데이터를 객체형 데이터처럼 사용하는것이 가능하다.

  - 주로 `Annotaion` 을 통해 ORM 을 사용한다.

  - Android 의 ORM 은 `aBatis`, `ORMLite`, `greenDAO` 등이 있는데, 보편적으로 많이 사용하는 `ORMLite` 를 사용하였다.

- Annotaion 설정 (Table 생성 및 Column 속성 변경)

  - Table 생성은 `@DatabaseTable` 을 사용한다.

    - `@DatabaseTable(tableName="")` : Table 명을 바꿀 수 있다.

  - Column 생성은 `@DatabaseField` 을 사용한다.

    - `@DatabaseField(columnName = "title")` : Column 명을 바꿀 수 있다.

    - `@DatabaseField(id = true)` : Column 의 `PK` 속성을 설정한다. default 값은 false 이다.

    - `@DatabaseField(generatedId = true)` : Column 을 `PK`로 만들어주고 Sequence로 만들어준다. default 값은 false 이다.

    - `@DatabaseField(canBeNull = true)` : Column 의 `Not Null` 속성을 변경한다. default 값은 true 이다.

    - `@DatabaseField(foreign = true)` : Column 의 `FK` 속성을 설정한다. default 값은 false 이다.

  - 예제

    ```java
    /**
     * Annotation 을 통해 table 과 column 을 생성한다.
     *
     * @DatabaseTable(tableName="")
     * 으로 생성될 Table 이름을 설정한다.
     *
     * 쿼리를 작성할 경우 명령문과 Table 이름을 구별하기 위해서(대소문자를 가리기 위해)
     * tableName 을 소문자로 작성한다.
     */
    @DatabaseTable(tableName = "picnote")
    public class PicNote{
        // 식별자
        @DatabaseField(generatedId = true)
        private long id;
        // 제목
        @DatabaseField(columnName = "title", canBeNull = true)
        private String title;
        // 그림

        // Bitmap 같은 것들은 Bitmap 자체를 저장하지 않고, 경로를 저장한다.
        @DatabaseField
        private String bitmapPath;
        // 날짜
        @DatabaseField
        private long dateTime;

    }
    ```

- Query 실행 메소드

  - Table 생성

    - `DBHelper` 클래스에서는 `SQLiteOpenHelper` 대신 `OrmLiteSqliteOpenHelper` 을 상속받는다.

    - `TableUtils.createTable(connectionSource, PicNote.class)` : PicNote.class 에서 설정한 Annotation 을 반영하여 Table 을 생성한다.

    - 예제

      ```java
      public class DBHelper extends OrmLiteSqliteOpenHelper{
        // 생성자 생략..
        @Override
        public void onCreate(SQLiteDatabase database, ConnectionSource connectionSource) {
            // PicNote.class 를 참조해서 테이블을 생성한다.
            try {
                TableUtils.createTable(connectionSource, PicNote.class);
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
      }
      ```

  - CRUD 사용

    -  기본적으로 `Dao<Model 객체명, ID 타입> dao = dbHelper.getDao(PicNote.class)` 를 선언하여 dao 객체를 통해 DB 관련 쿼리를 실행한다. 모든 쿼리는 `PicNote.class`의 객체에 기반하여 실행한다.

    - `dao.create(picNote 객체)` : picNote 에 담겨있는 값들을 통해 `INSERT` 한다.

    - `dao.queryForAll()` : 모든 데이터를 검색한다. Return Type 은 List<PickNote> 이다.

    - `dao.dao.queryForId(id)` : id 값을 통해 데이터를 검색한다. Return Type 은 PikcNote 이다.

    - `dao.update(picNote 객체)` : picNote 에 담겨있는 값들을 통해 자동으로 id 값을 찾아 `UPDATE` 한다.

    - `dao.delete(picNote 객체)` : picNote 에 담겨있는 값들을 통해 자동으로 id 값을 찾아 `DELETE` 한다. 물론 `dao.deleteById(id)` 로 id 값을 통해 삭제할 수 있다.

- 참조 : [ORM이란](https://ko.wikipedia.org/wiki/%EA%B0%9D%EC%B2%B4_%EA%B4%80%EA%B3%84_%EB%A7%A4%ED%95%91), [ORMLite Annotaion](http://ormlite.com/javadoc/ormlite-core/doc-files/ormlite_2.html#Local-Annotations)

##### __RecyclerView__

- RecyclerView 이란?

  > Android Lollipop 버전에서 나온 RecyclerView 는 ListView 의 장/단점을 적용한 위젯이다. ListView 를 사용하는 것처럼 Adapter를 통해 RecyclerView에 Item을 보여준다.

  - ListView 의 단점인 getView 메소드를 보완한 View 의 재사용과 Holder 패턴을 기본적으로 적용하였다. (참조 : [ListView 설명](https://github.com/Hooooong/DAY12_ListView))

  - `LayoutManager` 을 통하여 Item 의 배치 방법을 다양하게 할 수 있다.

    1. `LinearLayoutManager`

        - 리사이클러 뷰에서 가장 많이 쓰이는 레이아웃으로 수평, 수직 스크롤을 제공하는 리스트를 만들 수 있다.

    2. `StaggeredGridLayoutManagerm`

        - 이 레이아웃을 통해 뷰마다 크기가 다른 레이아웃을 만들 수 있다.

    3. `GridLayoutManager`

        - 갤러리(GridView) 같은 격자형 리스트를 만들 수 있다.

- ListView vs RecyclerView

    - ListView 와 RecyclerView 의 가장 큰 차이점은 `View 재사용`,  `ViewHolder 패턴` 적용이다.

    - `LayoutManager` 를 통해 ListView 보다 더 유연하게 RecyclerView 를 만들 수 있다.

    - RecyclerView 는 ListView 에 비해 애니메이션을 좀 더 유연하게 적용시킬 수 있다.

    - ListView 는 Footer 와 Header 를 기본적으로 제공하지만, RecyclerView 는 직접 만들어야 하는 단점이 있다.

- 참조 : [RecyclerView](https://developer.android.com/training/material/lists-cards.html?hl=ko)
