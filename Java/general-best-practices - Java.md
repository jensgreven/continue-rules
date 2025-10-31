---
name: General Coding Standards (Java)  
globs: "**/*.java"  
alwaysApply: true  
description: Universelle Java Coding Standards nach Clean Code und Best Practices  
---

Du bist Experte für Software-Engineering, Clean Code und Best Practices in Java-Projekten.

## Namenskonventionen

- Verwende beschreibende und aussagekräftige Namen
- Einheitliche Namensmuster im Projekt durchhalten
- Keine Abkürzungen oder Einzelbuchstaben (außer Schleifenindizes)
- Konstanten sollten suchbare, gut lesbare Namen haben
- Zweck muss am Namen ersichtlich sein

### Beispiele

```java
// Gut
public static final int MAX_RETRY_ATTEMPTS = 3;
private String userAuthenticationToken = generateToken();

public BigDecimal calculateCompoundInterest(BigDecimal principal, double rate, int years) { /* ... */ }

// Schlecht
public static final int max = 3;
private String tkn = generateToken();

public BigDecimal calc(BigDecimal p, double r, int t) { /* ... */ }
```

## Methodendesign

- Halte Methoden klein, klar und eindeutig (Single Responsibility)
- Idealerweise maximal 3 Parameter, bei mehr: Kapselung als Objekt erwägen
- Methodennamen als „Tätigkeitsbeschreibung“ und sprechend
- Vermeide Seiteneffekte (außer explizit dokumentiert)
- Frühzeitiges Rückgeben, um Verschachtelung zu minimieren

```java
// Gut
public BigDecimal calculateOrderTotal(List<OrderItem> items) {
    if (items == null || items.isEmpty()) return BigDecimal.ZERO;
    BigDecimal subtotal = items.stream()
        .map(item -> item.getPrice().multiply(BigDecimal.valueOf(item.getQuantity())))
        .reduce(BigDecimal.ZERO, BigDecimal::add);
    return applyTaxes(subtotal);
}

// Schlecht
public void process(List data) {
    // 200 Zeilen, viele Verantwortlichkeiten, versteckte Nebeneffekte
}
```

## Code-Organisation

- Nahe zusammenliegende Funktionalität gemeinsam gruppieren (Pakete)
- Konsistente Paket- und Ordnerstruktur nutzen
- Klassen und Dateien auf einen Verantwortungsbereich beschränken
- Keine „Mega“-Klassen und -Dateien
- Eindeutige und tiefe Ordner-Hierarchie

```
project/
├── src/
│   ├── main/
│   │   └── java/
│   │       └── de/provinzial/<projekt>/
│   │           ├── controller/
│   │           ├── service/
│   │           ├── model/
│   │           ├── util/
│   │           └── config/
├── test/
├── docs/
└── scripts/
```

## Fehler-Prävention

- Eingaben an Systemgrenzen validieren
- Typisierung und Java Typ-System konsequent nutzen
- Sonderfälle (Null, Grenzwerte, etc.) explizit behandeln
- Schnell mit aussagekräftigen Fehlermeldungen scheitern („fail fast“)
- Defensive Programmiertechniken anwenden

```java
public double divideNumbers(double dividend, double divisor) {
    if (divisor == 0) {
        throw new IllegalArgumentException("Division by zero is not allowed");
    }
    if (Double.isNaN(dividend) || Double.isNaN(divisor) 
            || Double.isInfinite(dividend) || Double.isInfinite(divisor)) {
        throw new IllegalArgumentException("Invalid number provided");
    }
    return dividend / divisor;
}
```

## Lesbarkeit

- Code vorrangig für Menschen schreiben
- Gleichbleibende Formatierung (idealerweise via Formatter/Checkstyle/PRETTIER)
- Sinnvolle Leerzeilen für Übersichtlichkeit
- Zeilenlänge begrenzen (80–120 Zeichen)
- Aussagekräftige Variablennamen verwenden

## DRY-Prinzip

- „Don’t Repeat Yourself“: Wiederholungen vermeiden
- Gemeinsame Logik extrahieren (Hilfsmethoden, Basisklassen oder Utilities)
- Laufzeitkonfiguration anstelle von Code-Redundanz
- Wiederverwendbare Komponenten schaffen
- Klarheit steht immer über maximaler DRY-Disziplin

## SOLID-Prinzipien

- **Single Responsibility**: Jede Klasse hat genau einen Grund zur Änderung
- **Open/Closed**: Für Erweiterung offen, für Änderung geschlossen
- **Liskov Substitution**: Subtypen sind überall dort einsetzbar, wo der Basistyp verwendet wird
- **Interface Segregation**: Viele spezifische statt weniger allgemeiner Schnittstellen
- **Dependency Inversion**: Immer gegen Abstraktionen programmieren

## Best Practices

- Komposition bevorzugen gegenüber Vererbung
- Schreibe (Unit-)Tests zu deinem Code
- Refactore regelmäßig – auch kleine Verbesserungen sofort angehen
- Nutze Versionsverwaltung (Git, Branch-Konventionen, PRs etc.)
- Dokumentiere wesentliche Architekturentscheidungen (z. B. als Architekturdokument, ADR)
