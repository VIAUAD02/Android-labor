# Labor 02 - Felhasználói felület készítés

## Bevezető

A labor során egy tömegközlekedési vállalat számára megálmodott alkalmazás vázát készítjük el. Az alkalmazással a felhasználók különböző járművekre vásárolhatnak majd bérleteket. Az üzleti logikát (az authentikációt, a bevitt adatok ellenőrzését, a fizetés lebonyolítását) egyelőre csak szimulálni fogjuk, a labor fókusza a felületek és a köztük való navigáció elkészítése lesz.

<p align="center">
<img src="./assets/login.jpg" width="160">
<img src="./assets/list.jpg" width="160">
<img src="./assets/details.jpg" width="160">
<img src="./assets/pass.jpg" width="160">
</p>

## IMSc pontok

A laborfeladatok sikeres befejezése után az IMSc feladat-ot megoldva 2 IMSc pont szerezhető.

## Értékelés

Osztályzás:
- Splash képernyő: 0.5 pont
- Login képernyő: 0.5 pont
- Lehetőségek listája: 1 pont
- Részletes nézet: 1 pont
- A bérlet: 1 pont
- Önálló feladat (hajó bérlet): 1 pont

IMSc:
- Különböző bérlet napi árak: 1 IMSc pont
- Százalékos kedvezmények: 1 IMSc pont

## Projekt létrehozása

Hozzunk létre egy új Android projektet! Az alkalmazás neve legyen `PublicTransport`, a Company domain pedig `aut.bme.hu`. Láthatjuk, hogy ez alapján automatikusan a `hu.bme.aut.publictransport` package-et kapja az alkalmazás. 

Az alkalmazást természetesen telefonra készítjük, és használhatjuk az alapértelmezett 15-ös minimum SDK szintet.

Az első Activity-nk legyen egy Empty Activity, és nevezzük el `LoginActivity`-nek. A hozzá tartozó layout fájl automatikusan megkapja az `activity_login.xml` nevet.

## Splash képernyő

Az első Activity-nk a nevéhez híven a felhasználó bejelentkezéséért lesz felelős, azonban még mielőtt ez megjelenik a felhasználó számára, egy splash képernyővel fogjuk üdvözölni. Ez egy elegáns megoldás arra, hogy az alkalmazás betöltéséig ne egy egyszínű képernyő legyen a felhasználó előtt, hanem egy tetszőleges saját design.

<p align="center"> 
<img src="./assets/splash.jpg" width="320">
</p>

Először töltsük le [az alkalmazáshoz képeit tartalmazó tömörített fájlt](./downloads/res.zip), ami tartalmazza az összes képet, amire szükségünk lesz. A tartalmát másoljuk be az `app/src/main/res` mappába (ehhez segít, ha Android Studio-ban bal fent a szokásos Android nézetről a Project nézetre váltunk).

Hozzunk létre egy új XML fájlt a `drawable` mappában `splash_background.xml` néven. Ez lesz a splash képernyőnkön megjelenő grafika. A tartalma az alábbi legyen:

```xml
<?xml version="1.0" encoding="utf-8"?>
<layer-list xmlns:android="http://schemas.android.com/apk/res/android">

    <item>
        <bitmap
            android:gravity="fill_horizontal|clip_vertical"
            android:src="@drawable/splash_image"/>
    </item>

</layer-list>
```

Jelen esetben egyetlen képet teszünk ide, de további `item`-ek felvételével komplexebb dolgokat is összeállíthatnánk itt. Tipikus megoldás például egy egyszínű háttér beállítása, amin az alkalmazás ikonja látszik.

Nyissuk meg a `values/styles.xml` fájlt. Ez definiálja az alkalmazásban használt különböző témákat. A splash képernyőhöz egy új témát fogunk létrehozni, amelyben az előbb létrehozott drawable-t állítjuk be az alkalmazásablakunk hátterének (mivel ez látszik valójában, amíg nem töltött be a UI többi része). Ezt így tehetjük meg:

```xml
<style name="SplashTheme" parent="Theme.AppCompat.NoActionBar">
    <item name="android:windowBackground">@drawable/splash_background</item>
</style>
```

Ennek használatához az alkalmazásunk manifest fájlját (`AndroidManifest.xml`) kell módosítanunk. Ezt megnyitva láthatjuk, hogy jelenleg a teljes alkalmazás az `AppTheme` nevű témát használja.

```xml
<application
    ...
    android:theme="@style/AppTheme" >
```

Mi ezt nem akarjuk megváltoztatni, hanem csak a `LoginActivity`-nek akarunk egy új témát adni. Ezt így tehetjük meg:

```xml
<activity
    android:name=".LoginActivity"
    android:theme="@style/SplashTheme">
    ...
</activity>
```

Mivel a betöltés után már nem lesz szükségünk erre a háttérre, a `LoginActivity.java` fájlban a betöltés befejeztével visszaállíthatjuk az eredeti témát, amely fehér háttérrel rendelkezik. Ezt az `onCreate` függvény elején tegyük meg:

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    setTheme(R.style.AppTheme);
    ...
}
```

Most már futtathatjuk az alkalmazást, és betöltés közben látnunk kell a berakott képet. A splash képernyő általában akkor hasznos, ha az alkalmazás inicializálása sokáig tart. Mivel a mostani alkalmazásunk még nagyon gyorsan indul el, szimulálhatunk egy kis töltési időt az alábbi módon:

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    try {
        Thread.sleep(1000);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    setTheme(R.style.AppTheme);
    ...
}
```

## Login

Most már elkészíthetjük a login képernyőt. A felhasználótól egy email címet, illetve egy számokból álló jelszót fogunk bekérni, és egyelőre csak azt fogjuk ellenőrizni, hogy beírt-e valamit a mezőkbe.

<p align="center"> 
<img src="./assets/login.jpg" width="320">
</p>

Az `activity_login.xml` fájlba kerüljön az alábbi kód:

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:layout_margin="16dp"
    android:orientation="vertical"
    tools:context="hu.bme.aut.publictransport.LoginActivity">

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center"
        android:layout_margin="16dp"
        android:text="Please enter your credentials" />

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Email" />

    <EditText
        android:id="@+id/etEmailAddress"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Password" />

    <EditText
        android:id="@+id/etPassword"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />

    <Button
        android:id="@+id/btnLogin"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center"
        android:text="Login" />

</LinearLayout>
```

- A használt elrendezés teljesen lineáris, csak egymás alá helyezünk el benne különböző View-kat egy `LinearLayout`-ban. 
- Az `EditText`-eknek és a `Button`-nek adtunk ID-kat, hogy később kódból elérjük őket.

Az alkalmazást újra futtatva megjelenik a layout, azonban most még bármilyen szöveget be tudnunk írni a két beviteli mezőbe. Az `EditText` osztály lehetőséget ad számos speciális input kezelésére, XML kódban az [`inputType`](https://developer.android.com/reference/android/widget/TextView.html#attr_android:inputType) attribútum megadásával. Jelen esetben az email címet kezelő `EditText`-hez a `textEmailAddress` értéket, a másikhoz pedig a `numberPassword` értéket használhatjuk.

```xml
<EditText
    android:id="@+id/etEmailAddress"
    ...
    android:inputType="textEmailAddress" />

<EditText
    android:id="@+id/etPassword"
    ...
    android:inputType="numberPassword" />
```

Ha most kipróbáljuk az alkalmazást, már látjuk a beállítások hatását: 
- A legtöbb billentyűzettel az első mezőhöz most már megjelenik a `@` szimbólum, a másodiknál pedig csak számokat írhatunk be.
- Mivel a második mezőt jelszó típusúnak állítottuk be, a karakterek a megszokott módon elrejtésre kerülnek a beírásuk után.

Még egy dolgunk van ezen a képernyőn, az input ellenőrzése. Ezt a `LoginActivity.java` fájlban tehetjük meg. A layout-unkat alkotó View-kat az `onCreate` függvényben lévő `setContentView` hívás után tudjuk először elérni. Itt az alábbi kóddal szerezhetünk referenciákat a szükséges View-kra, az XML kódban lévő ID-juk felhasználásával:

```java
final EditText etEmailAddress = findViewById(R.id.etEmailAddress);
final EditText etPassword = findViewById(R.id.etPassword);
final Button btnLogin = findViewById(R.id.btnLogin);
```

Ezeket használva már kezelni tudjuk a gomb lenyomását:

```java
btnLogin.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(final View view) {
        if (etEmailAddress.getText().toString().isEmpty()) {
            etEmailAddress.requestFocus();
            etEmailAddress.setError("Please enter your email address");
            return;
        }

        if (etPassword.getText().toString().isEmpty()) {
            etPassword.requestFocus();
            etPassword.setError("Please enter your password");
            return;
        }
        
        // TODO log in
    }
});
```

Amennyiben valamelyik `EditText` üres volt, a `requestFocus` függvény meghívásával aktívvá tesszük, majd a [`setError`](https://developer.android.com/reference/android/widget/TextView.html#setError(java.lang.CharSequence)) függvénnyel kiírunk rá egy hibaüzenetet. Ez egy kényelmes, beépített megoldás input hibák jelzésére. Így nem kell például egy külön `TextView`-t használnunk erre a célra, és abba beleírni a fellépő hibát. Ezt már akár ki is próbálhatjuk, bár helyes adatok megadása esetén még nem történik semmi.

## Lehetőségek listája

A következő képernyőn a felhasználó a különböző járműtípusok közül válaszhat. Egyelőre három szolgáltatás működik a fiktív vállalatunkban: biciklik, buszok, illetve vonatok.

<p align="center"> 
<img src="./assets/list.jpg" width="320">
</p>

Hozzunk ehhez létre egy új Activity-t (New -> Activity -> Empty Activity), nevezzük el `ListActivity`-nek. Most, hogy ez már létezik, menjünk vissza a `LoginActivity` kódjában lévő TODO-hoz, és indítsuk ott el ezt az új Activity-t:

```java
@Override
public void onClick(View view) {
    ...
    
    Intent intent = new Intent(LoginActivity.this, ListActivity.class);
    startActivity(intent);
}
```

Folytassuk a layout elkészítésével a munkát, az `activity_list.xml` tartalmát cseréljük ki az alábbira:

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:weightSum="3"
    tools:context="hu.bme.aut.publictransport.ListActivity">

    <FrameLayout
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1">

    </FrameLayout>

    <FrameLayout
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1">

    </FrameLayout>

    <FrameLayout
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1">

    </FrameLayout>

</LinearLayout>
```

Ismét egy függőleges LinearLayout-ot használunk, most azonban súlyokat adunk meg benne. A gyökérelemben megadjuk, hogy a súlyok összege (`weightSum`) `3` lesz, és mindhárom gyerekének `1`-es súlyt (`layout_weight`), és `0dp` magasságot adunk. Ezzel azt érjük el, hogy három egyenlő részre osztjuk a képernyőt, amit a három `FrameLayout` fog elfoglalni.

A `FrameLayout` egy nagyon egyszerű és gyors elrendezés, amely lényegében csak egymás tetejére teszi a gyerekeiként szereplő View-kat. Ezeken belül egy-egy képet, illetve azokon egy-egy feliratot fogunk elhelyezni. A három sávból az elsőt így készíthetjük el:

```xml
<FrameLayout
    android:layout_width="match_parent"
    android:layout_height="0dp"
    android:layout_weight="1">

    <ImageButton
        android:id="@+id/btnBike"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:gravity="center"
        android:padding="0dp"
        android:scaleType="centerCrop"
        android:src="@drawable/bikes" />

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center"
        android:text="Bike"
        android:textColor="#FFF"
        android:textSize="36sp" />

</FrameLayout>
```

Az itt használt `ImageButton` pont az, aminek hangzik: egy olyan gomb, amelyen egy képet helyezhetünk el. Azt, hogy ez melyik legyen, az `src` attribútummal adtuk meg. Az utána szereplő `TextView` fehér színnel és nagy méretű betűkkel a kép fölé fog kerülni, ebbe írjuk bele a jármű nevét.

Töltsük ki ehhez hasonló módon a másik két `FrameLayout`-ot is, ID-ként használjuk a `@+id/btnBus` és `@+id/btnTrain` értékeket, képnek pedig használhatjuk a korábban már bemásolt `@drawable/bus` és `@drawable/trains` erőforrásokat. Ne felejtsük el a `TextView`-k szövegét is értelemszerűen átírni.

Az Activity Java fájlját megnyitva az alábbi kóddal kikereshetjük a gombjainkat (most is az `onCreate` függvényben):

```java
ImageButton btnBike = findViewById(R.id.btnBike);
ImageButton btnBus = findViewById(R.id.btnBus);
ImageButton btnTrain = findViewById(R.id.btnTrain);
```

Ezek lenyomásának kezelésére később fogunk visszatérni.

Próbáljuk ki az alkalmazásunkat, bejelentkezés után a most elkészített lista nézethez kell jutnunk.

## Részletes nézet

Miután a felhasználó kiválasztotta a kívánt közlekedési eszközt, néhány további opciót fogunk még felajánlani számára. Ezen a képernyőn fogja kiválasztani a bérleten szereplő dátumokat, illetve a rá vonatkozó kedvezményt, amennyiben van ilyen.

<p align="center"> 
<img src="./assets/details.jpg" width="320">
</p>

Hozzuk létre ezt az új Activity-t `DetailsActivity` néven, a layout-ját kezdjük az alábbi kóddal:

```xml
<?xml version="1.0" encoding="utf-8"?>
<ScrollView xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:clipToPadding="false"
    android:padding="16dp"
    android:scrollbarStyle="outsideInset"
    tools:context="hu.bme.aut.publictransport.DetailsActivity">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical">

    </LinearLayout>

</ScrollView>
```

Az eddigiekhez képest itt újdonság, hogy a használt `LinearLayout`-ot egy `ScrollView`-ba tesszük, mivel sok nézetet fogunk egymás alatt elhelyezni, és alapértelmezetten egy `LinearLayout` nem görgethető, így ezek bizonyos eszközökön már a képernyőn kívül lennének.

---

Kezdjük el összerakni a szükséges layout-ot a `LinearLayout` belsejében. Az oldal tetejére elhelyezünk egy címet, amely a kiválasztott jegy típusát fogja megjeleníteni.

```
<TextView
    android:id="@+id/tvTicketType"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:layout_gravity="center"
    android:textSize="24sp" 
    tools:text="Bus ticket" />
```

Az itt használt `tools` névtérrel megadott `text` attribútum hatása csak az előnézetben fog megjelenni, az alkalmazásban ezt majd a Java kódból állítjuk be, az előző képernyőn megnyomott gomb függvényében.

---

Az első beállítás ezen a képernyőn a bérlet érvényességének időtartama lesz. 

Ezt az érvényesség első és utolsó napjának megadásával tesszük, amelyhez a `DatePicker` osztályt használjuk fel. Ez alapértelmezetten egy teljes havi naptár nézetet jelenít meg, azonban a `calendarViewShown="false"` és a `datePickerMode="spinner"` beállításokkal egy kompaktabb, "pörgethető" választót kapunk.

```xml
<TextView
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="Start date" />

<DatePicker
    android:id="@+id/dpStartDate"
    android:layout_width="match_parent"
    android:layout_height="160dp"
    android:calendarViewShown="false"
    android:datePickerMode="spinner" />

<TextView
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="End date" />

<DatePicker
    android:id="@+id/dpEndDate"
    android:layout_width="match_parent"
    android:layout_height="160dp"
    android:calendarViewShown="false"
    android:datePickerMode="spinner" />
```

Ezeknek a `DatePicker`-eknek is adtunk ID-kat, hiszen később szükségünk lesz a Java kódunkban a rajtuk beállított értékekre.

---

Még egy beállítás van hátra, az árkategória kiválasztása - nyugdíjasoknak és közalkalmazottaknak különböző kedvezményeket adunk a jegyek árából.

Mivel ezek közül az opciók közül egyszerre csak egynek akarjuk megengedni a kiválasztását, ezért `RadioButton`-öket fogunk használni, amelyeket Androidon egy `RadioGroup`-pal kell összefognunk, hogy jelezzük, melyikek tartoznak össze.

```
<TextView
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="Price category" />

<RadioGroup
    android:id="@+id/rgPriceCategory"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content">

    <RadioButton
        android:id="@+id/rbFullPrice"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:checked="true"
        android:text="Full price" />

    <RadioButton
        android:id="@+id/rbSenior"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Senior" />

    <RadioButton
        android:id="@+id/rbPublicServant"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Public servant" />

</RadioGroup>
```

Fontos, hogy adjunk ID-t a teljes csoportnak, és a benne lévő minden opciónak is, mivel később ezek alapján tudjuk majd megnézni, hogy melyik van kiválasztva.

---

Végül az oldal alján kiírjuk a kiválasztott bérlet árát, illetve ide kerül a megvásárláshoz használható gomb is. Az árnak egyelőre csak egy fix értéket írunk ki.

```
<TextView
    android:id="@+id/tvPrice"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:layout_gravity="center"
    android:layout_margin="8dp"
    android:text="42000" />

<Button
    android:id="@+id/btnPurchase"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:layout_gravity="center"
    android:layout_margin="8dp"
    android:text="Purchase pass" />
```

---

Meg kell oldanunk még azt, hogy az előző képernyőn tett választás eredménye elérhető legyen a `DetailsActivity`-ben. Ezt úgy tehetjük meg, hogy az Activity indításához használt `Intent`-be teszünk egy azonosítót, amiből kiderül, hogy melyik típust választotta a felhasználó.

Ehhez a `DetailsActivity`-ben vegyünk fel egy konstanst, ami ennek a paraméternek a kulcsaként fog szolgálni:

```java
public static final String KEY_TRANSPORT_TYPE = "KEY_TRANSPORT_TYPE";
```

Ezután menjünk a `ListActivity` kódjához, és vegyünk fel konstansokat a különböző támogatott járműveknek:

```java
public static final int TYPE_BUS = 1;
public static final int TYPE_TRAIN = 2;
public static final int TYPE_BIKE = 3;
```

Most már létrehozhatjuk a gombok listener-jeit, amelyek elindítják a `DetailsActivity`-t, extrának beletéve a kiválasztott típust. Az első gomb listenerjének beállítását így tehetjük meg:

```java
btnBike.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(final View view) {
        Intent intent = new Intent(ListActivity.this, DetailsActivity.class);
        intent.putExtra(DetailsActivity.KEY_TRANSPORT_TYPE, TYPE_BIKE);
        startActivity(intent);
    }
});
```

A másik két gomb listener-je ugyanerre a mintára működik, csupán az átadott típus konstanst kell megváltoztatni bennük. Hozzuk létre ezeket is! (Ezt a viselkedést érdemes lehet később kiszervezni egy külön osztályba, ami implementálja az `OnClickListener` interfészt, de ezt most nem tesszük meg.)

Még hátra van az, hogy a `DetailsActivity`-ben kiolvassuk ezt az átadott paramétert, és megjelenítsük a felhasználónak. Ezt az `onCreate` függvényében tehetjük meg, az Activity indításához használt `Intent` elkérésével, majd az előbbi kulcs használatával:

```java
Intent intent = getIntent();
final int transportType = intent.getIntExtra(KEY_TRANSPORT_TYPE, -1);
```

Ezt az átadott számot még le kell képeznünk egy stringre, ehhez vegyünk fel egy egyszerű segédfüggvényt:

```java
private String getTypeString(int transportType) {
    switch (transportType) {
        case ListActivity.TYPE_BUS:
            return "Bus pass";
        case ListActivity.TYPE_TRAIN:
            return "Train pass";
        case ListActivity.TYPE_BIKE:
            return "Bike pass";
        default:
            return "Unknown pass type";
    }
}
```

Végül pedig az `onCreate` függvénybe visszatérve meg kell keresnünk a megfelelő `TextView`-t, és beállítani a szövegének a függvény által visszaadott értéket:

```java
TextView tvTicketType = findViewById(R.id.tvTicketType);
tvTicketType.setText(getTypeString(transportType));
```

Próbáljuk ki az alkalmazást! A `DetailsActivity`-ben meg kell jelennie a hozzáadott beállításoknak, illetve a tetején a megfelelő jegy típusnak.

## A bérlet

Az alkalmazás utolsó képernyője már kifejezetten egyszerű lesz, ez magát a bérletet fogja reprezentálni. Itt a bérlet típusát és érvényességi idejét fogjuk megjeleníteni, illetve egy QR kódot, amivel ellenőrizni lehet a bérletetet.

<p align="center"> 
<img src="./assets/pass.jpg" width="320">
</p>

Hozzuk létre a szükséges Activity-t, `PassActivity` néven. Ennek az Activity-nek szüksége lesz a jegy típusára és a kiválasztott dátumokra - a QR kód az egyszerűség kedvéért egy fix kép lesz.

Az adatok átadásához először vegyünk fel két kulcsot a `PassActivity`-ben:

```java
public final static String KEY_DATE_STRING = "KEY_DATE_STRING";
public final static String KEY_TYPE_STRING = "KEY_TYPE_STRING";
```

Ezeket az adatokat a `DetailsActivity`-ben kell összekészítenünk és beleraknunk az `Intent`-be. Ehhez először keressük ki a `DetailsActivity` `onCreate` függvényében a View-kat, amikre szükségünk van:

```java
final DatePicker dpStartDate = findViewById(R.id.dpStartDate);
final DatePicker dpEndDate = findViewById(R.id.dpEndDate);
final Button btnPurchase = findViewById(R.id.btnPurchase);
```

Majd adjunk hozzá a vásárlás gombhoz egy listener-t:

```java
btnPurchase.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(final View view) {
        String typeString = getTypeString(transportType);
        String dateString = getDateFrom(dpStartDate) + " - " + getDateFrom(dpEndDate);

        Intent intent = new Intent(DetailsActivity.this, PassActivity.class);
        intent.putExtra(PassActivity.KEY_TYPE_STRING, typeString);
        intent.putExtra(PassActivity.KEY_DATE_STRING, dateString);
        startActivity(intent);
    }
});
```

Ebben összegyűjtjük a szükséges adatokat, és a megfelelő kulcsokkal elhelyezzük őket a `PassActivity` indításához használt `Intent`-ben.

A `getDateFrom` egy segédfüggvény lesz, ami egy `DatePicker`-t kap paraméterként, és formázott stringként visszaadja az éppen kiválasztott dátumot, ennek implementációja a következő:

```java
private String getDateFrom(final DatePicker picker) {
    return String.format(Locale.getDefault(), "%04d.%02d.%02d.",
            picker.getYear(), picker.getMonth() + 1, picker.getDayOfMonth());
}
```

(Itt a hónaphoz azért adtunk hozzá egyet, mert akárcsak a [`Calendar`](https://developer.android.com/reference/java/util/Calendar.html#MONTH) osztály esetében, a `DatePicker` osztálynál is 0 indexelésűek a hónapok.)

---

Most már elkészíthetjük a `PassActivity`-t. Kezdjük a layout-jával (`activity_pass.xml`), aminek már majdnem minden elemét használtuk, az egyetlen újdonság itt az `ImageView` használata.

```xml
<?xml version="1.0" encoding="utf-8"?>
<ScrollView xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context="hu.bme.aut.publictransport.PassActivity">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_margin="16dp"
        android:orientation="vertical">

        <TextView
            android:id="@+id/tvTicketType"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="center"
            android:textSize="24sp"
            tools:text="Train pass" />

        <TextView
            android:id="@+id/tvDates"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="center"
            android:layout_margin="16dp"
            tools:text="1999.11.22. - 2012.12.21." />

        <ImageView
            android:layout_width="300dp"
            android:layout_height="300dp"
            android:layout_gravity="center"
            android:src="@drawable/qrcode" />

    </LinearLayout>

</ScrollView>
```

Az Activity Java kódjában pedig csak a két `TextView` szövegét kell az `Intent`-ben megkapott értékekre állítanunk (természetesen az `onCreate` függvényben):

```java
Intent intent = getIntent();

TextView tvTicketType = findViewById(R.id.tvTicketType);
tvTicketType.setText(intent.getStringExtra(KEY_TYPE_STRING));

TextView tvDates = findViewById(R.id.tvDates);
tvDates.setText(intent.getStringExtra(KEY_DATE_STRING));
```

## Önálló feladat

### Hajók

Vállalatunk terjeszkedésével elindult a hajójáratokat ajánló szolgáltatásunk is. Adjuk hozzá ezt az új bérlet típust az alkalmazásunkhoz!

### Megoldás

A szükséges változtatások nagy része a `ListActivity`-ben lesz. Először frissítsük az Activity layout-ját: itt egy új `FrameLayout`-ot kell hozzáadnunk, amiben a gomb ID-ja legyen `@+id/btnBoat`. A szükséges képet már tartalmazza a projekt, ezt `@drawable/boat` néven találjuk meg.

Ne felejtsük el a gyökérelemünkként szolgáló `LinearLayout`-ban átállítani a `weightSum` attribútumot `3`-ról `4`-re, hiszen most már ennyi a benne található View-k súlyainak összege. (Kipróbálhatjuk, hogy mi történik, ha például `1`-re, vagy `2.5`-re állítjuk ezt a számot, a hatásának már az előnézetben is látszania kell.)

Menjünk az Activity Java fájljába, és következő lépésként vegyünk fel egy új konstanst a hajó típus jelölésére.

```java
public static final int TYPE_BOAT = 4;
```

Az előző három típussal azonos módon keressük ki a hajót kiválasztó gombot (`R.id.btnBoat`), és állítsunk be rá egy listener-t, amely elindítja a `DetailsActivity`-t, a `TYPE_BOAT` konstanst átadva az `Intent`-ben paraméterként.

Még egy dolgunk maradt, a `DetailsActivity` kódjában értelmeznünk kell ezt a paramétert. Ehhez a `getTypeString` függvényen belül vegyünk fel egy új ágat a `switch`-ben:

```java
case ListActivity.TYPE_BOAT:
    return "Boat pass";
```

## iMSc feladat

### Ár kiszámolása

Korábban a részletes nézetben egy fix árat írtunk ki a képernyőre. Írjuk meg a bérlet árát kiszámoló logikát, és ahogy a felhasználó változtatja a bérlet paramétereit, frissítsük a megjelenített árat.

Az árazás a következő módon működjön:

| Közlekedési eszköz | Bérlet ára naponta|
| -- | -- |
| Bicikli | 700 |
| Busz | 1000 |
| Vonat | 1500 |
| Hajó | 2500 |

Ebből még az alábbi kedvezményeket adjuk:

| Árkategória | Kedvezmény mértéke |
| -- | -- |
| Teljes árú | 0% |
| Nyugdíjas | 90% |
| Közalkalmazott | 50% |

A számolásokhoz és az eseménykezeléshez a [`Calendar`][calendar] osztályt, a `DatePicker` osztály [`init`][picker-init-link] függvényét, illetve a `RadioGroup` osztály [`setOnCheckedChangeListener`][radio-checked-changed] osztályát érdemes használni.

[calendar]: https://developer.android.com/reference/java/util/Calendar.html

[picker-init-link]: https://developer.android.com/reference/android/widget/DatePicker.html#init(int%2C%20int%2C%20int%2C%20android.widget.DatePicker.OnDateChangedListener)

[radio-checked-changed]: https://developer.android.com/reference/android/widget/RadioGroup.html#setOnCheckedChangeListener(android.widget.RadioGroup.OnCheckedChangeListener)
