一、时间戳
更新NoteList类：在NoteList类的PROJECTION数组中添加COLUMN_NAME_MODIFICATION_DATE字段，以便从数据库中检索修改时间。


private static final String[] PROJECTION = new String[] {
NotePad.Notes._ID, // 0
NotePad.Notes.COLUMN_NAME_TITLE, // 1
NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, // 添加修改时间
};


在notelist_item.xml中添加布局文件



<TextView
android:id="@+id/text2"
android:layout_width="match_parent"
android:layout_height="wrap_content"
android:paddingLeft="5dip"
android:singleLine="true"
android:gravity="center_vertical"/>

格式化时间戳：在NoteEditor类的updateNote方法中获取当前系统的时间，并对其进行格式化，以便以更易读的格式显示。



ContentValues values = new ContentValues();
Long now = Long.valueOf(System.currentTimeMillis());
SimpleDateFormat sf = new SimpleDateFormat("yy/MM/dd HH:mm");
Date d = new Date(now);
String format = sf.format(d);
values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, format);



时间戳的功能显示![eccf7cf56e2b3d6b4981b1671c1920d](https://github.com/user-attachments/assets/700c38e5-dcb5-4509-acf9-53ee3edf01fd)





二、笔记内容的搜索功能
添加搜索视图：
在list_options_menu.xml中添加搜索菜单项，使其在主界面上可见。



//<item
android:id="@+id/menu_search"
android:title="@string/menu_search"
android:icon="@drawable/ic_search"
android:showAsAction="ifRoom|collapseActionView"
android:actionViewClass="android.widget.SearchView" />
设置事件监听器：
//private Cursor mCursor;
@Override
public boolean onCreateOptionsMenu(Menu menu) {
super.onCreateOptionsMenu(menu);
MenuInflater inflater = getMenuInflater();

inflater.inflate(R.menu.list_options_menu, menu);

        // 找到搜索菜单项并设置其监听器
        MenuItem searchItem = menu.findItem(R.id.menu_search);
        SearchView searchView = (SearchView) searchItem.getActionView();
        searchView.setOnQueryTextListener(new SearchView.OnQueryTextListener() {
            @Override
            public boolean onQueryTextSubmit(String query) {
                // 处理搜索提交事件
                return false;
            }
            @Override
            public boolean onQueryTextChange(String newText) { 
                // 处理搜索文本变化事件
                filterNotesList(newText);
                return false;
            }
        });
        return true;
    }
![08ed699f33aa511c07713a1b930080b](https://github.com/user-attachments/assets/65adfb45-f1b5-472c-927b-059948c7bf47)
![f66a04d4e0a005e8fbe857e25a14833](https://github.com/user-attachments/assets/bf9ef17a-b8ec-4bab-8373-6cb75365b38b)搜索内容K
![d7f357b9391344275be9dbecb902ce1](https://github.com/user-attachments/assets/1f051f69-5207-4a16-9c5f-8f3956fde99c)搜索内容a


扩展功能：
一、排序
这项功能分为按标题排序和按日期排序
1.在list_options_menu.xml中添加排序菜单选项
<item
    android:id="@+id/menu_sort"
    android:title="@string/sort"
    android:showAsAction="never">
    <menu>
        <item
            android:id="@+id/sort_by_title"
            android:title="@string/sort_by_title" />
        <item
            android:id="@+id/sort_by_date"
            android:title="@string/sort_by_date" />
        <!-- 可以根据需要添加更多排序选项 -->
    </menu>
</item>
![199ccb34f4c5a684e4bc47ab3d12121](https://github.com/user-attachments/assets/14c8b0dd-8be9-4d71-a1b7-df56ba88fa92)

2.在NoteList类中添加排序功能
@Override
public boolean onCreateOptionsMenu(Menu menu) {
super.onCreateOptionsMenu(menu);
MenuInflater inflater = getMenuInflater();
inflater.inflate(R.menu.list_options_menu, menu);
        // 添加排序选项到菜单
        MenuItem sortItem = menu.findItem(R.id.menu_sort);
        SubMenu sortSubMenu = sortItem.getSubMenu();
        sortSubMenu.clear(); // 清除默认子菜单项
        // 添加按标题排序的菜单项
        sortSubMenu.add(0, R.id.sort_by_title, 0, R.string.sort_by_title)
                .setIcon(android.R.drawable.ic_menu_sort_by_size)
                .setOnMenuItemClickListener(new MenuItem.OnMenuItemClickListener() {
                    @Override
                    public boolean onMenuItemClick(MenuItem item) {
                        sortNotesByTitle();
                        return true;
                    }
                });
        sortSubMenu.add(0, R.id.sort_by_date, 0, R.string.sort_by_date)
                .setIcon(R.drawable.ic_menu_sort_by_date) // 确保你的 drawable 文件存在
                .setOnMenuItemClickListener(new MenuItem.OnMenuItemClickListener() {
                    @Override
                    public boolean onMenuItemClick(MenuItem item) {
                        sortNotesByDate();
                        return true;
                    }
                });
private void sortNotesByTitle() {
// 实现按标题排序的逻辑
String newSortOrder = NotePad.Notes.COLUMN_NAME_TITLE + " ASC"; // 按标题升序排序
applySortOrder(newSortOrder);
}
    private void sortNotesByDate() {
        // 实现按日期排序的逻辑
        String newSortOrder = NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE + " DESC"; // 按修改日期降序排序
        applySortOrder(newSortOrder);
    }
    private void applySortOrder(String newSortOrder) {
        // 保存新的排序顺序
        sortOrder = newSortOrder;
        // 重新加载笔记列表
        refreshNotesList();
        // 保存排序偏好
        saveSortPreference(sortOrder);
    }
    private void refreshNotesList() {
    // 重新查询数据并更新列表
    Cursor newCursor = managedQuery(getIntent().getData(), PROJECTION, null, null, sortOrder);
    ((SimpleCursorAdapter) getListAdapter()).changeCursor(newCursor);
}
private void saveSortPreference(String sortOrder) {
    SharedPreferences preferences = getSharedPreferences("NotePadPrefs", MODE_PRIVATE);
    SharedPreferences.Editor editor = preferences.edit();
    editor.putString("SortOrder", sortOrder);
    editor.apply();
}
按标题排序![4d1786e9abdd2f3d8a96a1252d17249](https://github.com/user-attachments/assets/2d017f0e-01bc-4c24-99d1-9d5ad02382e9)

按日期排序![56345ead4c6b08d5b8cfb5f2cc3e917](https://github.com/user-attachments/assets/4030e025-58ed-49fd-a8b6-2131074e470a)





二、内容变换颜色


添加颜色的字段：
        @Override
        public void onCreate(SQLiteDatabase db) {
            db.execSQL("CREATE TABLE " + NotePad.Notes.TABLE_NAME + "   ("
                    + NotePad.Notes._ID + " INTEGER PRIMARY KEY,"
                    + NotePad.Notes.COLUMN_NAME_TITLE + " TEXT,"
                    + NotePad.Notes.COLUMN_NAME_NOTE + " TEXT,"
                    + NotePad.Notes.COLUMN_NAME_CREATE_DATE + " INTEGER,"
                    + NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE + " INTEGER,"
                    + NotePad.Notes.COLUMN_NAME_BACK_COLOR + " INTEGER" //颜色
                    + ");");
        }
sNotesProjectionMap.put(
NotePad.Notes.COLUMN_NAME_BACK_COLOR,
NotePad.Notes.COLUMN_NAME_BACK_COLOR);

public class MyCursorAdapter extends SimpleCursorAdapter {
public MyCursorAdapter(Context context, int layout, Cursor c,
String[] from, int[] to) {
super(context, layout, c, from, to);
}
@Override
public void bindView(View view, Context context, Cursor cursor){
super.bindView(view, context, cursor);
//从数据库中读取的cursor中获取笔记列表对应的颜色数据，并设置笔记颜色
int x = cursor.getInt(cursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_BACK_COLOR));
/**
* 白 255 255 255
* 黄 247 216 133
* 蓝 165 202 237
* 绿 161 214 174
* 红 244 149 133
*/
switch (x){
case NotePad.Notes.DEFAULT_COLOR:
view.setBackgroundColor(Color.rgb(255, 255, 255));
break;
case NotePad.Notes.YELLOW_COLOR:
view.setBackgroundColor(Color.rgb(247, 216, 133));
break;
case NotePad.Notes.BLUE_COLOR:
view.setBackgroundColor(Color.rgb(165, 202, 237));
break;
case NotePad.Notes.GREEN_COLOR:
view.setBackgroundColor(Color.rgb(161, 214, 174));
break;
case NotePad.Notes.RED_COLOR:
view.setBackgroundColor(Color.rgb(244, 149, 133));
break;
default:
view.setBackgroundColor(Color.rgb(255, 255, 255));
break;
}
}
}
![image](https://github.com/user-attachments/assets/ab6696ec-01a4-4824-a977-aa71089933c1)


