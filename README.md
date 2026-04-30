# ESP32 WiFi Web Server Project

## Aperçu du projet

Projet IoT utilisant une carte ESP32 et un serveur Ubuntu Apache/PHP permettant :

- l’envoi de données en WiFi,
- le stockage des données dans des fichiers,
- l’affichage des valeurs sur une page web,
- le contrôle d’une LED et d’un buzzer depuis un navigateur.

---

# Structure du projet

```txt
/var/www/html/btsciel
│
├── data.php
├── index.php
├── valeur.txt
└── valeur.csv
```

---

# data.php

Fichier PHP permettant :
- de recevoir les données envoyées par l’ESP32,
- d’enregistrer la dernière valeur dans `valeur.txt`,
- d’ajouter un historique dans `valeur.csv`.

```php
<?php

date_default_timezone_set("Pacific/Noumea");

if (isset($_POST['valeur'])) {

    $valeur = $_POST['valeur'];

    file_put_contents("valeur.txt", $valeur);

    $date = date("Y-m-d");
    $heure = date("H:i:s");

    $fichier = fopen("valeur.csv", "a");

    fputcsv($fichier, [$date, $heure, $valeur]);

    fclose($fichier);

    echo "OK";
}

?>
```

---

# index.php

Page web permettant :
- d’afficher la dernière valeur reçue,
- de contrôler la LED,
- de contrôler le buzzer.

```php
<?php

$valeur = "Aucune donnée";

if (file_exists("valeur.txt")) {
    $valeur = file_get_contents("valeur.txt");
}

?>

<!DOCTYPE html>
<html lang="fr">

<head>

<meta charset="UTF-8">
<meta http-equiv="refresh" content="2">

<title>ESP32 Web Server</title>

<style>

body {
    font-family: Arial;
    text-align: center;
    margin-top: 50px;
}

.box {
    display: inline-block;
    padding: 20px 40px;
    border: 2px solid black;
    border-radius: 10px;
    font-size: 30px;
}

button {
    padding: 10px 20px;
    margin: 10px;
    font-size: 18px;
}

</style>

</head>

<body>

<h1>Valeur ESP32</h1>

<div class="box">
<?php echo htmlspecialchars($valeur); ?>
</div>

<br><br>

<h1>Commande ESP32</h1>

<button onclick="fetch('http://IP_ESP32/led')">
LED
</button>

<button onclick="fetch('http://IP_ESP32/son')">
SON
</button>

</body>

</html>
```

---

# Code ESP32 – Envoi de données

```cpp
#include <WiFi.h>
#include <HTTPClient.h>

const char* ssid = "TPSN035";
const char* password = "BTSSN2022";

const char* serverName = "http://IP_UBUNTU/btsciel/data.php";

void setup() {

  Serial.begin(115200);

  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
  }

  Serial.println(WiFi.localIP());
}

void loop() {

  if (WiFi.status() == WL_CONNECTED) {

    int valeur = random(0,100);

    HTTPClient http;

    http.begin(serverName);

    http.addHeader("Content-Type", "application/x-www-form-urlencoded");

    String donnee = "valeur=" + String(valeur);

    http.POST(donnee);

    http.end();
  }

  delay(5000);
}
```

---

# ESP32 – Serveur Web LED/Buzzer

```cpp
#include <WiFi.h>
#include <WebServer.h>

const char* ssid = "TPSN035";
const char* password = "BTSSN2022";

WebServer server(80);

int led = 23;
int buzzer = 26;

void gestionLED() {

  digitalWrite(led, HIGH);
  delay(1000);
  digitalWrite(led, LOW);

  server.send(200, "text/plain", "LED OK");
}

void gestionSON() {

  ledcWriteTone(buzzer, 1000);
  delay(500);

  ledcWriteTone(buzzer, 0);

  server.send(200, "text/plain", "SON OK");
}

void setup() {

  pinMode(led, OUTPUT);

  ledcAttach(buzzer, 2000, 8);

  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
  }

  Serial.println(WiFi.localIP());

  server.on("/led", gestionLED);
  server.on("/son", gestionSON);

  server.begin();
}

void loop() {

  server.handleClient();
}
```

---

# Fonctionnement du projet

## Partie 1 – Envoi de données

1. L’ESP32 se connecte au WiFi.
2. Une valeur est générée.
3. La donnée est envoyée au serveur Ubuntu avec HTTP POST.
4. `data.php` reçoit la donnée.
5. Les fichiers sont mis à jour.
6. `index.php` affiche la valeur.

---

## Partie 2 – Contrôle à distance

1. L’utilisateur clique sur un bouton.
2. Le navigateur utilise `fetch()`.
3. Une requête HTTP est envoyée à l’ESP32.
4. L’ESP32 active :
   - une LED,
   - un buzzer.

---

# Technologies utilisées

- ESP32
- Arduino IDE
- WiFi
- HTTP
- PHP
- Apache2
- Ubuntu Linux
- HTML / CSS
- JavaScript
- AJAX Fetch

---

# Matériel utilisé

- ESP32
- Breadboard
- LED
- Résistance
- Grove Buzzer
- Câbles Dupont

---

# Résultat

Projet IoT complet permettant :
- l’envoi de données,
- le stockage sur serveur,
- l’affichage dynamique,
- le contrôle d’actionneurs via navigateur web.
