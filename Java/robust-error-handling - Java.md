---
name: Robust Error Handling (Java)
globs: "**/*.java"
alwaysApply: true
description: Richtlinien für umfassende Fehlerbehandlung und resiliente Java-Anwendungen
---

Du bist Experte für Fehlerbehandlung, defensives Programmieren und den Bau belastbarer Java-Anwendungen.

## Prinzipien der Fehlerbehandlung

- Scheitere früh und mit klaren Fehlermeldungen („fail fast and fail clear“)
- Stelle handlungsrelevante Fehlermeldungen zur Verfügung
- Logge Fehler immer mit ausreichend Kontext (ggf. mit Logging-Framework: SLF4J, Log4j, etc.)
- Behandle Fehler auf der passenden Ebene (zentrale Fallbacks/mechanismen nutzen)
- Fehler niemals „still schlucken“

## Fehlerarten und Behandlung in Java

```java
// Eigene Fehlerklassen erstellen
public class ValidationException extends RuntimeException {
    private final String field;
    public ValidationException(String message, String field) {
        super(message);
        this.field = field;
    }
    public String getField() { return field; }
}

public class NetworkException extends RuntimeException {
    private final int statusCode;
    public NetworkException(String message, int statusCode) {
        super(message);
        this.statusCode = statusCode;
    }
    public int getStatusCode() { return statusCode; }
}

// Umfassendes Error-Handling (z.B. in Service-Methoden)
public User fetchUserData(String userId) {
    try {
        if (userId == null || !isValidUUID(userId))
            throw new ValidationException("Invalid user ID format", "userId");
        
        HttpResponse response = apiClient.get("/api/users/" + userId);

        if (!response.isOk()) {
            throw new NetworkException(
                "Failed to fetch user: " + response.getStatusText(),
                response.getStatusCode()
            );
        }

        User user = parseUser(response.getBody());
        return validateUser(user);

    } catch (Exception error) {
        // Mit Kontext loggen
        logger.error("Failed to fetch user data",
            // Kontextinfos - je nach Logger unterschiedlich
            Map.of(
                "userId", userId,
                "error", error.getMessage()
            ), error
        );
        
        // Ausnahme weiterreichen oder transformieren
        if (error instanceof ValidationException) {
            throw (ValidationException) error;
        }

        if (error instanceof NetworkException 
                && ((NetworkException) error).getStatusCode() == 404) {
            throw new RuntimeException("User not found", error);
        }
        
        throw new RuntimeException("Unable to retrieve user data. Please try again later.", error);
    }
}
```

## Ressourcen und Transaktionen sicher behandeln

```java
// Mit try-with-resources („context manager“ in Python äquivalent)
public void executeTransaction(TransactionCallback callback) {
    try (Connection conn = dataSource.getConnection()) {
        conn.setAutoCommit(false);

        try {
            callback.execute(conn);
            conn.commit();
        } catch (SQLException e) {
            conn.rollback();
            logger.error("Database transaction failed: {}", e.getMessage());
            throw new DataIntegrityException("Failed to complete transaction", e);
        } catch (Exception e) {
            conn.rollback();
            logger.error("Unexpected error in transaction: {}", e.getMessage());
            throw e;
        }

    } catch (SQLException e) {
        logger.error("Error during DB transaction setup: {}", e.getMessage());
        throw new RuntimeException("Database unavailable", e);
    }
}
```

## Graceful Degradation

```java
public UserPreferences getUserPreferences(String userId) {
    try {
        return cache.getUserPreferences(userId);
    } catch (CacheException e) {
        logger.warn("Cache miss for user {}, falling back to database", userId);
        try {
            return database.fetchUserPreferences(userId);
        } catch (DatabaseException ex) {
            logger.error("Failed to fetch preferences for user {}: {}", userId, ex.getMessage());
            return UserPreferences.getDefault();
        }
    }
}
```

## Fehler-Resilienz-Strategien

### Retry-Logik

```java
public <T> T resilientApiCall(Callable<T> fn, int maxRetries, long baseBackoffMs, Predicate<Exception> retryable) throws Exception {
    Exception lastError = null;
    for (int attempt = 0; attempt <= maxRetries; attempt++) {
        try {
            return fn.call();
        } catch (Exception e) {
            lastError = e;
            if (attempt == maxRetries || (retryable != null && !retryable.test(e))) {
                throw e;
            }
            long delay = (long) (baseBackoffMs * Math.pow(2, attempt));
            logger.info("Retry attempt {} after {} ms", (attempt + 1), delay);
            Thread.sleep(delay);
        }
    }
    throw lastError != null ? lastError : new Exception("Unknown failure in resilientApiCall()");
}
```

### Circuit Breaker Pattern (vereinfacht)

```java
public class CircuitBreaker<T> {
    private final Callable<T> fn;
    private final int threshold;
    private final long timeoutMs;

    private int failures = 0;
    private long nextAttempt = 0;

    public CircuitBreaker(Callable<T> fn, int threshold, long timeoutMs) {
        this.fn = fn;
        this.threshold = threshold;
        this.timeoutMs = timeoutMs;
    }

    public synchronized T call() throws Exception {
        long now = System.currentTimeMillis();
        if (failures >= threshold) {
            if (now < nextAttempt) throw new IllegalStateException("Circuit breaker is OPEN");
            failures = 0;
        }

        try {
            T result = fn.call();
            failures = 0;
            return result;
        } catch (Exception ex) {
            failures++;
            if (failures >= threshold) nextAttempt = now + timeoutMs;
            throw ex;
        }
    }
}
```

## Best Practices

- Setze `try-catch-finally`-Blöcke sinnvoll ein (und `try-with-resources` für Ressourcen)
- Implementiere zentrale Exception-Handler (z.B. via @ControllerAdvice in Spring)
- Validierung immer frühzeitig und explizit (Parameter, DTOs prüfen)
- Stelle nutzerfreundliche, aber technische sinnvolle Fehlermeldungen bereit
- Fehler mit ausreichend Kontext loggen (u.a. Stacktraces, IDs, Nutzerkontext)
- Erkenne, monitore und reagiere auf Fehler-Muster aktiv
- Teste Fehlerfälle (auch im Unit- und Integrationstest)
