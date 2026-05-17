# 📱 Proiect PIP — Smart Device Controller

> Aplicație Java client-server cu interfață grafică făcută în JavaFX și integrare AI (TinyLlama via Ollama) pentru controlul funcțiilor unui dispozitiv simulat: Wi-Fi, Bluetooth și Lanternă.

---

## 👥 Autori

| Nume | Rol |
|---|---|
| Alupei Victor | Developer |
| Herghelegiu Cristian-Gabriel | Developer |
| Mocanu Marius-Gabriel | Team-Leader |
| Partac Alexis-Matei | Developer |


---

## 📋 Descriere

Aplicația noastră este o aplicație care simulează controlul funcțiilor unui telefon mobil prin comenzi text procesate de un model AI local.

Utilizatorul poate trimite comenzi precum `open wifi` sau `close bluetooth` dintr-o interfață grafică care imită un smartphone. Comenzile sunt procesate de un server Java care le transmite modelului **TinyLlama** rulat local prin **Ollama**. Răspunsul AI este interpretat și reflectat vizual în interfață (iconiță + culoare).

### Flux de date

```
[GUIApp / App]  ──TCP:5000──►  [ServerApp]  ──HTTP──►  [Ollama / TinyLlama]
      ▲                              │
      └──────── răspuns AI ──────────┘
```

---

## 🧩 Structura proiectului

```
Proiect_PIP/
├── src/
│   └── main/java/ro/tuiasi/ac/Proiect_PIP/
│       ├── App.java          # Client consolă TCP
│       ├── GUIApp.java       # Client grafic JavaFX
│       └── ServerApp.java    # Server TCP + integrare Ollama
│   └── test/java/ro/tuiasi/ac/Proiect_PIP/
│       ├── AppTest.java
│       ├── GUIAppTest.java
│       └── ServerAppTest.java
│   └── target/classes/META-INF
│       └── MANIFEST.MF
├── resurse/                  # Imagini pentru iconițe (wi-fi, bluetooth, lanternă)
└── pom.xml
README.md
```

---

## ⚙️ Cerințe de sistem

| Componentă | Versiune minimă |
|---|---|
| Java (JDK) | 17 |
| Maven | 3.6+ |
| JavaFX | 20 |
| Ollama | orice versiune recentă |
| Model AI | `tinyllama` |
| OS | Windows (clasificatorul Maven setat pe `win`) |

> **Notă pentru Linux/macOS:** schimbați `<javafx.platform>win</javafx.platform>` în `linux` sau `mac` în `pom.xml`.

---

## 🚀 Instalare și rulare

### 1. Clonați repository-ul

```bash
git clone https://github.com/<user>/Proiect_PIP.git
cd Proiect_PIP
```

### 2. Instalați și porniți Ollama

Descărcați Ollama de la [https://ollama.com](https://ollama.com), apoi:

```bash
ollama pull tinyllama
ollama serve
```

Ollama va asculta implicit pe `http://localhost:11434`.

### 3. Compilați proiectul

```bash
mvn clean package
```

### 4. Porniți serverul

```bash
mvn exec:java -Dexec.mainClass="ro.tuiasi.ac.Proiect_PIP.ServerApp"
```

Serverul va asculta pe portul **5000**.

### 5. Porniți clientul grafic (JavaFX)

```bash
mvn javafx:run
```

Sau porniți clientul consolă:

```bash
mvn exec:java -Dexec.mainClass="ro.tuiasi.ac.Proiect_PIP.App"
```

---

## 🎮 Comenzi acceptate

| Comandă | Răspuns AI | Efect vizual |
|---|---|---|
| `open wifi` | `wi-fi_on` | Icoana Wi-Fi devine verde |
| `close wifi` | `wi-fi_off` | Icoana Wi-Fi devine albă |
| `open bluetooth` | `bluetooth_on` | Icoana Bluetooth devine verde |
| `close bluetooth` | `bluetooth_off` | Icoana Bluetooth devine albă |
| `status` | `the sistem is operating.` | Mesaj în consolă |
| `exit` | — | Închide conexiunea |
| orice altceva | `error: Order not accepted.` | — |

---

## 🏗️ Arhitectură

### `App.java` — Client consolă

Client TCP simplu. Citește comenzi de la tastatură, le trimite serverului pe portul 5000 și afișează răspunsurile în consolă.

### `GUIApp.java` — Client grafic JavaFX

Interfață grafică care simulează un smartphone. Trimite comenzile prin socket TCP și actualizează automat iconițele și culorile în funcție de răspunsul primit de la server. Rulează listener-ul de rețea pe un thread daemon separat față de JavaFX Application Thread.

### `ServerApp.java` — Server TCP + AI

Acceptă o conexiune TCP pe portul 5000. Pentru fiecare comandă primită, construiește un prompt RAW pentru TinyLlama (cu exemple few-shot) și îl trimite prin HTTP POST la API-ul Ollama. Extrage câmpul `response` din JSON-ul returnat și îl transmite clientului.

**Parametri Ollama:**
- `temperature: 0.0` — răspunsuri deterministe
- `num_predict: 15` — răspuns scurt și precis
- `stream: false` — răspuns complet într-un singur bloc

---

## 🧪 Teste

Testele sunt scrise cu **JUnit 3** și acoperă:

- **`AppTest`** — comunicare client-server cu mock server; host necunoscut; închidere corectă a socket-ului
- **`GUIAppTest`** — inițializarea obiectului; eșec conexiune; logica de construire a căii imaginii
- **`ServerAppTest`** — conexiune eșuată; adresă invalidă; port negativ

Rulați testele cu:

```bash
mvn test
```

---

## 📦 Dependențe principale (pom.xml)

| Librărie | Versiune | Scop |
|---|---|---|
| `junit:junit` | 3.8.1 | Testare |
| `javafx-base` | 20 | JavaFX core |
| `javafx-controls` | 20 | Componente UI |
| `javafx-fxml` | 20 | FXML support |

---

## ⚠️ Limitări cunoscute

- Serverul acceptă **o singură conexiune** la un moment dat (nu suportă multi-client).
- Modelul TinyLlama poate produce ocazional răspunsuri neașteptate chiar și cu `temperature: 0.0`; promptul few-shot minimizează acest risc.
- Clasificatorul JavaFX este setat pe `win` în `pom.xml` — necesită modificare pentru alte sisteme de operare.
