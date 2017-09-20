# Labor 05 - Rajzoló alkalmazás készítése

## Bevezető

A labor során egy egyszerű rajzoló alkalmazás elkészítése a feladat. Az alkalmazással egy vászonra lehet vonalakat vagy 
pontokat rajzolni. Ezen kívül szükséges a rajzolt ábrát perzisztensen elmenteni, hogy az alkalmazás újraindítása után 
is visszatöltődjön.

<p align="center">
<img src="./assets/main_screen.png" width="160">
<img src="./assets/color_selector.png" width="160">
<img src="./assets/style_selector.png" width="160">
</p>

## Feltöltés

Az elkészült megoldást `.zip` formátumban (teljes Android Studio projekt – build mappa kivehető) kell feltölteni a tárgy
oldalán, ahol a laborvezető tudja értékelni.

## A projekt előkészítése

### A projekt létrehozása

Hozzunk létre egy új Android projektet. Az _Application name_ mezőben válasszuk a `Simple Drawer` nevet az alkalmazás 
nevének, a _Company domain_ mezőnek pedig adjuk meg az `aut.bme.hu` értéket. Láthatjuk, hogy a _Package név_ ennek 
megfelelően a következőképp alakul: `hu.bme.aut.simpledrawer`. Ezután nyomjunk a **Next** gombra.

A következő képernyőn hagyjuk meg az alapértelmezett beállításokat (`Phone and tablet/API 15`), majd ismét nyomjunk a 
**Next** gombra.

Válasszuk ki az _Empty activity_-t, majd kattintsunk a **Next** gombra.

_Activity name_-nek adjuk meg, hogy `DrawingActivity`, és hagyjuk bepipálva azt, hogy generáljon _layout_ fájlt és azt, 
hogy használja az _AppCompat_-ot. Ha ezekkel megvagyunk, akkor rányomhatunk a **Finish**-re.

Miután létrejött a projekt, töröljük ki a teszt package-eket, mert most nem lesz rá szükségünk.

### A resource-ok hozzáadása

Először töltsük le [az alkalmazás képeit tartalmazó tömörített fájlt](./downloads/res.zip), ami tartalmazza az összes 
képet, amire szükségünk lesz. A tartalmát másoljuk be az `app/src/main/res` mappába (ehhez segít, ha _Android Studio_-ban 
bal fent a szokásos _Android_ nézetről a _Project_ nézetre váltunk erre az időre).

Az alábbi, alkalmazáshoz szükséges _string resource_-okat másoljuk be a `res/values/strings.xml` fájlba:

```xml
<resources>
   <string name="app_name">Simple Drawer</string>
   
   <string name="style">Stílus</string>
   <string name="line">Vonal</string>
   <string name="point">Pont</string>
   
   <string name="are_you_sure_want_to_exit">Biztosan ki akarsz lépni?</string>
   <string name="ok">OK</string>
   <string name="cancel">Mégse</string>
</resources>
```

### Álló layout kikényszerítése

Az alkalmazásunkban az egyszerűség kedvéért most csak az álló módot támogatjuk. Ehhez az `AndroidManifest.xml`-ben a 
`DrawingActivity` nyitótagjához kell hozzáadni egy sort a következő módon:

```xml
<activity
    android:name=".DrawingActivity"
    android:screenOrientation="portrait">
```

## Laborfeladatok

A labor során a következő feladatokat kell megvalósítani a laborvezető segítségével, illetve bizonyos feladatokat 
önállóan. A labor végén lehetőség van **iMSc** pontokat is szerezni a jelölt feladatok megoldásával.

### 1. feladat: A kezdő layout létrehozása (1 pont)

Első lépésként hozzunk létre egy új _package_-et az `hu.be.aut.simpledrawer`-en belül, aminek adjuk a `view` nevet.
Ebben hozzunk létre egy új osztályt, amit nevezzünk el `DrawingView`-nak, és származzon le a `View` osztályból.

Hozzuk létre a szükséges konstruktort ezen belül:

```java
public class DrawingView extends View {
    public DrawingView(final Context context, final AttributeSet attrs) {
         super(context, attrs);
    }
}
```

Miután létrehoztuk a `DrawingView`-t, nyissuk meg a `res/layout/activity_drawing.xml`-t, és hozzunk létre gyökérelemként 
egy `RelativeLayout`-ot, azon belül pedig felülre a frissen létrehozott `DrawingView`-nkból helyezzünk el egy példányt 
fekete háttérrel, alulra pedig egy `Toolbar`-t rakjunk ki. Technikai okokból a `Toolbar`-t kell előbb definiálni, és 
csak ezután a `DrawingView`-t, különben utóbbi nem látja az előbbi ID-ját. Végezetül a layoutnak így kell kinéznie:

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
   android:layout_width="match_parent"
   android:layout_height="match_parent">
   
   <android.support.v7.widget.Toolbar
       android:id="@+id/toolbar"
       android:layout_width="match_parent"
       android:layout_height="wrap_content"
       android:layout_alignParentBottom="true"
       android:background="@color/colorPrimary" />
   
   <hu.bme.aut.simpledrawer.view.DrawingView
       android:id="@+id/canvas"
       android:layout_width="match_parent"
       android:layout_height="wrap_content"
       android:layout_above="@id/toolbar"
       android:background="@android:color/black" />
</RelativeLayout>
```

### 2. feladat: Stílusválasztó (2 pont)

Miután létrehoztuk a rajzolás tulajdonságainak állításáért felelős `Toolbar`-t, hozzuk létre a menüt, amivel be lehet 
állítani, hogy pontot vagy vonalat rajzoljunk. Ehhez hozzunk létre egy új _Android resource directory_-t `menu` néven a `res` mappában, 
és _Resource type_-nak is válasszuk azt, hogy `menu`. Ezen belül hozzunk létre egy új _Menu resource file_-t 
`toolbar_menu.xml` néven. Ebben hozzunk létre az alábbi hierarchiát:

```xml
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">
    <item
        android:id="@+id/menu_style"
        android:checkableBehavior="single"
        android:icon="@drawable/ic_style"
        android:title="@string/style"
        app:showAsAction="ifRoom">
        <menu>
            <group android:checkableBehavior="single">
                <item
                    android:id="@+id/menu_style_line"
                    android:checked="true"
                    android:title="@string/line" />
                <item
                    android:id="@+id/menu_style_point"
                    android:checked="false"
                    android:title="@string/point" />
            </group>
        </menu>
    </item>
</menu>
```

Ezután kössük be a menüt, hogy megjelenjen a `Toolbar`-on. Ehhez a `DrawingActivity`-ben definiáljuk felül az _Activity_ 
`onCreateOptionsMenu()` és `onOptionsItemSelected()` függvényét az alábbi módon:

```java
@Override
public boolean onCreateOptionsMenu(final Menu menu) {
    final Toolbar toolbar = (Toolbar) findViewById(R.id.toolbar);
    final Menu toolbarMenu = toolbar.getMenu();
    getMenuInflater().inflate(R.menu.toolbar_menu, toolbarMenu);
    for (int i = 0; i < toolbarMenu.size(); i++) {
        final MenuItem menuItem = toolbarMenu.getItem(i);
        menuItem.setOnMenuItemClickListener(new MenuItem.OnMenuItemClickListener() {
            @Override
            public boolean onMenuItemClick(final MenuItem item) {
                return onOptionsItemSelected(item);
            }
        });
        if (menuItem.hasSubMenu()) {
            final SubMenu subMenu = menuItem.getSubMenu();
            for (int j = 0; j < subMenu.size(); j++) {
                subMenu.getItem(j).setOnMenuItemClickListener(new MenuItem.OnMenuItemClickListener() {
                    @Override
                    public boolean onMenuItemClick(final MenuItem item) {
                        return onOptionsItemSelected(item);
                    }
                });
            }
        }
    }
    return super.onCreateOptionsMenu(menu);
}
```
```java
@Override
public boolean onOptionsItemSelected(final MenuItem item) {
    switch (item.getItemId()) {
        case R.id.menu_style_line:
            item.setChecked(true);
            return true;
        case R.id.menu_style_point:
            item.setChecked(true);
            return true;
        default:
            return super.onOptionsItemSelected(item);
    }
}
```

### 3. feladat: A `DrawingView` osztály implementálása

#### A modellek létrehozása

A rajzprogramunk, ahogy az már az előző feladatban is kiderült, kétféle rajzolási stílust fog támogatni. Nevezetesen a 
pont- és vonalrajzolást. Ahhoz, hogy a rajzolt alakzatokat el tudjuk tárolni szükségünk lesz két új típusra, modellre, 
amihez hozzunk létre egy új _package_-et a `hu.be.aut.simpledrawer`-en belül `model` néven.

Ezen belül először hozzunk létre egy `Point` osztályt, ami értelemszerűen a pontokat fogja reprezentálni.

```java
public class Point {
    private float x;
    
    private float y;
    
    public Point() {
    }
    
    public Point(final float x, final float y) {
        this.x = x;
        this.y = y;
    }
    
    public float getX() {
        return x;
    }
    
    public void setX(final float x) {
        this.x = x;
    }
    
    public float getY() {
        return y;
    }
    
    public void setY(final float y) {
        this.y = y;
    }
}
```

Miután ezzel megvagyunk, hozzunk létre egy `Line` osztályt. Mivel egy vonalat a két végpontjának megadásával ki tudunk 
rajzoltatni, így elegendő két `Point`-ot tartalmaznia az osztálynak.

```java
public class Line {
    private Point start;
    
    private Point end;
    
    public Line() {
    }
    
    public Line(final Point start, final Point end) {
        this.start = start;
        this.end = end;
    }
     
    public Point getStart() {
        return start;
    }
    
    public void setStart(final Point start) {
        this.start = start;
    }
    
    public Point getEnd() {
        return end;
    }
    
    public void setEnd(final Point end) {
        this.end = end;
    }
}
```

#### A rajzolási stílus beállítása

Most, hogy megvannak a modelljeink el lehet kezdeni magának a rajzolás funkciójának fejlesztését. Ehhez a `DrawingView` 
osztályt fogjuk ténylegesen is elkészíteni. Először vegyünk fel az osztályon belül egy `enum`-ot `DrawingStyle` néven, 
ami a rajzolási stílust fogja meghatározni. Ehhez kapcsolódóan vegyünk fel egy új `field`-et az osztályunkba, amiben 
eltároljuk, hogy jelenleg milyen stílus van kiválasztva. Ehhez írjunk egy _setter_-t, amivel kívülről is be lehet 
állítani a rajzolási stílust.

```java
public enum DrawingStyle {
    LINE,
    POINT
}

private DrawingStyle currentDrawingStyle = DrawingStyle.LINE;

public void setDrawingStyle(final DrawingStyle drawingStyle) {
    this.currentDrawingStyle = drawingStyle;
}
```

Ha ezek megvannak, akkor egészítsük ki a `DrawingActivity`-ben a menükezelést, úgy, hogy a megfelelő függvények 
hívódjanak meg.
 
Ehhez először egy referenciát kell létrehoznunk az `xml`-ben definiált `canvas` ID-jú `DravingView`-ra. Ehhez létre kell 
hozni egy úgy `field`-et a `DrawingActivity`-n belül, majd az `onCreate()` függvényen belül ennek értéket kell adni.

```java
private DrawingView canvas;
 
@Override
protected void onCreate(final Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_drawing);
    canvas = (DrawingView) findViewById(R.id.canvas);
}
```
Ezután már csak az `onOptionsItemSelected()` függvégy megfelelő `case` ágában meg kell hívnunk a `canvas`-ra a 
`setDrawingStyle()` függvényt a megfelelő paraméterrel.

```java
@Override
public boolean onOptionsItemSelected(final MenuItem item) {
    switch (item.getItemId()) {
        case R.id.menu_style_line:
            canvas.setDrawingStyle(DrawingView.DrawingStyle.LINE);
            item.setChecked(true);
            return true;
        case R.id.menu_style_point:
            canvas.setDrawingStyle(DrawingView.DrawingStyle.POINT);
            item.setChecked(true);
            return true;
        default:
            return super.onOptionsItemSelected(item);
    }
}
```
#### Inicializálások

A rajzolási funkció megvalósításához fel kell vennünk néhány további `field`-et a `DrawingView` osztályban, amiket a konstruktorban inicializálnunk kell.

```java
private Paint paint;
 
private Point startPoint;
 
private Point endPoint;
 
private List<Line> lines;
 
private List<Point> points;
 
public DrawingView(final Context context, final AttributeSet attrs) {
    super(context, attrs);
    initPaint();
    initLists();
}
 
private void initPaint() {
    paint = new Paint();
    paint.setColor(Color.GREEN);
    paint.setStyle(Paint.Style.STROKE);
    paint.setStrokeWidth(5);
}
 
private void initLists() {
    lines = new ArrayList<>();
    points = new ArrayList<>();
}
```

#### Gesztusok kezelése

Ahhoz, hogy vonalat vagy pontot tudjunk rajzolni a `View`-nkra, kezelnünk kell a felhasználótól kapott gesztusokat, mint 
például amikor hozzáér a kijelzőhöz, elhúzza rajta, illetve felemeli róla az ujját. Szerencsére ezeket a gesztusokat nem 
szükséges manuálisan felismernünk és lekezelnünk, a `View` osztály `onTouchEvent()` függvényének felüldefiniálásával 
egyszerűen megolható a feladat.

```java
@Override
public boolean onTouchEvent(final MotionEvent event) {
    endPoint = new Point(event.getX(), event.getY());
    switch (event.getAction()) {
        case MotionEvent.ACTION_DOWN:
            startPoint = new Point(event.getX(), event.getY());
            break;
        case MotionEvent.ACTION_MOVE:
            break;
        case MotionEvent.ACTION_UP:
            switch (currentDrawingStyle) {
                case POINT:
                    addPointToTheList(endPoint);
                    break;
                case LINE:
                default:
                    addLineToTheList(startPoint, endPoint);
            }
            startPoint = null;
            endPoint = null;
            break;
        default:
            return false;
    }
    invalidate();
    return true;
}
 
private void addPointToTheList(final Point startPoint) {
    points.add(startPoint);
}
 
private void addLineToTheList(final Point startPoint, final Point endPoint) {
    lines.add(new Line(startPoint, endPoint));
}
```

Ahogy a fenti kódrészletből is látszik minden gesztusnál elmentjük az adott `TouchEvent` pontját, mint a rajzolás 
végpontját, illetve ha `MotionEvent.ACTION_DOWN` történt, tehát a felhasználó hozzáért a `View`-hoz, elmentjük ezt 
kezdőpontként is. Amíg a felhasználó mozgatja az ujját a `View`-n (`MotionEvent.ACTION_MOVe`), addig nem csinálunk 
semmit, de amint felemeli (`MotionEvent.ACTION_UP`), elmentjük az adott elemet a korábban már definiált listákba. Ezen 
kívül minden egyes alkalommal meghívjuk az `invalidate()` függvényt, ami kikényszeríti a `View` újrarajzolását.

#### A rajzolás

A rajzolás megvalósításához a `View` ősosztály `onDraw()` metódusát kell felüldefiniálnunk. Egyrészt ki kell rajzolnunk 
a már meglévő objektumokat (amiket a `MotionEvent.ACTION_UP` eseménynél beleraktunk a listába), valamint ki kell 
rajzolnunk az aktuális kezdőpont (a `MotionEvent.ACTION_DOWN` eseménytől) és a felhasználó ujja közötti vonalat.

```java
@Override
protected void onDraw(final Canvas canvas) {
    super.onDraw(canvas);
 
    for (final Point point : points) {
        drawPoint(canvas, point);
    }
    for (final Line line : lines) {
        drawLine(canvas, line.getStart(), line.getEnd());
    }
 
    switch (currentDrawingStyle) {
        case POINT:
            drawPoint(canvas, endPoint);
            break;
        case LINE:
        default:
            drawLine(canvas, startPoint, endPoint);
    }
}
 
private void drawPoint(final Canvas canvas, final Point point) {
    if (point == null) {
        return;
    }
    canvas.drawPoint(point.getX(), point.getY(), paint);
}
 
private void drawLine(final Canvas canvas, final Point startPoint, final Point endPoint) {
    if (startPoint == null || endPoint == null) {
        return;
    }
    canvas.drawLine(startPoint.getX(), startPoint.getY(), endPoint.getX(), endPoint.getY(), paint);
}
```

### 4. feladat: Perzisztencia megvalósítása _SQLite_ adatbázis segítségével (1 pont)

Ahhoz, hogy az általunk rajzolt objektumok megmaradjanak az alkalmazásból való kilépés után is, az adatainkat valahogy 
olyan formába kell rendeznünk, hogy azt könnyedén el tudjuk tárolni egy _SQLite_ adatbázisban. 

Hozzunk létre egy új _package_-et az `hu.be.aut.simpledrawer`-en belül, aminek adjuk az `sqlite` nevet.

#### Táblák definiálása

Az objektumaink adatait tároló táblák létrehozása előtt hozzunk létre egy új _package_-et az `sqlite`-on belül `table` 
néven. Ezen belül először a `Point` osztálynak hozunk létre egy új `PointsTable` nevű táblát. Ezen belül létrehozunk egy 
`enum`-ot, hogy könnyebben tudjuk kezelni a tábla oszlopait, majd konstansokban eltároljuk a tábla létrehozását 
szolgáló _SQL utasítást_ valamint a tábla nevét is. Végezetül elkészítjük azokat a függvényeket, amelyeket a tábla 
létrehozásakor, illetve upgrade-elésekor kell meghívni.

```java
public class PointsTable {
    public static final String TABLE_POINTS = "points";
 
    private static final String DATABASE_CREATE = "create table " + TABLE_POINTS + "("
            + Columns._id.name() + " integer primary key autoincrement, "
            + Columns.coord_x.name() + " real not null, "
            + Columns.coord_y.name() + " real not null" + ");";
 
    public static void onCreate(final SQLiteDatabase database) {
        database.execSQL(DATABASE_CREATE);
    }
 
    public static void onUpgrade(final SQLiteDatabase database, final int oldVersion, final int newVersion) {
        Log.w(PointsTable.class.getName(), "Upgrading from version " + oldVersion + " to " + newVersion);
        database.execSQL("DROP TABLE IF EXISTS " + TABLE_POINTS);
        onCreate(database);
    }
 
    public enum Columns {
        _id,
        coord_x,
        coord_y
    }
}
```

A `Line` osztálynak a fentiekhez hasonlóan készítsük el a `LinesTable` táblát.

```java
public class LinesTable {
    public static final String TABLE_LINES = "lines";
 
    private static final String DATABASE_CREATE = "create table " + TABLE_LINES + "("
            + Columns._id.name() + " integer primary key autoincrement, "
            + Columns.start_x.name() + " real not null, "
            + Columns.start_y.name() + " real not null, "
            + Columns.end_x.name() + " real not null, "
            + Columns.end_y.name() + " real not null" + ");";
 
    public static void onCreate(final SQLiteDatabase database) {
        database.execSQL(DATABASE_CREATE);
    }
 
    public static void onUpgrade(final SQLiteDatabase database, final int oldVersion, final int newVersion) {
        Log.w(LinesTable.class.getName(), "Upgrading from version " + oldVersion + " to " + newVersion);
        database.execSQL("DROP TABLE IF EXISTS " + TABLE_LINES);
        onCreate(database);
    }
 
    public enum Columns {
        _id,
        start_x,
        start_y,
        end_x,
        end_y
    }
}
```

#### A segédosztályok létrehozása

Az adatbázis létrehozásához szükség van egy olyan segédosztályra, ami létrehozza magát az adatbázist, és azon belül 
inicializálja a táblákat is. Esetünkben ez lesz a `DBHelper` osztály, ami az `SQLiteOpenHelper` osztályból származik. 
Konstansként felvesszük az adatbázis nevét és verzióját is. Ha az adatbázisunk sémáján szeretnénk változtatni, akkor ez 
utóbbit kell inkrementálnunk, így elkerülhetjük az inkompatibilitás miatti nem kívánatos hibákat.

```java
public class DBHelper extends SQLiteOpenHelper {
    private static final String DATABASE_NAME = "simpledrawer.db";
 
    private static final int DATABASE_VERSION = 1;
 
    public DBHelper(final Context context) {
        super(context, DATABASE_NAME, null, DATABASE_VERSION);
    }
 
    @Override
    public void onCreate(final SQLiteDatabase sqLiteDatabase) {
        LinesTable.onCreate(sqLiteDatabase);
        PointsTable.onCreate(sqLiteDatabase);
    }
 
    @Override
    public void onUpgrade(final SQLiteDatabase sqLiteDatabase, final int oldVersion, final int newVersion) {
        LinesTable.onUpgrade(sqLiteDatabase, oldVersion, newVersion);
        PointsTable.onUpgrade(sqLiteDatabase, oldVersion, newVersion);
    }
}
```

Ezen kívül szükségünk van még egy olyan segédosztályra is, ami ezt az egészet összefogja, és amivel egyszerűen tudjuk 
kezelni az adatbázisunkat. Ez lesz a `PersistentDataHelper`. Ebben olyan függényeket fogunk megvalósítani, mint pl. az 
`open()` és a `close()`, amikkel az adatbáziskapcsolatot nyithatjuk meg, illetve zárhatjuk le. Ezen kívül ebben az 
osztályban valósítjuk meg azokat a függvényeket is, amik az adatok adatbázisba való kiírásáért, illetve az onnan való 
kiolvasásáért felelősek.

```java
public class PersistentDataHelper {
    private SQLiteDatabase database;
 
    private DBHelper dbHelper;
 
    private String[] pointColumns = {
            PointsTable.Columns._id.name(),
            PointsTable.Columns.coord_x.name(),
            PointsTable.Columns.coord_y.name()
    };
 
    private String[] lineColumns = {
            LinesTable.Columns._id.name(),
            LinesTable.Columns.start_x.name(),
            LinesTable.Columns.start_y.name(),
            LinesTable.Columns.end_x.name(),
            LinesTable.Columns.end_y.name()
    };
 
    public PersistentDataHelper(final Context context) {
        dbHelper = new DBHelper(context);
    }
 
    public void open() throws SQLiteException {
        database = dbHelper.getWritableDatabase();
    }
 
    public void close() {
        dbHelper.close();
    }
 
    public void persistPoints(final List<Point> points) {
        clearPoints();
        for (final Point point : points) {
            final ContentValues values = new ContentValues();
            values.put(PointsTable.Columns.coord_x.name(), point.getX());
            values.put(PointsTable.Columns.coord_y.name(), point.getY());
            database.insert(PointsTable.TABLE_POINTS, null, values);
        }
    }
 
    public List<Point> restorePoints() {
        final List<Point> points = new ArrayList<>();
        final Cursor cursor = database.query(PointsTable.TABLE_POINTS, pointColumns, null, null, null, null, null);
        cursor.moveToFirst();
        while (!cursor.isAfterLast()) {
            final Point point = cursorToPoint(cursor);
            points.add(point);
            cursor.moveToNext();
        }
        cursor.close();
        return points;
    }
 
    public void clearPoints() {
        database.delete(PointsTable.TABLE_POINTS, null, null);
    }
 
    private Point cursorToPoint(final Cursor cursor) {
        final Point point = new Point();
        point.setX(cursor.getFloat(LinesTable.Columns.start_x.ordinal()));
        point.setY(cursor.getFloat(LinesTable.Columns.start_y.ordinal()));
        return point;
    }
 
    public void persistLines(final List<Line> lines) {
        clearLines();
        for (final Line line : lines) {
            final ContentValues values = new ContentValues();
            values.put(LinesTable.Columns.start_x.name(), line.getStart().getX());
            values.put(LinesTable.Columns.start_y.name(), line.getStart().getY());
            values.put(LinesTable.Columns.end_x.name(), line.getEnd().getX());
            values.put(LinesTable.Columns.end_y.name(), line.getEnd().getY());
            database.insert(LinesTable.TABLE_LINES, null, values);
        }
    }
 
    public List<Line> restoreLines() {
        final List<Line> points = new ArrayList<>();
        final Cursor cursor = database.query(LinesTable.TABLE_LINES, lineColumns, null, null, null, null, null);
        cursor.moveToFirst();
        while (!cursor.isAfterLast()) {
            final Line line = cursorToLine(cursor);
            points.add(line);
            cursor.moveToNext();
        }
        cursor.close();
        return points;
    }
 
    public void clearLines() {
        database.delete(LinesTable.TABLE_LINES, null, null);
    }
 
    private Line cursorToLine(final Cursor cursor) {
        final Line line = new Line();
        final Point startPoint = new Point();
        startPoint.setX(cursor.getFloat(LinesTable.Columns.start_x.ordinal()));
        startPoint.setY(cursor.getFloat(LinesTable.Columns.start_y.ordinal()));
        line.setStart(startPoint);
        final Point endPoint = new Point();
        endPoint.setX(cursor.getFloat(LinesTable.Columns.end_x.ordinal()));
        endPoint.setY(cursor.getFloat(LinesTable.Columns.end_y.ordinal()));
        line.setEnd(endPoint);
        return line;
    }
}
```

#### A `DrawingView` kiegészítése

Ahhoz, hogy a rajzolt objektumainkat el tudjuk menteni az adatbázisba, fel kell készíteni a `DrawingView` osztályunkat 
arra, hogy átadja, illetve meg lehessen adni neki kívülről is őket. Ehhez a következő függvényeket kell felvennünk:

```java
public void restoreObjects(final List<Point> points, final List<Line> lines) {
    this.points = points;
    this.lines = lines;
    invalidate();
}
 
public List<Point> getPoints() {
    return points;
}
 
public List<Line> getLines() {
    return lines;
}
```

#### A `DrawingActivity` kiegészítése

A perzisztencia megvalósításához már csak egy feladatunk maradt hátra, mégpedig az, hogy bekössük a frissen létrehozott 
osztályainkat az `DrawerActivity`-nkbe. Ehhez először is példányosítanunk kell a `PersistentDataHelper` osztályunkat. 
Mivel az adatbázishozzáférés drága erőforrás, ezért ne felejtsük el az `Activity` `onResume()` függvényében megnyitni, 
az `onPause()` függvényében pedig lezárni a vele való kapcsolatot. Alább találhatók a szükséges módosítások:

```java
private PersistentDataHelper dataHelper;
 
@Override
protected void onCreate(final Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_drawing);
    canvas = (DrawingView) findViewById(R.id.canvas);
    dataHelper = new PersistentDataHelper(this);
    dataHelper.open();
    restorePersistedObjects();
}
 
@Override
protected void onResume() {
    super.onResume();
    dataHelper.open();
}
 
@Override
protected void onPause() {
    dataHelper.close();
    super.onPause();
}
 
private void restorePersistedObjects() {
    canvas.restoreObjects(dataHelper.restorePoints(), dataHelper.restoreLines());
}
```

Végezetül szeretnénk, hogy amikor a felhasználó ki szeretne lépni az alkalmazásból, akkor egy dialógusablak jelenjen meg, hogy biztos kilép-e, és ha igen, csak abban az esetben mentsük el a rajzolt objektumokat, és lépjünk ki az alkalmazásból. Ehhez felül kell definiálnunk az `Activity` `onBackPressed()` függvényét.

```java
@Override
public void onBackPressed() {
    new AlertDialog.Builder(this)
            .setMessage(R.string.are_you_sure_want_to_exit)
            .setPositiveButton(R.string.ok, new DialogInterface.OnClickListener() {
                @Override
                public void onClick(final DialogInterface dialogInterface, final int i) {
                    onExit();
                }
            })
            .setNegativeButton(R.string.cancel, null)
            .show();
}
 
private void onExit() {
    dataHelper.persistPoints(canvas.getPoints());
    dataHelper.persistLines(canvas.getLines());
    dataHelper.close();
    finish();
}
```

### 5. (önálló) feladat: A vászon törlése (1 pont)

Vegyünk fel a vezérlők közé egy olyan gombot, amelynek segíségével a törölhetjük a vásznat.

## Kiegészítő iMSc feladat (2 iMSc pont)

Vegyünk fel az alkalmazásba egy olyan vezérlőt, amivel változtatni lehet a rajzolás színét a 3 fő szín között (_RGB_).

**Figyelem:** az adatbázisban is el kell menteni az adott objektum színét!
