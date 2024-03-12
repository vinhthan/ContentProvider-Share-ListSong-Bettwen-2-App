# ContentProvider-Share-ListSong-Bettwen-2-App

Để hiểu rõ hơn về cách ứng dụng A truy cập dữ liệu danh sách bài hát từ ContentProvider của ứng dụng B, hãy xem ví dụ sau:

Ứng dụng B (ContentProvider):
a. Triển khai Song và SongProvider:
// Song.kt
data class Song(val id: Long, val title: String, val artist: String)

// SongProvider.kt
class SongProvider : ContentProvider() {

    companion object {
        const val AUTHORITY = "com.example.appb.provider"
        val CONTENT_URI: Uri = Uri.parse("content://$AUTHORITY/songs")
    }

    private lateinit var dbHelper: SongDatabaseHelper

    override fun onCreate(): Boolean {
        dbHelper = SongDatabaseHelper(context!!)
        return true
    }

    override fun query(
        uri: Uri, projection: Array<String>?, selection: String?,
        selectionArgs: Array<String>?, sortOrder: String?
    ): Cursor? {
        val db = dbHelper.readableDatabase
        return db.query("songs", projection, selection, selectionArgs, null, null, sortOrder)
    }

    // ... (các phương thức khác của ContentProvider)

    private class SongDatabaseHelper(context: Context) :
        SQLiteOpenHelper(context, DATABASE_NAME, null, DATABASE_VERSION) {

        override fun onCreate(db: SQLiteDatabase?) {
            db?.execSQL(
                "CREATE TABLE songs (" +
                        "_id INTEGER PRIMARY KEY AUTOINCREMENT, " +
                        "title TEXT, " +
                        "artist TEXT);"
            )
        }

        override fun onUpgrade(db: SQLiteDatabase?, oldVersion: Int, newVersion: Int) {
            db?.execSQL("DROP TABLE IF EXISTS songs")
            onCreate(db)
        }

        companion object {
            const val DATABASE_NAME = "songs.db"
            const val DATABASE_VERSION = 1
        }
    }
}


b. Đăng ký ContentProvider trong AndroidManifest.xml:
<application>
    <!-- ... -->
    <provider
        android:name=".SongProvider"
        android:authorities="com.example.appb.provider"
        android:exported="false"/>
</application>

Ứng dụng A:
a. Sử dụng ContentProvider từ ứng dụng A:
class MainActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        // Xác định URI của ContentProvider cung cấp dữ liệu từ ứng dụng B
        val contentUri = Uri.parse("content://com.example.appb.provider/songs")

        // Sử dụng ContentResolver để truy vấn dữ liệu từ ứng dụng B
        val cursor = contentResolver.query(contentUri, null, null, null, null)

        // Xử lý dữ liệu từ Cursor
        if (cursor != null) {
            while (cursor.moveToNext()) {
                val id = cursor.getLong(cursor.getColumnIndex("_id"))
                val title = cursor.getString(cursor.getColumnIndex("title"))
                val artist = cursor.getString(cursor.getColumnIndex("artist"))

                // Xử lý dữ liệu ở đây
                Log.d("SongInfo", "ID: $id, Title: $title, Artist: $artist")
            }
            cursor.close()
        }
    }
}

Trong ví dụ này, ứng dụng A sử dụng ContentResolver để truy vấn dữ liệu từ ContentProvider của ứng dụng B thông qua URI content://com.example.appb.provider/songs.

Hãy chắc chắn rằng android:authorities trong AndroidManifest.xml của ứng dụng B là "com.example.appb.provider" và bạn đã đăng ký ContentProvider của mình đúng cách.

Lưu ý rằng việc giao tiếp giữa các ứng dụng thông qua ContentProvider yêu cầu quyền truy cập và khai báo chính xác trong tệp AndroidManifest.xml của ứng dụng cung cấp dữ liệu (ứng dụng B).
