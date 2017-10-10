# Labor 07 - Weather Info

## 1 Felkészülés a laborra

A labor célja egy olyan alkalmazás fejlesztése, melyen keresztül bemutatjuk az Android platformon megszokott UI tervezési mintákat, valamint a hálózati kommunikáció megvalósításának egy lehetséges módját.
Felhasznált technológiák:
- Activity,

- Fragment,

- ViewPager,

- Retrofit,

- Glide.

## 2 Feltöltés

Az elkészült megoldást egy ZIP formájában (**teljes Android Studio projekt – build mappa és app/build mappa kivételével**) kell feltölteni a tárgy oldalán. Az eredmények is itt lesznek.

## 3 Az elkészítendő megoldás

Az elkészítendő megoldás egy időjárás jelentést megjelenítő alkalmazás lesz. Az applikáció indításakor egy város lista fogadja a felhasználót. A lista egy elemére kattintva egy új *Activity* nyílik meg, ahol először a fő adatok látszódnak, de jobbra *swipe*-olással részletesebb információkhoz is juthatunk.

<p align="center">
<img src="./assets/list.png">
<img src="./assets/main.png">
<img src="./assets/details.png">
</p>

## 4 Laborfeladatok

A labor során egy kompelx időjárás alkalmazás készül el. A labor szűkös időkerete miatt szükség lesz nagyobb kódblokkok másolására, azonban minden esetben figyeljen a laborvezető magyarázatára, hogy a kódrészek érthetőek legyenek. A cél nem egy copy-paste labor végigvitele, hanem a bemutatott kódok elmagyarázása, kipróbálása és teljes értékű elsajátítására.

*Elnézést kérünk még egyszer a nagyobb kód blokkokért, de egy ilyen méretű feladatnál kisebb méretben nem oldható meg, illetve elveszítené a labor a lényegét, ha csak egy „hello world” hálózati kommunikációs lekérést valósítanánk meg. Köszönjük a megértést.*

### 4.1 Kezdő projekt létrehozása

Hozzon létre egy **WeatherInfo** nevű projektet Android Studioban **Basic Activity**-vel inicializálva. A laborvezetővel vizsgálja meg a létrejött projektet és annak felépítését. Vizsgálja meg, hogyan áll össze a jelenlegi felület (*CoordinatorLayout*, *AppBarLayout* a *Toolbar*-al és az include-olt *content_main*.xml).

**Törölje ki** a *res/menu* erőforrást és a MainActivity-ben a menü kezelésre vonatkozó függvényeket, mert ezekre nem lesz szükség.

Vegyük fel az alábbi függőségeket a modul-hoz tartozó **build.gradle**-be:

```java
    compile 'com.android.support:appcompat-v7:26.+'
    compile 'com.android.support.constraint:constraint-layout:1.0.2'
    compile 'com.android.support:design:26.+'
    compile 'com.squareup.retrofit2:retrofit:2.3.0'
    compile 'com.squareup.retrofit2:converter-gson:2.3.0'
    compile 'com.github.bumptech.glide:glide:3.7.0'
```

Ha ez megvan, akkor kattintsunk a jobb felső sarokban megjelent **Sync now** gombra.

Ezek a függőségek előadáson már elhangzottak, ha valamelyik nem ismerős, egyeztesse a laborvezetővel.

Az alkalmazásban szükségünk lesz internet elérésre. Vegyük tehát fel a Manifest állományban az **Internet permission**-t az application tagen **kívülre**:

```xml
<uses-permission android:name="android.permission.INTERNET" />
```

[Az alkalmazáshoz képeit tartalmazó tömörített fájlt](./assets/drawables.zip) tartalmát megfelelő módon tömörítse ki az erőforrás (**res**) mappába.

Vegyük fel az alábbi szöveges erőforrásokat a res/values/**strings.xml**-be, hogy ezekkel a későbbiekben már ne legyen gond:

```xml
<resources>
    <string name="app_name">WeatherInfo</string>
    <string name="action_settings">Settings</string>
    <string name="remove">Remove</string>
    <string name="new_city">New city</string>
    <string name="ok">OK</string>
    <string name="cancel">Cancel</string>
    <string name="title_activity_details">DetailsActivity</string>
    <string name="weather">Weather</string>
    <string name="temperature">Temperature</string>
    <string name="min_temperature">Min temperature</string>
    <string name="max_temperature">Max temperature</string>
    <string name="pressure">Pressure</string>
    <string name="humidity">Humidity</string>
    <string name="main">Main</string>
    <string name="details">Details</string>
</resources>

```

**Regisztráljunk saját felhasználót** az [OpenWeatherMap](https://openweathermap.org/) oldalon, hogy legyen saját kulcsunk az API használatához: API keys tab regisztráció után.

### 4.2 Városlista megvalósítása

Ebben a lépésben a *MainAcitivity*-t valósítjuk meg, amely gyakorlatilag egy *RecyclerView*-t jelenít meg a városok listájával. A város nevére kattintva jelenik meg egy részletező nézet (*DetailsAcitivity*), ahol az időjárás információk letöltése fog történni. Új város felvételére egy *FloatingActionButton* fog szolgálni.

A városlista megvalósításának első lépéseként adjuk hozzá a *MainActivity*-hez egy *RecyclerView*-t, vagyis cseréljük le a *content_main.xml* tartalmát:

```xml
<?xml version="1.0" encoding="utf-8"?>
<android.support.v7.widget.RecyclerView
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/MainRecyclerView"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    app:layout_behavior="@string/appbar_scrolling_view_behavior"
    />
```

Az *activity_main.xml*-ben cserélje le a *FloatingActionButton* ikonját a [kitömörített](./assets/drawables.zip) *ic_add_white_36dp* erőforrásra. A *layout* tartalma így az alábbi lesz:

```xml
<?xml version="1.0" encoding="utf-8"?>
<android.support.design.widget.CoordinatorLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:fitsSystemWindows="true"
    tools:context=".ui.main.MainActivity">

    <android.support.design.widget.AppBarLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:theme="@style/AppTheme.AppBarOverlay">

        <android.support.v7.widget.Toolbar
            android:id="@+id/toolbar"
            android:layout_width="match_parent"
            android:layout_height="?attr/actionBarSize"
            android:background="?attr/colorPrimary"
            app:popupTheme="@style/AppTheme.PopupOverlay"/>

    </android.support.design.widget.AppBarLayout>

    <include layout="@layout/content_main"/>

    <android.support.design.widget.FloatingActionButton
        android:id="@+id/fab"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="bottom|end"
        android:layout_margin="@dimen/fab_margin"
        android:src="@drawable/ic_add_white_36dp"/>
</android.support.design.widget.CoordinatorLayout>
```

A felhasználói felülethez kapcsolódó osztályokat külön *package*-be fogjuk szervezni. Ennek megfelelően kattintsunk jobb gombbal az alkalmazás *package*-re, majd válasszuk a *New* > *Package* opciót. A *package* nevének **ui**-t adjunk meg. Ezen belül hozzunk létre egy újabb *package*-t **main** néven, hogy az egyes *Activity*-khez tartozó osztályokat is el tudjuk különíteni. *Drag and drop*-pal helyezzük át a *MainActivity*-t az előbb létrehozott *package*-be, a felugró dialógusban pedig kattintsunk a **Refactor** gombra.

A *MainActivity* kezdő tartalma az alábbi legyen:

```java
public class MainActivity extends AppCompatActivity {

    private RecyclerView recyclerView;
    private CityAdapter adapter;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        initFab();
        initRecyclerView();
    }

    private void initFab() {
        FloatingActionButton fab = 
          (FloatingActionButton) findViewById(R.id.fab);
        fab.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                // TODO: új város dialógus megjelení™ése
            }
        });
    }

    private void initRecyclerView() {
        recyclerView = 
          (RecyclerView) findViewById(R.id.MainRecyclerView);
        recyclerView.setLayoutManager(new LinearLayoutManager(this));
        adapter = new CityAdapter(
          new OnCitySelectedListener() {
            @Override
            public void onCitySelected(String city) {
                // Todo: új DetailsActivity indítása és a 
                // kiválasztott város hozzáadása
            }
        });
        adapter.addCity("Budapest");
        adapter.addCity("Debrecen");
        adapter.addCity("Sopron");
        adapter.addCity("Szeged");
        recyclerView.setAdapter(adapter);
    }
}
```

A *MainActivity* jelen állapotában hibát jelez, mivel nincs *CityAdapter* osztály, illetve az *OnCitySelectedListener* interfész, melyet az *CityAdapter* vár a konstruktorában.
Hozzuk létre ebben a main *packageben*, egy külön fileban az *OnCitySelectedListener* interfészt, melyen keresztül a *CityAdapter* jelzi, ha egy város ki lett választva:

```java
public interface OnCitySelectedListener {
    void onCitySelected(String city);
}
```

Ezt követően hozzuk létre a *CityAdapter* osztályt:

```java
public class CityAdapter extends 
  RecyclerView.Adapter<CityAdapter.CityViewHolder> {

    private final List<String> cities;
    private OnCitySelectedListener listener;

    public CityAdapter(OnCitySelectedListener listener) {
        this.listener = listener;
        cities = new ArrayList<>();
    }

    @Override
    public CityViewHolder onCreateViewHolder(ViewGroup parent, 
      int viewType) {
        View view = LayoutInflater.from(parent.getContext()).inflate(
          R.layout.item_city, parent, false);
        CityViewHolder viewHolder = new CityViewHolder(view);
        return viewHolder;
    }

    @Override
    public void onBindViewHolder(CityViewHolder holder, int position) {
        holder.position = position;
        holder.nameTextView.setText(cities.get(position));
    }

    @Override
    public int getItemCount() {
        return cities.size();
    }

    public void addCity(String newCity) {
        cities.add(newCity);
        notifyItemInserted(cities.size() - 1);
    }
public void removeCity(int position) {
        cities.remove(position);
        notifyItemRemoved(position);
        if (position < cities.size()) {
            notifyItemRangeChanged(position, cities.size() - position);
        }
    }

    public class CityViewHolder extends RecyclerView.ViewHolder {

        int position;

        TextView nameTextView;
        Button removeButton;

        public CityViewHolder(View itemView) {
            super(itemView);
            nameTextView = 
              (TextView) itemView.findViewById(
              R.id.CityItemNameTextView);
            removeButton = (Button) itemView.findViewById(
              R.id.CityItemRemoveButton);
            itemView.setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick(View view) {
                    if (listener != null) {
                        listener.onCitySelected(cities.get(position));
                    }
                }
            });
        }
    }
}
```

A hivatkozott *item_city.xml* layout tartalma:

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="horizontal"
    android:paddingBottom="8dp"
    android:paddingLeft="16dp"
    android:paddingRight="16dp"
    android:paddingTop="8dp"
    >

    <TextView
        android:id="@+id/CityItemNameTextView"
        android:layout_width="wrap_content"
        android:layout_weight="1"
        android:layout_height="wrap_content"
        tools:text="Budapest"/>

    <Button
        android:id="@+id/CityItemRemoveButton"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@string/remove"/>

</LinearLayout>
```

Vizsgálja meg a laborvezetővel a *CityAdapter* osztályban az alábbiakat:
- Hogyan történik a lista tartalmi elemeinek kezelése?
- Hogyan épül fel egy lista elem?
- Hogyan történik a lista elemen a kiválasztás események kezelése? Hogyan értesítjük a *MainActivity*-t interfészen keresztül a kiválasztott városról?
- Vizsgáljuk meg hogyan kerültek megvalósításra az *addCity(...)* és *removeCity(…)* metódusok!

A *MainActivity*-vel kapcsolatos következő lépés az új város nevét bekérő dialógus (*DialogFragment*) megvalósítása és a lista (*RecyclerView*) bővítése az új városnévvel.

Hozzunk létre egy *AddCityDialogListener* **interfészt** külön fileban a *ui*/*main* packageban:

```java
public interface AddCityDialogListener {
    void onCityAdded(String city);
}
```

Implementáljuk a *MainActivity*-ben az *AddCityDialogListener*-t, melyen keresztül a *DialogFragment* értesíti majd a *MainActivity*-t az új városnévről:

```java
public class MainActivity extends AppCompatActivity 
  implements AddCityDialogListener {
...
```

Implementáljuk az *onCityAdded(…)* metódust a *MainActivity*-ben, mely gyakorlatilag az új várost felveszi a *RecyclerView*-ba az *adapter*-en keresztül:

```java
@Override
public void onCityAdded(String city) {
    adapter.addCity(city);
}
```

A dialógus megjelenítéséhez hozzunk létre egy *dialog_new_city.xml*-t a layout erőforrás mappában, mely gyakorlatilag az új dialógus felületét fogja jelképezni:

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="24dp">

    <EditText
        android:id="@+id/NewCityDialogEditText"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>

</LinearLayout>
```

Hozzuk létre a *ui*/*main* packageben az *AddCityDialogFragment* dialógust az alábbi módon:

```java
public class AddCityDialogFragment extends AppCompatDialogFragment {

    public static final String TAG = "AddCityDialogFragment";

    private AddCityDialogListener listener;
    private EditText editText;

    @Override
    public void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        if (getActivity() instanceof AddCityDialogListener) {
            listener = (AddCityDialogListener) getActivity();
        } else {
           throw new RuntimeException(
           "Activity must implement AddCityDialogListener interface!");
        }
    }

    @NonNull
    @Override
    public Dialog onCreateDialog(Bundle savedInstanceState) {
        return new AlertDialog.Builder(getContext())
                .setTitle(R.string.new_city)
                .setView(getContentView())
                .setPositiveButton(R.string.ok, 
                   new DialogInterface.OnClickListener() {
                    @Override
                    public void onClick(
                      DialogInterface dialogInterface, int i) {
                    listener.onCityAdded(
                      editText.getText().toString());
                    }
                })
                .setNegativeButton(R.string.cancel, null)
                .create();
    }

    private View getContentView() {
        View view = LayoutInflater.from(getContext()).inflate(
          R.layout.dialog_new_city, null);
        editText = (EditText) view.findViewById(
          R.id.NewCityDialogEditText);
        return view;
    }
}
```

A laborvezetővel vizsgálja meg az *AddCityDialogFragment* implementációjában az alábbiakat:
- Hogyan ellenőrizzük az *onCreate(…)*-ben (**ami a Fragmentnek is van!**), hogy az *Activity*, akihez csatoltak implementálja-e az *AddCityDialogListener* *interfacet*?
- Hogyan kerül beállításra az egyedi *layout* a *DialogFragment*-ben?
- Hogyan térünk vissza a beírt városnévvel?
- Kis házi feladat otthonra: ne engedje üres város létrehozását!
</t>- Tipp: [http://stackoverflow.com/questions/13746412/prevent-dialogfragment-from-dismissing-when-button-is-clicked](http://stackoverflow.com/questions/13746412/prevent-dialogfragment-from-dismissing-when-button-is-clicked)

Végül a *MainActivity* *initFab(…)* függvényét egészítse ki, hogy a gombra kattintva az új dialógus megjelenjen:

```java
private void initFab() {
    FloatingActionButton fab = 
        (FloatingActionButton) findViewById(R.id.fab);
    fab.setOnClickListener(new View.OnClickListener() {
        @Override
        public void onClick(View view) {
            new AddCityDialogFragment().show(
              getSupportFragmentManager(), AddCityDialogFragment.TAG);
        }
    });
}
```

Ezt követően teszteljük az alkalmazást, amely már képes városnevek kezelésére *RecyclerView*-n keresztül.

### 4.3	Részletes nézet létrehozása és bekötése

A következő lépésben a *ui* *package*-n belül hozzuk létre a **details** packaget, melyben hozzuk létre az *Empty Activity* típusú *DetailsActivity*-t.

A hozzá tartozó *activity_details.xml* layout az alábbi legyen:

```xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:paddingBottom="@dimen/activity_vertical_margin"
    android:paddingLeft="@dimen/activity_horizontal_margin"
    android:paddingRight="@dimen/activity_horizontal_margin"
    android:paddingTop="@dimen/activity_vertical_margin">

    <android.support.v4.view.ViewPager
        android:id="@+id/mainViewPager"
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <android.support.v4.view.PagerTabStrip
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_gravity="top" />

    </android.support.v4.view.ViewPager>

</RelativeLayout>
```

Hozzunk létre a hiányzó *dimen* erőforrásokat(*Alt+Enter* - > *Create dimen value...*), értékük legyen *16dp*!

A felület gyakorlatilag egy *ViewPager*-t tartalmaz, melyben két *Fragment*et fogunk megjeleníteni balra-jobbra lapozással. A *PagerTabStrip* biztosítja a *Tab* jellegű fejlécet a lapozáskor.

A *DetailsActivity.java* kezdő kódja az alábbi:

```java
public class DetailsActivity extends AppCompatActivity {

    private static final String TAG = "DetailsActivity";
    public static final String EXTRA_CITY_NAME = "extra.city_name";

    private String city;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_details);

        city = getIntent().getStringExtra(EXTRA_CITY_NAME);

        getSupportActionBar().setTitle(getString(R.string.weather,
          city));
        getSupportActionBar().setDisplayHomeAsUpEnabled(true);
    }


    @Override
    protected void onResume() {
        super.onResume();
    }

    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        if (item.getItemId() == android.R.id.home) {
            finish();
            return true;
        }
        return super.onOptionsItemSelected(item);
    }
}
```

Cseréljük le a *strings.xml*-ben a *weather* szöveges erőforrást.

```xml
<string name="weather">Weather in %s </string>
```

A **%s** használatával lehetővé válik egy **String argumentum** beillesztése, ahogy a fenti kódrészletben láthatjuk.

Figyeljük meg, hogy a *DetailsActivity* hogyan állítja be az *ActionBar* címét a paraméterül kapott város nevével, illetve, hogy az *ActionBar* bal felső sarkában a vissza gomb hogyan került megvalósításra.

Valósítsuk meg a *MainActivity* *initRecyclerView(…)* függvényében, hogy a megfelelő esemény kerüljön végrehajtásra az *OnCitySelectedListener* segítségével amikor egy városnév került kiválasztásra és *DetailsActivity* megfelelően felparaméterezve indulhasson el:

```java
adapter = new CityAdapter(new OnCitySelectedListener() {
        @Override
        public void onCitySelected(String city) {
            Intent showDetailsIntent = new Intent();
            showDetailsIntent.setClass(MainActivity.this, 
              DetailsActivity.class);
            showDetailsIntent.putExtra(
              DetailsActivity.EXTRA_CITY_NAME, city);
            startActivity(showDetailsIntent);
        }
    });
```

Próbáljuk ki az alkalmazást, kattintsunk egy város nevére.

### 4.4	Modell osztályok létrehozása

A modell osztályok számára hozzunk létre új *package*-et **model** néven. Az új osztály neve legyen *WeatherData*. Ez fog létrejönni az időjárás szolgáltatástól kapott *JSON* válasz alapján.

```java
public class WeatherData {

    public List<Weather> weather;
    public MainWeatherData main;
    public Wind wind;
}
```

Hozzuk létre a hivatkozott *Weather* osztályt.

```java
public class Weather {

    public long id;
    public String main;
    public String description;
    public String icon;

}
```

Majd ezt követően a *MainWeatherData* osztályt.

```java
public class MainWeatherData {

    public float temp;
    public float pressure;
    public float humidity;
    public float temp_min;
    public float temp_max;
}
```

Végül a *Wind* osztályt definiáljuk.

```java
public class Wind {

    public float speed;
    public float deg;
}
```

Hozzuk létre a *ui*/*details* *package*-be a *WeatherDataHolder* interfészt, ezen keresztül fogják a *Fragment*ek lekérni az *Activity*-től az időjárás adatokat.

```java
public interface WeatherDataHolder {
    public WeatherData getWeatherData();
}
```

A *DetailsActiviy*-be vegyünk fel egy *WeatherData* tagváltozót:

```java
private WeatherData weatherData = null;
```

**Implementálja** a *DetailsActivity* a *WeatherDataHolder* **interfészt**:

```java
public class DetailsActivity extends AppCompatActivity implements WeatherDataHolder {
…
```

Implementáljuk a szükséges metódust:

```java
@Override
public WeatherData getWeatherData() {
    return weatherData;
}
```

A használt *weatherData* változónak fogunk később értéket adni amikor visszaérkezett az értéke a hálózati hívás eredményeként. A *ViewPager* két lapján levő *Fragment*ek a *WeatherDataHolder* interfészen keresztül fogják az *Activity*-től lekérni a *weatherData* objekutmot a megjelenítéshez.

### 4.5	Hálózati kommunikáció megvalósítása

A hálózati kommunikáció megvalósításához a [Retrofit 2](http://square.github.io/retrofit/) *library*-t fogjuk használni, amit már a *build.gradle*-be felvettünk. 

Hozzuk létre a **network** *package*-t, amely a hálózati kommunikációhoz kapcsolódó osztályokat fogja tartalmazni. Ezen belül hozzuk létre a *WeatherApi* interfészt. Ehhez jobb gombbal kattintsunk a *network* *package*-re, majd válasszuk a *New* > *Java class* lehetőséget, és a **kind** *dropdown*-ból válasszuk ki az *Interface*-t. Másoljuk be az alábbi kódot. Látható, hogy **annotációkon** keresztül tudjuk megmondani, hogy a hívás egy *GET* kérést jelent, és hogy a szerver *url*-en belül milyen címre küldjük a kérést. A függvény argumentumait a *@Query* annotáció a kéréshez fűzi paraméterként, az annotációban megadott kulccsal. A visszatérési érték pedig egy *Call<WeatherData>* objektum lesz, vagyis egy olyan hívás, aminek *WeatherData* a visszatérési értéke.

```java
public interface WeatherApi {

    @GET("/data/2.5/weather")
    Call<WeatherData> getWeather(@Query("q") String cityName,
      @Query("units") String units, @Query("appid") String appId);
}
```

Hozzuk létre a *NetworkManager* osztályt. Ez az osztály lesz felelős a hálózati kérések lebonyolításáért. Az osztály a **Singleton pattern**-t valósítja meg, hiszen egyetlen példány elég belőle. Konstansokban van tárolva a szerver címe, valamint a szolgáltatás használatához szükséges API kulcs. A *WeatherApi* interfészből a *Retrofit* osztály segítségével tudunk működő implementációt generálni. A *retrofit.create()* függvény eredményeképpen visszaadott objektum megvalósítja a *WeatherApi* interfészt, és metódusát meghívva el is végzi a hálózati kommunikációt. Az *APP_ID* paramétert elfedjük az időjárást lekérdező osztályok elől, ezért a *NetworkManager* is tartalmaz egy *getWeather()* függvényt, ami a *WeatherApi* implementációba hív tovább.

```java
public class NetworkManager {

    private static final String ENDPOINT_ADDRESS = "http://api.openweathermap.org";
    private static final String APP_ID = "f3d694bc3e1d44c1ed5a97bd1120e8fe";

    private static NetworkManager instance;

    public static NetworkManager getInstance() {
        if (instance == null) {
            instance = new NetworkManager();
        }
        return instance;
    }

    private Retrofit retrofit;
    private WeatherApi weatherApi;

    private NetworkManager() {
        retrofit = new Retrofit.Builder()
                .baseUrl(ENDPOINT_ADDRESS)
                .client(new OkHttpClient.Builder().build())
                .addConverterFactory(GsonConverterFactory.create())
                .build();
        weatherApi = retrofit.create(WeatherApi.class);
    }

    public Call<WeatherData> getWeather(String city) {
        return weatherApi.getWeather(city, "metric", APP_ID);
    }
}
```

**Cseréljük le** az *APP_ID* értékét az [OpenWeatherMap](https://openweathermap.org/) oldalon regisztrált saját értékre (bejelentkezés után -> API keys tab).

### 4.6	Részletes nézet továbbfejlesztése

A modell elemek és a hálózati kommunikáció megvalósítása után a részletes nézetet fogjuk továbbfejleszteni.

A *ViewPager* megfelelő működéséhez létre kell hoznunk egy *FragmentPagerAdapter* objektumot, mely kezeli, hogy melyik „lapon” melyik fragment jelenjen meg:

```java
public class DetailsPagerAdapter extends FragmentPagerAdapter {
    private Context context;

    public DetailsPagerAdapter(FragmentManager fm, Context context) {
        super(fm);
        this.context = context;
    }
    
    // Ez a függvény csak egyszer hívódik meg, nem fog feleslegesen sok 
    // ugyanolyan Fragment-et létrehozni!
    @Override
    public Fragment getItem(int position) {
        Fragment ret = null;
        switch (position) {
            case 0:
                ret = new DetailsMainFragment();
                break;
            case 1:
                ret = new DetailsMoreFragment();
                break;
        }
        return ret;
    }

    @Override
    public CharSequence getPageTitle(int position) {
        String title;
        switch (position) {
            case 0:
                title = context.getString(R.string.main);
                break;
            case 1:
                title = context.getString(R.string.details);
                break;
            default:
                title = "";
        }
        return title;
    }

    @Override
    public int getCount() {
        return 2;
    }
}
```

A *DetailsPagerAdapter* jelenleg hibás, mivel nem létezik a két *Fragment* (*DetailsMainFragment* és *DetailsMoreFragment*), hozzuk ezeket létre a felülettel együtt:

*res/layout/fragment_details_main.xml*:

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="16dp">

    <TextView
        android:id="@+id/tvMain"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        tools:text="Clear"/>

    <TextView
        android:id="@+id/tvDescription"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="16dp"
        tools:text="Clear sky"/>

    <ImageView
        android:id="@+id/ivIcon"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout_marginTop="16dp"/>

</LinearLayout>
```

*ui/details packageben a DetailsMainFragment.java*:

```java
public class DetailsMainFragment extends Fragment {
    private TextView tvMain;
    private TextView tvDescription;
    private ImageView ivIcon;

    private WeatherDataHolder weatherDataHolder;

    @Override
    public void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        if (getActivity() instanceof WeatherDataHolder) {
            weatherDataHolder = (WeatherDataHolder) getActivity();
        } else {
            throw new RuntimeException(
              "Activity must implement WeatherDataHolder interface!");
        }
    }

    @Nullable
    @Override
    public View onCreateView(LayoutInflater inflater, 
      @Nullable ViewGroup container,
      @Nullable Bundle savedInstanceState) {
        View view = inflater.inflate(R.layout.fragment_details_main, 
          container, false);
        tvMain = (TextView) view.findViewById(R.id.tvMain);
        tvDescription = (TextView) view.findViewById(
          R.id.tvDescription);
        ivIcon = (ImageView) view.findViewById(R.id.ivIcon);
        if (weatherDataHolder.getWeatherData() != null) {
          displayWeatherData();
        }
        return view;
    }

    private void displayWeatherData() {
        Weather weather = weatherDataHolder.getWeatherData()
          .weather.get(0);
        tvMain.setText(weather.main);
        tvDescription.setText(weather.description);
        Glide.with(this)
                .load("http://openweathermap.org/img/w/" +
                  weather.icon + ".png")
                .crossFade()
                .into(ivIcon);
    }
}
```

Figyelje meg, hogy a **Glide library** hogyan kerül felhasználásra képek betöltésére. Az *OpenWeatherMap* API-tól a képek lekérhetők a visszakapott adatok alapján, pl:

[http://openweathermap.org/img/w/10d.png](http://openweathermap.org/img/w/10d.png) 

A *DetailsMoreFragment* megalósításáshoz tegyük az alábbiakat:

*res/layout/fragment_details_more.xml*:

```xml
<?xml version="1.0" encoding="utf-8"?>
<TableLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:padding="16dp"
    android:stretchColumns="0">
    <TableRow>
        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@string/temperature"/>
        <TextView
            android:id="@+id/tvTemperature"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            tools:text="25 °C"/>
    </TableRow>
    <TableRow>
        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@string/min_temperature"/>
        <TextView
            android:id="@+id/tvMinTemp"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            tools:text="24 °C"/>
    </TableRow>
    <TableRow>
        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@string/max_temperature"/>
        <TextView
            android:id="@+id/tvMaxTemp"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            tools:text="26 °C"/>
    </TableRow>
    <TableRow>
        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@string/pressure"/>
        <TextView
            android:id="@+id/tvPressure"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            tools:text="100 Pa"/>
    </TableRow>
    <TableRow>
        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@string/humidity"/>
        <TextView
            android:id="@+id/tvHumidity"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            tools:text="50 %"/>
    </TableRow>
</TableLayout>
```

*ui/details/DetailsMoreFragment.java*:

```java
public class DetailsMoreFragment extends Fragment {
    private TextView tvTemperature;
    private TextView tvMinTemp;
    private TextView tvMaxTemp;
    private TextView tvPressure;
    private TextView tvHumidity;

    private WeatherDataHolder weatherDataHolder;

    @Override
    public void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        if (getActivity() instanceof WeatherDataHolder) {
            weatherDataHolder = (WeatherDataHolder) getActivity();
        } else {
            throw new RuntimeException(
              "Activity must implement WeatherDataHolder interface!");
        }
    }

    @Nullable
    @Override
    public View onCreateView(LayoutInflater inflater, 
      @Nullable ViewGroup container,
      @Nullable Bundle savedInstanceState) {
        View view = inflater.inflate(R.layout.fragment_details_more,
          container, false);
        tvTemperature = 
          (TextView) view.findViewById(R.id.tvTemperature);
        tvMinTemp = (TextView) view.findViewById(R.id.tvMinTemp);
        tvMaxTemp = (TextView) view.findViewById(R.id.tvMaxTemp);
        tvPressure = (TextView) view.findViewById(R.id.tvPressure);
        tvHumidity = (TextView) view.findViewById(R.id.tvHumidity);
        if (weatherDataHolder.getWeatherData() != null) {
          showWeatherData();
        }
        return view;
    }

    private void showWeatherData() {
        WeatherData weatherData = weatherDataHolder.getWeatherData();
        tvTemperature.setText("" + weatherData.main.temp);
        tvMinTemp.setText("" + weatherData.main.temp_min);
        tvMaxTemp.setText("" + weatherData.main.temp_max);
        tvPressure.setText("" + weatherData.main.pressure);
        tvHumidity.setText("" + weatherData.main.humidity);
    }
}
```

Figyeljük meg, hogyan ellenőrzi a *DetailsMainFragment* és a *DetailsMoreFragment*, hogy az *Activity* implementálja-e a *WeatherDataHolder* interfészt. Fontos, hogy ezt a két *Fragment*et majd csak aztán kerül valójában a *DetailsActivity*-re a *ViewPager*-en keresztül, amikor az időjárás lekérdezés hálózati kérés már adott vissza eredményt.

Ideiglenesen a *DetailsActivity* *onResume()* függvénye legyen az alábbi:

```java
@Override
protected void onResume() {
    super.onResume();
    ViewPager mainViewPager = 
      (ViewPager) findViewById(R.id.mainViewPager);
    DetailsPagerAdapter detailsPagerAdapter =
      new DetailsPagerAdapter(getSupportFragmentManager(), this);
    mainViewPager.setAdapter(detailsPagerAdapter);
}
```

Próbáljuk ki az alkalmazást, kattintsunk egy városnevére, jelenleg még nem jelennek meg valós adatok.

### 4.7	Hálózati hívás bekötése

Az időjárás adatok lekérdezésére valósítsunk meg egy *loadWeatherData()* nevű függvényt a *DetailsActivity*-ben:

```java
private void loadWeatherData() {
    NetworkManager.getInstance().getWeather(city).enqueue(new Callback<WeatherData>() {
        @Override
        public void onResponse(Call<WeatherData> call, 
          Response<WeatherData> response) {
            Log.d(TAG, "onResponse: " + response.code());
            if (response.isSuccessful()) {
                displayWeatherData(response.body());
            } else {
                Toast.makeText(DetailsActivity.this, 
                  "Error: "+response.message(),
                  Toast.LENGTH_SHORT).show();
            }
        }

        @Override
        public void onFailure(Call<WeatherData> call, Throwable t) {
            t.printStackTrace();
            Toast.makeText(DetailsActivity.this, 
              "Error in network request, check LOG",
              Toast.LENGTH_SHORT).show();
        }
    });
}
```

Illetve a *displayWeatherData(..)*-t, mely siker esetén megjeleníti az eredményt beállítva a *ViewPagert* megfelelően a két *Fragment*tel:

```java
private void displayWeatherData(WeatherData receivedWeatherData) {
    weatherData = receivedWeatherData;
    ViewPager mainViewPager = 
      (ViewPager) findViewById(R.id.mainViewPager);
    DetailsPagerAdapter detailsPagerAdapter = 
      new DetailsPagerAdapter(getSupportFragmentManager(), this);
    mainViewPager.setAdapter(detailsPagerAdapter);
}
```

A *DetailsActivity* *onResume(…)* függvényében hívjuk meg a *loadWeatherData()* függvényt:

```java
@Override
protected void onResume() {
    super.onResume();
    loadWeatherData();
}
```

### 4.8	Önálló feladat: város listában a törlés megvalósítása

Valósítsa meg az elemek törlését a remove gomb megfelelő bekötésével.