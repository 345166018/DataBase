Github地址：[greenDAO](https://github.com/greenrobot/greenDAO)


# 1 简介
# 1.1 ORM框架介绍

对象关系映射（Object Relational Mapping，简称ORM）是通过使用描述对象和数据库之间。

映射的元数据，将程序中的对象自动持久化到关系数据库中。
>- java中的类------------------------> 表
>- java里面的类属性---------------> 字段
>- java里面的类属性的值---------> 数据库表里面的1条数据

## 1.2 GreenDao介绍
GreenDao 是一个对象关系映射（ORM） 的框架， 能够提供一个接口通过操作对象的方式去操作关系型数据库， 它能够让你操作数据库时更简单、 更方便。 如下图所示：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190722094137694.png)

## 1.3 GreenDao表现
<img src="https://img-blog.csdnimg.cn/20190722111606914.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2hvbmd4dWU4ODg4,size_16,color_FFFFFF,t_70" width="500"/>

# 2 注解解释


1. @Entity：告诉GreenDao该对象为实体，只有被@Entity注释的Bean类才能被dao类操作。

2. @Id：对象的Id，使用Long类型作为EntityId，否则会报错。(autoincrement = true)表示主键会自增，如果false就会使用旧值。

3. @Property：可以自定义字段名，注意外键不能使用该属性。

4. @NotNull：属性不能为空。

5. @Transient：使用该注释的属性不会被存入数据库的字段中。

6. @Unique：该属性值必须在数据库中是唯一值。

7. @Generated：编译后自动生成的构造函数、方法等的注释，提示构造函数、方法等不能被修改。


# 3 使用
Github地址：[HxGreenDao](https://github.com/345166018/DataBase/tree/master/HxGreenDao)




root build.gradle：

```
buildscript {
    repositories {
        jcenter()
        mavenCentral() // add repository
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:3.1.1'
        classpath 'org.greenrobot:greendao-gradle-plugin:3.2.2' // add plugin
    }
}
```
 app modules app/build.gradle:
 

```
apply plugin: 'com.android.application'
apply plugin: 'org.greenrobot.greendao' // apply plugin

greendao {
    schemaVersion 1
    targetGenDir 'src/main/java'
    daoPackage 'com.hongx.greendao.dao'
} 

dependencies {
    implementation 'org.greenrobot:greendao:3.2.2' // add library
}
```

 Make project 自定生成DaoMaster和DaoSession，如下图：
 
 <img src="https://img-blog.csdnimg.cn/20190722104623462.png" width="200"/>

创建Note.java：
```
@Entity(indexes = {
        @Index(value = "text, date DESC", unique = true)
})
public class Note {

    @Id
    private Long id;

    @NotNull
    private String text;
    private String comment;
    private java.util.Date date;

    @Convert(converter = NoteTypeConverter.class, columnType = String.class)
    private NoteType type;
}
```
创建NoteType：
```
public enum NoteType {
    TEXT, LIST, PICTURE
}
```
App中初始化：
```
public class App extends Application {

    private DaoSession daoSession;

    @Override
    public void onCreate() {
        super.onCreate();

        // regular SQLite database
        DaoMaster.DevOpenHelper helper = new DaoMaster.DevOpenHelper(this, "notes-db");
        Database db = helper.getWritableDb();

        // encrypted SQLCipher database
        // note: you need to add SQLCipher to your dependencies, check the build.gradle file
        // DaoMaster.DevOpenHelper helper = new DaoMaster.DevOpenHelper(this, "notes-db-encrypted");
        // Database db = helper.getEncryptedWritableDb("encryption-key");

        daoSession = new DaoMaster(db).newSession();
    }

    public DaoSession getDaoSession() {
        return daoSession;
    }
}
```
查询：

```
        // get the note DAO
        DaoSession daoSession = ((App) getApplication()).getDaoSession();
        noteDao = daoSession.getNoteDao();

        // query all notes, sorted a-z by their text
        notesQuery = noteDao.queryBuilder().orderAsc(NoteDao.Properties.Text).build();
```
增加：

```
       Note note = new Note();
        note.setText(noteText);
        note.setComment(comment);
        note.setDate(new Date());
        note.setType(NoteType.TEXT);
        noteDao.insert(note);
```
删除：

```
   Note note = notesAdapter.getNote(position);
   Long noteId = note.getId();
   noteDao.deleteByKey(noteId);
```
效果如下：

<br>
<img src="https://img-blog.csdnimg.cn/20190722110519879.gif" width="250"/>

