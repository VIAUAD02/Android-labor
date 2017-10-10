# Labor 06 - Bevásárló alkalmazás készítése

## Bevezető
A labor célja az Android alapvető felületi elemeivel történő megismerkedés, valamint a perzisztencia megvalósítási lehetőségei közül az ORM bemutatása egy konkrét példán keresztül

Felhasznált technológiák:
- *Activity*
- *Fragment*
- *RecyclerView*
- *FloatingActionButton*
- *SugarORM*

## Feltöltés
Az elkészült megoldást egy ZIP formájában (teljes Android Studio projekt – build mappa kivehető) kell feltölteni a tárgy oldalán, ahol a laborvezető tudja értékelni.

## Az elkészítendő megoldás
Az elkészítendő alkalmazás egy bevásárló listát valósít meg. A listaelemeket adhatunk hozzá, törölhetünk és szerkeszthetünk. Az alkalmazás a bevásárló lista elemeit perzisztensen tárolja.
A képernyők mintaképei:

<p align="center">
<img src="./assets/shopping_list.png" width="320">
<img src="./assets/new_item.png" width="320">
</p>

Az alkalmazás egy *Activity*-ből áll, mely a listát jeleníti meg. Új elemet a *FloatingActionButton* segítségével adhatunk hozzá, mely a jobb alsó sarokban található. Erre kattintva egy dialógus jelenik meg, melynek segítségével megadhatjuk a vásárolni kívánt áru nevét, leírását, kategóriáját és becsült árát.
Az adatok a lista elemen megjelennek. *CheckBox* segítségével jelezhetjük a már megvásárolt elemeket. A kuka ikonra kattintva törölhetjük az egyes elemeket.
A menüben található „Remove all” opcióval az összes listaelemet törölhetjük.

## Laborfeladatok
A labor során az alábbi feladatokat kell megvalósítani a laborvezető segítségével, illetve a jelölt feladatoknál önállóan.

### Kezdő projekt létrehozása
Hozzon létre egy `ShoppingList` nevű projektet Android Studioban. A Company domain-nek `aut.bme.hu`-t adjon meg. Az alkalmazás típusának válassza ki a **Phone and Tablet** opciót, a minimum SDK-nak pedig **API 15-öt (default)**. Az Activity típusok közül a **Basic Activity**-t válassza. Ezután kattintson a **Finish** gombra.
Másolja be a labor anyagban található *res.zip* könyvtár tartalmát a projekt erőforrás mappájába megfelelő módon.

### Perzisztencia megvalósítása (2 pont)
Az adatok perzisztens tárolásához a Sugar ORM (http://satyan.github.io/sugar/) könyvtárat fogjuk használni.

#### Sugar ORM importálása
Ehhez az app modulhoz tartozó build.gradle fájlban a *dependencies* részhez adja hozzá a következő sort:
```gradle
compile 'com.github.satyan:sugar:1.4'
```
Ezután kattintson a jobb felső sarokban a **Sync now** gombra.

A **Sugar ORM** számára a *Manifest*-ben elhelyezett meta-data tag-ek segítségével adhatjuk meg az adatbázist tartalmazó fájl nevét és verzióját, hogy logolja-e a query-ket, illetve hogy milyen domain-t használunk. Frissítsük az *AndroidManifest.xml*-t az alábbiak szerint:
```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest package="hu.bme.aut.shoppinglist"
          xmlns:android="http://schemas.android.com/apk/res/android">

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:supportsRtl="true"
        android:theme="@style/AppTheme"
        android:name=".ShoppingApplication">
        <activity
            android:name=".MainActivity"
            android:label="@string/app_name"
            android:theme="@style/AppTheme.NoActionBar">
            <intent-filter>
                <action android:name="android.intent.action.MAIN"/>

                <category android:name="android.intent.category.LAUNCHER"/>
            </intent-filter>
        </activity>

        <meta-data android:name="DATABASE" android:value="shopping_list.db" />
        <meta-data android:name="VERSION" android:value="2" />
        <meta-data android:name="QUERY_LOG" android:value="true" />
        <meta-data android:name="DOMAIN_PACKAGE_NAME" android:value="hu.bme.aut.shoppinglist" />

    </application>

</manifest>
```
Hozzon létre új Java osztályt: kattintson jobb gombbal az alkalmazás package-re, majd válassza a **New** > **Java class** opciót. Az osztály neve legyen `ShoppingApplication`.
```java
public class ShoppingApplication extends SugarApp {
}
```
Ennek a `SugarApp`-ból kell származnia, ezzel biztosítjuk a Sugar ORM megfelelő inicializálását, illetve lezárását.
Később ha bármikor szükségünk van saját *Application* osztályra ezt az osztályt bővíthetjük.

#### Modell osztály létrehozása
Hozzon létre új Java osztályt: kattintson jobb gombbal az alkalmazás package-re, majd válassza a **New** > **Java class** opciót. Az osztály neve legyen `ShoppingItem`. Másolja be az osztály kódját.
```java
public class ShoppingItem extends SugarRecord {

    public enum Category {
        FOOD, ELECTRONIC, BOOK;

        public static Category getByOrdinal(int ordinal) {
            Category ret = null;
            for (Category cat : Category.values()) {
                if (cat.ordinal() == ordinal) {
                    ret = cat;
                    break;
                }
            }
            return ret;
        }
    }

    public String name;
    public String description;
    public Category category;
    public int estimatedPrice;
    public boolean isBought;
}
```
Látható, hogy ősosztályként a `SugarRecord`-ot használtuk, ez biztosítja, hogy az osztály példányait adatbázisba lehessen menteni.

#### Adapter létrehozása
Következő lépésként az adaptert fogjuk létrehozni, mely a modell elemek megjelenítéséért felel. Hozzunk létre új Java osztályt `ShoppingAdapter` néven. Mivel a listát *RecyclerView* segítségével szeretnénk megjeleníteni, ezért az adapter a `RecyclerView.Adapter` osztályból fog leszármazni. A modell elemeket egy listában fogjuk tárolni. Szükség lesz még egy *ViewHolder* osztály megadására is, ezen keresztül érhetjük majd el a listaelemekhez tartozó *View*-kat.
Az ősosztály három abstract függvényt definiál, amelyeket meg kell valósítanunk. Az `onCreateViewHolder()`-ben hozzuk létre az adott sort megjelenítő *View*-t és a hozzá tartozó *ViewHolder*-t, az `onBindViewHolder()`-ben kötjük hozzá a modell elemhez a nézetet, a `getItemCount()` pedig a listában található elemek számát adja vissza.
```java
public class ShoppingAdapter extends RecyclerView.Adapter<ShoppingAdapter.ShoppingViewHolder> {

    private final List<ShoppingItem> items;

    public ShoppingAdapter() {
        items = new ArrayList<>();
    }
    
    @Override
    public ShoppingViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
        View itemView = LayoutInflater.from(parent.getContext()).inflate(R.layout.item_shopping_list, parent, false);
        ShoppingViewHolder viewHolder = new ShoppingViewHolder(itemView);
        return viewHolder;
    }

    @Override
    public void onBindViewHolder(ShoppingViewHolder holder, int position) {

    }

    @Override
    public int getItemCount() {
        return items.size();
    }


    public class ShoppingViewHolder extends RecyclerView.ViewHolder {

        public ShoppingViewHolder(View itemView) {
            super(itemView);
        }
    }
}
```
Az `R.layout.item_shopping`-ra hibát dog a fordító, hiszen még nem hoztuk létre a layout erőforrást. Kattintson rá, majd nyomja meg az **Alt + Enter** billentyű kombinációt. Válassza az első lehetőséget: **„Create layout resource file item_shopping_list.xml”**. Cseréljük le az újonnan létrehozott fájl tartalmát az alábbira:
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="horizontal"
    android:paddingTop="8dip"
    android:paddingBottom="8dip"
    android:paddingLeft="16dip"
    android:paddingRight="16dip">

    <CheckBox
        android:id="@+id/ShoppingItemIsBoughtCheckBox"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center_vertical"
        android:text="@string/bought"/>

    <ImageView
        android:id="@+id/ShoppingItemIconImageView"
        android:layout_width="64dip"
        android:layout_height="64dip"
        android:layout_marginLeft="8dip"
        android:padding="8dip"
        tools:src="@drawable/open_book"/>

    <LinearLayout
        android:layout_width="0dip"
        android:layout_height="wrap_content"
        android:layout_weight="1"
        android:layout_marginLeft="8dip"
        android:orientation="vertical">


    </LinearLayout>

    <ImageButton
        android:id="@+id/ShoppingItemRemoveButton"
        android:layout_width="50dp"
        android:layout_height="50dp"
        android:layout_gravity="center_vertical"
        android:src="@drawable/ic_delete_grey600_48dp"
        android:scaleType="fitXY"
        style="@style/Widget.AppCompat.Button.Borderless"
        />

</LinearLayout>
```
Ilessze be a belső `LinearLayout`-ba a szövegmezőket:
```xml
<TextView
    android:id="@+id/ShoppingItemNameTextView"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    tools:text="Apple"/>

<TextView
    android:id="@+id/ShoppingItemDescriptionTextView"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    tools:text="My favorite fruit"/>

<TextView
    android:id="@+id/ShoppingItemCategoryTextView"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    tools:text="Food"/>

<TextView
    android:id="@+id/ShoppingItemPriceTextView"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    tools:text="20 Ft"/>
```
Adja hozzá a szükséges képi erőforrásokat.
Hozzuk létre a `@string/bought` erőforrást, ehhez kattintsunk rá a hivatkozásra, majd **Alt + Enter** lenyomása után válasszuk a **„Create string value resource ’bought’”** lehetőséget. A felugró ablakban adjuk meg a kívánt értéket.

Térjünk vissza az adapterhez, és adjuk hozzá a *ViewHolder*-hez a megfelelő mezőket. Ezeken a mezőkön keresztül fogjuk tudni elérni az egyes sorokhoz tartozó nézeteket.
```java
public class ShoppingViewHolder extends RecyclerView.ViewHolder {

    ImageView iconImageView;
    TextView nameTextView;
    TextView descriptionTextView;
    TextView categoryTextView;
    TextView priceTextView;
    CheckBox isBoughtCheckBox;
    ImageButton removeButton;

    public ShoppingViewHolder(View itemView) {
        super(itemView);
        iconImageView = (ImageView) itemView.findViewById(R.id.ShoppingItemIconImageView);
        nameTextView = (TextView) itemView.findViewById(R.id.ShoppingItemNameTextView);
        descriptionTextView = (TextView) itemView.findViewById(R.id.ShoppingItemDescriptionTextView);
        categoryTextView = (TextView) itemView.findViewById(R.id.ShoppingItemCategoryTextView);
        priceTextView = (TextView) itemView.findViewById(R.id.ShoppingItemPriceTextView);
        isBoughtCheckBox = (CheckBox) itemView.findViewById(R.id.ShoppingItemIsBoughtCheckBox);
        removeButton = (ImageButton) itemView.findViewById(R.id.ShoppingItemRemoveButton);
    }
}
```
Valósítsuk meg az onBindViewHolder metódust, vagyis kössük hozzá a nézeteket a megfelelő modell elemhez:
```java
@Override
public void onBindViewHolder(ShoppingViewHolder holder, int position) {
    ShoppingItem item = items.get(position);
    holder.nameTextView.setText(item.name);
    holder.descriptionTextView.setText(item.description);
    holder.categoryTextView.setText(item.category.name());
    holder.priceTextView.setText(item.estimatedPrice + " Ft");
    holder.iconImageView.setImageResource(getImageResource(item.category));
    holder.isBoughtCheckBox.setChecked(item.isBought);
}
```
Adjuk hozzá az adapter osztályhoz a `getImageResource()` metódust:
```java
private @DrawableRes int getImageResource(ShoppingItem.Category category) {
    @DrawableRes int ret;
    switch (category) {
        case BOOK:
            ret = R.drawable.open_book;
            break;
        case ELECTRONIC:
            ret = R.drawable.lightning;
            break;
        case FOOD:
            ret = R.drawable.groceries;
            break;
        default:
            ret = 0;
    }
    return ret;
}
```
Biztosítsunk lehetőséget elem hozzáadására, valamint a teljes lista frissítésére:
```java
public void addItem(ShoppingItem item) {
    items.add(item);
    notifyItemInserted(items.size() - 1);
}

public void update(List<ShoppingItem> shoppingItems) {
    items.clear();
    items.addAll(shoppingItems);
    notifyDataSetChanged();
}
```

### RecyclerView hozzáadása (1 pont)
A lista megjelenítéséhez a `MainActivity` tartalmát egy *RecyclerView*-ra kell lecserélnünk.
Ehhez a `content_main.xml` tartalmát kell frissíteni:
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
Adjuk hozzá az alábbi mezőket a `MainActivity`-hez:
```java
private RecyclerView recyclerView;
private ShoppingAdapter adapter;
```
Adjuk hozzá a *RecyclerView*-t inicializáló kódrészletet. A `loadItemsInBackground()` függvény háttérszálon betölti az adatbázisba mentett elemeket, majd a UI szálon megjeleníti őket (átadja az adapter-nek). Ehhez az *AsyncTask* osztályt használjuk, melynek `doInBackground()` metódusa háttérszálon fut le, az `onPostExecute()` pedig már a UI szálon hívódik meg.
```java
private void initRecyclerView() {
    recyclerView = (RecyclerView) findViewById(R.id.MainRecyclerView);
    adapter = new ShoppingAdapter();
    loadItemsInBackground();
    recyclerView.setLayoutManager(new LinearLayoutManager(this));
    recyclerView.setAdapter(adapter);
}

private void loadItemsInBackground() {
    new AsyncTask<Void, Void, List<ShoppingItem>>() {

        @Override
        protected List<ShoppingItem> doInBackground(Void... voids) {
            return ShoppingItem.listAll(ShoppingItem.class);
        }

        @Override
        protected void onPostExecute(List<ShoppingItem> shoppingItems) {
            super.onPostExecute(shoppingItems);
            adapter.update(shoppingItems);
        }
    }.execute();
}
```
Hívjuk meg a fenti függvényt az `onCreate()`-ből:
```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    Toolbar toolbar = (Toolbar) findViewById(R.id.toolbar);
    setSupportActionBar(toolbar);

    FloatingActionButton fab = (FloatingActionButton) findViewById(R.id.fab);
    fab.setOnClickListener(new View.OnClickListener() {
        @Override
        public void onClick(View view) {
            Snackbar.make(view, "Replace with your own action", Snackbar.LENGTH_LONG)
                    .setAction("Action", null).show();
        }
    });
    initRecyclerView();
}
```
Ezzel az adatbázisban szereplő elemeket meg tudná jeleníteni a program, azonban sajnos még egy elemünk sincs. A következő lépés az új elem létrehozása lesz.
### Dialógus megvalósítása új elem hozzáadásához (1 pont)
A dialógus megjelenítéséhez *DialogFragment*-et fogunk használni. Hozzunk létre új Java osztályt `NewShoppingItemDialogFragment` néven. Az osztályban definiálunk egy listener interface-t, amelyen keresztül az *Activity* értesülhet az új elem létrehozásáról.
```java
public class NewShoppingItemDialogFragment extends AppCompatDialogFragment {

    public static final String TAG = "NewShoppingItemDialogFragment";

    public interface INewShoppingItemDialogListener {
        void onShoppingItemCreated(ShoppingItem newItem);
    }

    private INewShoppingItemDialogListener listener;

    @Override
    public void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        FragmentActivity activity = getActivity();
        if (activity instanceof INewShoppingItemDialogListener) {
            listener = (INewShoppingItemDialogListener) activity;
        } else {
            throw new RuntimeException("Activity must implement the INewShoppingItemDialogListener interface!");
        }
    }

    @Override
    public Dialog onCreateDialog(Bundle savedInstanceState) {
        return new AlertDialog.Builder(getContext()).create();
    }
}
```
A megjelenő dialógust az `onCreateDialog()` függvényben fogjuk összeállítani. Ehhez az `AlertDialog.Builder` osztályt hívjuk segítségül. 
```java
@NonNull
@Override
public Dialog onCreateDialog(Bundle savedInstanceState) {
    return new AlertDialog.Builder(getContext())
            .setTitle(R.string.new_shopping_item)
            .setView(getContentView())
            .setPositiveButton(R.string.ok, new DialogInterface.OnClickListener() {
                @Override
                public void onClick(DialogInterface dialogInterface, int i) {
                    
                }
            })
            .setNegativeButton(R.string.cancel, null)
            .create();
}
```
Hiányzik még a dialógus tartalmát létrehozó `getContentView()` függvény. Adjuk hozzá ezt is:
```java
private View getContentView() {
    View contentView = LayoutInflater.from(getContext()).inflate(R.layout.dialog_new_shopping_item, null);
    nameEditText = (EditText) contentView.findViewById(R.id.ShoppingItemNameEditText);
    descriptionEditText = (EditText) contentView.findViewById(R.id.ShoppingItemDescriptionEditText);
    estimatedPriceEditText = (EditText) contentView.findViewById(R.id.ShoppingItemEstimatedPriceEditText);
    categorySpinner = (Spinner) contentView.findViewById(R.id.ShoppingItemCategorySpinner);
    categorySpinner.setAdapter(new ArrayAdapter<>(getContext(), android.R.layout.simple_spinner_dropdown_item, getResources().getStringArray(R.array.category_items)));
    alreadyPurchasedCheckBox = (CheckBox) contentView.findViewById(R.id.ShoppingItemIsPurchasedCheckBox);
    return contentView;
}
```
Ezt követően hozzuk létre a dialógushoz tartozó layout file-t. Ehhez kattintson a `getContentView()` első sorában található `R.layout.dialog_new_shopping_item`-re, majd **Alt + Enter**-t nyomva válassza az első lehetőséget: **Create layout resource file …**, majd kattintsunk az **Ok**-ra. Másolja be a layout fájl tartalmát:
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="24dip">

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@string/name"/>

    <EditText
        android:id="@+id/ShoppingItemNameEditText"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="16dip"
        android:text="@string/description"/>

    <EditText
        android:id="@+id/ShoppingItemDescriptionEditText"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="16dip"
        android:text="@string/category"/>

    <Spinner
        android:id="@+id/ShoppingItemCategorySpinner"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="16dip"
        android:text="@string/estimated_price"/>

    <EditText
        android:id="@+id/ShoppingItemEstimatedPriceEditText"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:inputType="numberDecimal"/>

    <CheckBox
        android:id="@+id/ShoppingItemIsPurchasedCheckBox"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="@string/already_purchased"/>

</LinearLayout>
```
Adjuk hozzá a hiányzó szöveges erőforásokat a `strings.xml` fájlhoz:
```xml
<string name="new_shopping_item">New shopping item</string>
<string name="ok">Ok</string>
<string name="cancel">Cancel</string>
```
Hozzuk létre a hiányzó tagváltozókat:
```java
private EditText nameEditText;
private EditText descriptionEditText;
private EditText estimatedPriceEditText;
private Spinner categorySpinner;
private CheckBox alreadyPurchasedCheckBox;
```
Iratkozzunk fel a CheckBox onClick eseményére, hogy a vásárlás állapotát frissíteni lehessen az onBindViewHolder(…) függvényben:
```java
holder.isBoughtCheckBox.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View view) {
        boolean checked = holder.isBoughtCheckBox.isChecked();
        Log.d(TAG, "onClick: checked = " + checked + " in position " 
         + holder.getAdapterPosition());
        ShoppingItem item = items.get(holder.getAdapterPosition());
        item.isBought = checked;
        item.save();
    }
});
```
Adja hozzá a `strings.xml`-hez a hiányzó szöveges erőforrásokat:
```xml
<string name="name">Name</string>
<string name="description">Description</string>
<string name="category">Category</string>
<string name="estimated_price">Estimated price</string>
<string name="already_purchased">Already purchased</string>
<string-array name="category_items">
    <item>Food</item>
    <item>Electronic</item>
    <item>Book</item>
</string-array>
```
Az új elemet az positiveButton *onClickListener*-jében fogjuk létrehozni, amennyiben a bevitt adatok validak, vagyis a név ki van töltve.
```java
@Override
public void onClick(DialogInterface dialogInterface, int i) {
    if (isValid()) {
        listener.onShoppingItemCreated(getShoppingItem());
    }
}

private boolean isValid() {
    return nameEditText.getText().length() > 0;
}

private ShoppingItem getShoppingItem() {
    ShoppingItem shoppingItem = new ShoppingItem();
    shoppingItem.name = nameEditText.getText().toString();
    shoppingItem.description = descriptionEditText.getText().toString();
    try {
        shoppingItem.estimatedPrice = Integer.parseInt(estimatedPriceEditText.getText().toString());
    } catch (NumberFormatException e) {
        shoppingItem.estimatedPrice = 0;
    }
    shoppingItem.category = ShoppingItem.Category.getByOrdinal(categorySpinner.getSelectedItemPosition());
    shoppingItem.isBought = alreadyPurchasedCheckBox.isChecked();
    shoppingItem.save();
    return shoppingItem;
}
```
A `MainActivity` `onCreate()`-jében frissítsük a `FloatingActionButton` `OnClickListener`-jét, hogy az a fentebb megvalósított dialógust dobja fel:
```java
fab.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View view) {
        new NewShoppingItemDialogFragment().show(getSupportFragmentManager(), NewShoppingItemDialogFragment.TAG);
    }
});
```
Frissítse a `MainActivity`-t, hogy implementálja a dialógusban definiált interface-t:
```java
public class MainActivity extends AppCompatActivity implements NewShoppingItemDialogFragment.INewShoppingItemDialogListener {

//…

@Override
public void onShoppingItemCreated(ShoppingItem newItem) {
    adapter.addItem(newItem);
}
}
```
Frissítse az `activity_main.xml` layout fájlban a `FloatingActionButton` ikonját:
```xml
<android.support.design.widget.FloatingActionButton
    android:id="@+id/fab"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:layout_gravity="bottom|end"
    android:layout_margin="@dimen/fab_margin"
    android:src="@drawable/ic_add_white_36dp"/>
```
Próbálja ki az alkalmazást.

### Önálló feladat: törlés megvalósítása (1 pont)
- ViewHolder-ben elem pozíciójának tárolása
- Adapter-ben érintésre elem törlése
- Adapter értesítése, hogy frissítse a nézetet
- Menú kiegészítése: összes elem törlése
- Összes elem törlésének megvalósítása

### IMSc feladatok
#### Megerősítő dialógus
Az alkalmazás jelenítsen meg egy megerősítő dialógust, amikor a felhasználó a Remove all menüpontra kattint. A dialógus tartalmazzon egy rövid szöveges figyelmeztetést, hogy minden elem törlődni fog, egy pozitív és negatív gombot (**OK** és **Cancel**). A pozitív gomb lenyomásakor törlődjenek csak az elemek.
#### Elemek szerkesztése
Valósítsa meg a listaelem szerkesztését. A lista elemre helyezzen egy szerkesztés gombot, melynek hatására nyíljon meg a már korábban implementált felviteli dialógus, a beviteli mezők pedig legyenek előre kitöltve a mentett értékekkel. Az **OK** gomb hatására a meglévő listaelem módosuljon az adatbázisban és a nézeten is.





