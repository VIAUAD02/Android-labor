# Labor 04 - Felhasználói felület készítése - Fragmentek, Chartok

## Bevezető

A labor során egy HR alkalmazást készítünk el, amelybe belépve a felhasználó meg tudja tekinteni személyes adatait, illetve szabadságot tud rögzíteni. Az alkalmazás nem használ perzisztens adattárolást és valós bejelentkeztetést, csak demo adatokkal dolgozik. A labor fő témája a Fragmentekkel való felületkészítés lesz.

<p align="center">
<img src="./assets/menu.png" width="160">
<img src="./assets/profile1.png" width="160">
<img src="./assets/profile2.png" width="160">
<img src="./assets/holiday1.png" width="160">
<img src="./assets/datepicker.png" width="160">
</p>

## IMSc pontok

A laborfeladatok sikeres befejezése után az IMSc feladatot megoldva 2 IMSc pont szerezhető.

## Értékelés

Osztályzás:
- Főmenü képernyő: 1 pont
- Profil képernyő: 1 pont
- Szabadság képernyő: 1 pont
- Dátumválasztó, napok csökkentése: 1 pont
- Önálló feladat (szabadság továbbfejlesztése): 1 pont

IMSc: Fizetés menüpont megvalósítása
- Kördiagram: 1 IMSc pont
- Oszlopdiagram: 1 IMSc pont

## Projekt létrehozása

Hozzunk létre egy új Android projektet! Az alkalmazás neve legyen `WorkplaceApp`, a Company domain pedig `aut.bme.hu`.

Az alkalmazást természetesen telefonra készítjük, és használhatjuk az alapértelmezett 15-ös minimum SDK szintet.

Az első Activity-nk legyen egy Empty Activity, és nevezzük el `MenuActivity`-nek. A hozzá tartozó layout fájl automatikusan megkapja az `activity_menu.xml` nevet.

Előzetesen töltsük le az alkalmazás képeit tartalmazó [tömörített fájlt](./downloads/res.zip) és bontsuk ki. A benne lévő drawable könyvtárat másoljuk be az app/src/main/res mappába (Studio-ban res mappán állva `Ctrl+V`).

## Főmenü képernyő

Az első Activity amit elkészítünk a navigációért lesz felelős. A labor során 2 funkciót fogunk megvalósítani, ezek a Profil és a Szabadság.

A projekt készítésekor már létrejött `activity_menu.xml` tartalmát cseréljük ki az alábbira:

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:layout_margin="16dp"
    android:gravity="center"
    android:orientation="vertical">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal">

        <FrameLayout
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1">

        </FrameLayout>

        <FrameLayout
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1">

        </FrameLayout>
    </LinearLayout>

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal">

        <FrameLayout
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1">

        </FrameLayout>

        <FrameLayout
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1">

        </FrameLayout>

    </LinearLayout>

</LinearLayout>
```

Egy függőleges LinearLayout-ba tettünk bele 2 vízszintes LinearLayout-ot, mindkettő 2 gombot fog tartalmazni. Súlyozás segítségével 2 részre osztottuk a vízszintes LinearLayout-okat.
A gombon a háttér és a felirat elhelyezéséhez a korábbi laboron már látotthoz hasonlóan FrameLayout-ot fogunk használni.

Az első gombot például így készíthetjük el (a `FrameLayout` tagbe írjuk):
```xml
<ImageButton
    android:id="@+id/btnProfile"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:adjustViewBounds="true"
    android:scaleType="fitCenter"
    android:src="@drawable/profile" />

<TextView
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:layout_gravity="center"
    android:text="Profile"
    android:textSize="34sp" />
```

A további 3 gombot ennek a mintájára készítsük el ezekkel az értékekkel:

| Szöveg | ID | Kép |
| -- | -- | -- |
| Holiday | `@+id/btnHoliday` | `@drawable/holiday` |
| Payment | `@+id/btnPayment` | `@drawable/payment` |
| Cafeteria | `@+id/btnCafeteria` | `@drawable/cafeteria` |

Ne felejtsük el a szövegeket kiszervezni erőforrásba! (a szövegen állva `Alt+Enter`)

Hozzunk létre a két új Empty Activity-t (`ProfileActivity` és `HolidayActivity`)

A MenuActivity Java fájljában (`MenuActivity.java`) keressük ki a gombokat és rendeljünk a lenyomásukhoz eseménykezelőt az onCreate metódusban:

```java
ImageButton btnProfile = findViewById(R.id.btnProfile);
btnProfile.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View view) {
        Intent profileIntent = new Intent(MenuActivity.this, ProfileActivity.class);
        startActivity(profileIntent);
    }
});

ImageButton btnHoliday = findViewById(R.id.btnHoliday);
btnHoliday.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View view) {
        Intent holidayIntent = new Intent(MenuActivity.this, HolidayActivity.class);
        startActivity(holidayIntent);
    }
});
```

Próbáljuk ki az alkalmazást! 4 gombnak kell megjelennie és a felső kettőn működnie kell a navigációnak a (még) üres Activity-kbe.

## Profil képernyő elkészítése

A Profil képernyő két lapozható oldalból fog állni, ezen a név, email, lakcím (első oldal), illetve a személyigazolvány szám, TAJ szám, adószám és törzsszám (második oldal) fognak megjelenni.

Hozzunk létre egy `data` package-et, azon belül egy `Person` osztályt, ebben fogjuk tárolni az oldalakon megjelenő adatokat:

```java
public class Person {
    private String name;
    private String email;
    private String address;
    private String id;
    private String socialSecurityNumber;
    private String taxId;
    private String registrationId;

    public Person(String name, String email, String address, String id,
                  String socialSecurityNumber, String taxId, String registrationId) {
        this.name = name;
        this.email = email;
        this.address = address;
        this.id = id;
        this.socialSecurityNumber = socialSecurityNumber;
        this.taxId = taxId;
        this.registrationId = registrationId;
    }

    public String getName() {
        return name;
    }

    public String getEmail() {
        return email;
    }

    public String getAddress() {
        return address;
    }

    public String getId() {
        return id;
    }

    public String getSocialSecurityNumber() {
        return socialSecurityNumber;
    }

    public String getTaxId() {
        return taxId;
    }

    public String getRegistrationId() {
        return registrationId;
    }
}
```

A Person osztály példányának elérésére hozzunk létre egy `DataManager` osztályt (szintén a `data` package-en belül), ezzel fogjuk szimulálni a valós adatelérést (Singleton mintát használunk, hogy az alkalmazás minden részéből egyszerűen elérhető legyen):

```java
public class DataManager {
    private static DataManager instance;

    private Person person;

    private DataManager() {
        person = new Person(
                "Test User", "testuser@domain.com",
                "1234 Test, Random Street 1.",
                "123456AB",
                "123456789",
                "1234567890",
                "0123456789");
    }

    public static DataManager getInstance() {
        if (instance == null) {
            instance = new DataManager();
        }

        return instance;
    }

    public Person getPerson() {
        return person;
    }
}
```

Ezután elkészíthetjük a két oldalt, Fragmentekkel. Hozzuk létre egy új `fragments` package-ben a két Fragmentet (New -> Java Class), ezek neve legyen `MainProfileFragment` és `DetailsProfileFragment`.

A két Fragmentben származzunk le a Fragment osztályból (androidx-es verziót válasszuk) és definiáljuk felül az onCreateView metódust. Ebben betöltjük a layout-ot és a Person objektum adatait kiírjuk a TextView-kra.

`MainProfileFragment.java`:
```java
public class MainProfileFragment extends Fragment {
    @Nullable
    @Override
    public View onCreateView(LayoutInflater inflater, @Nullable ViewGroup container,
                             @Nullable Bundle savedInstanceState) {
        View rootView = inflater.inflate(R.layout.profile_main, container, false);

        TextView tvName = rootView.findViewById(R.id.tvName);
        TextView tvEmail = rootView.findViewById(R.id.tvEmail);
        TextView tvAddress = rootView.findViewById(R.id.tvAddress);

        Person person = DataManager.getInstance().getPerson();

        tvName.setText(person.getName());
        tvEmail.setText(person.getEmail());
        tvAddress.setText(person.getAddress());

        return rootView;
    }
}
```

`DetailsProfileFragment.java`:
```java
public class DetailsProfileFragment extends Fragment {
    @Nullable
    @Override
    public View onCreateView(LayoutInflater inflater, @Nullable ViewGroup container,
                             @Nullable Bundle savedInstanceState) {
        View rootView = inflater.inflate(R.layout.profile_detail, container, false);

        TextView tvId = rootView.findViewById(R.id.tvId);
        TextView tvSSN = rootView.findViewById(R.id.tvSSN);
        TextView tvTaxId = rootView.findViewById(R.id.tvTaxId);
        TextView tvRegistrationId = rootView.findViewById(R.id.tvRegistrationId);

        Person person = DataManager.getInstance().getPerson();

        tvId.setText(person.getId());
        tvSSN.setText(person.getSocialSecurityNumber());
        tvTaxId.setText(person.getTaxId());
        tvRegistrationId.setText(person.getRegistrationId());

        return rootView;
    }
}
```

Készítsük el a megfelelő layout-okat a Fragmentekhez (`R.layout.profile_main` és `R.layout.profile_detail`-en állva `Alt+Enter`).

`profile_main.xml`:
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="16dp">

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Name:"
        android:textAllCaps="true"
        android:textSize="20sp" />

    <TextView
        android:id="@+id/tvName"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginBottom="20dp"
        android:textColor="#000000"
        android:textSize="34sp" />

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Email:"
        android:textAllCaps="true"
        android:textSize="20sp" />

    <TextView
        android:id="@+id/tvEmail"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginBottom="20dp"
        android:textColor="#000000"
        android:textSize="34sp" />

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Address:"
        android:textAllCaps="true"
        android:textSize="20sp" />

    <TextView
        android:id="@+id/tvAddress"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginBottom="20dp"
        android:textColor="#000000"
        android:textSize="34sp" />

</LinearLayout>
```

`profile_detail.xml`:
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="16dp">

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="ID:"
        android:textAllCaps="true"
        android:textSize="20sp" />

    <TextView
        android:id="@+id/tvId"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginBottom="20dp"
        android:textColor="#000000"
        android:textSize="34sp" />

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Social Security ID:"
        android:textAllCaps="true"
        android:textSize="20sp" />

    <TextView
        android:id="@+id/tvSSN"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginBottom="20dp"
        android:textColor="#000000"
        android:textSize="34sp" />

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Tax ID:"
        android:textAllCaps="true"
        android:textSize="20sp" />

    <TextView
        android:id="@+id/tvTaxId"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginBottom="20dp"
        android:textColor="#000000"
        android:textSize="34sp" />

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Registration ID:"
        android:textAllCaps="true"
        android:textSize="20sp" />

    <TextView
        android:id="@+id/tvRegistrationId"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginBottom="20dp"
        android:textColor="#000000"
        android:textSize="34sp" />

</LinearLayout>
```

(Szervezzük ki a szövegeket erőforrásba)

Már csak a lapozás megvalósítása maradt hátra, ezt a ViewPager osztállyal fogjuk megvalósítani.

Az `activity_profile.xml` fájlba hozzunk létre egy `ViewPager`-t:
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context="hu.bme.aut.workplaceapp.ProfileActivity">

    <androidx.viewpager.widget.ViewPager
        android:id="@+id/vpProfile"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />
</LinearLayout>
```

A ViewPager osztály egy PagerAdapter osztály segítségével tudja az oldalakat létrehozni. Hozzunk létre egy új `adapter` package-be egy PagerAdaptert a két Fragmentünkhöz.
`ProfilePagerAdapter.java`:
```java
public class ProfilePagerAdapter extends FragmentPagerAdapter {
    private static final int NUM_PAGES = 2;

    public ProfilePagerAdapter(FragmentManager fm) {
        super(fm);
    }

    @Override
    public Fragment getItem(int position) {
        switch (position) {
            case 0:
                return new MainProfileFragment();
            case 1:
                return new DetailsProfileFragment();
            default:
                return new MainProfileFragment();
        }
    }

    @Override
    public int getCount() {
        return NUM_PAGES;
    }
}
```

A ProfileActivity-ben rendeljük hozzá a ViewPagerhez a most elkészített adaptert (onCreate metódus): 
```java
ViewPager vpProfile = findViewById(R.id.vpProfile);
vpProfile.setAdapter(new ProfilePagerAdapter(getSupportFragmentManager()));
```

Próbáljuk ki az alkalmazást. A Profile gombra kattinva megjelennek a felhasználó adatai és lehet lapozni is.

## Szabadság képernyő elkészítése

A Szabadság képernyőn egy kördiagramot fogunk megjeleníteni, ami mutatja, hogy mennyi szabadságot vettünk már ki és mennyi maradt. Ezen kívül egy gomb segítségével új szabadnap kivételét is megengedjük a felhasználónak.

Először egészítsük ki a DataManager osztályunkat, hogy kezelje a szabadsághoz kapcsolódó adatokat is:
```java
private static final int HOLIDAY_MAX_VALUE = 20;
private static final int HOLIDAY_DEFAULT_VALUE = 15;

...

private int holidays = HOLIDAY_DEFAULT_VALUE;

...

public int getHolidays() {
    return holidays;
}

public void setHolidays(int holidays) {
    this.holidays = holidays;
}

public int getRemainingHolidays() {
    return HOLIDAY_MAX_VALUE - getHolidays();
}
```

A PieChart kirajzolásához az [MPAndroidChart](https://github.com/PhilJay/MPAndroidChart) library-t fogjuk használni.

Projekt szintű build.gradle:
```groovy
allprojects {
    repositories {
        ...
        maven { url "https://jitpack.io" }
    }
}
```

App szintű build.gradle:
```groovy
dependencies {
    ...
    implementation 'com.github.PhilJay:MPAndroidChart:v3.0.3'
}
```

Ezután kattinsunk az Android Studioban jobb fent megjelenő `Sync Now` feliratra vagy a fejlécen szereplő mérges gradle elefánt gombra, hogy a library fájljai letöltődjenek.

Ha a library fájljai letöltődtek, akkor írjuk meg az Activity layout-ját (`activity_holiday.xml`):
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context="hu.bme.aut.workplaceapp.HolidayActivity">

    <com.github.mikephil.charting.charts.PieChart
        android:id="@+id/chartHoliday"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1" />

    <Button
        android:id="@+id/btnTakeHoliday"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Take Holiday"
        android:layout_gravity="center" />

</LinearLayout>
```

Írjuk meg az Activity kódját (`HolidayActivity.java`):
```java
public class HolidayActivity extends AppCompatActivity {
    private PieChart chartHoliday;
    private Button btnTakeHoliday;

    private DataManager dataManager;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_holiday);

        dataManager = DataManager.getInstance();

        chartHoliday = findViewById(R.id.chartHoliday);

        btnTakeHoliday = findViewById(R.id.btnTakeHoliday);
        btnTakeHoliday.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                // TODO: DatePickerDialogFragment megjelenítése
            }
        });

        loadHolidays();
    }

    private void loadHolidays() {
        List<PieEntry> entries = new ArrayList<>();

        entries.add(new PieEntry(dataManager.getHolidays(), "Taken"));
        entries.add(new PieEntry(dataManager.getRemainingHolidays(), "Remaining"));

        PieDataSet dataSet = new PieDataSet(entries, "Holidays");
        dataSet.setColors(ColorTemplate.MATERIAL_COLORS);

        PieData data = new PieData(dataSet);
        chartHoliday.setData(data);
        chartHoliday.invalidate();
    }
}
```

Próbáljuk ki az alkalmazást! A PieChart most már megjelenik, de a gomb még nem kell, hogy működjön.
 
## Dátumválasztó megvalósítása

A következő lépésben a Take Holiday gombra megjelenő dátumválasztó működését fogjuk megvalósítani. A gomb lenyomására megjelenik egy dátumválasztó és a dátum kiválasztása után a szabad napok eggyel csökkennek.

Hozzunk létre egy DatePickerDialogFragment osztályt:
```java
public class DatePickerDialogFragment extends DialogFragment
        implements DatePickerDialog.OnDateSetListener {
    private OnDateSelectedListener onDateSelectedListener;

    @Override
    public void onAttach(Context context) {
        super.onAttach(context);

        if (!(context instanceof OnDateSelectedListener)) {
            throw new RuntimeException("The activity does not implement the" +
                    "OnDateSelectedListener interface");
        }

        onDateSelectedListener = (OnDateSelectedListener) context;
    }

    @Override
    public Dialog onCreateDialog(Bundle savedInstanceState) {
        Calendar c = Calendar.getInstance();
        int year = c.get(Calendar.YEAR);
        int month = c.get(Calendar.MONTH);
        int day = c.get(Calendar.DAY_OF_MONTH);

        return new DatePickerDialog(getActivity(),this,
                year, month, day);
    }

    @Override
    public void onDateSet(DatePicker datePicker, int year, int month, int day) {
        onDateSelectedListener.onDateSelected(year, month, day);
    }

    public interface OnDateSelectedListener {
        void onDateSelected(int year, int month, int day);
    }
}
```

Az importoknál a `java.util`-t válasszuk a Calendarhoz, a Fragment-hez pedig az `androidx`-es verziót.

A laborvezetővel vizsgáljuk meg az `OnDateSelectedListener` interface működését. Az osztályt használóknak ezt az interface-t kell implementálnia és a megvalósított `onDateSelected` metódus kapja meg a dátumot.

Állítsuk be a gomb eseménykezelőjét a HolidayActivity-ben, hogy lenyomáskor jelenítse meg a dátumválasztót:
```java
btnTakeHoliday.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View view) {
        new DatePickerDialogFragment().show(getSupportFragmentManager(), "DATE_TAG");
    }
});
```

A kiválasztott dátum feldolgozásához implementáljuk az OnDateSelectedListener-t a HolidayActivity-ben:
```java
public class HolidayActivity extends AppCompatActivity
        implements DatePickerDialogFragment.OnDateSelectedListener {

...

@Override
public void onDateSelected(int year, int month, int day) {
    int numHolidays = dataManager.getHolidays();

    if (dataManager.getRemainingHolidays() > 0) {
        dataManager.setHolidays(numHolidays + 1);
    }

    // Update chart
    loadHolidays();
}
```

Próbáljuk ki az alkalmazást! Most már a gomb is jól kell, hogy működjön, a napok számának is csökkennie kell a diagramon.

## Önálló feladat

- Csak akkor engedjünk új szabadságot kivenni, ha a kiválasztott nap a mai napnál későbbi.
- Ha elfogyott a szabadságkeretünk, akkor a Take Holiday gomb legyen letiltva.

## iMSc feladat

### Fizetés menüpont megvalósítása

A Payment menüpontra kattintva jelenjen meg egy PaymentActivity rajta egy ViewPager-rel és 2 Fragment-tel (A Profile menüponthoz hasonlóan):
- `PaymentTaxesFragment`: kördiagram, aminek a közepébe van írva az aktuális fizetés és mutatja a nettó jövedelmet illetve a levont adókat (adónként külön)
- `MonthlyPaymentFragment`: egy oszlopdiagramot mutasson 12 oszloppal, a havi szinten lebontott fizetéseket mutatva - érdemes az adatokat itt is a DataManager osztályban tárolni

[Segítség](https://github.com/PhilJay/MPAndroidChart/wiki)
