---
name: SOLID Design Principles (Java)
---

# SOLID Design Principles – Java Coding Assistant Guidelines

Beim Erstellen, Überprüfen oder Ändern von Code befolge diese Richtlinien, um die SOLID-Prinzipien im Java-Projekt sicherzustellen:

---

## 1. Single Responsibility Principle (SRP)

- Jede Klasse sollte nur einen einzigen Grund für Änderungen haben.
- Begrenze die Klasse klar auf einen Funktionsbereich oder eine Abstraktion.
- Prüfe bei Überschreiten von 100–150 Zeilen pro Klasse, ob eine Aufteilung sinnvoll ist.
- Trenne Querschnittsbelange (Logging, Validierung, Fehlerbehandlung) von Geschäftslogik.
- Implementiere separate Klassen für z.B. Datenzugriff, Business Rules, UI.
- Methodenbezeichnungen müssen ihren Einzelzweck klar widerspiegeln.
- Enthält eine Methodendokumentation ein "und/oder", droht ein SRP-Verstoß.
- Komposition (z.B. durch Member-Objekte, Delegation) ist Vererbung vorzuziehen.

**Beispiel:**
```java
// GUT: einzelne Verantwortung
public class UserRepository {
    public Optional<User> findById(String id) { /* ... */ }
}

// SCHLECHT: vermischt viele Verantwortungen
public class UserManager {
    // Datenhaltung, Validierung, Logging usw. gemischt
}
```

---

## 2. Open/Closed Principle (OCP)

- Klassen sollen für Erweiterung offen, für Modifikation geschlossen sein.
- Verwende abstrakte Klassen und Interfaces für stabile Verträge.
- Biete gezielte Extension Points (z. B. Strategie-Muster) für Vielfalt.
- Reduziere Verzweigungen (switch/if-else) auf Grundlage von Typprüfungen.
- Nutze Dependency Injection und Konfiguration als Erweiterungsmechanismus.
- Setze auf Polymorphie statt bedingter Logik zur Laufzeit.

**Beispiel:**
```java
public interface PaymentStrategy {
    void pay(BigDecimal amount);
}

public class PaypalPayment implements PaymentStrategy { /* ... */ }
public class CreditCardPayment implements PaymentStrategy { /* ... */ }

// Erweiterbar ohne bestehende Klassen zu ändern
```

---

## 3. Liskov Substitution Principle (LSP)

- Subklassen müssen überall einsetzbar sein wie ihre Oberklasse.
- Bewahre alle Zusicherungen (Invarianten) der Basisklasse.
- Verschärfe keine Vorbedingungen, schwäche keine Nachbedingungen in Overrides ab.
- Überschreibe Methoden nicht, um sie nutzlos zu machen oder nur Exceptions zu werfen.
- Vermeide Methoden, die Typprüfungen oder Downcasts auf Subtypen machen.
- Nutze Komposition, wenn vollständige Austauschbarkeit mit Basistypen nicht sinnvoll ist.

**Beispiel:**
```java
// GUT
public class Rectangle {
    public int area() { /* ... */ }
}
public class Square extends Rectangle {
    // area() bleibt korrekt, keine Einschränkung oder Änderung der Basisklassenverträge
}
```

---

## 4. Interface Segregation Principle (ISP)

- Erstelle fokussierte, minimalistische Schnittstellen mit zusammengehörigen Methoden.
- Splitte große ("fette") Interfaces in kleinere spezialisierte Interfaces auf.
- Richte Interfaces nach Client-Bedürfnissen, nicht nach Bequemlichkeit der Implementierung.
- Nutze Rollen-/Verhaltens-Interfaces, nicht Objektarten.
- Nutze Interface-Komposition für komplexes Verhalten.
- Entferne Methoden aus Interfaces, die nicht für alle Implementierungen relevant sind.

**Beispiel:**
```java
// SCHLECHT: "Fettes" Interface
public interface Device {
    void print();
    void scan();
    void fax();
}

// Besser: Segregation
public interface Printer { void print(); }
public interface Scanner { void scan(); }
public interface Fax { void fax(); }
```

---

## 5. Dependency Inversion Principle (DIP)

- High-Level-Module sollen sich auf Abstraktionen, nicht auf konkrete Implementierungen beziehen.
- Mache Abhängigkeiten durch Konstruktorinjektion explizit.
- Nutze Dependency Injection Frameworks bei Bedarf (z.B. Spring).
- Code gegen Interfaces, nicht gegen konkrete Klassen.
- Trenne Abstraktionen (Interfaces) und Implementierungen (z.B. unterschiedliche Pakete).
- Vermeide Konstruktionen wie `new ServiceImpl()` im Code der Geschäftslogik.
- Abstraktionsgrenzen an Layerübergängen einziehen.

**Beispiel:**
```java
public interface EmailService {
    void sendEmail(String to, String message);
}

public class NotificationManager {
    private final EmailService emailService;

    public NotificationManager(EmailService emailService) {
        this.emailService = emailService;
    }
}
```

---

## Implementierungsleitfaden

- Definiere beim Anlegen einer neuen Klasse schriftlich deren Einzelverantwortung (z.B. JavaDoc).
- Dokumentiere gezielt Erweiterungspunkte und erwartetes Subklassierungsverhalten.
- Schreibe Interface-Verträge inkl. Invarianten und Erwartungen.
- Reflektiere kritisch bei Klassen, die viele konkrete Implementierungen direkt referenzieren.
- Setze Fabriken, Dependency Injection oder Service Locator Patterns für die Verwaltung von Instanzen ein.
- Prüfe Vererbungshierarchien regelmäßig auf LSP-Konformität.
- Refaktoriere Code gezielt in Richtung SOLID, insb. bei Erweiterungen und Wartung.
- Nutze relevante Design Patterns (Strategy, Factory, Decorator, Observer etc.), um SOLID zu realisieren.

---

## Warnsignale

- „God Classes“, die alle Aufgaben übernehmen
- Methoden mit vielen/Boolean-Parametern, die Verhalten drastisch ändern
- Tiefe oder nicht nachvollziehbare Vererbungshierarchien
- Klassen kennen Detailimplementierungen ihrer Abhängigkeiten
- Zirkuläre Abhängigkeiten zwischen Modulen/Klassen
- Starke Kopplung zwischen fachlich nicht zusammenhängenden Komponenten
- Klassen, die rasant wachsen, wenn Funktionen ergänzt werden
- Methoden mit ungewöhnlich vielen Parametern
