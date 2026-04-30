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
- d’afficher la dernière valeur reçue,
- de contrôler la LED et le buzzer depuis la page web.

```php
<?php
error_reporting(E_ALL);
ini_set('display_errors', 1);

$file = __DIR__ . "/valeur.txt";

// Si c'est une requête POST, on enregistre la valeur
if ($_SERVER['REQUEST_METHOD'] === 'POST') {

    if (isset($_POST['valeur'])) {

        $valeur = $_POST['valeur'];

        $result = file_put_contents($file, $valeur);

        if ($result === false) {

            echo "Erreur écriture fichier";

        } else {

            echo "Valeur enregistrée : " . $valeur;
        }

    } else {

        echo "POST reçu mais pas de valeur";
    }

    exit;
}

// Si c'est une requête GET, on lit et affiche la dernière valeur
if (file_exists($file)) {

    $valeurAffiche = file_get_contents($file);

} else {

    $valeurAffiche = "Aucune valeur reçue pour l'instant.";
}
?>

<!DOCTYPE html>
<html lang="fr">

<head>

<meta charset="UTF-8">

<title>Valeur Arduino</title>

<meta http-equiv="refresh" content="5">

<style>

body {
    font-family: Arial, sans-serif;
    text-align: center;
    margin-top: 50px;
}

.valeur {
    font-size: 2em;
    color: #007BFF;
}

button {
    padding: 10px 20px;
    font-size: 18px;
    margin: 10px;
    cursor: pointer;
}

</style>

</head>

<body>

<h1>Dernière valeur reçue de l'Arduino</h1>

<div class="valeur">
<?php echo htmlspecialchars($valeurAffiche); ?>
</div>

<p>
La page se rafraîchit automatiquement toutes les 5 secondes.
</p>

<button onclick="fetch('http://192.168.100.91/led')">
Allumer Led
</button>

<button onclick="fetch('http://192.168.100.91/son')">
Allumer Buzzer
</button>

</body>
</html>
```
# index.php

Page web permettant :
- d’afficher la dernière valeur reçue,
- de contrôler la LED,
- de contrôler le buzzer depuis l’ESP32.

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
<title>Donnée ESP32</title>

<style>

body {
font-family: Arial, sans-serif;
text-align: center;
margin-top: 50px;
}

.box {
display: inline-block;
padding: 20px 40px;
border: 2px solid #333;
border-radius: 12px;
font-size: 28px;
background-color: #f2f2f2;
}

button {
padding: 10px 20px;
font-size: 18px;
margin: 10px;
cursor: pointer;
}

</style>

</head>

<body>

<h1>Valeur reçue depuis l'ESP32</h1>

<div class="box">
<?php echo htmlspecialchars($valeur); ?>
</div>

<PARTIE MESURE A LAISSER>

<h2>Commande ESP32</h2>

<h3>Commande LED</h3>

<button onclick="fetch('http://192.168.100.109/led')">
Allumer
</button>

<h3>Commande son</h3>

<button onclick="fetch('http://192.168.100.109/son')">
Allumer
</button>

</body>
</html>
```

# ESP32 – Serveur Web LED/Buzzer

#include <WiFi.h>
#include <WebServer.h>
#include <HTTPClient.h>
const char* ssid = "TPSN035";    // à modifier
const char* password = "BTSSN2022"; // à modifier aussi
const char* serverName = "http://192.168.100.34/btsciel/TP07_2/data.php";
int myrandom = 0 ;

WebServer server(80);

int sortieled = 23;
int buzzer = 26;
int frequence = 2000;
void handleLED();
void handleSON();

void gestionLED() {
  Serial.println("reception info client led");
  digitalWrite(sortieled, HIGH);
  delay(3000);
  digitalWrite(sortieled, LOW);
  delay(3000);
  
  
  server.send(200, "text/plain", "LED test");
}

void gestionSON() {
  Serial.println("reception info client son");
  ledcWriteTone(buzzer, 1000);
  delay(500);
  ledcWriteTone(buzzer, 500); 
  delay(500);
  ledcWriteTone(buzzer, 1000); 
  delay(500);
  ledcWriteTone(buzzer, 500); 
  delay(500);
  ledcWriteTone(buzzer, 0); 

  Serial.println("reception info client son");
  server.send(200, "text/plain", "Son test");
}

void setup() {

  Serial.begin(115200);
  pinMode(sortieled, OUTPUT);
  ledcAttach(buzzer, frequence,8);
  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  Serial.println(WiFi.localIP());

  server.on("/led", gestionLED);
  server.on("/son", gestionSON);

  server.begin();
}

void loop() {
  server.handleClient();
  if (WiFi.status() == WL_CONNECTED) {
    int valeur = random(0,100);
    Serial.print("Valeur envoyer:");
    Serial.print(valeur);
    HTTPClient http;
    http.begin(serverName);
    http.addHeader("Content-Type", "application/x-www-form-urlencoded");

    String data = "valeur=" + String(valeur);
    int httpReponse = http.POST(data);
    Serial.print("HTTP Reponse code: ");
    Serial.println(httpReponse);

    http.end();
  }

  delay(2000);
}
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
