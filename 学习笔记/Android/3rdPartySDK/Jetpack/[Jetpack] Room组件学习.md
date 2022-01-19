# [Jetpack] Room组件学习

## 简介
Room为SQLite提供一个抽象层，在充分利用SQLite的同时，实现更强大的数据库访问。

## build.gradle配置

```xml
dependencies {
  def room_version = "2.2.0-alpha02"

  implementation "androidx.room:room-runtime:$room_version"
  annotationProcessor "androidx.room:room-compiler:$room_version" // For Kotlin use kapt instead of annotationProcessor

  // optional - Kotlin Extensions and Coroutines support for Room
  implementation "androidx.room:room-ktx:$room_version"

  // optional - RxJava support for Room
  implementation "androidx.room:room-rxjava2:$room_version"

  // optional - Guava support for Room, including Optional and ListenableFuture
  implementation "androidx.room:room-guava:$room_version"

  // Test helpers
  testImplementation "androidx.room:room-testing:$room_version"
}
```
由于在公司使用2.2.0-alpha02版本，会无法下载依赖，因此使用2.1.0了。

更多配置请参考： [官网说明配置](https://developer.android.com/jetpack/androidx/releases/room)


## 基本使用

### 了解三个主要注解

1. @Database: 用于注解一个继承于RoomDatabase的抽象类，通过注解参数导入一系列的entities及其版本号，该类主要作用是创建数据库和创建Daos（data access objects，数据访问对象）。

2. @Entity: @Entity用来注解实体类, @Database通过entities属性引用被@Entity注解的类，并利用该类的所有字段作为表的列名来创建表。

3. @Dao: @Dao用来注解一个接口或者抽象方法，该类的作用是提供访问数据库的方法。在使用@Database注解的类中必须定一个不带参数的方法，这个方法返回使用@Dao注解的类.

Room这些注解组件与app的关系：
![Image](https://developer.android.com/images/training/data-storage/room_architecture.png)

### 基本使用Demo

1, 我们先用@Entity创建一个数据库的Table的item:

```java
@Entity(tableName = "UserDataTable") //默认情况下，Room使用类名作为数据库的表名,但你可以另起名字:"UserDataTable"为table的名字
public class UserTableEntity {
    @PrimaryKey(autoGenerate = true) // 单个主键设置为自增长
    private int id;

    @ColumnInfo(name = "user_name") // 默认使用变量名作为列名，但可以使用name属性来重命名；
    private String mUserName;

    @ColumnInfo(name = "age")
    private int mAge;

    //Setting/getting代码在这里略过
}
```

@PrimaryKey： 每个entity必须定义至少一个字段作为主键。如果需要动态分配ID的话，需要设置autoGenerate = true；如果entity有个组合的主键，你可以使用@Entity注解的primaryKeys的属性: @Entity(primaryKeys = {"firstName", "lastName"}); 

2， 使用@Dao定义数据库的接口的

```java
@Dao
public interface UserDao {
    @Query("SELECT * FROM UserDataTable")
    List<UserTableEntity> getAll();

    @Query("SELECT * FROM UserDataTable WHERE id IN (:userIds)")
    List<UserTableEntity> getUserById(int[] userIds);

    @Insert
    void insertAll(List<UserTableEntity> users);

    @Insert
    void insertAll(UserTableEntity user);

    @Delete
    void delete(UserTableEntity user);
}
```

3, 使用@Database定义一个数据库，

```java
//entities定义包含的table有哪些；
@Database(entities = {UserTableEntity.class}, version = 1)
public abstract class AppDatabase extends RoomDatabase {
    public abstract UserDao UserDao();
}
```

4， 对于数据库来说，必须遵守单例模式当初始化一个AppDatabase对象；因为多实例的话，代价很高，也不太需要多实例；

```java
public class DataBaseManager {
    private static AppDatabase mAppDatabase;
    private static Object LOCK = new Object();
    public static AppDatabase getAppDatabase(Context context) {
        if (mAppDatabase == null) {
            synchronized (LOCK) {
                if (mAppDatabase == null) {
                    //app_data_base为数据库的命名
                    mAppDatabase = Room.databaseBuilder(context.getApplicationContext()
                            , AppDatabase.class, "app_data_base").build();
                }
            }
        }
        return mAppDatabase;
    }
}
```
这里整个数据就搭建直接的，剩下就是使用了；

5， 获取数据库实例并使用：

```java
 UserTableEntity user = new UserTableEntity();
 user.setAge(18);
 user.setUserName("Yufen.Zhi");
 user.setWork("Android");
 DataBaseManager.getAppDatabase(RoomTestingActivity.this).UserDao().insertAll(user);//插入数据
 DataBaseManager.getAppDatabase(RoomTestingActivity.this).UserDao().getAll(); //获取数据
```

注： @insert等操作不允许在UI线程里操作，否则会抛出异常；

#### indices属性
可通过在@Entity属性中包含indices属性来加快数据的查询:

```java
@Entity(indices = {@Index("name"), @Index("last_name", "address")})
class User {
    @PrimaryKey
    public int id;

    public String firstName;
    public String address;

    @ColumnInfo(name = "last_name")
    public String lastName;

    @Ignore
    Bitmap picture;
}
```
有时，确切的字段和组字段必须是独一无二的。你可以强加这个独一无二的特性通过设置一个@Index注解的unique属性为true。

```java
@Entity(indices = {@Index(value = {"first_name", "last_name"},
        unique = true)})
class User {
    @PrimaryKey
    public int id;

    @ColumnInfo(name = "first_name")
    public String firstName;

    @ColumnInfo(name = "last_name")
    public String lastName;

    @Ignore
    Bitmap picture;
}
```

#### @ForeignKey

```java
@Entity(tableName = "UserDataTable")
public class UserTableEntity {
    @PrimaryKey(autoGenerate = true) // 单个主键设置为自增长
    private int id;

    @ColumnInfo(name = "user_name") // 定义列名
    private String mUserName;

    @ColumnInfo(name = "age")
    private int mAge;
}

@Entity(foreignKeys = @ForeignKey(entity = UserTableEntity.class,
        parentColumns = "id",
        childColumns = "user_id",
        onDelete = ForeignKey.CASCADE))
 public class Book {

    @PrimaryKey(autoGenerate = true)
    public int bookId;

    public String title = "Android learn";

    @ColumnInfo(name = "user_id")
    public int userId;
 }
```
ForeignKey可以指明两个表之间的关系；onDelete = ForeignKey.CASCADE: 例如这个人有一本书，当人的数据被删除时，对应的Book也会被自动删除；

但该用法,我还没尝试写出成功例子, 后续有时候再写一个; 暂时先略过;

#### @Ingore
当你有一些列名的数据不需要再添加到数据库里的话， 则可直接使用@Ignore注解告知；

#### @Embedded
使用 @Embedded 注释来表示要分解到表中子字段的对象（此时数据库的列为两个类中所有的字段）; 

该功能还是非常实用，我们获取数据时，可直接转化为我们需要的变量实例；

注：使用@Embedded的对象中，不能使用@PrimaryKey，使用的话， 该变量会被无视掉；

```java
@Entity(tableName = "UserDataTable")
public class UserTableEntity {
    @PrimaryKey(autoGenerate = true) // 单个主键设置为自增长
    private int id;

    @ColumnInfo(name = "user_name") // 定义列名
    private String mUserName;

    @ColumnInfo(name = "age")
    private int mAge;

    @Embedded
    private Book mBook;
}

@Entity
 public class Book {
    public String title = "Android learn";

    @ColumnInfo(name = "user_id")
    public int userId;
 }

```

### 数据操作详解
我们使用@Dao来定义数据操作的接口；

#### Insert

使用@Insert注解时， Room注解处理器会自动生成相应的方法来把参数的对象的数据插入数据库中；

```java
@Dao
public interface MyDao {
    @Insert(onConflict = OnConflictStrategy.REPLACE)
    public void insertUsers(User... users);

    @Insert
    public void insertBothUsers(User user1, User user2);

    @Insert
    public void insertUsersAndFriends(User user, List<User> friends);
}
```

其中onConflict的值表示，把数据冲突时， 应该采用哪个策略： REPLACE， ROLLBACK， ABORT， IGNORE；

注：SQLite处理@Insert(OnConflict=REPLACE) 作为一个REMOVE和REPLACE操作而不是单独的UPDATE操作。这个替换冲突值的方法能够影响你的外键约束。

#### Update && Delete

 在更新或删除时，会以Promarykey为匹配对象，相同则更新该对象为新数据或删除该对象；

```java
@Dao
public interface MyDao {
    @Update
    public void updateUsers(User... users);
    @Delete
    public void deleteUsers(User... users);
}
```

#### Query
@Query 是DAO类中使用的主要注解，它允许你在数据库中执行读/写操作。每个@Query方法在编译时被校验，所以如果有问题，将在编译时出现编译错误而不是运行时；

   它给出警告如果仅有一些字段匹配；

   它报错如果没有字段匹配

```java
@Dao
public interface MyDao {
    //装载所有数据
    @Query("SELECT * FROM user")
    public User[] loadAllUsers();
    
    //通过参数来过滤数据(单个参数)
    @Query("SELECT * FROM user WHERE age > :minAge")
    public User[] loadAllUsersOlderThan(int minAge);
    
    
    //多个参数
    @Query("SELECT * FROM user WHERE age BETWEEN :minAge AND :maxAge")
    public User[] loadAllUsersBetweenAges(int minAge, int maxAge);

    @Query("SELECT * FROM user WHERE first_name LIKE :search " +
           "OR last_name LIKE :search")
    public List<User> findUserWithName(String search);
    
    //可变参数
    @Query("SELECT first_name, last_name FROM user WHERE region IN (:regions)")
    public List<NameTuple> loadUsersFromRegions(List<String> regions);
}
```

#### 返回一部分数据
大部分时间，我们并不需要每次都返回所有数据，我们只需要其中一部分数据而已，这样查询速度也会更快；Room允许你返回任何java对象从查询中只要列结果集能够被映射到返回的对象中。例如：

你能够创建如下POJO通过拿取用户的姓和名：
```java
public class NameTuple {
    @ColumnInfo(name="first_name")
    public String firstName;

    @ColumnInfo(name="last_name")
    public String lastName;
}
```
然后在查询方法中： 
```java
@Dao
public interface MyDao {
    @Query("SELECT first_name, last_name FROM user")
    public List<NameTuple> loadFullName();
}
```

#### Observable请求
当我们执行查询时， 我们经常希望我们的UI能观察到数据变化从而自动更新， 为了这个目标，我们需要查询时，我们需要返回一个LiveData类型的实例，我们就可以对其进行订阅， 而RoomRoom生成所有需要的代码， 当数据库被更新时，Room会去更新LiveData。

```java
@Dao
public interface MyDao {
    @Query("SELECT first_name, last_name FROM user WHERE region IN (:regions)")
    public LiveData<List<User>> loadUsersFromRegionsSync(List<String> regions);
}
```

#### RxJava
Room也能返回RxJava2 Publisher和Flowable对象从你定义的查询当中。为了使用这个功能，添加android.arch.persistence.room:rxjava2 到你的build Gradle依赖。你能够返回Rxjava2定义的对象; 
```xml
dependencies {
    def room_version = "2.1.0"
    implementation 'androidx.room:room-rxjava2:$room_version'
}
```

```java
@Dao
@Dao
public interface MyDao {
    @Query("SELECT * from user where id = :id LIMIT 1")
    public Flowable<User> loadUserById(int id);

    // Emits the number of users added to the database.
    @Insert
    public Maybe<Integer> insertLargeNumberOfUsers(List<User> users);

    // Makes sure that the operation finishes successfully.
    @Insert
    public Completable insertLargeNumberOfUsers(User... users);

    /* Emits the number of users removed from the database. Always emits at
       least one user. */
    @Delete
    public Single<Integer> deleteUsers(List<User> users);
}
```
 更多详细，请参考： ![Room and RxJava](https://medium.com/google-developers/room-rxjava-acb0cd4f3757)

#### 使用Cursor访问

如果你的应用逻辑直接访问返回的行，你可以返回一个Cursor对象从你的查询当中， 但不推荐使用；

```java
@Dao
public interface MyDao {
    @Query("SELECT * FROM user WHERE age > :minAge LIMIT 5")
    public Cursor loadRawUsersOlderThan(int minAge);
}
```

#### 同时搜索多个表

Room允许你写任何查询，所以你也能连接表格。另外Roomm返回的是Flowable或LiveData的话， Room会监控所有返回的数据是否有效。

```java
@Dao
public interface MyDao {
    @Query("SELECT * FROM book " +
           "INNER JOIN loan ON loan.book_id = book.id " +
           "INNER JOIN user ON user.id = loan.user_id " +
           "WHERE user.name LIKE :userName")
   public List<Book> findBooksBorrowedByNameSync(String userName);
}
```

也可以专门写一个POJOs来装载数据：
```java
@Dao
public interface MyDao {
   @Query("SELECT user.name AS userName, pet.name AS petName " +
          "FROM user, pet " +
          "WHERE user.id = pet.user_id")
   public LiveData<List<UserPet>> loadUserAndPetNames();

   // You can also define this class in a separate file, as long as you add the
   // "public" access modifier.
   static class UserPet {
       public String userName;
       public String petName;
   }
}
```

#### 类型转换
有时候，app需要使用自定义数据保存在数据库列中，为了使用这部分，Room提供使用@typeConverter的注解来转换自定义数据为room已知的类型进行保存或读取；
比如我们需要保存Date对象， 那边我们就需要用@typeConverter转换成时间戳及其互换： 
```java
public class Converters {
    @TypeConverter
    public static Date fromTimestamp(Long value) {
        return value == null ? null : new Date(value);
    }

    @TypeConverter
    public static Long dateToTimestamp(Date date) {
        return date == null ? null : date.getTime();
    }
}
```

定义好转换方法后，我们就需要在数据库中使用@TypeConverters告诉它这类数据转换方法：
```java
@Database(entities = {User.class}, version = 1)
@TypeConverters({Converters.class})
public abstract class AppDatabase extends RoomDatabase {
    public abstract UserDao userDao();
}
```

最后你就可以在Enitity中和Dao的接口定义中使用该Date对象进行操作了：
```java
@Entity
public class User {
    private Date birthday;
}

@Dao
public interface UserDao {
    @Query("SELECT * FROM user WHERE birthday BETWEEN :from AND :to")
    List<User> findUsersBornBetweenDates(Date from, Date to);
}
```



### Migrating Room databases

当我们升级应用的版本，功能改变时，如果你的数据entity类也需要修改的话，我们可以使用Migration类来把旧版本的数据迁移到当前最新数据库版本。

```java
static final Migration MIGRATION_1_2 = new Migration(1, 2) {//由1升级到版本2
    @Override
    public void migrate(SupportSQLiteDatabase database) {
        database.execSQL("CREATE TABLE `Fruit` (`id` INTEGER, "
                + "`name` TEXT, PRIMARY KEY(`id`))");
    }
};

static final Migration MIGRATION_2_3 = new Migration(2, 3) {//由2升级到版本3
    @Override
    public void migrate(SupportSQLiteDatabase database) {
        database.execSQL("ALTER TABLE Book "
                + " ADD COLUMN pub_year INTEGER"); //添加pub_year列，类型为INTEGER
    }
};

//在获取数据库里
Room.databaseBuilder(getApplicationContext(), MyDb.class, "database-name")
        .addMigrations(MIGRATION_1_2, MIGRATION_2_3).build();
```

对于数据库迁移是很重要的事， 一个不小心就要把用户的数据给弄没了（特别是没有云服务数据库的应用），这样就无法恢复了；

Room同样提供相关的数据库迁移的单元测试；

### Room JUnit Test
对于添加JUnit的测试是非常有必要的，并且这个不需要实现界面，因此实现也会更简单；

在测试Room时， 你应该创建一个使用的内存的数据来让你的测试更加封装性：Room.inMemoryDatabaseBuilder(xxx).build();

```java
@RunWith(AndroidJUnit4.class)
public class SimpleEntityReadWriteTest {
    private UserDao userDao;
    private TestDatabase db;

    @Before
    public void createDb() {
        Context context = ApplicationProvider.getApplicationContext();
        db = Room.inMemoryDatabaseBuilder(context, TestDatabase.class).build();
        userDao = db.getUserDao();
    }

    @After
    public void closeDb() throws IOException {
        db.close();
    }

    @Test
    public void writeUserAndReadInList() throws Exception {
        User user = TestUtil.createUser(3);
        user.setName("george");
        userDao.insert(user);
        List<User> byName = userDao.findUsersByName("george");
        assertThat(byName.get(0), equalTo(user));
    }
}
```

##### 测试数据库升级
1，需要配置room.schemaLocation属性，Room会把你的数据库的结构保存到JSON文件中；
```xml
android {
    ...
    defaultConfig {
        ...
        javaCompileOptions {
            annotationProcessorOptions {
                arguments = ["room.schemaLocation":
                             "$projectDir/schemas".toString()]
            }
        }
    }
}
```
另外你应该存储导出的JSON文件-代表了你数据库schema的历史-在你的版本控制系统中，正如它允许创建老版本的数据库去测试。

为了测试这些migrations，添加 android.arch.persistence.room:testing Maven artifac从Room当中到你的测试依赖当中，并且把schema 位置当做一个asset文件添加，如下所示：
```xml
android {
    ...
    sourceSets {
        androidTest.assets.srcDirs += files("$projectDir/schemas".toString())
    }
}
```

2，测试package提供一个可以读取这些schema文件的MigrationTestHelper类。它也是Junit4 TestRule类，所以它能管理创建的数据库。

```java
@RunWith(AndroidJUnit4.class)
public class MigrationTest {
    private static final String TEST_DB = "migration-test";

    @Rule
    public MigrationTestHelper helper;

    public MigrationTest() {
        helper = new MigrationTestHelper(InstrumentationRegistry.getInstrumentation(),
                MigrationDb.class.getCanonicalName(),
                new FrameworkSQLiteOpenHelperFactory());
    }

    @Test
    public void migrate1To2() throws IOException {
        SupportSQLiteDatabase db = helper.createDatabase(TEST_DB, 1);

        // db has schema version 1. insert some data using SQL queries.
        // You cannot use DAO classes because they expect the latest schema.
        db.execSQL(...);

        // Prepare for the next version.
        db.close();

        // Re-open the database with version 2 and provide
        // MIGRATION_1_2 as the migration process.
        db = helper.runMigrationsAndValidate(TEST_DB, 2, true, MIGRATION_1_2);

        // MigrationTestHelper automatically verifies the schema changes,
        // but you need to validate that the data was migrated properly.
    }
}
```

### 相关问题

问题点：

```xml
Cannot figure out how to save this field into database. You can consider adding a type converter for it.
```

原因是： Jetpack的Room不支持直接存储列表的功能, 需要添加一个转换器：

比如： val idList: MutableList<String>, 就编译失败； 

解决方案：

实现一个转换器：

```kotlin
class StringListTypeConverter {
    val gson by lazy { Gson() }

    @TypeConverter
    fun translateToStringList(data: String):  List<String> {
        val listType = object : TypeToken<List<String>>() {}.type
        return gson.fromJson(data, listType)
    }

    @TypeConverter
    fun translateToString(list: List<String>): String {
        return gson.toJson(list)
    }
}
```

使用时：

```kotlin
@Keep
@Entity(tableName = "table_name")
@TypeConverters(StringListTypeConverter::class)
class DatabaseItem(
    val idList: MutableList<String>
)
```



本文参考文章：

> [Andrdoi Room 官网](https://developer.android.com/training/data-storage/room)

> [Android Jetpack架构组件之 Room（使用、源码篇）](https://blog.csdn.net/Alexwll/article/details/83033460)

> [Android Room Orm框架学习](https://www.jianshu.com/p/29e5e8c75450)
