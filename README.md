## Human Feedback

Erst einmal danke, dass du uns bei der Evaluation unseres Sprachmodells hilst.
Hier eine kurze Einführung in die Evaluation mit diesem Tool.

## Setup

Dies ist eine Vue Anwendung, die node.js benötigt. Die Anwendung wurde in Chrome getestet, daher ist es empfehlenswert auch bei der Benutzung Chrome zu nutzen.

### Installation Mac

1. Installiere Node `brew install node`
2. Teste ob Node installiert wurde `npm --version`

### Instalation Windows

Installiere node.js und npm mit der Hilfe dieser Anleitung:

- https://learn.microsoft.com/en-us/windows/dev-environment/javascript/nodejs-on-windows

### Anwendung ausführen

1. In Ordner "llm_human_feedback": `npm install`
2. Starte die Anwendung: `npm run dev`
3. Die Anwendung sollte im Browser unter `http://localhost:5173/` erreichbar sein

## Usage

- Im allgemeinen werden die Fortschritte der Evaluation im Browser gespeichert und sind daher auch nach Beenden des Browsers gesichert. Außer es wird so etwas wie ein Inkognitomodus genutzt.
- Nach der Evaluation können die Ergebnisse heruntergeladen werden.
- Du wirst nacheinander verschiedene Fragen sehen und dazu jeweils einen Antwort von einem Modell.
- Es wird verschiedene Antworten zu jeder Frage geben, die du nacheinander bewerten sollst.
- Auf der linken Seite siehst du hilfreiche Informationen zum Kontext der Frage und auf der rechten Seite die Antwort des Modells.
- Die Antworten sind in Sätze aufgeteilt, die du einzeln bewerten sollst.
- Jeder Satz hat zwei Bewertungen, eine für den Trust und einen für helpfullness der Satz.
- Bewerte jeden Satz dabei bitte unabhängig von den anderen Sätzen.
  - Eine Ausnahme besteht wenn ein Satz von dem Tool auf eine merkwürdige Art getrennt wurde oder wenn der Sinn oder die Quelle erst in einem späteren Satz deutlich wird. Dann können diese Aspekte in die Bewertung einfließen. Beispiele: Ein Satz besteht nur aus "a." und der nächste Satz enthält einen Stichpunkt. Dann gehört "a." zu dem Stichpunkt und kann die selbe Bewertung bekommen. Oder ein Satz enthält keine Quelle, aber später am Ende des Abschnittes oder am Ende der Antwort gibt es eine Quellenangabe, dann kann auch schon ein früherer Satz als Belegt gewertet werden.
- Um einen Satz zu bewerten, klicke auf die entsprechende Zahl auf der rechten Seite oder drücke die entsprechende
  Zahl auf deiner Tastatur.
- Hast du alle Sätze bewertet, kannst du unten auf "Next >" klicken, um zur nächsten Frage zu kommen. Dabei werden deine
  Bewertungen automatisch gespeichert.
- Solltest du dich vertippt haben, kannst du die Bewertung eines Satzes auch ändern, indem du auf die entsprechende Bewertung klickst.
- Solltest du Fragen haben, kannst du dich gerne an uns wenden.
- Bitte sende und seine Ergebnisse zu wenn du die Evaluation abgeschlossen hast. Wir benötigen dabei nur die Datei die du in der Anwendung herunterladen kannst.

## Rating scale

### Trust:

- Belegt: Der Satz wird durch eine Quellenangabe belegt, die auch im Kontext aufgeführt ist. Und die Informationen im Satz passen zum Kontext.
  - Die gegebenen Informationen können auch über die Angaben im Kontext hinausgehen, solange sie zu diesem passen (z.B. im Kontext wird ein Algorithmus erwähnt und der Satz gibt eine allgemeine Erklärung zu diesem Algorithmus)
- Teilweise belegt: Der Satz wird nicht durch eine Quellenangabe belegt, passt jedoch zu den Informationen aus dem Kontext. (Wie Belegt, jedoch ohne Quellenangabe)
- Allgemeinwissen: Die Informationen in dem Satz passen nicht zu den Inhalten aus dem Kontext sind aber wahr.
- Falschaussage: Der Satz lässt sich widerlegen.
- Quatsch: Der Satz ist offensichtlich falsch, da er z.B. keinen Sinn ergibt, wiedersprüchlich ist oder nicht verständlich ist.
  - Auch Sätze mit falschen Quellenangaben, die nicht im Kontext aufgelistet sind sollten als "Quatsch" bewertet werden.

### Helpfullness:

- Hilfreich: Der Satz hilft klar bei der Beantwortung der Frage. Ein Satz kann nur als hilfreich bewertet werden, wenn zuvor keine wichtigen Teile für die Antwort ausgelassen wurden.
  - z.B. Die Frage besteht aus Teil a und b. Der Satz bezieht sich auf Teil b, ohne dass Teil a bereits behandelt wurde. Der Satz kann dann maximal als eingeschränkt hilfreich bewertet werden.
- Eingeschränkt: Der Satz hilft nur teilweise bei der Beantwortung der Frage.
- Unklar: Der Satz passt zwar zum Thema der Frage, hilft aber nicht bei der Beantwortung der Frage.
- Wiederholung: Der Satz wiederholt hilfreiche oder thematisch passende Informationen aus vorherigen Sätzen. Die Information ist also nicht neu.
- Nicht hilfreich: Der Satz ist nicht hilfreich, irreführend oder unverständlich.
