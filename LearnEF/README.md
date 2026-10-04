# Booking Management System

## Obiettivo

Realizzare una Web API in **.NET con ASP.NET Core** per la gestione di prenotazioni di servizi.

L’applicazione dovrà permettere agli utenti di visualizzare i servizi disponibili e creare prenotazioni, mentre un amministratore dovrà poter gestire i servizi e consultare tutte le prenotazioni.

Il progetto dovrà utilizzare **Entity Framework Core** per la persistenza dei dati e un database relazionale, preferibilmente **PostgreSQL** o **SQL Server**.

---

# Architettura del progetto

Il progetto è organizzato seguendo una struttura ispirata ai principi della **Clean Architecture**, separando dominio, logica applicativa, infrastruttura e presentation layer.

```text
BookingSystem
│
├── BookingSystem.Web
│   ├── Controllers
│   ├── Middleware
│   ├── Program.cs
│   └── appsettings.json
│
├── BookingSystem.Application
│   ├── DTOs
│   ├── Interfaces
│   └── Services
│
├── BookingSystem.Domain
│   ├── Entities
│   └── Enums
│
├── BookingSystem.Infrastructure
│   ├── Persistence
│   │   ├── AppDbContext.cs
│   │   ├── Configurations
│   │   └── Migrations
│   └── DependencyInjection.cs
│
└── BookingSystem.Tests
```

## BookingSystem.Web

Rappresenta il **Presentation Layer** dell'applicazione.

Contiene:

- API Controller;
- middleware;
- configurazione dell'applicazione;
- configurazione della Dependency Injection;
- autenticazione e autorizzazione;
- Swagger/OpenAPI.

Questo layer riceve le richieste HTTP e delega la logica applicativa al progetto `BookingSystem.Application`.

---

## BookingSystem.Application

Contiene i **casi d'uso** e la logica applicativa.

Esempi:

```text
DTOs/
    CreateBookingDto.cs
    UpdateBookingDto.cs
    BookingDto.cs
    CreateServiceDto.cs
    ServiceDto.cs

Interfaces/
    IBookingService.cs
    IServiceService.cs

Services/
    BookingService.cs
    ServiceService.cs
    AuthService.cs
```

Questo layer coordina le operazioni dell'applicazione senza dipendere direttamente dai dettagli infrastrutturali.

---

## BookingSystem.Domain

Contiene il **dominio principale dell'applicazione**.

Esempio:

```text
Entities/
    User.cs
    Service.cs
    Booking.cs

Enums/
    BookingStatus.cs
    UserRole.cs
```

Il Domain contiene:

- entità;
- enum;
- eventuali regole di dominio;
- modelli indipendenti dal database e dal framework Web.

Il progetto Domain non deve dipendere da:

- ASP.NET Core;
- Entity Framework Core;
- PostgreSQL;
- SQL Server;
- servizi esterni.

---

## BookingSystem.Infrastructure

Contiene tutti i dettagli infrastrutturali dell'applicazione.

In particolare, **Entity Framework Core** sarà configurato in questo progetto.

```text
Persistence/
    AppDbContext.cs

    Configurations/
        BookingConfiguration.cs
        ServiceConfiguration.cs
        UserConfiguration.cs

    Migrations/
```

Questo layer si occupa di:

- Entity Framework Core;
- DbContext;
- configurazione delle entità;
- migrations;
- accesso al database;
- eventuali API esterne;
- eventuali servizi email;
- file system.

---

## BookingSystem.Tests

Contiene i test automatici.

Il progetto utilizzerà principalmente **xUnit**.

I test potranno essere suddivisi in:

```text
Unit/
Integration/
```

I test unitari verificheranno principalmente la logica applicativa.

I test di integrazione potranno verificare:

- endpoint HTTP;
- Entity Framework Core;
- database;
- autenticazione;
- regole di business.

---

## Dipendenze tra i layer

Le dipendenze devono essere mantenute il più possibile nella seguente direzione:

```text
Domain
   ↑
Application
   ↑
Web
```

Il progetto Infrastructure implementa i dettagli necessari all'Application:

```text
Infrastructure
      ↓
Application
      ↓
Domain
```

Il Domain non deve conoscere Infrastructure.

Ad esempio:

```text
BookingSystem.Domain
```

non deve dipendere da:

```text
BookingSystem.Infrastructure
```

e le Entity non devono dipendere direttamente da Entity Framework Core.

---

## Requisiti principali

L'applicazione dovrà gestire tre entità principali:

### User

Un utente deve avere almeno:

- Id
- Nome
- Email
- Password
- Ruolo

I ruoli previsti sono:

- User
- Admin

### Service

Un servizio deve contenere:

- Id
- Nome
- Descrizione
- Prezzo
- Durata in minuti

Esempi di servizi:

- Taglio capelli
- Consulenza
- Revisione veicolo
- Riparazione
- Visita

### Booking

Una prenotazione deve contenere:

- Id
- Data e ora di inizio
- Data e ora di fine
- Stato della prenotazione
- Utente che ha effettuato la prenotazione
- Servizio prenotato

Gli stati possibili possono essere:

- Pending
- Confirmed
- Cancelled

---

# Funzionalità richieste

## Gestione servizi

L'amministratore deve poter:

- creare un servizio;
- visualizzare tutti i servizi;
- modificare un servizio;
- eliminare un servizio.

Endpoint indicativi:

```http
GET /api/services

GET /api/services/{id}

POST /api/services

PUT /api/services/{id}

DELETE /api/services/{id}
```

---

## Gestione prenotazioni

Un utente deve poter:

- creare una prenotazione;
- visualizzare le proprie prenotazioni;
- modificare una prenotazione;
- cancellare una prenotazione.

L'amministratore deve poter visualizzare tutte le prenotazioni.

Endpoint indicativi:

```http
GET /api/bookings

GET /api/bookings/{id}

POST /api/bookings

PUT /api/bookings/{id}

DELETE /api/bookings/{id}
```

---

# Regola di business principale

Il sistema non deve consentire due prenotazioni sovrapposte nello stesso intervallo temporale.

Prima di creare una prenotazione deve quindi essere verificato che non esistano altre prenotazioni attive che occupano lo stesso intervallo.

Esempio:

```text
Prenotazione esistente

10:00 -------- 11:00
```

Non devono essere accettate prenotazioni come:

```text
10:30 -------- 11:30
```

oppure:

```text
09:30 -------- 10:30
```

mentre deve essere possibile creare:

```text
11:00 -------- 12:00
```

Questa verifica deve essere effettuata tramite una query Entity Framework Core.

---

# Entity Framework Core

Il progetto deve utilizzare **EF Core Code First**.

Devono essere configurati:

- DbContext;
- DbSet;
- relazioni tra entità;
- foreign key;
- navigation properties;
- Fluent API;
- migrations.

Il database dovrà essere creato utilizzando le migration.

Esempio:

```bash
dotnet ef migrations add InitialCreate
dotnet ef database update
```

Il `DbContext` sarà posizionato nel progetto:

```text
BookingSystem.Infrastructure/Persistence/AppDbContext.cs
```

Le configurazioni EF Core delle singole Entity saranno preferibilmente separate:

```text
BookingSystem.Infrastructure/Persistence/Configurations/
```

Ad esempio:

```text
BookingConfiguration.cs
ServiceConfiguration.cs
UserConfiguration.cs
```

---

# Relazioni

Le relazioni previste sono:

```text
User
  |
  | 1
  |
  | N
Booking
  |
  | N
  |
  | 1
Service
```

Quindi:

```text
User 1 ---- N Booking

Service 1 ---- N Booking
```

Un utente può avere più prenotazioni.

Un servizio può essere associato a più prenotazioni.

---

# Query con LINQ

Implementare almeno alcune query utilizzando LINQ ed Entity Framework Core.

Ad esempio:

- visualizzare tutte le prenotazioni di una determinata data;
- visualizzare le prenotazioni di un determinato utente;
- visualizzare le prenotazioni ordinate per data;
- visualizzare le prenotazioni associate a uno specifico servizio;
- verificare la presenza di prenotazioni sovrapposte.

Esempio:

```csharp
var bookings = await context.Bookings
    .Include(b => b.Service)
    .Where(b => b.StartDate.Date == date.Date)
    .OrderBy(b => b.StartDate)
    .ToListAsync();
```

Esempio di verifica di sovrapposizione:

```csharp
var hasConflict = await context.Bookings
    .AnyAsync(b =>
        b.StartDate < newBooking.EndDate &&
        b.EndDate > newBooking.StartDate);
```

---

# DTO

Gli endpoint non devono utilizzare direttamente le Entity come input e output.

Creare quindi dei DTO, per esempio:

```text
CreateServiceDto
UpdateServiceDto
ServiceDto

CreateBookingDto
UpdateBookingDto
BookingDto

LoginDto
RegisterDto
```

I DTO saranno posizionati principalmente in:

```text
BookingSystem.Application/DTOs
```

---

# Dependency Injection

La logica dell'applicazione non deve essere contenuta interamente nei controller.

Creare dei servizi applicativi, per esempio:

```text
BookingService
ServiceService
AuthService
```

e registrarli tramite il sistema di Dependency Injection di ASP.NET Core.

Le registrazioni relative all'infrastruttura potranno essere centralizzate in:

```text
BookingSystem.Infrastructure/DependencyInjection.cs
```

---

# Gestione errori

L'API deve restituire status code HTTP appropriati.

Esempi:

```text
200 OK

201 Created

400 Bad Request

401 Unauthorized

403 Forbidden

404 Not Found

409 Conflict
```

Se un utente prova a creare una prenotazione sovrapposta a una già esistente, l'API dovrebbe restituire:

```http
409 Conflict
```

---

# Autenticazione

Come seconda fase del progetto, implementare autenticazione tramite **JWT**.

Devono essere disponibili almeno:

```http
POST /api/auth/register

POST /api/auth/login
```

Gli utenti autenticati devono poter gestire esclusivamente le proprie prenotazioni.

Gli utenti con ruolo Admin devono poter accedere agli endpoint amministrativi.

---

# Swagger

Configurare Swagger/OpenAPI per poter testare l'intera API dal browser.

Swagger deve permettere di testare anche gli endpoint protetti tramite JWT.

---

# Testing

Implementare alcuni test automatici utilizzando **xUnit**.

Scrivere almeno test per:

- creazione di una prenotazione valida;
- tentativo di prenotazione sovrapposta;
- servizio inesistente;
- prenotazione inesistente.

---

# Bonus

Una volta completati i requisiti principali è possibile aggiungere:

- Docker;
- Docker Compose;
- logging;
- pagination;
- filtering;
- ricerca servizi;
- cancellazione logica;
- refresh token;
- FluentValidation;
- frontend React, Angular, Vue o Astro;
- dashboard amministrativa.

---

# Risultato finale

Al termine del progetto dovrà essere possibile eseguire un flusso simile:

```text
Registrazione utente
        ↓
Login
        ↓
Visualizzazione servizi
        ↓
Scelta servizio
        ↓
Controllo disponibilità
        ↓
Creazione prenotazione
        ↓
Salvataggio tramite EF Core
        ↓
Visualizzazione proprie prenotazioni
```

## Stack consigliato

```text
.NET
C#
ASP.NET Core Web API
Entity Framework Core
PostgreSQL
LINQ
JWT
Swagger
xUnit
Docker
```

Lo scopo dell'esercizio non è soltanto creare un CRUD, ma realizzare una piccola applicazione backend che includa **persistenza dei dati, relazioni, query, regole di business, autenticazione e gestione degli errori**, mantenendo una separazione chiara tra i diversi layer dell'applicazione.