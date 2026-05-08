# AGENTS.md - Schach-Spiel Projekt

## Projektübersicht

Dies ist ein Schachspiel mit KI-Gegner, vollständig in einer einzigen HTML-Datei implementiert. Das Spiel nutzt Unicode-Schachfiguren, ein ELO-Bewertungssystem und einen Minimax-Algorithmus für höhere Schwierigkeitsstufen.

## Build/Lint/Test Befehle

### Anwendung starten
```bash
# Im Browser öffnen (einfachste Methode)
xdg-open Schach0.02.html

# Oder mit einem lokalen Server (empfohlen für Entwicklung)
python -m http.server 8000
# Dann öffnen: http://localhost:8000/Schach0.02.html
```

### Tests
Es gibt keine automatisierten Tests. Das Spiel muss manuell im Browser getestet werden:
- Figurauswahl und Zugvalidierung
- KI-Züge auf verschiedenen Schwierigkeitsstufen
- Schach/Schachmatt-Erkennung
- Bauernumwandlung
- ELO-Berechnung

### Lint/Formatierung
Keine konfigurierten Linter. Bei Änderungen manuell auf sauberen Code achten:
```bash
# Optional: HTML validieren
tidy -e Schach0.02.html

# Optional: JavaScript mit ESLint prüfen (falls installiert)
eslint Schach0.02.html --ext .html
```

## Projektstruktur

```
Schach/
├── Schach0.02.html      # Hauptdatei (aktuelle Version)
├── Schach0.01.html      # Vorgängerversion (im Archiv/)
├── icon.png             # Projektsymbol
├── info.txt             # Projektbeschreibung
└── Archiv/              # Ältere Versionen
```

## Code-Stilrichtlinien

### HTML
- Deutsche Sprache für UI-Texte (`lang="de"`)
- Semantische HTML5-Elemente verwenden
- IDs und Klassen in englischer Sprache (z.B. `game-container`, `chess-board`)
- Inline-Event-Handler sind akzeptabel (z.B. `onclick="startNewGame()"`)

### CSS
- BEM-ähnliche Namenskonvention: `.game-container`, `.chess-board`, `.square.light`
- CSS-Variablen bevorzugen bei wiederholten Werten
- Farben als Hex oder rgba
- Animationen mit `@keyframes` definieren
- Responsive Design mit Media Queries

```css
/* Empfohlene Farbschema-Struktur */
.square.light { background-color: #f0d9b5; }
.square.dark { background-color: #b58863; }
```

### JavaScript
- Funktionsdeklarationen mit `function` (keine Arrow Functions für Hauptfunktionen)
- Variablen mit `let` für veränderliche Werte, `const` für Konstanten
- Keine TypeScript-Typen oder JSDoc erforderlich, aber hilfreich

```javascript
// Globale Zustandsvariablen am Anfang definieren
let board = [];
let currentPlayer = 'w';
let selectedSquare = null;
let gameOver = false;
let difficulty = 3;
let soundEnabled = true;
let playerElo = 1200;
let aiElo = 1200;
let moveHistory = [];
let lastMove = null;
let kingInCheck = null;
```

#### Benennungskonventionen
- Funktionen: camelCase (`getPossibleMoves`, `isValidMove`, `aiMove`)
- Konstanten: camelCase für Objekte/Arrays (`pieces`)
- Boolesche Variablen: mit Präfix wie `is`, `has`, `should` (`gameOver`, `soundEnabled`)
- Event-Handler: mit Präfix `handle` (`handleSquareClick`, `handlePromotion`)

#### Figur-Repräsentation
```javascript
// Zwei-Zeichen-Format: Farbe + Typ
// 'w' = weiß, 'b' = schwarz
// 'K' = König, 'Q' = Dame, 'R' = Turm, 'B' = Läufer, 'N' = Springer, 'P' = Bauer
// Beispiele: 'wK', 'bQ', 'wP', 'bN'
const pieces = {
    'wK': '♔', 'wQ': '♕', 'wR': '♖', 'wB': '♗', 'wN': '♘', 'wP': '♙',
    'bK': '♚', 'bQ': '♛', 'bR': '♜', 'bB': '♝', 'bN': '♞', 'bP': '♟'
};
```

#### Koordinatensystem
- Board: 8x8 Array `[row][col]`
- Zeilen (row): 0 = schwarze Grundreihe, 7 = weiße Grundreihe
- Spalten (col): 0 = a-Date, 7 = h-Date
- Rückgabe von Positionen als Array `[row, col]`

### Fehlerbehandlung
- Keine Try-Catch-Blöcke erforderlich für dieses Projekt
- Validierung durch bedingte Prüfungen
- Rückgabe von `null` für nicht gefundene Elemente

```javascript
function findKing(color) {
    for (let row = 0; row < 8; row++) {
        for (let col = 0; col < 8; col++) {
            const piece = board[row][col];
            if (piece === color + 'K') {
                return [row, col];
            }
        }
    }
    return null;
}
```

### Audio/Sound
- Web Audio API verwenden (nicht HTML5 Audio Elemente)
- AudioContext lazy initialisieren

```javascript
let audioContext;

function initAudio() {
    if (!audioContext) {
        audioContext = new (window.AudioContext || window.webkitAudioContext)();
    }
}
```

## KI-Implementierung

### Schwierigkeitsstufen
1. **Anfänger (ELO ~800)**: Zufällige Züge
2. **Leicht (ELO ~1000)**: Züge mit Score > -50 bevorzugen
3. **Mittel (ELO ~1200)**: Beste Züge mit etwas Zufälligkeit
4. **Schwer (ELO ~1500)**: Minimax mit Tiefe 2
5. **Experte (ELO ~1800)**: Minimax mit Tiefe 3

### Bewertungsfunktion
```javascript
// Materialwerte
const pieceValues = { P: 1, N: 3, B: 3, R: 5, Q: 9, K: 100 };

// Positionsbewertung
// - Zentrale Kontrolle
// - Figurenentwicklung
// - Sicherheit (Anzahl Angreifer)
```

## Wichtige Funktionen

| Funktion | Beschreibung |
|----------|--------------|
| `initBoard()` | Initialisiert das Brett mit Startposition |
| `renderBoard()` | Rendert das Brett im DOM |
| `getPossibleMoves(row, col)` | Gibt alle möglichen Züge für eine Figur |
| `isValidMove(from, to)` | Prüft Zugvalidität inkl. Schach-Vermeidung |
| `makeMove(from, to)` | Führt einen Zug aus |
| `aiMove()` | Löst KI-Zug aus |
| `checkGameEnd()` | Prüft auf Schachmatt/Patt |
| `updateElo(playerWon)` | Aktualisiert ELO-Punkte |

## Tastenkürzel

- `N` - Neues Spiel starten
- `S` - Sound ein/ausschalten

## Hinweise für neue Features

1. **Rochade**: Aktuell nicht implementiert. Müsste in `getKingMoves()` ergänzt werden.
2. **En Passant**: Aktuell nicht implementiert. Müsste in `getPawnMoves()` ergänzt werden.
3. **Remis-Angebot**: Könnte als Button hinzugefügt werden.
4. **Spielstand speichern**: localStorage könnte für Persistenz genutzt werden.

## Sprache

Alle UI-Texte und Meldungen sind auf Deutsch. Kommentare im Code können auf Englisch oder Deutsch sein.
