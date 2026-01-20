<!-- https://github.com/othneildrew/Best-README-Template -->
<a id="readme-top"></a>

<!-- PROJECT LOGO -->
<br />
<div align="center">
  <a href="https://github.com/Tuchila-Adi-Bogdan/utcn-OpenGL-StarwarsDemo">
    <img src="docs/Screenshot.png" alt="Logo">
  </a>
  
<h3 align="center">utcn-OpenGL-StarwarsDemo</h3>

  <p align="center">
    O simulăre 3D interactivă a unei bătălii spațiale, inspirată din universul "Star Wars". Implementată cu OpenGL.
    <br />
    Pentru materia PG (prelucrare grafica). 
    <br />
    <a href="https://github.com/Tuchila-Adi-Bogdan/utcn-OpenGL-StarwarsDemo/issues/new?labels=bug&template=bug-report---.md">Report Bug</a>
    &middot;
    <a href="https://github.com/Tuchila-Adi-Bogdan/utcn-OpenGL-StarwarsDemo/issues/new?labels=enhancement&template=feature-request---.md">Request Feature</a>
  </p>
</div>

# 1. Cuprins

1. [Cuprins](#1-cuprins)
2. [Prezentarea temei](#2-prezentarea-temei)
3. [Scenariul](#3-scenariul)
   1. [Descrierea scenei și a obiectelor](#31-descrierea-scenei-și-a-obiectelor)
   2. [Funcționalități](#32-funcționalități)
4. [Detalii de implementare](#4-detalii-de-implementare)
   1. [Funcții și algoritmi](#41-funcții-și-algoritmi)
      1. [Soluții posibile](#411-soluții-posibile)
      2. [Motivarea abordării alese](#412-motivarea-abordării-alese)
   2. [Modelul grafic](#42-modelul-grafic)
   3. [Structuri de date](#43-structuri-de-date)
   4. [Ierarhia de clase](#44-ierarhia-de-clase)
5. [Prezentarea interfeței grafice utilizator / Manual de utilizare](#5-prezentarea-interfeței-grafice-utilizator--manual-de-utilizare)
6. [Concluzii și dezvoltări ulterioare](#6-concluzii-și-dezvoltări-ulterioare)
7. [Referințe](#7-referințe)
   
<!-- ABOUT THE PROJECT -->
# 2. Prezentarea temei

Proiectul constă în realizarea unei simulări 3D interactive a unei bătălii spațiale, inspirată din universul "Star Wars". Scopul principal este demonstrarea conceptelor fundamentale de grafică computerizată utilizând biblioteca OpenGL și limbajul C++. Aplicația pune accent pe manipularea obiectelor 3D în timp real, iluminare dinamică, transformări geometrice complexe și animație procedurală. Scena surprinde confruntarea dintre Flota Imperială (incluzând Death Star II și Imperial Star Destroyers) și Alianța Rebelă (nave de tip X-Wing, A-Wing și Mon Calamari Cruisers), oferind utilizatorului control asupra camerei și asupra declanșării evenimentelor de luptă.

# 3. Scenariul
## 3.1 Descrierea scenei și a obiectelor 
Scena este plasată în spațiu adânc (Deep Space), randat prin intermediul unui Skybox cubic texturat.
1. Sursă de lumină globală: Există o sursă de lumină in spatele Death Star II.
2. The empire:
   1. Death Star II: Obiect masiv, parțial construit, poziționat în fundal.
   2. Imperial Star Destroyers (ISD): O formație de 7 nave capitale, dispuse în "V", care acționează ca platforme de tragere pentru utilizator.
3. Rebelii:
   1. Crucișătoare MC: Nave capitale care servesc drept ținte pentru Death Star.
   2. Flota de vânătoare (Fighters): Escadrile de X-Wings și A-Wings care execută manevre de atac și evaziune.
4. Elemente dinamice:
   1. Lasere verzi (Death Star și ISD).
   2. Lasere roșii (Nave rebele).
   3. Explozii animate.

## 3.2 Funcționalități
Aplicația permite:
-	Navigare liberă: Utilizatorul poate explora scena folosind o cameră de tip "Fly Camera".
-	Animație procedurală: Navele mici (X-Wings, A-Wings) nu stau statice; ele urmează o mașină de stări (State Machine) pentru a simula un atac: apropiere, viraj evaziv, fugă și regrupare.
-	Sistem de luptă interactiv:
 -	Declanșarea "Superlaser-ului" Death Star asupra crucișătoarelor Mon Calamari.
 -	Controlul turelelor de pe Star Destroyers (tastele 1-7) pentru a ataca ținte aleatorii din flota inamică.
 -	"Volley Fire" din partea rebelilor (tasta P).
 -	Manipularea timpului: Acțiunile se desfășoară doar cât timp tasta SPACE este apăsată, permițând vizualizarea detaliată a traiectoriilor laserelor și a detaliilor.

# 4. Detalii de implementare
## 4.1. Funcții și Algoritmi
Ca și regulă, folosesc funcția processMovement() , pentru toată mișcarea. În această funcție verific dacă space bar este apăsat. Dacă da, se actualizează mai multe variabile care reprezintă poziția pentru mai multe obiecte din scenă. Odată ce s-a actualizat poziția, obiectele sunt translatate cu diferite viteze, așa are loc mișcarea.
### 4.1.1 Soluții posibile
Aici am prezentat pe scurt algoritmii mai interesanți din proiect.
### 1. Algoritmul de mișcare a navelor
- Pentru navele mari, așa are loc mișcarea:
	- ImperialFleetPosition se actualizează când e ținut apăsat space bar.
	- Adaugăm idsOffsets ca să își păstreze formația
	  ```cpp
	  ImperialFleetPosition.z += capitalShipSpeed; //în if (pressedKeys[GLFW_KEY_SPACE])
	  ```
	  ```cpp
	  isdModel = glm::translate(isdModel, ImperialFleetPosition + isdOffsets[i]);
	  ```
- Pentru navele mici, s-a implementat un algoritm bazat pe Mașini de Stări Finite (FSM - Finite State Machine).
	- Fiecare navă are o stare curentă (APPROACHING, EVASIVE_TURN, EVASIVE_RUN, STABILIZE, RETURNING, RESET_TURN).
	- Tranzițiile între stări se fac pe bază de cronometru (stateTimer) sau poziție (atingerea unei linii imaginare, care e defapt coordonata pe Z a unei anumite flote).
    - La fiecare cadru, în funcție de stare, se actualizează poziția și rotația (Euler Angles).
	<img src="docs/StateDiagramPG.png" alt="Logo">
	</br>
1. Starea APPROACHING
- Logica pentru "Random Jitter" - mișcarea arată mult mai naturală
	```cpp
	float rPitch = ((rand() % 1000) / 1000.0f - 0.5f) * 2.0f;
	float rYaw = ((rand() % 1000) / 1000.0f - 0.5f) * 2.0f;
	float rRoll = ((rand() % 1000) / 1000.0f - 0.5f) * 2.0f;
	ship.rotation.x += rPitch * deltaTime * 2.5f;
	ship.rotation.y += rYaw * deltaTime * 2.5f;
	ship.rotation.z += rRoll * deltaTime * 4.0f;
	```
- Logica ca să ma asigur că mișcarea navei este în direcția "nose of the ship" - Face mișcarea mai naturală.
	```cpp
	glm::vec3 forward;
	forward.x = sin(ship.rotation.y) * cos(ship.rotation.x);
	forward.y = -sin(ship.rotation.x);
	forward.z = cos(ship.rotation.y) * cos(ship.rotation.x);
	forward = glm::normalize(forward);
	ship.position += forward * (ship.speed * 60.0f * deltaTime);
	```
- Tranziția spre următoarea stare : Verifică dacă a ajuns la navele Empire
	```cpp
	if (canDoManeuver && ship.position.z < empireLine)
		ship.state = EVASIVE_TURN;
	```
2. Starea EVASIVE_TURN : Rotire 180 cu "drift" pentru ca mișcarea să arate mai natural
	```cpp
	// Pentru notația cu unghiuri Euler, defapt rotim "yaw" cu PI (180 grade)
	ship.rotation.y += 3.14159f * deltaTime; // 1 second turn
	ship.position.z -= 10.0f * deltaTime;    // Momentum drift
	```
3. Starea EVASIVE_RUN : Aici navele fac un fel de "swarm" în jurul navelor Empire, pentru a simula un atac. La bază este doar o mișcare random scurtă, cu o viteză mai mare.
 	```cpp
   ship.position.z += (ship.speed * 80.0f) * deltaTime; // Zboară spre +Z
	// Jitter
	ship.rotation.x += sin(glfwGetTime() * 15.0f) * 3.0f * deltaTime;
	ship.rotation.z += cos(glfwGetTime() * 10.0f) * 3.0f * deltaTime;
	```
4. Starea STABILIZE : Încerc să stabilizez navele pentru zborul înapoi spre flota Rebelă, dar în practică nu funcționează așa bine. Este totuși necesară pentru că forțez navele să zboare în direcția "nose of the ship" (pentru ca mișcarea să fie naturală) - deci navele trebuie orientate puțin
   ```cpp
   float lerpSpeed = 2.0f * deltaTime;
	ship.rotation.x = ship.rotation.x * (1.0f - lerpSpeed);
	ship.rotation.z = ship.rotation.z * (1.0f - lerpSpeed);
	ship.position.z += (ship.speed * 80.0f) * deltaTime;
   ```
5. Starea RETURNING : Mișcare spre +Z, întoarcere la flota rebelă. La fel ca și APPROACHING, dar direcție inversă.
6. Starea RESET_TURN : La fel ca și EVASIVE_TURN, dar în plus resetez orientarea navelor.
### 2. Super-laser (proiectilul death star): Am folosit un model bazat pe interpolare liniară (LERP). Poziția laserului este determinată de variabila beamProgress (0.0 la 1.0).
Ca și origine este dsDishPos, ca și target folosim locația cruiserului. Când are loc mișcare, beamProgress crește.
- Pozitia curentă:
```cpp
// în renderSuperlaser
glm::vec3 currentPos = start + (target - start) * beamProgress;
```
```cpp
//Var. globală
float beamProgress = 0.0f; // 0.0 = Death Star, 1.0 = Cruiser
```
- Pentru traiectoria laserului :
```cpp
// în renderSuperlaser
glm::vec3 start = dsDishPos;
glm::vec3 target = cruiserFleetPosition;
```
- Se întămplă doar când are loc mișcare (este ținut apăsat space bar):
```cpp
// în processMovement
if (!hasHit) {
    beamProgress += 0.3f * deltaTime;
    if (beamProgress >= 1.0f) {
        beamProgress = 1.0f;
        hasHit = true;
    }
}
```
- Desenarea proiectilului
```cpp
// în renderSuperlaser
if (beamProgress > 0.01f && beamProgress < 1.0f) {
    glm::mat4 modelBeam = glm::mat4(1.0f);
    // Translație la poziția curentă
    modelBeam = glm::translate(modelBeam, currentPos);
```
-Iluminare : Laserul este o sursă de lumină dinamică (Point Light) care se mișcă odată cu geometria.
```cpp
// în renderSuperlaser
glUniform3fv(glGetUniformLocation(shader.shaderProgram, "pointLightPos"), 1, glm::value_ptr(currentPos));
glm::vec3 greenColor = glm::vec3(0.0f, 5.0f, 0.0f);
```
-Mai au loc câteva rotații în renderSuperlaser pentru orientarea corectă a modelului.
### 3. Laserele mici - trase de navele empire (Imperial Star Destroyers) și interceptoarele rebele (x-wings și a-wings): De asemenea un model bazat pe interpolare liniară.
- Sunt controlate de funcțiile fireRebelVolley() și fireISDLaser() - unde stabilim startpos, progress, target - fiecare laser este un struct
- De asemenea am funcțiile renderRebelLasers() și renderISDLaser(), foarte similare, diferă doar poziția de start, target-ul, culoarea și mărimea acestora.
- Target-ul este ales mereu random
```cpp
// în renderRebelLasers
for (const auto& laser : rebelLasers) // Pentru fiecare laser
```  
```cpp
// în renderRebelLasers
// Calculate Target Position: Fleet Pos + ISD Offset
// offset (0, 10, 50) to hit the hull, not the center pivot
glm::vec3 target = ImperialFleetPosition + isdOffsets[laser.targetISDIndex] + glm::vec3(0.0f, 10.0f, 50.0f);
glm::vec3 currentPos = start + (target - start) * laser.progress; // Similar cu logica pentru laserul Death Star
```
```cpp
// în renderRebelLasers
// RED LIGHT
glUniform3fv(glGetUniformLocation(shader.shaderProgram, "pointLightPos"), 1, glm::value_ptr(currentPos));
glm::vec3 redColor = glm::vec3(5.0f, 0.0f, 0.0f); // Bright Red
glUniform3fv(glGetUniformLocation(shader.shaderProgram, "pointLightColor"), 1, glm::value_ptr(redColor));
```
```cpp
// în renderRebelLasers
// RENDER BEAM
glm::mat4 modelBeam = glm::mat4(1.0f);
modelBeam = glm::translate(modelBeam, currentPos);
```
```cpp
// în renderRebelLasers
// Orientation
if (glm::length(target - start) > 0.1f) {
    glm::mat4 rotation = glm::inverse(glm::lookAt(currentPos, start, glm::vec3(0, 1, 0)));
    rotation[3] = glm::vec4(0, 0, 0, 1);
    modelBeam = modelBeam * rotation;
}
```  
### 4.1.2 Motivarea abordării alese
 - Sistemul de mișcare este bazat pe FSM (Finite State Machine).
 	- Motiv: Această arhitectură oferă scalabilitate și modularitate. Permite gestionarea unui număr mare de entități simultan cu un cost computațional redus, generând un comportament organic care ar fi fost dificil și ineficient de implementat prin animație statică (keyframe).
 - Modelul laserelor este bazat pe Interpolare Liniară (LERP): Traiectoria proiectilelor este calculată folosind un factor de progres (0.0 - 1.0), actualizat în funcție de deltaTime
 	- Motiv: Această metodă maximizează eficiența. Elimină necesitatea unor calcule fizice complexe (precum integrarea numerică a accelerației) pentru obiecte care nu își schimbă traiectoria, garantând în același timp precizia impactului și o sincronizare perfectă între logică și randare.

## 4.2 Modelul grafic
Aplicația folosește OpenGL 3.3 Core Profile.
-	Pipeline de randare: Vertex Shader (transformă coordonatele din spațiul local -> lume -> vizualizare -> clip) și Fragment Shader (calculează culoarea finală).
-	Iluminare: Modelul Blinn-Phong.
    - Lumină direcțională (de la steaua cea mai apropriata).
    - Lumini punctiforme (Point Lights) dinamice atașate de vârful laserelor și de centrul exploziilor. Acestea apar și dispar în funcție de logica simulării.
- Texturare: Modelele 3D au coordonate UV și texturi difuze mapate.
### Fragment shader:
În fragment shader calculez ambientTotal, diffuseTotal, specularTotal.
 - ambientTotal = ambientDir + ambientPoint
 - diffuseTotal = diffuseDir + diffusePoint
 - specularTotal = specularDir + specularPoint
```cpp
//pentru texturi transparente
    if(texColor.x == 0)
        discard;
```
```cpp
// Lumini directionale
    computeDirLight(fPosEye, normalEye, viewDir, texDiffuse, texSpecular);
```
```cpp
// Lumini punctiforme (de la lasere)
    // Calculam lumina punctiforma doar daca are culoare (optimizare)
    if (length(pointLightColor) > 0.0) {
        computePointLight(fPosEye, normalEye, viewDir, texDiffuse, texSpecular);
    }
```
```cpp
// Combinare finală
vec3 finalColor = min(ambientTotal + diffuseTotal + specularTotal, 1.0f);
```
## 4.3 Structuri de date
Pentru gestionarea eficientă a obiectelor, s-au folosit structuri C++ și containere STL:
- struct Ship: Stochează poziția (glm::vec3), rotația, viteza și starea curentă a fiecărei nave de luptă.
```cpp
struct Ship {
    glm::vec3 position; // World Position
    glm::vec3 rotation; // Euler Angles: x=Pitch, y=Yaw, z=Roll
    float speed;        // Individual speed
    ShipState state = APPROACHING;
    float stateTimer = 0.0f;
};
```
- struct LaserShot / RebelLaserShot: Reține starea activă, progresul (0.0 - 1.0), poziția de start și un pointer către nava țintă (Ship* targetShip). Aceasta permite laserului să urmărească ținta chiar dacă nava se mișcă.
```cpp
struct LaserShot {
    bool active = false;
    float progress = 0.0f;      // 0.0 = Start, 1.0 = Target
    glm::vec3 startPos;         // Origin
    Ship* targetShip = nullptr; // Target
};
```
```cpp
struct RebelLaserShot {
    bool active = false;
    float progress = 0.0f;
    glm::vec3 startPos;
    int targetISDIndex; // Index 0-7 of the ISD array
};
```
- std::vector<Ship>: Gestionarea dinamică a flotelor (XWings, AWings).
- std::vector<RebelLaserShot>: Permite un număr variabil de proiectile simultane

## 4.4 Ierarhia de clase
Deși logica jocului este procedurală (în main.cpp), s-au folosit clase wrapper pentru abstractizarea OpenGL:
-	gps::Window: Inițializarea ferestrei GLFW și contextului OpenGL.
-	gps::Camera: Gestionează matricea de vizualizare (View Matrix), mișcarea (WASD) și rotația din mouse (Pitch/Yaw).
-	gps::Model3D: Încărcarea modelelor .obj folosind tiny_obj_loader și desenarea lor.
-	gps::Shader: Încărcarea, compilarea și link-area shaderelor GLSL.
-	gps::SkyBox: Clasa dedicată randării cubului de fundal.

# 5. Prezentarea interfeței grafice utilizator / Manual de utilizare

Interfața este vizuală, controlul realizându-se prin tastatură și mouse. Nu există meniuri 2D suprapuse (HUD), imersiunea fiind prioritară.
Controale:
-	W, A, S, D: Deplasare cameră (Înainte, Stânga, Înapoi, Dreapta).
- Mouse: Orientare privire.
-	Q / E: Rotația scenei pentru a observa umbrele/modelele din alte unghiuri.
-	SPACE (Apăsat):
    -	Activează curgerea timpului (navele se mișcă).
    -	Încarcă și trage cu Superlaser-ul Death Star.
- Tastele 1 - 7 (În timp ce ții SPACE): Fiecare tastă comandă un Star Destroyer specific să tragă un laser verde spre o navă rebelă aleatorie.
-	Tasta P (În timp ce ții SPACE): Comandă "Rebel Volley": Toate navele mici rebele trag simultan lasere roșii către distrugătoarele imperiale.
-	Tasta K: Resetarea completă a scenei (poziții nave, lasere, explozii).

# 6. Concluzii și dezvoltări ulterioare
Concluzii: Proiectul a reușit simularea unei scene complexe de luptă spațială, integrând cu succes concepte de transformări matriceale, iluminare dinamică (lasere care emit lumină) și inteligență artificială rudimentară pentru mișcarea flotelor. Utilizarea structurilor de date dinamice (std::vector) a permis scalarea numărului de nave fără a modifica logica de bază.

## Dezvoltări ulterioare:
Pentru a crește realismul aplicației, se pot implementa:
1.	Shadow Mapping: Implementarea umbrelor dinamice (navele să lase umbră una pe alta), folosind randarea în două treceri (Depth Map pass + Render pass).
2.	Coliziuni Avansate: Înlocuirea verificării simple (progress >= 1.0) cu volume de coliziune (AABB sau sfere) pentru a permite distrugerea navelor la impact.
3.	Sistem de Particule: Pentru a genera explozii volumetrice și urme ale motoarelor (engine trails).
4.	Audio: Adăugarea efectelor sonore spațiale 3D folosind o bibliotecă precum OpenAL.

# 7. Referințe:
1.	LearnOpenGL: https://learnopengl.com/ - Resursă principală pentru teorie (iluminare, transformări, cameră).
2.	Laboratoare PG - Matematica vectorilor și matricelor.
3.	Star Wars 3D Models: sketchfab (https://sketchfab.com) - Sursa modelelor (Death Star, X-Wing, ISD).
5.  Skybox: http://alexcpeterson.com/spacescape/ - un program excelent pentru skybox-uri spatiale.
Project Link: [https://github.com/Tuchila-Adi-Bogdan/utcn-Investor_Centre](https://github.com/Tuchila-Adi-Bogdan/utcn-Investor_Centre)

<p align="right">(<a href="#readme-top">back to top</a>)</p>
