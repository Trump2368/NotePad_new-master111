一、时间戳
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
![c2f0c1ec0e7929dcd4c758575f5e7b54_](https://github.com/user-attachments/assets/ae62776b-704a-4ba6-8221-53e177c5a1f1)



二、笔记内容的搜索功能
添加搜索视图（SearchView）：

在res/menu/list_options_menu.xml中添加搜索菜单项，使其在主界面上可见。
<item
android:id="@+id/menu_search"
android:title="@string/menu_search"
android:icon="@drawable/ic_search"
android:showAsAction="ifRoom|collapseActionView"
android:actionViewClass="android.widget.SearchView" />
private Cursor mCursor;
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


三、内容变换颜色
先将背景色改为白色
android:theme="@android:style/Theme.Holo.Light"
public static final String COLUMN_NAME_BACK_COLOR = "color";
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
![img_1.png](img_1.png)效果图

四、背景更换
private static final String[] PROJECTION =
new String[] {
NotePad.Notes._ID,
NotePad.Notes.COLUMN_NAME_TITLE,
NotePad.Notes.COLUMN_NAME_NOTE,
NotePad.Notes.COLUMN_NAME_BACK_COLOR
};
添加更换背景的图标
<item android:id="@+id/menu_color"
android:title="@string/menu_color"
android:icon="@drawable/ic_menu_color"
android:showAsAction="always"/>
换背景颜色选项
case R.id.menu_color:
changeColor();
break;

主要类
public class NoteColor extends Activity {
private Cursor mCursor;
private Uri mUri;
private int color;
private static final int COLUMN_INDEX_TITLE = 1;
private static final String[] PROJECTION = new String[] {
NotePad.Notes._ID, // 0
NotePad.Notes.COLUMN_NAME_BACK_COLOR,
};
public void onCreate(Bundle savedInstanceState) {
super.onCreate(savedInstanceState);
setContentView(R.layout.note_color);
//从NoteEditor传入的uri
mUri = getIntent().getData();
mCursor = managedQuery(
mUri,        // The URI for the note that is to be retrieved.
PROJECTION,  // The columns to retrieve
null,        // No selection criteria are used, so no where columns are needed.
null,        // No where columns are used, so no where values are needed.
null         // No sort order is needed.
);
}
@Override
protected void onResume(){
//执行顺序在onCreate之后
if (mCursor != null) {
mCursor.moveToFirst();
color = mCursor.getInt(COLUMN_INDEX_TITLE);
}
super.onResume();
}
@Override
protected void onPause() {
//执行顺序在finish()之后，将选择的颜色存入数据库
super.onPause();
ContentValues values = new ContentValues();
values.put(NotePad.Notes.COLUMN_NAME_BACK_COLOR, color);
getContentResolver().update(mUri, values, null, null);
}
public void white(View view){
color = NotePad.Notes.DEFAULT_COLOR;
finish();
}
public void yellow(View view){
color = NotePad.Notes.YELLOW_COLOR;
finish();
}
public void blue(View view){
color = NotePad.Notes.BLUE_COLOR;
finish();
}
public void green(View view){
color = NotePad.Notes.GREEN_COLOR;
finish();
}
public void red(View view){
color = NotePad.Notes.RED_COLOR;
finish();
}
}
换背景色
<activity android:name="NoteColor"
android:theme="@android:style/Theme.Holo.Light.Dialog"
android:label="ChangeColor"
android:windowSoftInputMode="stateVisible"/>
![img_2.png](img_2.png)效果图
