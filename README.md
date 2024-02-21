# Mobile Learning API Tool

Willkommen zum Mobile Learning App API Tool!

Dieses Repository beinhaltet ein Skript, mit dem Sie **automatisch** Ihre Kurse der Mobile Learning App in Form von JSON-Dateien **hinzufügen und ändern** können. 

Die Datei `courses.json` enthält zur Orientierung eine beispielhafte Struktur, die einfach ersetzt werden kann.

Sie können dieses Repository forken, um den Versionsverlauf Ihrer Kurse zu verwalten. 


## Vorgehen

### 1. Setup

Das Ausführen des Skripts erfordert eine installierte Javascript Laufzeitumgebung z.B. [Node](https://nodejs.org/en)

1. `npm install` 

2. Erstellen Sie eine `.env` Datei im Stammverzeichnis des Projekts und fügen Sie die folgenden Umgebungsvariablen hinzu (siehe .env.example):
```env
BACKEND_URL=<URL_des_Backends (z. B. http://localhost:8080 oder http://admin:pass@loco.in.htwg-konstanz.de)>
HTWG_USERNAME=<HTWG_Benutzername>
HTWG_PASSWORD=<HTWG_Passwort>
```
3. Kurse inkl. Feedback & Quizzes in `courses.json` anlegen

### 2. Kurse in `courses.json` erstellen

Nun können die gewünschten Inhalte (Kurse, Feedbacks, Quizzes) in der `courses.json` Datei angelegt werden.
Modifizieren Sie hierzu die Beispieldaten in der `courses.json` Datei.
Wenn Sie VSCode verwenden, wird ihnen der Editor beim Bearbeiten der Datei mit IntelliSense helfen.

> ⚠ **Wichtig:**
> Die `key`-Felder der Kurse, Feedbacks, Quizzes und Questions müssen eindeutig sein!
> Wenn Sie ein Objekt ändern möchten, dürfen Sie den `key` nicht ändern. Durch den Key erkennt der Server, dass es sich um ein bestehendes Objekt handelt, das geändert werden soll.

> ℹ **Hinweis Moodle:**
> Wenn Sie einen Kurs erstellen müssen Sie ihm einen Moodle-Kurs zuweisen, um Studenten darin einzuschreiben.
> Dafür brauchen Sie die ID des Moodle-Kurses. Diese finden Sie in der URL des Moodle-Kurses
> (z. B. `https://moodle.htwg-konstanz.de/moodle/course/view.php?id=940` → Die ID ist `940`)
> Diese ID wird als `moodleCourseId` in der `courses.json` Datei benötigt (`... "moodleCourseId": "940" ...`)

#### Grobe Struktur der `courses.json` Datei
 
- Kurs 1
  - Feedback-Formular 1
    - Frage 1
      - Antwortmöglichkeit 1 
      - Antwortmöglichkeit 2
    - Frage 2
      - ...
  - Feedback-Formular 2
    - ...
  - Quiz-Formular 1
    - Frage 1
      - Antwortmöglichkeit 1
- Kurs 2
  - ...

### 4. Skript Ausführen

Die Kurse können jetzt durch das Ausführen eines Scripts mit dem Server synchronisiert werden:
`node sync-courses.js`

## Detailliertere Informationen

### Kurse sind JSON-Objekte bestehend aus: 

- `key` **(required)**: Eindeutiger Schlüssel für den Kurs. 
- `name` **(required)**: Name des Kurses.
- `description` **(required)**: Beschreibung des Kurses.
- `moodleCourseId` **(required)**: ID des Moodle-Kurses, in dem der Kurs erstellt werden soll.
- `feedbackForms`: Array aus Feedback-Forumular Objekten, die mit dem Kurs verbunden sind.
- `quizForms`: Array aus Quiz-Formular Objekten, die mit dem Kurs verbunden sind.

### Feedback-Formulare sind JSON-Objekte bestehend aus:

- `key` **(required)**: Eindeutiger Schlüssel für das Feedback-Formular.
- `name` **(required)**: Name des Feedback-Formulars.
- `description` **(required)**: Beschreibung des Feedback-Formulars.
- `questions` **(required)**: Array aus Feedback-Frage Objekten, die im Feedback-Formular gestellt werden.

### Feedback Fragen sind JSON-Objekte bestehend aus:

- `key` **(required)**: Eindeutiger Schlüssel für die Frage.
- `name` **(required)**: Name der Frage.
- `description` **(required)**: Beschreibung der Frage.
- `type` **(required)**: Typ der Frage.
  - Optionen für `type`: `SLIDER`, `SINGLE_CHOICE`, `FULLTEXT`, `YES_NO`, `STARS`
- `options`: Array aus Strings mit Antwortmöglichkeiten, die für `SINGLE_CHOICE` Fragen zur Verfügung stehen.


### Quiz-Formulare sind JSON-Objekte bestehend aus:

- `key` **(required)**: Eindeutiger Schlüssel für das Quiz-Formular.
- `name` **(required)**: Name des Quiz-Formulars.
- `description` **(required)**: Beschreibung des Quiz-Formulars.
- `questions` **(required)**: Array aus Quiz Frage Objekten, die im Quiz-Formular gestellt werden.

### Quiz Fragen sind JSON-Objekte bestehend aus:

- `name` **(required)**: Name der Frage.
- `description` **(required)**: Beschreibung der Frage.
- `type` **(required)**: Typ der Frage
  - Optionen für `type`: `SINGLE_CHOICE`, `YES_NO`, `MULTIPLE_CHOICE`, `FULLTEXT`
- `key` **(required)**: Eindeutiger Schlüssel für die Frage.
- `options`: Array aus Strings mit Antwortmöglichkeiten, die für `SINGLE_CHOICE` Fragen zur Verfügung stehen. 
- `hasCorrectAnswers` **(required)**: Boolean, der angibt, ob die Frage eine richtige Antwort hat.
- `correctAnswers` **(required)**: Array aus Strings mit den richtigen Antworten
  - bei `SINGLE_/MULTIPLE_CHOICE`: Indizes der Antwortmöglichkeiten. (z. B. `["0", "2"]`)
  - bei `YES_NO`: `["yes"]` oder `["no"]`


## Funktionsweise

Das Skript führt die folgenden Schritte aus:

1. Lädt die Umgebungsvariablen aus der `.env` Datei & führt Login durch.

2. Gibt aktuell existierende Kurse inklusive exisitierender Feedbacks & Quizzes aus

3. Die Datei `courses.json` wird eingelesen.

4. PATCH-Request wird an den Server gesendet.
   - Neu hinzugefügte Kurse, Feedbacks, Quizzes & Antwortmöglichkeiten werden angelegt (z. B. auch neue Antwortmöglickeiten bei bestehenden Quizzes/Feedbacks). Führt Änderungen bei bestehenden Kursen, Feedbacks, Quizzes & Antwortmöglichkeiten durch (z. B. Änderungen von Beschreibung, Formulierung von Antwormöglichkeiten, etc.)