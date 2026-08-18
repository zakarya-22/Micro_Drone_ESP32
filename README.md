# 🚁 DIY ESP32-C3 Micro-Drone

*(Insérer ici une photo de ton drone monté)*
`![Photo du drone monté](images/nom_de_la_photo.jpg)`

## 1. Introduction et Objectif du Projet
L'objectif de ce projet est la conception de A à Z d'un mini-drone quadricoptère basé sur le microcontrôleur ESP32-C3. Ce projet multidisciplinaire couvre le prototypage mécanique, la conception électronique et le développement logiciel embarqué. Il implique le contrôle PWM de 4 moteurs à courant continu via des MOSFETs, la lecture de capteurs d'attitude (IMU) via I2C, et la gestion d'une alimentation par batterie LiPo.

## 2. 🛠️ Architecture Matérielle (Hardware)

*(Insérer ici une photo de ton PCB nu ou de l'impression 3D)*
`![Photo PCB et Châssis](images/nom_de_la_photo.jpg)`

* **Microcontrôleur :** ESP32-C3 Super Mini (Wi-Fi/Bluetooth, monocœur RISC-V).
* **Actionneurs :** 4 x Moteurs à courant continu (Coreless) avec hélices profilées.
* **Électronique de puissance :** Transistors MOSFETs pour la commutation des moteurs.
* **Capteur :** IMU (Inertial Measurement Unit) communicant en I2C pour la lecture de l'assiette du drone.
* **Alimentation :** Batterie LiPo Tattu 300mAh (avec alimentation de laboratoire utilisée pour les phases de test).
* **Mécanique :** Châssis conçu sur-mesure via **FreeCAD** et imprimé en 3D.
* **Circuit imprimé :** Routage réalisé sur **KiCad** et usinage d'un PCB personnalisé à l'aide d'une fraiseuse LPKF.

## 3. 💻 Architecture Logicielle (Software)
* **Environnement de développement :** Arduino IDE.
* **Modularité :** Le code est actuellement développé en modules séparés pour isoler et valider chaque sous-système de manière unitaire :
  * Un module dédié à l'acquisition des données de l'IMU via le bus I2C.
  * Un module dédié à la génération des signaux PWM (via `ledcWrite`) pour le contrôle de la vitesse des moteurs.

## 4. 🐛 Historique de Débogage et Résolution des Problèmes

Au cours du développement, plusieurs défis techniques ont été rencontrés et résolus méthodiquement :

### Défi 1 : Blocage de l'ESP32-C3 au démarrage (Erreur ESP-ROM)
* **Symptôme :** Après le téléversement du code, la carte refusait de lancer le programme et affichait uniquement le message `ESP-ROM` sur le port série.
* **Diagnostic :** La broche GPIO 9 (broche de "Strapping" définissant le mode de démarrage) mesurait une tension flottante de 0.2V au lieu d'un état haut (3.3V), forçant l'entrée en mode flash (*Download Mode*).
* **Solution :** Un reset électrique complet (décharge des condensateurs en court-circuitant 3V3 et GND hors tension) a purgé les charges résiduelles, validant la nécessité d'états électriques stables sur les broches de strapping.

### Défi 2 : Redémarrages intempestifs en charge (Brownout detector was triggered)
* **Symptôme :** Dès l'activation des moteurs, l'ESP32 plantait avec l'erreur `Brownout`.
* **Diagnostic :** L'appel de courant brutal des moteurs faisait chuter la tension d'alimentation sous le seuil critique (environ 2.8V).
* **Solution temporaire :** Passage sur une alimentation de laboratoire (3.65V - 4A) pour garantir la puissance nécessaire.

### Défi 3 : Problème de poussée inversée (Le drone "frappe" le sol)
* **Symptôme :** Les moteurs tournaient, mais le flux d'air plaquait le drone au sol au lieu de le soulever.
* **Diagnostic :** Sens de rotation incorrect et profil aérodynamique des hélices non respecté. Les MOSFETs unidirectionnels ne permettent pas de changer le sens logiciellement.
* **Solution :** Retournement physique des hélices (face bombée vers le haut) et inversion du câblage matériel des moteurs sur le PCB.

### Défi 4 : Comportement aléatoire des moteurs au démarrage
* **Symptôme :** Activation erratique des moteurs lors de la mise sous tension.
* **Diagnostic :** Les grilles (Gates) des MOSFETs captaient des signaux parasites à cause de la haute impédance (état flottant) des GPIOs pendant la séquence de boot de l'ESP32.
* **Solution :** Ajout de résistances de pull-down (10kΩ) entre les grilles et la masse (GND) pour forcer l'état bas tant que le PWM n'est pas envoyé logiciellement.

### Défi 5 : Capteur IMU non détecté sur le bus I2C
* **Symptôme :** Échec du scanner I2C, chute de la tension 3.3V du capteur.
* **Diagnostic :** Court-circuit ou inversion de polarité (VCC / GND) mettant l'alimentation en sécurité et bloquant le bus.
* **Action :** Isolement immédiat du circuit et inspection du câblage pour prévenir la destruction des composants.

*(Insérer ici des captures d'écran du moniteur série ou d'autres photos de debug)*
`![Débogage](images/nom_de_la_photo.jpg)`

## 5. 🚧 Problèmes Actuels et Améliorations Matérielles

Actuellement, les sous-systèmes fonctionnent parfaitement de manière isolée. Cependant, lors de la combinaison des codes, le microcontrôleur n'arrive pas à gérer les moteurs de façon stable (perte de puissance ou redémarrage). Voici les deux pistes de résolutions matérielles prioritaires en cours :

1. **Le seuil de commutation des MOSFETs (Logic-Level) :** L'ESP32-C3 fonctionne en logique 3.3V. Si les MOSFETs utilisés nécessitent une tension de grille de 5V ou plus pour saturer complètement (s'ouvrir à 100%), ils fonctionnent en régime ohmique (comme une résistance). Ils brident alors le courant destiné aux moteurs et dissipent de la chaleur. L'amélioration consistera à s'assurer d'utiliser des MOSFETs "Logic-Level" (ex: SI2302) ayant une très faible tension de seuil de grille ($V_{GS(th)}$).
2. **Filtrage et découplage de puissance (Brownout matériel) :** Pour absorber le gigantesque pic de courant généré au démarrage par 4 moteurs *coreless* et protéger l'ESP32 des chutes de tension, l'ajout d'un condensateur de forte valeur (ex: 470µF) au plus près de la ligne d'alimentation générale, combiné à des condensateurs de découplage proches de l'ESP32, est nécessaire.

## 6. 🚀 État Actuel et Prochaines Étapes
- [x] Modélisation du châssis sur FreeCAD.
- [x] Routage et usinage du PCB sur KiCad/LPKF.
- [x] Validation unitaire du code de l'IMU.
- [x] Validation unitaire du code PWM pour les moteurs.
- [ ] Résolution du problème d'alimentation/contrôle moteur avec l'ESP32 (Code combiné).
- [ ] Fusion propre du code dans un `flight_controller` unifié.
- [ ] Remplacement de l'alimentation de labo par la batterie Tattu 300mAh.
- [ ] Implémentation et réglage de la boucle d'asservissement PID.
