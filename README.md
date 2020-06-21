# NotePad
This is an AndroidStudio rebuild of google SDK sample NotePad

## 功能拓展：

### 一.添加笔记时间戳

(1). 在notelist_item.xml中添加一个TextView来显示时间

     原xml中只有一个TextView显示标题，修改xml代码如下:
```
 <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:paddingLeft="6dip"
    android:paddingRight="6dip"
    android:paddingBottom="3dip">
    
    <TextView
        android:id="@android:id/text1"
        android:layout_width="match_parent"
        android:layout_height="?android:attr/listPreferredItemHeight"
        android:textAppearance="?android:attr/textAppearanceLarge"
        android:gravity="center_vertical"
        android:paddingLeft="5dip"
        android:singleLine="true"
        android:text="test"
        />
    <TextView
        android:id="@+id/text2"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:textAppearance="?android:attr/textAppearanceLarge"
        android:gravity="center_vertical"
        android:singleLine="true"
        android:text="time"
        />
</LinearLayout>
```


### (2).在NotesList.java中的String[] PROJECTION中添加修改笔记时间项
```
private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_TITLE, //
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE
    };
```
  
### (3).在dataColumns和ViewId中添加时间项
```
String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE,NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE } ;
int[] viewIDs = { android.R.id.text1 ,R.id.text2};
```
### (4).将时间戳转换为我们设定格式的时间
         在NoteEditor.java中添加一个Long变量获取系统时间戳，然后通过SimpleDateFormat转换为时间
```
//获取系统时间戳
        Long now = System.currentTimeMillis();
        SimpleDateFormat sf = new SimpleDateFormat("yyyy年MM月dd日 HH时mm分ss秒");
        //把时间戳转换为时间
        String format = sf.format(now);

        values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, format);
```

### (5).运行效果
![效果1](app/src/main/res/screenshoots/1.png)

## 二.笔记查询

### (1).在list_options_menu.xml菜单布局文件中添加一个item用于搜索笔记
         icon就设置为系统自带的搜索icon
```
<item
        android:id="@+id/menu_search"
        android:title="Search"
        android:showAsAction="always"
        android:icon="@android:drawable/ic_search_category_default">

    </item>
```
### (2).添加一个用于显示搜索页面的布局文件
        命名为search_list，其中包括一个SearchView显示搜索框和ListView显示笔记
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical" android:layout_width="match_parent"
    android:layout_height="match_parent">
    <SearchView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:id="@+id/search_view"
        android:queryHint="输入搜索内容">

    </SearchView>
    <ListView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:id="@android:id/list">

    </ListView>

</LinearLayout>
```
            其中ListView的id不能设置为"@+id/"，否则导致应用闪退
### (3).新建一个名为NoteSearch的activit用于NoteList跳转和实现搜索效果
        NoteSearch要继承ListView和实现SearchView.OnQueryTextListener，匹配模式为模糊匹配
```
 private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_TITLE, // 1
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, // 2
    };
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.search_list);
        Intent intent = getIntent();
        if (intent.getData() == null) {
            intent.setData(NotePad.Notes.CONTENT_URI);
        }
        SearchView searchview = (SearchView)findViewById(R.id.search_view);
        //给查询文本框设置监听器
        searchview.setOnQueryTextListener(NoteSearch.this);
    }
    @Override
    public boolean onQueryTextSubmit(String query) {
        return false;
    }
    @Override
    public boolean onQueryTextChange(String newText) {
        String selection = NotePad.Notes.COLUMN_NAME_TITLE + " Like ? ";
        String[] selectionArgs = { "%"+newText+"%" };
        Cursor cursor = managedQuery(
                getIntent().getData(),            // Use the default content URI for the provider.
                PROJECTION,                       // Return the note ID and title for each note. and modifcation date
                selection,                        // 条件左边
                selectionArgs,                    // 条件右边
                NotePad.Notes.DEFAULT_SORT_ORDER  // Use the default sort order.
        );
        String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE ,  NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE };
        int[] viewIDs = { android.R.id.text1 , R.id.text2 };
        SimpleCursorAdapter adapter = new SimpleCursorAdapter(
                this,
                R.layout.noteslist_item,
                cursor,
                dataColumns,
                viewIDs
        );
        setListAdapter(adapter); //加载主界面布局
        return true;
    }
    @Override
    protected void onListItemClick(ListView l, View v, int position, long id) { //点击笔记进入编辑
        // Constructs a new URI from the incoming URI and the row ID
        Uri uri = ContentUris.withAppendedId(getIntent().getData(), id);
        // Gets the action from the incoming Intent
        String action = getIntent().getAction();
        // Handles requests for note data
        if (Intent.ACTION_PICK.equals(action) || Intent.ACTION_GET_CONTENT.equals(action)) {
            // Sets the result to return to the component that called this Activity. The
            // result contains the new URI
            setResult(RESULT_OK, new Intent().setData(uri));
        } else {
            // Sends out an Intent to start an Activity that can handle ACTION_EDIT. The
            // Intent's data is the note ID URI. The effect is to call NoteEdit.
            startActivity(new Intent(Intent.ACTION_EDIT, uri));
        }
    }
```
### (4).在NotesList的OnOptionsItemSelected方法中添加新增的search项的点击后要执行的代码
        显示调用intent用于跳转到NoteSearch
```
case R.id.menu_search:
                Intent intent=new Intent();
                intent.setClass(this,NoteSearch.class);
                NotesList.this.startActivity(intent);
                return true;
```
### (5).运行效果
点击菜单的搜索
![效果2](app/src/main/res/screenshoots/2.png)

点击搜索框
![效果3](app/src/main/res/screenshoots/3.png)

