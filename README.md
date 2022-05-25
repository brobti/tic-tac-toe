# 05 OpenManipulator - Amőba

A projekt a BME-MOGI Kognitív robotika és Robotrendszerek laboratórium tantárgyak projektfeladataként valósult meg.

**Fejlesztők:**

Bartalos Róbert

Dobó Viktor

Galovics Bálint

Szabó Bendegúz

**Oktatók:**

Dudás Dávid

Nagy Balázs


## Migrálás, setup

A BME-MOGI Mesterséges Intelligencia és Kognitív Robotika Laboratóriumában található Ubuntu 20.04.3 operációs rendszert futtató desktop gépre telepítésre került a ROS Noetic (1.15.13) disztribúciója egy korábbi Telemanipulátor projekt migrálása céljából. A ROBOTIS hivatalos honlapja alapján telepítettük az OpenManipulatort.

https://emanual.robotis.com/docs/en/platform/openmanipulator_x/overview/

A felhasznált, eredeti csomagok telepítési utasításai:
```
sudo apt install ros-noetic-ros-controllers 
```
```
sudo apt install ros-noetic-gazebo 
```
```
sudo apt install ros-noetic-moveit 
```
```
sudo apt install ros-noetic-industrial-core
```
```
sudo apt install ros-noetic-dynamixel-sdk 
```
```
sudo apt install ros-noetic-dynamixel-workbench
```
```
sudo apt install ros-noetic-joint-state-publisher-gui
```
```
sudo apt install ros-noetic-robotis-manipulator
```
```
git clone -b noetic-devel https://github.com/ROBOTIS-GIT/open_manipulator.git #nem szükséges telepíteni, lecserélésre került
```
```
git clone -b noetic-devel https://github.com/ROBOTIS-GIT/open_manipulator_msgs.git
```
```
git clone -b noetic-devel https://github.com/ROBOTIS-GIT/open_manipulator_simulations.git
```
```
git clone https://github.com/ROBOTIS-GIT/open_manipulator_dependencies.git
```
```
<git clone https://github.com/ROBOTIS-GIT/open_manipulator_controls.git #nem szükséges telepíteni, lecserélésre került
```

## Amőbázó robot
### Rövid összefoglalás
A projektünk során Robotis OpenMANIPULATOR-X robottal valósítottunk meg egy színfeismeréssel működő amőbaprogramot, aminek során a robot lejátszik magával egy amőbajátszmát, és minden lépés előtt megnézi és feldolgozza a jelenlegi állást. Azért nem a korábbi lépések elmentését használtuk a következő lépés eldöntéséhez, mert így a projekt átalakítható később olyanra, amelyben a robot ember ellen játszik.
A játék pályája 3x3-as, a robot emellől veszi fel a bábukat, 4 kéket és 4 pirosat. A projekt 3 fő részből áll, a következőkben ezeket vesszük végig.
### További szükséges package-ek
A fentebb felsorolt repositorykon kívül három további került telepítésre.
```
git clone https://github.com/MOGI-ROS/open_manipulator_tools.git #nem szükséges telepíteni, lecserélésre került
```
```
git clone https://github.com/MOGI-ROS/gazebo_ros_link_attacher.git
```
```
git clone https://github.com/MOGI-ROS/open_manipulator_ikfast_plugin
```

Összesen három csomagban végeztünk módosításokat a projekt kivitelezéséhez. Ezeket forkoltuk, és létrehoztunk új brancheket a projektünknek. Az amőbázó robot futtatásához ezek szükségesek.
Az általunk létrehozott branchek telepítési utasításai:
```
git clone -b noetic-devel-mod https://github.com/brobti/open_manipulator.git
```
```
git clone -b telemanipulator https://github.com/brobti/open_manipulator_controls.git
```
```
git clone -b ticTacToe https://github.com/brobti/open_manipulator_tools.git
```

### Futtatás
#### Szimuláció
```
[PC terminal #1] $ roscore
[PC terminal #2] $ roslaunch open_manipulator_controllers joint_trajectory_controller.launch
[PC terminal #3] $ rosrun open_manipulator_controller colorRecognition.py sim
[PC terminal #4] $ rosrun open_manipulator_tools inverse_kinematics.py sim
[PC terminal #5] $ rosrun open_manipulator_controller ticTacToe.py sim
```
Videó a szimulációról:

[![szimuláció](https://img.youtube.com/vi/HVNNPpgNrNY/0.jpg)](https://www.youtube.com/watch?v=HVNNPpgNrNY)

#### Valós robot futtatása
Valós kamera elindítása:
Feltéve, hogy a felhasznált Rasberry Pi kapcsolódik a wifi hálózatra, csatlakozzunk rá, ez lesz a `[PI terminal]`:

IP: 10.0.0.11

Jelszó: turtlebot

```
[PI terminal] $ ssh ubuntu@10.0.0.11
```
A robot:
```
[PC terminal #1] $ roscore
[PI terminal   ] $ rosrun mecanum_anomaly_det StreamCam.py
[PC terminal #2] $ roslaunch open_manipulator_controllers joint_trajectory_controller.launch sim:=false
[PC terminal #3] $ rosrun open_manipulator_controller colorRecognition.py
[PC terminal #4] $ rosrun open_manipulator_tools inverse_kinematics.py
[PC terminal #5] $ rosrun open_manipulator_controller ticTacToe.py
```

Videó a valós robottal történő futtatásról:

[![futtatás valós robottal](https://img.youtube.com/vi/ySCi5xbLK7A/0.jpg)](https://www.youtube.com/watch?v=ySCi5xbLK7A)

### Színfelismerés
A színfelismeréshez a https://github.com/MOGI-ROS/Week-5-6-Gazebo-sensors 5. pontjában használt kódból indultunk ki. A szimulált és a valós kamera a `head_camera/image_raw` Ros topicon keresztül folyamatosan publisholja a képet, erre csatlakozik az `open_manipulator/open_manipulator_controller/scripts/color_recognition.py` subscribere.
Az implementált színfelésmerés OpenCV használatával két, előre beállított RGB-értéket keres a képen. Az ismert RGB-értékek alapján `binary treshold` algoritmussal mindhárom színcsatornára elkészíti a maszkot, majd ezek metszetéből meghatározza a tényleges bináris képet.
A bináris képből az open_cv beépített `cv2.findContours()` algoritmusával meghatározzuk a bábuink felületét és azok középpontjait.
A megharározott középpontokat az egyszerűség kedvéért egy stringbe összefűzve publisholjuk a `/color_recognition` topicra. Ezen felül a bináris képeket elhelyeztük a nyers képen, és azt vizuálisan megjelenítettük a könnyű kezelhetőség érdekében.
#### A színfelismerés elindítása:
Szimulációval:
```
[PC terminal #1] $ roscore
[PC terminal #2] $ roslaunch open_manipulator_controller open_manipulator_controller.launch
[PC terminal #3] $ rosrun open_manipulator_controller color_recognition.py
```

![alt text](/images/color_recognition_simulation.png?raw=true)

Valós kamerával:
```
[PC terminal #1] $ roscore
[PI terminal   ] $ rosrun mecanum_anomaly_det StreamCam.py
[PC terminal #2] $ rosrun open_manipulator_controller color_recognition.py
```

![alt text](/images/valos_kamera.PNG?raw=true)

### Inverz kinematika
Az inverz kinematika tényleges implementációja már a https://github.com/MOGI-ROS/open_manipulator_tools repositoryban megtörtént. Ebből készítettük a https://github.com/brobti/open_manipulator_tools forkot, majd ennek a ticTacToe branch-ét. Ezen a branchen került implementálásra az `open_manipulator_tools/inverse_kinematics.py`, a robot és az amőbázó szkript közötti kommunikációt megvalósító `action server`.
#### Action Server működése
Az általunk definiált `kinematicsAction` message a következőképpen épül fel:
 - input: x, y, z, angle
 - feedback: time_elapsed
 - result: result

Az action server az `/arm_controller/command` node-ra publisholja az inverz kinematikával számolt csuklószögeket, majd a `/joint_states` node-ra feliratkozik, és innen a csuklószögeket kinyeri. A szkript folyamatosan fut, amíg a kiküldött és az érkező joint szögek különbsége egy delta érték alá nem esik, ekkor leáll és a `result` értéket `True`-ra állítja. Emellett a folyamat során a `time_elapsed` számlálót folyamatosan inkrementálja egy maximális értékig. Amennyiben a számláló eléri ezt az értéket, a folyamat timeouttal leáll, és `False` lesz a `result` értéke.
### A main function
#### A pálya feldolgozása
A `/color_recognition` topicról a `rospy.wait_for_message('/color_recognition', String, 5)` paranccsal kérjük le a táblán látható bábuk képkoordinátáit és színeit, mikor egy megadott rátekintési pozícióból figyel a robot. Ezekre a koordinátákra megnézzük sor és oszlop szerint, hogy az előre meghatározott keresési intervallumon belül vannak-e, és ezek alapján töltünk fel egy 9 elemű vektort, amely a jelenlegi játszma állását tartalmazza. Minden lépés előtt megnézzük az állást, és ez alapján választunk pozíciót (a következő alfejezetben részletezett módon), ahova le szeretnénk tenni a bábut. Ez a módszer azért jobb, mint ha elmentenénk a korábbi lépéseinket, mert át lehet alakítani olyan formára, ahol a robot emberrel játszik.
#### Amőba algoritmus
Az amőba algoritmushoz használt kiindulási alap: https://www.techwithtim.net/tutorials/python-programming/tic-tac-toe-tutorial/.
Az algoritmus kap egy pályát és játékost (piros vagy kék) bemenetként, és választ hozzá lépést. Első lépésként elmenti az üres pozíciók indexeit. Amennyiben a pálya üres, véletlenszerűen választ. Ha már vannak fent bábuk, megnézi, meg tudja-e nyerni az adott játékos egy lépéssel a játszmát, vagy ha nem, meg tudja-e akadályozni azt, hogy az ellenfél nyerjen. Mikor ilyen helyzetek nincsenek, először a pálya közepére próbál rakni, ha az foglalt, akkor véletlenszerűen választ magának egy üres sarkot, ha pedig ilyen nincsen, az egyik oldalra rak.
Abban az esetben, ha valaki nyer, vagy elfogynak az asztal mellől a bábuk, a szimuláció vagy a fizikai robot megáll.
#### Pick and place állapotautomata
A bábuk felvétele és lerakása egy mozgássorozat következménye. A mozgássorozat lépéseinek egymás utáni lefutását egy állapotautomata szabályozza. Erre a rospy node folyamatos zavartalan futása miatt is szükség van, nem szeretnénk, hogy a node hosszabb ideig leálljon, amíg várakozik. Az állapotautomata leegyszerűsített felépítése látható az alábbi ábrán:

![az állapotgép blokkvázlata](/images/pick-place.png?raw=true)

#### Move függvény
Egy teljes lépés 3 fő részből áll: Először a robot a felvétel helyére mozog, és felveszi a megfelelő bábut. Ezután a megfelelő pozícióra mozog a tábla fölé, majd lehelyezi a bábut. Az utolsó lépés a kiindulási pozícióba való visszatérés. A folyamat az alábbi ábrán követhető:

![egy lépés folyamata](/images/move.png?raw=true)

### Modellek és alkatrészek
A kameratartó STL-modelljének forrása: https://cad.onshape.com/documents/317f41cd6ef3f111631e9f97/w/be4693711e5767f7686bfed7/e/3490ca5c51c70a615d19ad93. A kameramodell forrása: https://www.thingiverse.com/thing:2376448. A könnyebb kezelhetőség miatt a kamera és a tartó modelljeit az Inventor CAD szoftverben összeállítottuk, és a szimulációban együtt kezeltük. A robotmodellben alapértelmezetten szerepelnek a rögzítő csavarok, ezért az ütközések elkerülése érdekében a furatok átmérőjét megnöveltük.
A kék és piros bábuk saját tervezésűek. 
Mindhárom modellt a Blender programmal alakítottuk át .dae kiterjesztésű Collada mesh-sé. A ticTacToe_camera_test_final.world fájl ezekre hivatkozik.
A valós robotra szerelt kameratartó és a bábuk PLA-ból készültek, FDM technológiás 3D nyomtató berendezéssel. A gyártáshoz a Polimertechnika Tanszék biztosította az eszközöket, az alapanyagot és a felkészítést.

### A szimuláció és a valós robotirányítás közti eltérések és azok magyarázata
open_manipulator/open_manipulator_controler/scripts/ticTacToe.py
- A megfogó kinyitása és becsukása különböző tartományban állítható a szimulációban és a valóságban
- A megfogóhoz a valóságban nem kell `attach` és `detach` parancs a működéshez, ez csak a modellezés miatt szükséges.
- A valóságban a gripper helyes alapállapotba állításához szükség van egy becsukás-kinyitás sorozatra.
- A szimulációban az asztal pontos magassága nem ismert, ezért a robotkar nem pontosan az asztallap felületén, hanem pár milliméterrel afölött helyezkedik el. Ezért a bábuk letevéséhez a valósághoz képest kicsit eltérő magassági koordinátát kellett megadni.

open_manipulator/open_manipulator_controler/scripts/colorRecognition.py
- A valóságban a bábuk más színűek, és a megvilágítás függvényében a valós színüket is másnak látja a kamera, ezért a bábuk színének RGB értékei külön vannak kódolva a két esetre.
- A valós kamera más pozícióból néz rá a táblára, közelebbről látja azt. Emiatt állítottunk a rátekintési magasságon, így hasonló képet lát a két esetben.

open_manipulator_tools/scripts/inverse_kinematics.py
- A szimulációban a `/joint_states` topicon tudjuk figyelni a jointok helyzetét, viszont a valós működéskor ezen a topicon nem érkezik adat. A működést utóbbi esetben nem tudjuk biztosítani a visszajelzés ismeretében, ezért a parancs kiadása után egy ideig várunk, majd bízva abban, hogy a robot elért a megfelelő pozícióba, továbbhaladunk a következő utasításra.
## Releváns módosított fájlok teljes listája:
- open_manipulator `branch: noetic-devel-mod`
  - open_manipulator_controller
    - scripts
      - colorRecognition.py
      - ticTacToe.py
    - srv
      - kinematicsMessage.srv
  - open_manipulator_description
    - meshes
      - hengeres_babu_kek.dae 
      - hengeres_babu_piros.dae
      - tartos_kamera3.dae 
    - urdf
      - open_manipulator.gazebo.xacro
      - open_manipulator.urdf.xacro 
- open_manipulator_controls `branch: telemanipulator`
  - open_manipulator_controllers
    - launch
      - joint_trajectory_controller.launch
  - open_manipulator_hw
    - launch
      - open_manipulator_gazebo.launch 
  - open_manipulator_moveit_config
    - config
      - kinematics.yaml
    - launch
      - moveit.rviz 
- open_manipulator_tools `branch: ticTacToe`
  - action
    - kinematicsAction.action
  - scripts
    - close_gripper.py 
    - inverse_kinematics.py
    - open_gripper.py 
  - worlds
    - model.config
    - model.sdf
    - ticTacToe_camera_test_final.world
  - CMakeLists.txt
  - package.xml 
### Ismert bugok
- A rostopic pub /option std_msgs/String "print_open_manipulator_setting" nem írja ki az információkat a controlleres terminálablakba.
- A teleop_keyboard néha random lefagy, iylenkor újra kell indítani a controllert és a teleop_keyboardot is.
- Gazebóban a robot megfogója nyitott állapotban ugrál, zárt állapotban nem.
- A gravitáció kompenzálásához Dynamixel Wizardban kéne beállítani valamit, amit nem tudtunk telepíteni.
- A szimulált robotkar nem pontosan az asztallap felületén helyezkedik el, henem kicsivel fölötte.
- Mivel a szimulált bábuk kinetikai modellek, és bizonyos távolságon túl a robot már nem tudja elérni a megadott letevési magasságot, a legtávolabbi sorba helyezett bábukat kicsit beleteszi az asztalba.
- Amikor a robot a valóságban nagyobb távolságra tesz le egy bábut, a pálya egyenetlensége miatt esetenként beleütközik abba.
## Fejlesztési lehetőségek
- [ ] A módosított scriptek kiszervezése különálló csomagba.
- [ ] Ember-robot játék megvalósítása.
- [ ] Színfelismerés javítása, hogy bármilyen pozícióban meg tudja határozni, hogy a látóterében lévő tárgy milyen koordinátán van.
- [ ] A robot mozgásának folyamatosabbá tétele egy jobb, pályakövető szabályozással.
- [ ] A robot az utolsó lépés után megáll hibaüzenettel, mert nincs több bábu. Valamilyen elegánsabb megoldással ez kiküszöbölhető lenne.
