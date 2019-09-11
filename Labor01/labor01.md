

# Labor 01 - Hello World

## Általános tudnivalók

### iMSC pontok

iMSC pontok a ZH-n, a nagyHF-n és várhatóan néhány laboron szerezhetők. Az első laboron nem szerezhető iMSc pont. A későbbi laborokon ha lesz ilyen lehetőség, az iMSc-s feladatok megoldásainak AUT portálra kell feltölteni ugyanúgy a megfelelő labor alá, de érdemes megjezésként odaírni, hogy mely iMSc feladatok lettek megoldva. Ha egy feladatban kérdések szerepelnek, a pontok csak akkor fogadhatók el, ha mellékletben egy igényes jegyzőkönyv is szerepel a kérdésekre vonatkozó válaszokkal. iMSc pont szerzésére bármely hallgató jogosult, aki az előtte lévő feladatokkal már végzett (laborvezető ellenőrzi a haladást).

### Laborok értékelése

Minden labor leírása tartalmazza, hogy mi a megoldandó feladat. A megoldásokat minden esetben az AUT portálra kell feltölteni.

### Beugró

Az első labort kivéve minden labor előtt lesz beugró, mely feltétele a labor teljesítésének.

### Kis ZH-k

A félév során hat alkalommal kis zárthelyit íratunk a laboratórium alkalmakon. Ezek közül a négy legjobban sikerült kis zárthelyi pontszámnak egyenként el kell érje a szerezhető pontszám 40%-át.

A kisZH-k kettő vagy három hetente lesznek, ezek időpontját az első vagy második előadáson kihirdetjük. A mostani Labor 01-en nincs kisZH.

## Labor 01 információk

A mérés célja, hogy bemutassa az Android fejlesztőkörnyezetet, az alkalmazáskészítés, illetve a tesztelés és fordítás folyamatát, az alkalmazás felügyeletét, valamint az emulátor és a fejlesztőkörnyezet funkcióit. Továbbá, hogy ismertesse egy Hello World alkalmazás elkészítésének módját.
A mérés során a laborvezető részletesen bemutatja az eszközöket.

A mérés az alábbi témákat érinti:

*   Az Android platform alapfogalmainak ismerete
*   Android Studio fejlesztőkörnyezet alapok
*   Android Emulátor tulajdonságai
*   Monitoring és log nézet
*   Android projekt létrehozása és futtatása emulátoron
*   Manifest állomány felépítése

###  A labor értékelése
A feladat a leírás végén szereplő feladatok megoldása. **A megoldásokról készítsen jegyzőkönyvet (képernyőképekkel és lényegre törő szöveggel), melyet a tárgy oldalára töltsön fel a labor végén!**


###  Fordítás menete Android platformon

A projekt létrehozása után a forráskód az _src_ könyvtárban, míg a felhasználói felület leírására szolgáló XML állományok a _res_ könyvtárban találhatók. Az erőforrás állományokat egy “_R.java”_ állomány köti össze a forráskóddal, így könnyedén elérhetjük Java oldalról az XML-ben definiált felületi elemeket. Az Android projekt fordításának eredménye egy APK állomány, melyet közvetlenül telepíthetünk mobil eszközre.

1.  A fejlesztő elkészíti a Java forráskódot, valamint az XML alapú felhasználói felületleírást a szükséges erőforrás állományokkal.

2.  A fejlesztőkörnyezet az erőforrás állományokból folyamatosan naprakészen tartja az „_R.java_” erőforrás fájlt a fejlesztéshez és a fordításhoz. **FONTOS: az “_R.java_” állomány generált, kézzel SOHA ne módosítsuk!**

3.  A fejlesztő a Manifest állományban beállítja az alkalmazás hozzáférési jogosultságait (pl. Internet elérés, szenzorok használata, stb.).

4.  A fordító a forráskódból, az erőforrásokból és a külső könyvtárakból előállítja a Dalvik/[ART](https://hu.wikipedia.org/wiki/Android_Runtime) virtuális gép gépi kódját.

5.  A gépi kódból és az erőforrásokból előáll a nem aláírt APK állomány.

6.  Végül a rendszer végrehajtja az aláírást és előáll a készülékekre telepíthető aláírt APK.

Az Android Studio Gradle-t használ, ami lehetőséget biztosít köztes állapot produkálására, erről részletesebben a Gradle részben lesz szó (későbbi labor anyaga).

**Megjegyzések:**

*   A teljes folyamat a fejlesztői gépen megy végbe, a készülékekre már csak bináris állomány jut el.

*   A külső könyvtárak általában JAR állományként, vagy egy másik projekt hozzáadásával illeszthetők az aktuális projekthez.

*   Az APK állomány leginkább a Java világban ismert JAR állományokhoz hasonlítható.

*   A Manifest állományban meg kell adni a támogatni kívánt Android verziót, mely felfele kompatibilis az újabb verziókkal, régebbi verzióra azonban már nem telepíthető.

*   Az Android folyamatosan frissülő verziói nagy gondot jelentenek a fejlesztőknek.

*   Az Android alkalmazásokat tipikusan az Android Playben szokták publikálni, így az APK formátumban való terjesztés nem annyira elterjedt.

*   A teljes folyamat a szoftverfejlesztők számítógépein megy végbe, az ügyfélhez a bájtkódokat tartalmazó kész alkalmazás jut el.

![](assets/lab-1-compile.png)

Fordítás menete Android platformon


### SDK és könyvtárai

A [d.android.com/sdk](https://developer.android.com/studio) oldalról letölthető az IDE és az SDK. Ennek fontosabb mappáit, eszközeit **tekintsék át a laborvezető segítségével**!

![](assets/ide_android.png)

SDK szerkezet


*   **docs:** Dokumentáció
*   **extras:** különböző extra szoftverek helye. Maven repository, support libes anyagok, analytics sdk, google [android usb driver](https://developer.android.com/studio/run/win-usb.html) (amennyiben SDK managerrel ezt is letöltöttük) stb.
*   **platform-tools:** fastboot és adb binárisok helye (legtöbbet használt eszközök)
*   **platforms, samples, sources, system-images:** minden API levelhez külön almappában a platform anyagok, források, példaprojektek, OS image-k
*   **tools:** fordítást és tesztelést segítő eszközök, SDK manager, 9Patch drawer, emulátor binárisok stb.

### AVD és SDK manager

Az SDK kezelésére az SDK managert használjuk, ezzel lehet letölteni és frissen tartani, az eszközeinket. Indítása a *tools/android binárissal*, paraméter nélkül történhet, vagy fejlesztői környezeten keresztül.

SDK manager ikon:

![](assets/sdk_manager_icon_new.PNG)

SDK manager felülete:

![](assets/sdk_manager_new.PNG)

Indítsuk el, és vizsgáljuk meg a laborvezetővel, rendelkezésre áll-e minden, ami az első alkalmazásunkhoz kelleni fog.

### AVD

Az AVD az Android Virtual Device rövidítése. Ahogy arról már előadáson is szó esett, nem csak valódi eszközön futtathatjuk a kódunkat, hanem emulátoron is. (Mi is a különbség szimulátor és emulátor között?) Az AVD indítása a *tools/android bináris ‘avd’* paraméterrel, vagy fejlesztői környezeten keresztül lehetséges.

AVD ikon:

![](assets/avd_icon_new.PNG)

![](assets/avd.png)

Az AVD megnyitásakor a létező virtuális eszközök listáját láthatjuk. A létrehozás gombra kattintva elkezdhetjük egy új virtuális készülék összeállítását. Néhány előre elkészített sablon áll rendelkezésre, magunk is készíthetünk ilyet, ha tipikusan adott eszközre szeretnénk fejleszteni (pl. galaxy s4). Készítsünk új emulátort (értelemszerűen csak olyan API szintű eszközt készíthetünk, amilyenek rendelkezésre állnak az SDK manageren keresztül)!

1.  Kattintsunk a "Create Virtual Device..." gombra!
2. Válasszuk ki a kiindulási eszköz definíciót!
    1. Kategória: "Phone"
    2. Az eszköz legyen például Pixel 2.
3. A következő lapon a megfelelő rendszer image-et tudjuk kiválasztani.
    1. Válasszunk az ajánlottak ("Recommended") közül olyan image-et, ami már le van töltve a gépünkre! 
4. A konfiguráció véglegesítése oldalon részletesebben is testre szabhatjuk a virtuális készülékünket.
    1. A készülék neve legyen mondjuk “Labor_1″.
    2. Beállíthatjuk, hogy látszódjon-e a készülék kerete az emulátoron.
    3. A "Show Advanced Settings" gombra kattintva egyéb, részletesebb beállításokat módosíthatunk:
        Kamera(ák): WebcamX, hardveres kamera, ami a számítógépre van csatlakoztatva; Emulated, egy szoftveres megoldás, most legalább az egyik kamera legyen ilyen.
        Szimulált hálózati sebesség.
        Virtuális eszköz indulása: újonnani indítás, legutóbbi, vagy tetszőleges mentett állapot alapján. 
        Memória mérete. A laborszámítógépeken, mivel kevés a rendszermemóriánk nem érdemes 768 MB-nál többet adni, könnyen futhat az ember problémákba. Ha az emulátor lefagy, vagy az egész OS megáll működés közben, akkor állítsuk alacsonyabbra az értéket (saját laptop esetén 8GB vagy több rendszermemória esetén nyugodtan állíthatjuk az emulátor memóriáját 1024/2048MB-ra). VM heap, az alkalmazások virtuális gépének szól, maradhat az alapérték. Tudni kell, hogy készülékek esetében gyártónként változik.
        Belső flash memória és SD kártya mérete.
        Billentyűzet: gépelhessünk-e a csatlakoztatott hardveres billentyűzettel.
5.  Ha mindent rendben talál az ablak, akkor Finish!

![](assets/avd_create.png)

Új emulátor készítése

Indítsuk el az új emulátort! A felbukkanó Launch options ablakban lehetőségünk van leskálázni a felbontást (például arra az esetre, ha esetleg nem 1:1 arányban szeretnénk megfeleltetni  1920×1080 pixelt az 1366×768-as kijelzőnkön), törölhetjük az adatokat (wipe user data ~ gyári visszaállítás) illetve befolyásolhatjuk a snapshot állapotát.

Az elindított emulátoron próbálják ki az “API Demos” és “Dev Tools” alkalmazásokat!

Megjegyzés: A gyári emulátoron kívül több alternatíva is létezik, a Genymotion az egyik legjobb, nagyon gyors AMD processzorokon is (igaz Windows 10 esetén problémás a használata).

## Fejlesztői környezet

Android fejlesztésre a labor során a JetBrains IntelliJ alapjain nyugvó Android Studio-t fogjuk használni. A Studio-val ismerkedők számára kivételesen hasznos funkció a “Tip of the day”, érdemes egyből kipróbálni, megnézni az adott funkciót. Induláskor a legutóbbi projekt nyílik meg, ha nincs ilyen, vagy minden projektünket bezártuk, a menüpontok értelemszerűek.


## Hello World

A laborvezető segítségével készítsenek egy egyszerű Hello World alkalmazást!

###  Android Studio

Ez a rész azoknak szól, akik korábban már használták az Eclipse nevű IDE-t, és szeretnék megismerni a különbségeket az Android Studio-hoz képest.

*   **Import régi projektekből:** Android Studioban lehetséges a projekt importálása régebbi verziójú projektekből és a régi Eclipse projektekből is.
*   **Projektstruktúra:**  A Studio Gradle-lel fordít, és más felépítést használ. Projekten belül:
    *   .idea: IDE fájlok
    *   app: forrás
        *   build: fordított állományok
        *   libs: libraryk
        *   src: forráskód, azon belül is külön projekt a tesztnek, és azon belül pedig “_res_” könyvtár, illetve “_java_“. Java-n belül már a csomagok vannak.
    *   gradle: gradle fájlok
*   **Hasznos funkciók:**
    *   IntelliSense, fejlett refaktoráció támogatás
    *   Ha egy sorban színre, vagy képi erőforrásra hivatkozunk, a sor elejére kitesz egy miniatűr változatot.
    *   Ha közvetve hivatkozott erőforrást (akár getResources.get…, akár R…..) adunk meg, összecsukja a hivatkozást és a tényleges értéket mutatja. Ha rávisszük az egeret felfedi, ha kattintunk kibontja a hivatkozást.
    *   Névtelen belső osztályokkal is hasonlót tud, javítva a kód olvashatóságát.
    *   Kódkiegészítésnél szabad a kereső, a szótöredéket keresi, nem pedig a szóval kezdődő lehetőségeket (lásd képen)
    *   Változónév ajánlás: amikor változónévre van szükségünk, nyomjunk CTRL+SPACE-t. Ha adottak a körülmények, a Studio egész jó neveket tud felajánlani.
    *   Szigorú lint. A Studio megengedi a warningot. Ezért szigorúbb a lint, több mindenre figyelmeztet (olyan apróságra is, hogy egy view egyik oldalán van padding, a másikon nincs)
    *   Layout szerkesztés. A grafikus layout építés lehetséges, 2016-os béta verziós újítás: *CoordinatorLayout*.
    *   CTRL-t lenyomva navigálhatunk a kódban (pl. osztályra, metódushívásra kattintva). Ezt a navigációt (és az egyszerű másik osztályba kattintást is) rögzíti, és a history előre-hátra gombokkal lehet lépkedni benne. Ha van az egerünkön/billentyűzetünkön ilyen gomb, és netes böngészés közben aktívan használjuk, ezt a funkciót nagyon hasznosnak fogjuk találni.
    *   Szín ikonja a sor elején; kiemelve jobb oldalon, hogy melyik nézeten vagyunk; szabadszavas kiegészítés; a “Hello world” igazából _“@string/very_very_very_long_hello_world”_

![](assets/Nice-studio.png)

### Billentyűkombinációk

*   **CTRL + ALT + L:** Kódformázás
*   **CTRL + SPACE:** Kódkiegészítés
*   **SHIFT + F6:** Átnevezés (Mindenhol)
*   **F2:** A következő error-ra ugrik. Ha nincs error, akkor warningra.
*   **CTRL + Z** illetve **CTRL + SHIFT + Z:** Visszavonás és Mégis
*   **CTRL + P:** Paraméterek mutatása
*   **ALT + INSERT:** Metódus generálása
*   **CTRL + O:** Metódus felüldefiniálása
*   **CTRL + F9:** Fordítás
*   **SHIFT + F10:** Fordítás és futtatás
*   **SHIFT SHIFT:** Keresés mindenhol

Érdemes otthon tanulmányozni a további billentyű kombinációkat.

### Eszközök, szerkesztők

A View menü Tool Windows menüpontjában lehetőség van különböző ablakok ki- és bekapcsolására. Laborvezető segítségével tekintsék át az alábbi eszközöket!

*   Project
*   Structure
*   Debug
*   TODO
*   Terminal
*   Messages
*   Gradle

Lehetőség van felosztani a szerkesztőablakot, ehhez kattinsunk egy megnyitott fájl tabfülére jobb gombbal!

### Hasznos beállítások

A laborvezető segítségével állítsák be a következő hasznos funkciókat:

*   kis- nagybetű érzékenység kikapcsolása a kódkiegészítőben (settingsben keresés: sensitive)
*   "laptop mód" ki- és bekapcsolása (File/Power Saver mode)
*   sorszámozás bekapcsolása (settingsben keresés: Show line numbers)
*   beírás közbeni autoimport bekapcsolása (settingsben keresés: import, utána Editor/Auto import)

### Generálható elemek

A Studio sok sablont tartalmaz, röviden tekintsék át a lehetőségeket:

*   Projektfában, projektre jobb gombbal kattintva -> new -> module
*   Projektfában, modulon belül, Java-ra kattintva jobb gombbal -> new
*   Forráskódban ALT+INSERT billentyűkombinációra

## LogCat

Keressük meg az Android Studioban a LogCat nézetet, vizsgáljuk meg hol lehet megtekinteni a log és hiba üzeneteket, hogyan lehet képernyőképet és videot készíteni az emulátorról és csatlakoztatott készülékről.

## Monitoring

Vizsgáljuk meg megnyitott projekt esetén az Android Studio Tools menüpontját, a laborvezetővel elemezzük az itt látható funkciókat.
Hasonlóképpen próbáljuk ki a Run->Profile lehetőséget, valamint az Analyze->Inspect code-ot.
Összegezzük a tapasztalatokat.

## Feladatok:

**A megoldásokról készítsen jegyzőkönyvet (képernyőképekkel és lényegre törő szöveggel), melyet a tárgy oldalára töltsön fel a labor végén!**

1.  Az új alkalmazást futtassák emulátoron (akinek saját készüléke van, az is próbálja ki)!
2.  Helyezzenek breakpointot a kódba, és debug módban indítsák az alkalmazást! (érdemes megyfigyelni, hogy most másik Gradle Task fut a képernyő alján)
3.  Indítsanak hívást és küldjenek SMS-t az emulátoron! Mit tapasztalnak?
4.  Tekintse át az emulátor vezérlésének lehetőségeit a laborvezető segítségével!
5.  Hajtson végre egy bejövő telefonhívást emulátorban!
6.  Küldjön egy SMS-t az emulátor vezérlő nézetről!
7.  Változtassa meg az emulátor fölrajzi koordinátáit!
8.  Vizsgálja meg az elindított HelloWorld projekt nyitott szálait, memóriafoglalását!
9.  Vizsgálja meg a LogCat nézet tartalmát!
10. Vizsgálja meg az Analyze->Inspect code eredményét.
11. Keresse ki a létrehozott HelloWorld projekt mappáját és az app/build/outputs/apk útvonalon található könyvtáron belül
    vizsgálja meg az .apk állomány tartalmát! Hol található a lefordított kód?
