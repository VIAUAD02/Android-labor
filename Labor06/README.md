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
<img src="./assets/shopping_list.png" width="160">
<img src="./assets/new_item.png" width="160">
</p>

Az alkalmazás egy Activity-ből áll, mely a listát jeleníti meg. Új elemet a FloatingActionButton segítségével adhatunk hozzá, mely a jobb alsó sarokban található. Erre kattintva egy dialógus jelenik meg, melynek segítségével megadhatjuk a vásárolni kívánt áru nevét, leírását, kategóriáját és becsült árát.
Az adatok a lista elemen megjelennek. CheckBox segítségével jelezhetjük a már megvásárolt elemeket. A kuka ikonra kattintva törölhetjük az egyes elemeket.
A menüben található „Remove all” opcióval az összes listaelemet törölhetjük.


