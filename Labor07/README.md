# Labor 07 - Weather Info alkalmazás készítése

## Bevezető

A labor során egy időjárás információkat megjelenítő alkalmazás elkészítése a feladat. A korábban látott UI elemek használata mellett láthatunk majd példát hálózati kommunkáció hatékony megvalósítására is a [`Retrofit`](https://square.github.io/retrofit/) library felhasználásával.

Az alkalmazás városok listáját jeleníti meg egy [`RecyclerView`](https://developer.android.com/guide/topics/ui/layout/recyclerview)-ban, egy kiválasztott város részletes időjárás adatait pedig az [OpenWeatherMap](https://openweathermap.org/) REST API-jának segítségével kérdezi le. A részetező nézeten egy [`ViewPager`](https://developer.android.com/training/animation/screen-slide)-ben két [`Fragment`](https://developer.android.com/guide/components/fragments)-en lehet megtekinteni a részleteket. Új város hozzáadására egy  [`FloatingActionButton`](https://developer.android.com/guide/topics/ui/floating-action-button) megnyomásával van lehetőség. 

> REST = [**Re**presentational **S**tate **T**ransfer](https://en.wikipedia.org/wiki/Representational_state_transfer)

<p align="center">
<img src="./assets/list.png">
<img src="./assets/main.png">
<img src="./assets/details.png">
</p>

Felhasznált technológiák: 

- [`Activity`](https://developer.android.com/guide/components/activities/intro-activities)
- [`Fragment`](https://developer.android.com/guide/components/fragments)
- [`RecyclerView`](https://developer.android.com/guide/topics/ui/layout/recyclerview)
- [`ViewPager`](https://developer.android.com/training/animation/screen-slide)
- [`Retrofit`](https://square.github.io/retrofit/)
- [`Gson`](https://github.com/google/gson)
- [`Glide`](https://github.com/bumptech/glide)

## Feltöltés

Az elkészült megoldást `.zip` formátumban (teljes Android Studio projekt – build mappa kivehető) kell feltölteni a tárgy oldalán, ahol a laborvezető tudja értékelni.

## Az alkalmazás specifikációja

Az alkalmazás két`Activity`-ből áll. 

Az alkalmazás indulásakor megjelenő `Activity` a felhasználó által felvett városok listáját jeleníti meg. Minden lista elemhez tartozik egy *Remove* gomb, aminek a megnyomására az adott város törlődik a listából. Új várost a nézet jobb alsó sarkában található `FloatingActionButton` megnyomásával lehet felvenni.

Egy városra való kattintás hatására megnyílik egy új `Activity` két `Fragment`-tel, amik között `ViewPager`-rel lehet váltani. Az első `Fragment` a kiválasztott város időjárásának leírását és az ahhoz tartozó ikont jeleníti meg. A második `Fragment`-en a városban mért átlagos, minimum és maximum hőmérséklet, a légnyomás és a páratartalom értéke látható.

## Laborfeladatok

A labor során az alábbi feladatokat a laborvezető segítségével, illetve a jelölt feladatokat önállóan kell megvalósítani.

1. Város lista megvalósítása: 1 pont
2. Részletező nézet létrehozása és bekötése a navigációba: 1 pont
3. Hálózati kommunikáció megvalósítása: 1 pont
4. A hálózati réteg bekötése a részletező nézetbe: 1 pont
5. Önálló feladat: város listából törlés megvalósítása: 1 pont

A labor során egy kompelx időjárás alkalmazás készül el. A labor szűkös időkerete miatt szükség lesz nagyobb kódblokkok másolására, azonban minden esetben figyeljünk a laborvezető magyarázatára, hogy a kódrészek érthetőek legyenek. A cél a bemutatott kódok megértése és a felhasznált libraryk használatának elsajátítása.

*Elnézést kérünk  az eddigieknél nagyobb kód blokkokért, de egy ilyen, bemutató jellegű feladat kisebb méretben nem oldható meg, illetve a labor elveszítené a lényegét, ha csak egy „hello world” hálózati kommunikációs lekérést valósítanánk meg. Köszönjük a megértést.*

### Projekt létrehozása

Hozzunk létre egy `WeatherInfo` nevű projektet Android Studioban, `Add no activity` opcióval ! A *Company domain* legyen `aut.bme.hu`! Az alkalmazást telefonra és tabletre készítjük, tehát válasszuk ki a **Phone and Tablet** lehetőséget, minimum SDK-nak pedig válasszuk az **API 15**-öt! Első `Activity`-ként hozzunk létre egy *Basic Activity*, `Fragment` használata **nélkül** és nevezzük el `CityActivity`-nek, legyen ez a **Launcher Activity**-nk majd kattintsunk a *Finish* gombra!

Töltsük le és tömörítsük ki [az alkalmazáshoz szükséges erőforrásokat](./assets/drawables.zip) , majd másoljuk be őket a projekt *app/src/main/res* mappájába (Studio-ban a *res* mappa kijelölése után *Ctrl+V*)!

Az *app* modulhoz tartozó `build.gradle` fájlban a `dependencies` blokkhoz adjuk hozzá a `Retrofit` és `Glide` libraryket:

```groovy
dependencies{
    //...
    def retrofit_version = "2.4.0"
    implementation "com.squareup.retrofit2:retrofit:$retrofit_version"
    implementation "com.squareup.retrofit2:converter-gson:$retrofit_version"
    implementation 'com.github.bumptech.glide:glide:4.8.0'
}
```

Ezután kattintsunk a jobb felső sarokban megjelenő **Sync now** gombra.

>  A `Retrofit` a fejlesztő által leírt egyszerű, megfelelően annotált interfészek alapján kódgenerálással állít elő HTTP hivásokat lebonyolító implementációt. Kezeli az URL-ben inline módon adott paramétereket, az URL queryket, stb. Támogatja a legnépszerűbb szerializáló/deszerializáló megoldásokat is (pl.: [`Gson`](https://github.com/google/gson), [`Moshi`](https://github.com/square/moshi), [`Simple XML`](simple.sourceforge.net), stb.), amikkel Java objektumok, és JSON vagy XML formátumú adatok közötti kétirányú átalakítás valósítható meg. A laboron ezek közül a Gsont fogjuk használni a JSON-ban érkező időjárás adatok konvertálására.

> A `Glide`  egy hatékny képbetöltést és -cahce-elést megvalósító library Androidra. Egyszerű interfésze és hatékonysága miatt használjuk.

Az alkalmazásban szükségünk lesz internet elérésre. Vegyük fel az `AndroidManifest.xml` állományban az *Internet permission*-t az `application` tagen *kívülre*:

```xml
<uses-permission android:name="android.permission.INTERNET" />
```

> Androidon API 23-tól (6.0, Marshmallow) az engedélyek két csoportba lettek osztva. A *normal* csoportba tartozó engedélyeket elég felvenni az `AndroidManifest.xml` fájlba az előbb látott módon és az alkalmazás automatikusan megkapja őket. A *dangerous* csoportba tartozó engedélyek esetén ez már nem elég, futás időben explicit módon el kell kérni őket a felhasználótól, aki akármikor meg is tagadhatja az alkalmazástól a kért engedélyt. Az engedélyek kezeléséről bővebben a [developer.android.com](https://developer.android.com/guide/topics/permissions/overview) oldalon lehet tájékozódni

Vegyük fel az alábbi szöveges erőforrásokat a `res/values/strings.xml`-be:

```xml
<resources>
    <string name="app_name">WeatherInfo</string>
    
    <string name="action_settings">Settings</string>
    
    <string name="title_activity_city">Cities</string>
    <string name="remove">Remove</string>
    
    <string name="new_city">New city</string>
   	<string name="new_city_hint">City</string>
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

#### OpenWeatherMap API kulcs

Regisztráljunk saját felhasználót az [OpenWeatherMap](https://openweathermap.org/) oldalon, és hozzunk létre egy API kulcsot, aminek a segítségével ahasználhatjuk majd a szolgáltatást az alkalmazásunkban! 

1. Kattintsunk a *Sign up* gombra
2. Töltsük ki a regisztrációs formot
3. A *Company* mező értéke legyen "BME", a *Purpose* értéke legyen "Education/Science"
4. Sikeres regisztráció után az *API keys* tabon található az alapértelmezettként létrehozott API kulcs.

A kapott API kulcsra később szükségünk lesz az időjárás adatokat lekérő API hívásnál.

### 1. Városlista megvalósítása (1 pont)

Valósítsuk meg az egy `RecyclerView`-ból álló, városok listáját megjelenítő `CityAcitivity`-t! 

A város nevére kattintva jelenik majd meg egy részletező nézet (*DetailsAcitivity*), ahol az időjárás információk letöltése fog történni. Új város felvételére egy *FloatingActionButton* fog szolgálni.

Cseréljük le a *content_city.xml* tartalmát egy `RecyclerView`-ra:

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.recyclerview.widget.RecyclerView
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/MainRecyclerView"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    app:layout_behavior="@string/appbar_scrolling_view_behavior"
    />
```

Cseréljük le a `FloatingActionButton` ikonját az  `activity_city.xml`-ben:

```xml
<com.google.android.material.floatingactionbutton.FloatingActionButton
    android:id="@+id/fab"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:layout_gravity="bottom|end"
    android:layout_margin="@dimen/fab_margin"
    android:src="@drawable/ic_add_white_36dp"/>
```

Az egyes funkciókhoz tartozó osztályokat külön package-ekbe fogjuk szervezni. Előfordulhat, hogy a másolások miatt az Android Studio nem ismeri fel egyből a package szerkezetet, így ha ilyen problémánk lenne, az osztály néven állva Alt+Enter után állítassuk be a megfelelő package nevet.

A `hu.bme.aut.weatherinfo` package-ben hozzunk létre egy `feature` nevű package-et. A `feature` package-ben hozzunk létre egy `city` nevű package-et. *Drag and drop* módszerrel helyezzük át a `CityActivity`-t a `city` *package*-be, a felugró dialógusban pedig kattintsunk a *Refactor* gombra.

A `CityActivity` kódját cseréljük le a következőre:

```java
public class CityActivity extends AppCompatActivity 
		implements CityAdapter.OnCitySelectedListener, AddCityDialogFragment.AddCityDialogListener {

    private RecyclerView recyclerView;
    private CityAdapter adapter;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_city);

        initFab();
        initRecyclerView();
    }

    private void initFab() {
        FloatingActionButton fab = findViewById(R.id.fab);
        fab.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                // TODO: Show new city dialog
            }
        });
    }

    private void initRecyclerView() {
        recyclerView = findViewById(R.id.MainRecyclerView);
        recyclerView.setLayoutManager(new LinearLayoutManager(this));
        adapter = new CityAdapter(this);
    adapter.addCity("Budapest");
        adapter.addCity("Debrecen");
        adapter.addCity("Sopron");
        adapter.addCity("Szeged");
        recyclerView.setAdapter(adapter);
    }

    @Override
    public void onCitySelected(String city) {
        // Todo: Start DetailsActivity with the selected city
    }

    @Override
    public void onCityAdded(String city) {
        adapter.addCity(city);
    }
}
```

A `city` package-ben hozzuk létre a `CityAdapter` osztályt:

```java
public class CityAdapter extends RecyclerView.Adapter<CityAdapter.CityViewHolder> {

    private final List<String> cities;

    private OnCitySelectedListener listener;

    public interface OnCitySelectedListener {
        void onCitySelected(String city);
    }

    CityAdapter(OnCitySelectedListener listener) {
        this.listener = listener;
        cities = new ArrayList<>();
    }

    @NonNull
    @Override
    public CityViewHolder onCreateViewHolder(@NonNull ViewGroup parent, int viewType) {
        View view = LayoutInflater.from(parent.getContext()).inflate(R.layout.item_city, parent, false);
        return new CityViewHolder(view);
    }

    @Override
    public void onBindViewHolder(@NonNull CityViewHolder holder, int position) {
        String item = cities.get(position);
        holder.nameTextView.setText(cities.get(position));
        holder.item = item;

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

    class CityViewHolder extends RecyclerView.ViewHolder {

        TextView nameTextView;

        Button removeButton;

        String item;

        CityViewHolder(View itemView) {
            super(itemView);

            nameTextView = itemView.findViewById(R.id.CityItemNameTextView);
            removeButton = itemView.findViewById(R.id.CityItemRemoveButton);

            itemView.setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick(View view) {
                    if (listener != null) {
                        listener.onCitySelected(item);
                    }
                }
            });
        }
    }
}
```

Hozzuk létre a `res/layout` mappában az  `item_city.xml` layoutot:

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="horizontal"
    android:paddingBottom="8dp"
    android:paddingLeft="16dp"
    android:paddingRight="16dp"
    android:paddingTop="8dp"
    android:weightSum="3">

    <TextView
        android:id="@+id/CityItemNameTextView"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_weight="2"
        tools:text="Budapest" />

    <Button
        android:id="@+id/CityItemRemoveButton"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_weight="1"
        android:text="@string/remove" />

</LinearLayout>
```

Igény szerint vizsgáljuk meg a laborvezetővel a `CityAdapter` osztályban az alábbiakat:
- Hogyan történik a lista tartalmi elemeinek kezelése?
- Hogyan épül fel egy lista elem?
- Hogyan történik a lista elemen a kiválasztás események kezelése? Hogyan értesítjük a `CityActivity`-t egy elem kiválasztásáról?
- Hogyan kerültek megvalósításra az `addCity(...)` és `removeCity(…)` metódusok!

A `CityActivity`-vel kapcsolatos következő lépés az új város nevét bekérő dialógus (`DialogFragment`) megvalósítása és bekötése.

Hozzunk létre egy `dialog_new_city.xml` nevű layout fájlt a `res/layout` mappában a következő tartalommal:

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="24dp">

    <EditText
        android:id="@+id/NewCityDialogEditText"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="@string/new_city_hint"
        android:inputType="text" />

</LinearLayout>
```

A `city` package-ben hozzuk létre az `AddCityDialogFragment` osztályt:

```java
public class AddCityDialogFragment extends AppCompatDialogFragment {

    private AddCityDialogListener listener;

    private EditText editText;

    public interface AddCityDialogListener {
        void onCityAdded(String city);
    }

    @Override
    public void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        if (getActivity() instanceof AddCityDialogListener) {
            listener = (AddCityDialogListener) getActivity();
        } else {
            throw new RuntimeException("Activity must implement AddCityDialogListener interface!");
        }
    }

    @NonNull
    @Override
    public Dialog onCreateDialog(Bundle savedInstanceState) {
        return new AlertDialog.Builder(requireContext())
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
        View view = LayoutInflater.from(getContext()).inflate(R.layout.dialog_new_city, null);
        editText = view.findViewById(R.id.NewCityDialogEditText);
        return view;
    }
}
```

Igény szerint vizsgáljuk meg a laborvezetővel az `AddCityDialogFragment` implementációjában az alábbiakat:
- Hogyan ellenőrizzük az `onCreate(…)` függvényben azt, hogy az `Activity`, amihez a `DialogFragment` felcsatolódott implementálja-e az `AddCityDialogListener` interfészt?
- Hogyan kerül beállításra az egyedi layout a `DialogFragment`-ben?
- Hogyan térünk vissza a beírt városnévvel?

> Szorgalmi feladat otthonra: az alkalmazás ne engedje a város létrehozsát, ha a városnév mező üres!
> Tipp: [http://stackoverflow.com/questions/13746412/prevent-dialogfragment-from-dismissing-when-button-is-clicked](http://stackoverflow.com/questions/13746412/prevent-dialogfragment-from-dismissing-when-button-is-clicked)

Végül egészítsük ki a `CityActivity` `initFab(…)` függvényét úgy, hogy a gombra kattintva jelenjen meg az új dialógus:

```java
private void initFab() {
    FloatingActionButton fab = findViewById(R.id.fab);
    fab.setOnClickListener(new View.OnClickListener() {
        @Override
        public void onClick(View view) {
            new AddCityDialogFragment()
                    .show(getSupportFragmentManager(), AddCityDialogFragment.class.getSimpleName());
        }
    });
}
```

Indítsuk el az alkalmazást, amely már képes városnevek bekérésére és megjelenítésére.

### 2. Részletező nézet létrehozása és bekötése a navigációba (1 pont)

A következő lépésben a `hu.bme.aut.weatherinfo.feature`  package-en belül hozzunk létre egy `details` nevű packaget.

A `details` package-ben hozzunk létre egy *Empty Activity* típusú `Activity`-t  `DetailsActivity` néven.

A hozzá tartozó `activity_details.xml` layout kódja:

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:paddingBottom="@dimen/activity_vertical_margin"
    android:paddingLeft="@dimen/activity_horizontal_margin"
    android:paddingRight="@dimen/activity_horizontal_margin"
    android:paddingTop="@dimen/activity_vertical_margin">

    <androidx.viewpager.widget.ViewPager
        android:id="@+id/mainViewPager"
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <androidx.viewpager.widget.PagerTabStrip
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_gravity="top" />

    </androidx.viewpager.widget.ViewPager>

</RelativeLayout>
```

Hozzunk létre a hiányzó *dimen* erőforrásokat (*Alt+Enter* -> *Create dimen value...*), értékük legyen *16dp*!

A felület gyakorlatilag egy `ViewPager`-t tartalmaz, melyben két `Fragment`-et fogunk megjeleníteni. A `PagerTabStrip` biztosítja a *Tab* jellegű fejlécet.

A `DetailsActivity.java`  kódja legyen a következő:

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

        getSupportActionBar().setTitle(getString(R.string.weather, city));
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

Cseréljük le a `strings.xml`-ben a *weather* szöveges erőforrást:

```xml
<string name="weather">Weather in %s </string>
```

A string erőforrásba írt *%s* jelölő használatával lehetővé válik egy *String argumentum* beillesztése a stringbe, ahogy a fenti kódrészletben láthatjuk.

> Figyeljük meg, hogy a `DetailsActivity` hogyan állítja be az `ActionBar` címét a paraméterül kapott város nevével, illetve és azt, hogy az `ActionBar` bal felső sarkában a *vissza gomb* kezelése hogyan került megvalósításra.

Valósítsuk meg a `CityActivity` `onCitySelected(…)` függvényében azt, hogy egy városnév kiválasztásakor a `DetailsActivity` megfelelően felparaméterezve induljon el:

```java
@Override
public void onCitySelected(String city) {
    Intent showDetailsIntent = new Intent();
    showDetailsIntent.setClass(CityActivity.this, DetailsActivity.class);
    showDetailsIntent.putExtra(DetailsActivity.EXTRA_CITY_NAME, city);
    startActivity(showDetailsIntent);
}
```

Próbáljuk ki az alkalmazást, kattintsunk egy város nevére!

### 3. Hálózati kommunikáció megvalósítása (1 pont)

#### Modell osztályok létrehozása 

A modell osztályok számára a `hu.bme.aut.weatherinfo` package-ben hozzunk létre új package-et `model` néven. 

A `model` package-ben hozzunk létre egy új osztályt `WeatherData` néven:

```java
public class WeatherData {

    public List<Weather> weather;
    public MainWeatherData main;
    public Wind wind;
}
```

 Az időjárás szolgáltatástól kapott *JSON* válasz alapján egy ilyen `WeatherData` példány fog létrejönni a `Retrofit` és a `Gson` együttműködésének köszönhetően.

A `model` package-ben hozzuk létre a `Weather` osztályt:

```java
public class Weather {

    public long id;
    public String main;
    public String description;
    public String icon;

}
```

Szintén a `model` package-ben hozzuk létre a `MainWeatherData` osztályt:

```java
public class MainWeatherData {

    public float temp;
    public float pressure;
    public float humidity;
    public float temp_min;
    public float temp_max;
}
```

Végül hozzuk létre a `Wind` osztályt is:

```java
public class Wind {

    public float speed;
    public float deg;
}
```

A `details` *package*-ben hozzuk létre a `WeatherDataHolder` interfészt:

```java
public interface WeatherDataHolder {
    public WeatherData getWeatherData();
}
```

 A `WeatherDataHolder` -en keresztül fogják lekérni a `Fragment`-ek az `Activity`-től az időjárás adatokat.

Vegyünk fel egy `WeatherData` típusú tagváltozót a `DetailsActiviy`-be:

```java
private WeatherData weatherData = null;
```

Módosítsuk úgy a `DetailsActivity` -t, hogy implementálja a `WeatherDataHolder` interfészt:

```java
public class DetailsActivity extends AppCompatActivity implements WeatherDataHolder {
```

Implementáljuk a szükséges függvényt:

```java
@Override
public WeatherData getWeatherData() {
    return weatherData;
}
```

A használt `weatherData` változónak fogunk később értéket adni, amikor visszaérkezett az értéke a hálózati hívás eredményeként. A `ViewPager` két lapján levő `Fragment`-ek a `WeatherDataHolder` interfészen keresztül fogják lekérni az `Activity`-től a `weatherData` objekutmot a megjelenítéshez.

#### A hálózati réteg megvalósítása

A `hu.bme.aut.weatherinfo` package-ben hozzuk létre egy `network` nevű package-et, amely a hálózati kommunikációhoz kapcsolódó osztályokat fogja tartalmazni. 

A `network` package-en belül hozzuk létre egy `WeatherApi` nevű Java interfészt. 

```java
public interface WeatherApi {

    @GET("/data/2.5/weather")
    Call<WeatherData> getWeather(
            @Query("q") String cityName, 
            @Query("units") String units,
            @Query("appid") String appId
    );
}
```

Látható, hogy *annotációk* alkalmazásával tuduk jelezni, hogy az adott függvényhívás milyen hálózati hívásnak fog megfelelni. A `@GET` annotáció *HTTP GET* kérést jelent, a paraméterként adott string pedig azt jelzi, hogy hogy a szerver alap *URL*-éhez képest melyik végpontra szeretnénk küldeni a kérést.

> Hasonló módon tudjuk leírni a többi HTTP kérés típust is: @POST, @UPDATE, @PATCH, @DELETE

A függvény paremétereit a `@Query` annotációval láttuk el. Ez azt jelenti, hogy a `Retrofit` az adott paraméter értékét a kéréshez fűzi *query paraméterként* az annotációban megadott kulccsal.

> További említésre méltó annotációk a teljesség igénye nélkül: @HEAD, @Multipart, @Field

A hálózati hívást jelölő interfész függvény visszatérési értéke egy`Call<WeatherData>` típusú objektum lesz. Ez egy olyan hálózati hívást ír le, aminek a válasza `WeatherData` típusú objektummá alakítható.

Hozzunk létre a `network` package-ben egy `NetworkManager` osztályt:

```java
public class NetworkManager {

    private static final String SERVICE_URL = "https://api.openweathermap.org";
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
                .baseUrl(SERVICE_URL)
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

Ez az osztály lesz felelős a hálózati kérések lebonyolításáért. Egyetlen példányra lesz szükségünk belőle, így [singleton](https://en.wikipedia.org/wiki/Singleton_pattern)ként implementáltuk. Konstansokban tároljuk a szerver alap címét, valamint a szolgáltatás használatához szükséges API kulcsot.

A `Retrofit.Builder()` hívással kérhetünk egy pareméterehzető `Builder` példányt. Ebben adhatjuk meg a hálózati hívásaink tulajdonságait. Jelen példában beállítjuk az elérni kívánt szolgáltatás címét, a HTTP kliens implementációt ([OkHttp](http://square.github.io/okhttp/)), valamint a JSON és objektum reprezentációk közötti konvertert (Gson).

A `WeatherApi` interfészből a `Builder`-rel létrehozott `Retrofit` példány segítségével tudjuk elkérni a fordítási időben generált, paraméterezett implementációt.

 A `retrofit.create(WeatherApi.class)` hívás eredményeként kapott objektum megvalósítja a `WeatherApi` interfészt.  Ha ezen az objektumon meghívjuk a `getWeather(...)` függvényt, akkor megtörténik az általunk az interfészben definiált hálózati hívás. 

Az `APP_ID` paramétert elfedjük az időjárást lekérdező osztályok elől, ezért a `NetworkManager` is tartalmaz egy `getWeather(...)` függvényt, ami a `WeatherApi` implementációba hív tovább.

**Cseréljük le** az `APP_ID` értékét az [OpenWeatherMap](https://openweathermap.org/) oldalon kapott saját API kulcsunkra!

### 4. A hálózati réteg bekötése a részletező nézetbe (1 pont)

A modell elemek és a hálózati réteg megvalósítása után a részletező nézetet fogjuk a specifikációnak megfelelően implementálni, majd bekötjük a hálózati réteget is.

#### A részletező nézetek továbbfejlesztése

A `ViewPager` megfelelő működéséhez létre kell hoznunk egy `FragmentPagerAdapter`-ből származó osztályt a `details` package-ben, ami az eddig látott adapterekhez hasonlóan azt határozza meg, hogy milyen elemek jelenjenek meg a hozzájuk tartozó nézeten (jelen esetben az elemek `Fragment`-ek lesznek):

```java
public class DetailsPagerAdapter extends FragmentPagerAdapter {
    private Context context;

    public DetailsPagerAdapter(FragmentManager fm, Context context) {
        super(fm);
        this.context = context;
    }
    
    // This method is called whenever the adapter needs a Fragment for a certain position
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

Implementáljuk a hiányzó `Fragment`-eket a hozzájuk tartozó néztekkel együtt:

##### DetailsMainFragment

`res/layout/fragment_details_main.xml`:

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

A `details` package-ben a `DetailsMainFragment`:

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
    public View onCreateView(@NonNull LayoutInflater inflater, @Nullable ViewGroup container,
            @Nullable Bundle savedInstanceState) {
        View view = inflater.inflate(R.layout.fragment_details_main,
                container, false);
        tvMain = view.findViewById(R.id.tvMain);
        tvDescription = view.findViewById(
                R.id.tvDescription);
        ivIcon = view.findViewById(R.id.ivIcon);
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
                .load("https://openweathermap.org/img/w/" + weather.icon + ".png")
                .transition(new DrawableTransitionOptions().crossFade())
                .into(ivIcon);
    }
}
```

Figyeljük meg, hogy hogy használjuk a kódban a `Glide` libraryt!

> Az *OpenWeatherMap* API-tól a képek lekérhetők a visszakapott adatok alapján, pl: [https://openweathermap.org/img/w/10d.png](http://openweathermap.org/img/w/10d.png) 

##### DetailsMoreFragment

`res/layout/fragment_details_more.xml`:

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

A `details` package-ben a `DetailsMoreFragment`:

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
            throw new RuntimeException("Activity must implement WeatherDataHolder interface!");
        }
    }

    @Nullable
    @Override
    public View onCreateView(@NonNull LayoutInflater inflater,
            @Nullable ViewGroup container,
            @Nullable Bundle savedInstanceState) {
        View view = inflater.inflate(R.layout.fragment_details_more,
                container, false);
        tvTemperature = view.findViewById(R.id.tvTemperature);
        tvMinTemp = view.findViewById(R.id.tvMinTemp);
        tvMaxTemp = view.findViewById(R.id.tvMaxTemp);
        tvPressure = view.findViewById(R.id.tvPressure);
        tvHumidity = view.findViewById(R.id.tvHumidity);
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

Figyeljük meg, hogyan ellenőrzi a `DetailsMainFragment` és a `DetailsMoreFragment` azt, hogy az `Activity` implementálja-e a `WeatherDataHolder` interfészt. Fontos, hogy ezt a két `Fragment` majd csak azután kerül a `DetailsActivity`-re a `ViewPager`-en keresztül, amikor az adatokat lekérő hálózati kérés már adott vissza eredményt.

Ideiglenesen a `DetailsActivity` `onResume()` függvénye legyen az alábbi:

```java
@Override
protected void onResume() {
    super.onResume();
    ViewPager mainViewPager = findViewById(R.id.mainViewPager);
    DetailsPagerAdapter detailsPagerAdapter = new DetailsPagerAdapter(getSupportFragmentManager(), this);
    mainViewPager.setAdapter(detailsPagerAdapter);
}
```

Próbáljuk ki az alkalmazást, kattintsunk egy városra! jelenleg még nem jelennek meg valós adatok, mivel még nem kötöttük be a az adatok lekéréséért felelős hívást.

#### Hálózati hívás bekötése

Az időjárás adatok lekérdezésének bekötéséhez implementáljunk egy `loadWeatherData()` nevű függvényt a `DetailsActivity`-ben:

`(amennyiben a Callback-et nem ismerné fel a studio, Alt+Enter után importáljuk a retrofit2-es megoldást)`

```java
private void loadWeatherData() {
        NetworkManager.getInstance().getWeather(city).enqueue(new Callback<WeatherData>() {
            @Override
            public void onResponse(@NonNull Call<WeatherData> call,
                                   @NonNull Response<WeatherData> response) {
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
            public void onFailure(@NonNull Call<WeatherData> call, @NonNull Throwable throwable) {
                throwable.printStackTrace();
                Toast.makeText(DetailsActivity.this,
                               "Network request error occurred, check LOG",
                               Toast.LENGTH_SHORT).show();
            }
        });
}
```

Implementáljuk a hiányzó `displayWeatherData(...)` függvényt, ami sikeres API hívás esetén megjeleníti az eredményt:

```java
private void displayWeatherData(WeatherData receivedWeatherData) {
    weatherData = receivedWeatherData;
    ViewPager mainViewPager = findViewById(R.id.mainViewPager);
    DetailsPagerAdapter detailsPagerAdapter = new DetailsPagerAdapter(getSupportFragmentManager(), this);
    mainViewPager.setAdapter(detailsPagerAdapter);
}
```

A `DetailsActivity` `onResume()` függvényében hívjuk meg a `loadWeatherData()` függvényt:

```java
@Override
protected void onResume() {
    super.onResume();
    loadWeatherData();
}
```

Futtassuk az alkalmazást és figyeljük meg a működését! Próbáljuk ki azt is, hogy mi történik akkor, ha megszakítjuk a futtató eszköz internet kapcsolatát és megpróbáljuk megnyitni a részletező nézetet!

### 5. Önálló feladat: város listából törlés megvalósítása (1 pont)

Valósítsuk meg a városok törlését a *Remove* gomb megnyomásának hatására.
