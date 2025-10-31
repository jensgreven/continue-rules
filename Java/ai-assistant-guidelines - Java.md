---
name: AI Assistant Guidelines (Java)
globs: "**/*.java"
alwaysApply: true
description: Guidelines for working effectively with AI coding assistants in Java projects
---

Diese Guidelines unterstützen produktive und qualitativ hochwertige Code-Erstellung mit AI-Coding-Assistants in Java-Projekten.

## Code-Kontext und Struktur

- Schreibe klar strukturierten, selbstdokumentierenden Code
- Verwende aussagekräftige Datei- und Paketnamen
- Gruppiere zusammengehörende Funktionalität (Paketstruktur)
- Ergänze komplexe Verzeichnisse um README.md
- Verwende eindeutige Schnittstellen (Interfaces) und ggf. DTOs

## Dokumentation für AI-Verständnis

### Methoden-Dokumentation

```java
/**
 * Berechnet die Versandkosten abhängig von Gewicht, Distanz und Versandtyp.
 *
 * Geschäftsregeln:
 * - Standardversand: 0,50 €/kg + 0,10 €/km
 * - Express: 1,00 €/kg + 0,20 €/km
 * - Overnight: 25 € pauschal + Standardtarif
 * - Kostenloser Versand ab 100 € Bestellwert (nur Standardversand)
 *
 * @param weight   Paketgewicht in kg
 * @param distance Versanddistanz in km
 * @param type     Versandtyp: "STANDARD", "EXPRESS", "OVERNIGHT"
 * @return Versandkosten als BigDecimal
 * @throws IllegalArgumentException bei ungültigem Typ/negativen Werten
 */
public BigDecimal calcShippingCost(
    double weight,
    double distance,
    String type
) {
    // Implementierung ...
}
```

## Projektstruktur-Best-Practices

### Klare Architektur

```
project/
├── README.md
├── ARCHITECTURE.md
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── de/provinzial/<projekt>/
│   │   │       ├── api/       // REST-Controller
│   │   │       ├── service/   // Geschäftslogik
│   │   │       ├── model/     // Entitäten/DTOs
│   │   │       ├── util/      // Utilities
│   │   │       └── config/    // Konfigurationen
│   └── test/
│       └── java/
│           └── de/provinzial/<projekt>/
└── docs/
```

### Konfigurationsdokumentation

```java
// src/main/resources/application.properties
# Datenbankkonfiguration
# DB_HOST: Datenbank-Host (default: localhost)
# DB_PORT: Port (default: 5432)
# DB_NAME: Name der Datenbank (erforderlich)
# DB_USER: Benutzername (erforderlich)
# DB_PASSWORD: Passwort (erforderlich)
# Pool: max=20, Timeout=30s
spring.datasource.url=jdbc:postgresql://${DB_HOST:localhost}:${DB_PORT:5432}/${DB_NAME}
spring.datasource.username=${DB_USER}
spring.datasource.password=${DB_PASSWORD}
spring.datasource.hikari.maximum-pool-size=20
spring.datasource.hikari.idle-timeout=30000
```

## Namenskonventionen

```java
public class UserDto {
    private String id;
    private String email;
    private String name;
    // Getter und Setter...
}

public interface UserService {
    UserDto createUser(CreateUserDto dto);
    UserDto getUserById(String id);
    UserDto updateUser(String id, UpdateUserDto dto);
    void deleteUser(String id);
}

// Event-Namenskonvention als Enum
public enum UserEvent {
    USER_CREATED,
    USER_UPDATED,
    USER_DELETED
}
```

## Fehlerkontext

```java
public class OrderProcessingException extends Exception {
    private final String orderId;
    private final Stage stage;

    public enum Stage { VALIDATION, PAYMENT, FULFILLMENT }

    public OrderProcessingException(String message, String orderId, Stage stage, Throwable cause) {
        super(message, cause);
        this.orderId = orderId;
        this.stage = stage;
    }

    // Getter...
}

// Verwendung mit Kontext:
throw new OrderProcessingException(
    "Payment failed",
    order.getId(),
    OrderProcessingException.Stage.PAYMENT,
    new RuntimeException("Insufficient funds")
);
```

## Test-Patterns

### Aussagekräftige Testfälle

```java
import org.junit.jupiter.api.*;

class ShoppingCartTest {

    /**
     * Testet das Hinzufügen eines Artikels in einen leeren Warenkorb.
     * Erwartung: Ein Artikel im Warenkorb, korrekter Gesamtpreis.
     */
    @Test
    void testAddItemToEmptyCart() {
        ShoppingCart cart = new ShoppingCart();
        Product item = new Product("123", "Widget", BigDecimal.valueOf(10));
        cart.addItem(item, 1);

        Assertions.assertEquals(1, cart.getItems().size());
        Assertions.assertEquals(BigDecimal.valueOf(10), cart.getTotal());
    }

    /**
     * Testet Hinzufügen eines doppelten Artikels.
     * Erwartung: Erhöhung der Menge statt Duplikat.
     */
    @Test
    void testAddDuplicateItemIncreasesQuantity() {
        // Implementierung ...
    }

    /**
     * Überprüft das Entfernen eines nicht vorhandenen Artikels.
     * Erwartung: IllegalArgumentException
     */
    @Test
    void testRemoveItemNotInCartThrows() {
        // Implementierung ...
    }
}
```

## AI-freundliche Kommentare

### Was kommentieren?

```java
// DO: Geschäftslogik erklären
// Rabattberechnung: 
// - 0-99 Stück: 0%
// - 100-499 Stück: 10%
// - ab 500 Stück: 15%
private double calculateBulkDiscount(int quantity) {
    if (quantity < 100) return 0;
    if (quantity < 500) return 0.10;
    return 0.15;
}

// DO: Workarounds dokumentieren
// WORKAROUND: Spezielles Handling für IE11-Kompatibilität bis Migration abgeschlossen ist
// ...

// DON'T: Offensichtliches kommentieren
// DON'T: i um 1 erhöhen
i++;
```

## Best Practices

- Halte Methoden klein und fokussiert
- Einheitlichen Stil überall einhalten (z.B. mittels Checkstyle)
- Komplexe Algorithmen und Geschäftsregeln dokumentieren
- Abhängigkeiten in Build-Tools (z.B. Maven/Gradle) pflegen
- Beispielnutzung dokumentieren (z.B. JavaDoc @see, @example)
- Integrationstests ergänzen
- API-Verträge klar dokumentieren (OpenAPI, JavaDoc)
- Fehlermeldungen möglichst präzise gestalten
