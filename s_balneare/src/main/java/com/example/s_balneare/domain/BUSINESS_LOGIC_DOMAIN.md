# BUSINESS RULES

## Overview

Questo documento definisce le Business Rules delle classi presenti nel package **domain**.

Tutte le regole vanno gestite a livello Domain o Application. Il livello Infrastructure NON deve contenere NESSUNA BUSINESS LOGIC.

---

## Package beach

### Classe Beach.java

Definisce un'istanza di una spiaggia. Nel DDD, viene considerata una **Aggregate Root**, poiché gestisce l'intero ciclo di vita di una spiaggia e le sue entità/value object associati.

Una spiaggia, appena creata, deve avere:

> - `id` (intero; può essere `null` se deve essere salvato nel DB, poi messo uguale all'ID assegnato dal DB),
> - `addressId` (obbligatorio, non modificabile direttamente),
> - `seasons` (lista di stagioni; possono essere aggiunte o rimosse tramite metodi dedicati),
> - `zones` (lista di zone; possono essere aggiunte, rinominate o rimosse solo se non sono in uso in nessuna stagione),
> - `ownerId` (obbligatorio, può essere modificato),
> - `beachGeneral` (obbligatorio, modificabile tramite i suoi metodi),
> - `beachInventory` (opzionale alla creazione, modificabile tramite i suoi metodi),
> - `beachServices` (opzionale alla creazione, modificabile tramite i suoi metodi),
> - `parking` (opzionale alla creazione, modificabile tramite i suoi metodi),
> - `extraInfo` (informazioni testuali aggiuntive),
> - `active` (stato di attivazione della spiaggia),
> - `closed` (stato di chiusura della spiaggia).

BUSINESS LOGIC:

> - Una spiaggia può essere attivata (`active = true`) solo se sono stati definiti `beachInventory`, `beachServices`, `parking` e se esistono almeno una stagione (`Season`) e una zona (`Zone`).
> - Se una di queste condizioni viene a mancare dopo l'attivazione, la spiaggia viene disattivata automaticamente.
> - La spiaggia può essere chiusa definitivamente tramite `closeBeach()`, impostando `closed = true` e `active = false`.
> - Le zone possono essere rimosse solo se non sono associate a nessuna stagione esistente.

### Classe BeachGeneral.java

Contiene gli attributi generali di una spiaggia. Nel DDD, è considerata un **Value Object**.

Ogni oggetto creato contiene:

> - `name` (nome della spiaggia; non può essere vuoto o superare i 100 caratteri),
> - `description` (breve descrizione dello stabilimento; max 512 caratteri),
> - `phoneNumber` (numero di telefono, con validazione: deve iniziare con '+' seguito da cifre, senza spazi).

### Classe BeachInventory.java

Contiene l'inventario degli oggetti extra disponibili in una spiaggia. Nel DDD, è considerata un **Value Object**.

Ogni oggetto creato contiene:

> - `countExtraSdraio` (numero di sdraio extra*; >= 0),
> - `countExtraLettini` (numero di lettini extra*; >= 0),
> - `countExtraSedie` (numero di sedie extra*; >= 0),
> - `countCamerini` (numero di camerini; >= 0).

\* Con "extra" si intendono gli oggetti non inclusi nel noleggio di ombrelloni/tende.

### Classe BeachServices.java

Definisce i servizi offerti da una spiaggia. Nel DDD, è considerata un **Value Object**.

Ogni oggetto creato contiene attributi booleani che indicano la disponibilità di:

> - `bathrooms` (WC),
> - `showers` (docce),
> - `pool` (piscina),
> - `bar` (bar),
> - `restaurant` (ristorante),
> - `wifi` (Wi-Fi),
> - `volleyballField` (campo da beach volley).

### Classe Parking.java

Descrive la struttura del parcheggio di una spiaggia. Nel DDD, è considerata un **Value Object**.

Ogni oggetto creato contiene:

> - `nAutoPark` (numero di posti auto; >= 0),
> - `nMotoPark` (numero di posti moto; >= 0),
> - `nElectricPark` (numero di posti per auto elettriche; >= 0),
> - `CCTV` (presenza di videosorveglianza).

### Classe Pricing.java

Definisce il listino prezzi per gli extra di una stagione. Nel DDD, è considerata un **Value Object**.

Ogni oggetto creato contiene:

> - `id` (identificativo del listino; `null` se nuovo),
> - `priceLettino` (prezzo di un lettino extra; >= 0),
> - `priceSdraio` (prezzo di una sdraio extra; >= 0),
> - `priceSedia` (prezzo di una sedia extra; >= 0),
> - `priceParking` (prezzo di un posto parcheggio, unico per tutti i tipi di veicolo; >= 0),
> - `priceCamerino` (prezzo di un camerino; >= 0).

\* Tutti i prezzi sono espressi in `double`.

### Classe Season.java

Definisce una stagione specifica per una spiaggia. Nel DDD, è considerata una **Entity**.

Ogni oggetto creato contiene:

> - `name` (nome della stagione; max 50 caratteri),
> - `startDate` (data di inizio),
> - `endDate` (data di fine),
> - `pricing` (riferimento all'oggetto `Pricing` con le tariffe della stagione),
> - `zoneTariffs` (lista di `ZoneTariff` che definiscono i prezzi per ogni zona in quella stagione).

BUSINESS LOGIC:

> - `startDate` deve essere precedente a `endDate`.
> - Le stagioni non possono sovrapporsi temporalmente per la stessa spiaggia.
> - Ogni stagione deve avere almeno una `ZoneTariff` definita.
> - Include metodi per verificare se una data rientra nella stagione (`includes(LocalDate date)`).

### Classe ZoneTariff.java

Definisce le tariffe per gli spot (ombrelloni/tende) in una specifica zona per una stagione. Nel DDD, è considerata un **Value Object**.

Ogni oggetto creato contiene:

> - `zoneName` (riferimento al nome della `Zone`),
> - `priceOmbrellone` (prezzo di un ombrellone in quella zona; >= 0),
> - `priceTenda` (prezzo di una tenda in quella zona; >= 0).

---

## Package Booking

### Classe Booking.java

Definisce una prenotazione effettuata da un cliente per una specifica spiaggia e data. Nel DDD, viene considerata una **Aggregate Root**, poiché gestisce l'intero ciclo di vita di una prenotazione e le sue entità/value object associati.

Una prenotazione, appena creata, deve avere:

> - `id` (intero; `null` se nuovo),
> - `beachId` (ID della spiaggia a cui si riferisce la prenotazione),
> - `customerId` (ID del cliente che ha effettuato la prenotazione),
> - `callerName` (nome di chi prenota, utile se diverso dal customer o per note rapide),
> - `callerPhone` (numero di telefono di chi prenota, validato),
> - `date` (data della prenotazione),
> - `spotIds` (lista degli ID degli spot prenotati; non può essere vuota),
> - `extraSdraio` (numero di sdraio extra prenotate; >= 0),
> - `extraLettini` (numero di lettini extra prenotati; >= 0),
> - `extraSedie` (numero di sedie extra prenotate; >= 0),
> - `camerini` (numero di camerini prenotati; >= 0),
> - `parking` (oggetto `BookingParking` che definisce i posti auto prenotati per tipo; non può essere null),
> - `totalPrice` (prezzo totale della prenotazione),
> - `status` (stato iniziale della prenotazione, di default `PENDING`).

BUSINESS LOGIC:

> - `beachId` e `customerId` devono essere validi (maggiori di 0).
> - `date` non può essere null.
> - `spotIds` non può essere null o vuota, e ogni `spotId` deve essere valido (maggiore di 0).
> - Tutte le quantità extra e i posti auto devono essere >= 0.
> - `callerPhone` segue la stessa validazione di `BeachGeneral`.
> - Lo stato di una prenotazione può essere modificato solo se rispetta determinate condizioni:
>   - `confirmBooking()`: solo se lo stato è `PENDING`.
>   - `rejectBooking()`: solo se lo stato è `PENDING`.
>   - `cancelBooking()`: solo se lo stato è `PENDING` o `CONFIRMED`.
> - L'aggiunta di quantità extra o posti auto tramite `update...` o `updateSpotsAndParking` è consentita solo se lo stato è `PENDING` o `CONFIRMED`. Se lo stato era `CONFIRMED`, torna a `PENDING` dopo la modifica per richiedere nuova approvazione.
> - La quantità aggiunta deve essere maggiore di 0.

### Classe BookingParking.java

Definisce i posti parcheggio prenotati. Nel DDD, è considerata un **Value Object** e rappresenta i dettagli del parcheggio all'interno di una prenotazione.

Ogni oggetto creato contiene:

> - `autoPark` (numero di posti auto prenotati; >= 0),
> - `motoPark` (numero di posti moto prenotati; >= 0),
> - `electricPark` (numero di posti per auto elettriche prenotati; >= 0).

### Enum BookingStatus.java

Definisce i possibili stati di una prenotazione. Nel DDD, è considerata un **Value Object**.

Gli stati possibili sono:

> - `PENDING` (in attesa di conferma),
> - `CONFIRMED` (confermata),
> - `REJECTED` (rifiutata),
> - `CANCELLED` (annullata).

### Classe PriceCalculator.java

È un **Domain Service** che calcola il prezzo totale di una prenotazione. Essendo un servizio, non ha stato e contiene solo logica.

Il metodo `calculateTotal` richiede:

> - `booking` (l'oggetto `Booking` per cui calcolare il prezzo).
> - `beach` (l'oggetto `Beach` completo, che contiene le informazioni su stagioni, zone e tariffe).

BUSINESS LOGIC:

> 1. **Trova la stagione attiva**: identifica la stagione corretta in base alla data della prenotazione. Se non esiste una stagione attiva per quella data, lancia un'eccezione.
> 2. **Calcola il costo degli extra**: somma i costi di sdraio, lettini, sedie e camerini extra, usando il listino prezzi (`Pricing`) della stagione attiva.
> 3. **Calcola il costo del parcheggio**: somma il numero totale di veicoli prenotati e li moltiplica per il prezzo fisso del parcheggio (`priceParking`) definito nel listino.
> 4. **Calcola il costo degli spot**:
>    - Per ogni `spotId` nella prenotazione, identifica la `Zone` e lo `SpotType` corrispondenti.
>    - Se uno `spotId` non esiste, lancia un'eccezione.
>    - Trova la tariffa di zona (`ZoneTariff`) per quella `Zone` nella stagione attiva.
>    - Aggiunge al totale il prezzo corretto in base al tipo di spot.
> 5. **Restituisce il totale**: ritorna il prezzo finale calcolato.

---

## Package common

### Classe Address.java

Definisce un indirizzo geografico. Nel DDD, è considerata un **Value Object**.

Ogni oggetto creato contiene:

> - `id` (identificativo dell'indirizzo; `null` o 0 se nuovo),
> - `street` (nome della via),
> - `streetNumber` (numero civico),
> - `city` (città),
> - `zipCode` (codice postale),
> - `country` (nazione).

BUSINESS LOGIC:

> - Tutti i campi stringa (`street`, `streetNumber`, `city`, `zipCode`, `country`) non possono essere nulli o vuoti.
> - Le lunghezze massime per i campi sono:
>   - `street`: 255 caratteri
>   - `streetNumber`: 10 caratteri
>   - `city`: 100 caratteri
>   - `zipCode`: 20 caratteri
>   - `country`: 100 caratteri

---

### Enum ObjectStatus.java

Definisce i possibili stati di un oggetto generico (es. parcheggi, spot). Nel DDD, è considerata un **Value Object**.

Gli stati possibili sono:

> - `PENDING` (in attesa),
> - `OCCUPIED` (occupato),
> - `FREE` (libero).

### Interfaccia TransactionContext.java

È un'interfaccia "marker" o "token" utilizzata per rappresentare il contesto di una transazione. Nell’Hexagonal Architecture, facilita l'isolamento del dominio dall'infrastruttura, permettendo ai servizi di dominio di richiedere un contesto transazionale senza dipendere direttamente da implementazioni specifiche (es. JDBC `Connection`).

BUSINESS LOGIC:

> - Non contiene metodi o attributi. La sua presenza come parametro indica che l'operazione deve essere eseguita all'interno di una transazione.

---

## Package layout

### Classe Spot.java

Definisce un singolo spot (ombrellone o tenda) all'interno di una zona della spiaggia. Nel DDD, è considerata una **Entity**.

Ogni oggetto creato contiene:

> - `id` (identificativo dello spot; `null` se nuovo),
> - `type` (tipo di spot: `UMBRELLA` o `TENT`),
> - `row` (riga in cui si trova lo spot; >= 0),
> - `column` (colonna in cui si trova lo spot; >= 0).

BUSINESS LOGIC:

> - `type` non può essere `null`.
> - `row` e `column` non possono essere valori negativi.

### Enum SpotType.java

Definisce i possibili tipi di spot disponibili in una spiaggia. Nel DDD, è considerata un **Value Object**.

I tipi possibili sono:

> - `UMBRELLA` (ombrellone),
> - `TENT` (tenda).

### Classe Zone.java

Definisce una zona specifica all'interno della spiaggia, che raggruppa un insieme di spot. Nel DDD, è considerata una **Entity**.

Ogni oggetto creato contiene:

> - `name` (nome della zona; max 50 caratteri),
> - `spots` (lista degli `Spot` che appartengono a questa zona).

BUSINESS LOGIC:

> - `name` non può essere nullo o vuoto e non può superare i 50 caratteri.
> - La lista `spots` viene copiata (immutabilità) se non è nulla.

---

## Package moderation

### Classe Ban.java

Rappresenta un'azione di ban emessa da un amministratore. Nel DDD, è considerata una **Entity**.

Ogni oggetto creato contiene:

> - `id` (identificativo del ban),
> - `bannedId` (ID dell'utente bannato),
> - `banType` (tipo di ban: `BEACH` o `APPLICATION`),
> - `bannedFromBeachId` (ID della spiaggia da cui l'utente è stato bannato, se `banType` è `BEACH`),
> - `adminId` (ID dell'amministratore che ha emesso il ban),
> - `reason` (motivazione del ban),
> - `createdAt` (data e ora di creazione del ban, di tipo `Instant`).

BUSINESS LOGIC:

> - `bannedId` e `adminId` devono essere validi e maggiori di 0.
> - `reason` non può essere vuota.
> - `createdAt` non può essere `null`.
> - Se `banType` è `BEACH`, `bannedFromBeachId` deve essere specificato (> 0).
> - Se `banType` è `APPLICATION`, `bannedFromBeachId` deve essere `null`.

### Enum BanType.java

Definisce il tipo di ban. Nel DDD, è considerata un **Value Object**.

I tipi possibili sono:

> - `BEACH` (ban da una specifica spiaggia),
> - `APPLICATION` (ban dall'intera applicazione).

### Classe Report.java

Rappresenta una segnalazione (report) inviata da un utente. Nel DDD, è considerata una **Aggregate Root**.

Ogni oggetto creato contiene:

> - `id` (identificativo del report),
> - `reporterId` (ID dell'utente che ha inviato la segnalazione),
> - `reportedId` (ID dell'utente o della spiaggia segnalata),
> - `reportedType` (tipo di entità segnalata: `USER` o `BEACH`),
> - `description` (descrizione della segnalazione),
> - `createdAt` (data e ora di creazione, di tipo `Instant`),
> - `status` (stato del report, di default `PENDING`),
> - `bookingId` (ID della prenotazione a cui si riferisce la segnalazione; obbligatorio).

BUSINESS LOGIC:

> - `reporterId` e `reportedId` devono essere validi (> 0) e non possono essere uguali.
> - `description` non può essere vuota e ha una lunghezza massima di 1024 caratteri.
> - `createdAt` non può essere una data futura.
> - `bookingId` deve essere sempre presente.
> - Lo stato di un report può essere modificato (`approve()` o `reject()`) solo se è `PENDING`.

### Enum ReportStatus.java

Definisce i possibili stati di un report. Nel DDD, è considerata un **Value Object**.

Gli stati possibili sono:

> - `PENDING` (in attesa di revisione),
> - `APPROVED` (approvato),
> - `REJECTED` (rifiutato).

### Enum ReportTargetType.java

Definisce il tipo di entità che può essere segnalata. Nel DDD, è considerata un **Value Object**.

I tipi possibili sono:

> - `USER` (un utente),
> - `BEACH` (una spiaggia).

---

## Package review

### Classe Review.java

Rappresenta una recensione lasciata da un cliente per una specifica spiaggia. Nel DDD, è considerata una **Entity**.

Ogni oggetto creato contiene:

> - `id` (identificativo della recensione),
> - `beachId` (ID della spiaggia recensita),
> - `customerId` (ID del cliente che ha lasciato la recensione),
> - `rating` (punteggio da 1 a 5),
> - `comment` (commento testuale; max 1024 caratteri),
> - `createdAt` (data e ora di creazione, di tipo `Instant`).

BUSINESS LOGIC:

> - `beachId` e `customerId` devono essere validi (> 0).
> - `rating` deve essere un valore intero compreso tra 1 e 5.
> - `comment` non può essere vuoto e ha una lunghezza massima di 1024 caratteri. Viene rimosso lo spazio bianco iniziale e finale.
> - `createdAt` non può essere `null`.

---

## Package user

### Classe User.java

È una classe astratta che rappresenta un utente generico del sistema. Nel DDD, è considerata una **Entity**.

Ogni utente contiene:

> - `id` (identificativo dell'utente),
> - `email` (indirizzo email dell'utente),
> - `username` (nome utente),
> - `name` (nome anagrafico),
> - `surname` (cognome anagrafico).

BUSINESS LOGIC:

> - `email` non può essere nulla, vuota, deve avere una lunghezza tra 6 e 80 caratteri, contenere '@' e non contenere '..'.
> - `username` non può essere nullo, vuoto e non può superare i 50 caratteri.
> - `name` non può essere nullo, vuoto e non può superare i 100 caratteri.
> - `surname` non può essere nullo, vuoto e non può superare i 50 caratteri.
> - Contiene un metodo astratto `getRole()` che restituisce il ruolo specifico dell'utente.
> - Contiene un metodo astratto `isOTP()` che indica se l'utente deve cambiare la password al prossimo login.
> - Fornisce metodi per aggiornare email, nome, cognome e username rispettando le validazioni.

### Classe Admin.java

Rappresenta un utente con privilegi di amministratore. Estende `User`. Nel DDD, è considerata una **Entity**.

Ogni amministratore contiene, oltre agli attributi di `User`:

> - `OTP` (boolean che indica se l'amministratore deve cambiare la password al prossimo login).

BUSINESS LOGIC:

> - Il ruolo restituito da `getRole()` è `ADMIN`.
> - `isOTP()` restituisce il valore dell'attributo `OTP`.

### Classe Customer.java

Rappresenta un cliente del sistema. Estende `User`. Nel DDD, è considerata una **Entity**.

Ogni cliente contiene, oltre agli attributi di `User`:

> - `phoneNumber` (numero di telefono del cliente),
> - `addressId` (ID dell'indirizzo associato al cliente),
> - `active` (boolean che indica se l'account del cliente è attivo).

BUSINESS LOGIC:

> - Il ruolo restituito da `getRole()` è `CUSTOMER`.
> - `isOTP()` restituisce sempre `false` per un cliente.
> - `phoneNumber` non può essere nullo, vuoto, non può superare i 50 caratteri e deve iniziare con '+' seguito da cifre, senza spazi.
> - `addressId` deve essere presente.
> - Un cliente può chiudere il proprio account tramite `closeAccount()`, impostando `active` a `false`.

### Classe Owner.java

Rappresenta il proprietario di una spiaggia. Estende `User`. Nel DDD, è considerata una **Entity**.

Ogni proprietario contiene, oltre agli attributi di `User`:

> - `active` (boolean che indica se l'account del proprietario è attivo),
> - `OTP` (boolean che indica se il proprietario deve cambiare la password al prossimo login).

BUSINESS LOGIC:

> - Il ruolo restituito da `getRole()` è `OWNER`.
> - `isOTP()` restituisce il valore dell'attributo `OTP`.
> - Un proprietario può chiudere il proprio account tramite `closeAccount()`, impostando `active` a `false`.
> - Può aggiornare il proprio stato `OTP` tramite `updateOTP(boolean OTP)`.

### Enum Role.java

Definisce i possibili ruoli che un utente può avere nel sistema. Nel DDD, è considerata un **Value Object**.

I ruoli possibili sono:

> - `CUSTOMER` (cliente),
> - `OWNER` (proprietario di spiaggia),
> - `ADMIN` (amministratore del sistema).