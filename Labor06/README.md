# Labor 06 - Bevásárló alkalmazás készítése

## Bevezető
A labor célja az Android alapvető felületi elemeivel történő megismerkedés, valamint a perzisztencia megvalósítási lehetőségei közül az ORM bemutatása egy konkrét példán keresztül

Felhasznált technológiák:
- Activity
- Fragment
- RecyclerView
- FloatingActionButton
- SugarORM

## Feltöltés
Az elkészült megoldást egy ZIP formájában (teljes Android Studio projekt – build mappa kivehető) kell feltölteni a tárgy oldalán, ahol a laborvezető tudja értékelni.

## Az elkészítendő megoldás
Az elkészítendő alkalmazás egy bevásárló listát valósít meg. A listaelemeket adhatunk hozzá, törölhetünk és szerkeszthetünk. Az alkalmazás a bevásárló lista elemeit perzisztensen tárolja.
A képernyők mintaképei:

<p align="center">
<img src="./assets/shopping_list.png" width="320">
<img src="./assets/new_item.png" width="320">
</p>

Az alkalmazás egy Activity-ből áll, mely a listát jeleníti meg. Új elemet a FloatingActionButton segítségével adhatunk hozzá, mely a jobb alsó sarokban található. Erre kattintva egy dialógus jelenik meg, melynek segítségével megadhatjuk a vásárolni kívánt áru nevét, leírását, kategóriáját és becsült árát.
Az adatok a lista elemen megjelennek. CheckBox segítségével jelezhetjük a már megvásárolt elemeket. A kuka ikonra kattintva törölhetjük az egyes elemeket.
A menüben található „Remove all” opcióval az összes listaelemet törölhetjük.

## Laborfeladatok
A labor során az alábbi feladatokat kell megvalósítani a laborvezető segítségével, illetve a jelölt feladatoknál önállóan.

### Kezdő projekt létrehozása
Hozzon létre egy ShoppingList nevű projektet Android Studioban. A Company domain-nek aut.bme.hu-t adjon meg. Az alkalmazás típusának válassza ki a Phone and Tablet opciót, a minimum SDK-nak pedig API 15-öt (default). Az Activity típusok közül a Basic Activity-t válassza. Ezután kattintson a Finish gombra.
Másolja be a labor anyagban található drawables.zip könyvtár tartalmát a projekt erőforrás mappájába megfelelő módon.

### Perzisztenica megvalósítása
Az adatok perzisztens tárolásához a Sugar ORM (http://satyan.github.io/sugar/) könyvtárat fogjuk használni.

#### Sugar ORM importálása
Ehhez az app modulhoz tartozó build.gradle fájlban a dependencies részhez adja hozzá a következő sort:
```gradle
compile 'com.github.satyan:sugar:1.4'
```
Ezután kattintson a jobb felső sarokban a Sync now gombra.

A Sugar ORM számára a Manifest-ben elhelyezett meta-data tag-ek segítségével adhatjuk meg az adatbázist tartalmazó fájl nevét és verzióját, hogy logolja-e a query-ket, illetve hogy milyen domain-t használunk. Frissítsük az AndroidManifest.xml-t az alábbiak szerint:
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
Hozzon létre új Java osztályt: kattintson jobb gombbal az alkalmazás package-re, majd válassza a New > Java class opciót. Az osztály neve legyen ShoppingApplication.
```java
public class ShoppingApplication extends SugarApp {
}
```
Ennek a SugarApp-ból kell származnia, ezzel biztosítjuk a Sugar ORM megfelelő inicializálását, illetve lezárását.
Később ha bármikor szükségünk van saját Application osztályra ezt az osztályt bővíthetjük.




