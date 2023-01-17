## Paging框架



#### **相关资料**

+ [Paging库概览](https://developer.android.com/topic/libraries/architecture/paging)
+ [反思|Android 列表分页组件Paging的设计与实现：系统概述](https://juejin.cn/post/6844903976777809928#heading-15)
+ 
+ https://juejin.cn/post/6844903849946415118  临时记录



#### Paging框架作用

一个为**无限滚动**的分页模式（即加载一定量的数据以后会继续加载下一页的数据，而不必等用户滑动到页面底部，给用户使用时就像一次性加载完所有数据一样）设计的框架。



#### Paging框架使用

**设置**

根据文档进行依赖添加。

**库结构**

+ PagedList

  列表数据容器，每一页的数据量等都可以进行配置。

+ 数据

  + 为PagedList提供分页数据，是`数据库数据或者服务端`数据的一个**快照**，我们需要提供的不是DataSource，而是**DataSource的工厂**。
    1. DataSource(类型)：DataSource<Key,Value>，Key对应加载数据的条件信息，Value对应数据实体类。
       + PageKeyedDataSource<Key,Value>：适用于目标数据根据页信息请求数据的场景。
       + ItemKeyedDataSource<Key,Value>：适用于目标数据的加载依赖特定的item的信息。
       + PositionnalDataSource<Key,Value>：适用于目标数据总数固定，通过特定的位置加载数据。
  + 与PagedList的关系通过PagedListBuilder串联。
  + 更多配置可以通过**PagedList.Config**完成，它可以作为PagedListBuilder的参数。
    + pageSize 每页加载数据量。
    + initialLoadSizeHint 首次加载的数据量，默认为pageSize 三倍。
    + prefetchDistance 当距离加载边缘多远时进行分页的请求，默认值为pageSize ，也就是还有一页数据的时候开始下一页的数据加载。
    + enablePlaceholders 是否启用占位符，不开启占位符时条目的数量和数据容器PagedList中的数量一致；但是开启以后条目的数量和Database的数据量（如果数据量是确定的话）一致，列表中所展示的一些条目还没有渲染完成。

+ 界面

  使用 `PagedListAdapter` 将项加载到 `RecyclerView`。

**支持不同的数据结构**

> 分页库支持多种数据结构
>
> ![image-20210404153502818](D:\云盘备份\学习总结\安卓\Paging框架.assets\image-20210404153502818.png)

+ 仅限网络

+ 仅限数据库

+ 网络和数据库

  在开始观察数据库之后，您可以使用 `PagedList.BoundaryCallback`监听数据库中的数据何时耗尽。然后，您可以从网络中获取更多项并将它们插入到数据库中。

**处理网络连接错误**

+ 关注网络状态
+ 向用户提供重试选择。

**更新现有应用**

+ 自定义分页解析

+ 使用列表而不是页面加载的数据

+ 使用CursorAdapter将数据游标与列表视图相关联

  + 使用 CursorAdapter 将 Cursor 的数据与 ListView 相关联
  + 如果迁移到 RecyclerView，就将 Cursor 组件替换为 Room 或 PositionalDataSource，具体取决于 Cursor 实例是否会访问 SQLite 数据库。

+ 使用AsyncListUtil异步加载内容

  如果使用 AsyncListUtil 对象来异步加载和显示信息组，则通过 Paging 库可以更轻松地加载数据。

  + 数据不需要固定位置
  + 可以加载庞大数量的数据
  + 数据可观察

**数据库实例**

+ 使用LiveData观察分页数据

   LiveData<PagedList>

  ``` kotlin
  /*
  随着在数据库中添加、移除或更改 concert 事件，RecyclerView 中的内容会自动且高效地更新
  */
  @Dao
  public interface ConcertDao {
      // The Integer type parameter tells Room to use a PositionalDataSource
      // object, with position-based loading under the hood.
      @Query("SELECT * FROM concerts ORDER BY date DESC")
      DataSource.Factory<Integer, Concert> concertsByDate();
  }
  
  public class ConcertViewModel extends ViewModel {
      private ConcertDao concertDao;
      public final LiveData<PagedList<Concert>> concertList;
  
      public ConcertViewModel(ConcertDao concertDao) {
          this.concertDao = concertDao;
          concertList = new LivePagedListBuilder<>(
              concertDao.concertsByDate(), /* page size */ 50).build();
      }
  }
  
  public class ConcertActivity extends AppCompatActivity {
      @Override
      public void onCreate(@Nullable Bundle savedInstanceState) {
          super.onCreate(savedInstanceState);
          ConcertViewModel viewModel =
                  new ViewModelProvider(this).get(ConcertViewModel.class);
          RecyclerView recyclerView = findViewById(R.id.concert_list);
          ConcertAdapter adapter = new ConcertAdapter();
          viewModel.concertList.observe(this, adapter::submitList);
          recyclerView.setAdapter(adapter);
      }
  }
  
  public class ConcertAdapter
          extends PagedListAdapter<Concert, ConcertViewHolder> {
      protected ConcertAdapter() {
          super(DIFF_CALLBACK);
      }
  
      @Override
      public void onBindViewHolder(@NonNull ConcertViewHolder holder,
              int position) {
          Concert concert = getItem(position);
          if (concert != null) {
              holder.bindTo(concert);
          } else {
              // Null defines a placeholder item - PagedListAdapter automatically
              // invalidates this row when the actual object is loaded from the
              // database.
              holder.clear();
          }
      }
  
      private static DiffUtil.ItemCallback<Concert> DIFF_CALLBACK =
              new DiffUtil.ItemCallback<Concert>() {
          // Concert details may have changed if reloaded from the database,
          // but ID is fixed.
          @Override
          public boolean areItemsTheSame(Concert oldConcert, Concert newConcert) {
              return oldConcert.getId() == newConcert.getId();
          }
  
          @Override
          public boolean areContentsTheSame(Concert oldConcert,
                  Concert newConcert) {
              return oldConcert.equals(newConcert);
          }
      };
  }
  ```

+ 使用RxJava2观察分页数据

  Observable<List>





#### Paging框架原理