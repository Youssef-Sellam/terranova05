# Sistema di Prenotazione dei Laboratori Scolastici

Progetto PCTO 2025/26 — Classe 5FI — Istituto Terranova  
Tarbacea · Wu · Sellam

---

## Descrizione

Sistema web per la prenotazione dei laboratori scolastici. L'accesso avviene tramite il registro elettronico dell'istituto — **nessuna registrazione locale**. Il ruolo dell'utente (studente o professore) viene rilevato direttamente dal servizio scolastico terzo al momento del login. Gli studenti prenotano rispettando i vincoli orari di ogni laboratorio. I professori possono annullare prenotazioni e inviare messaggi agli studenti.

---

## File del progetto

```
progetto_terranova/
├── index.html            Home page del sistema con link a login/orari
├── login.html            Pagina di accesso tramite servizio scolastico terzo
├── orario.html           Visualizzazione degli orari dei laboratori
├── prenotazione.html     la pagina dove prenotare i laboratori
├── dashboard-professore.html    la page del professore dopo essere acceduto
├── dashboard-studente.html      la page dello studente dopo essere acceduto
```

> `register.html`gli utenti non si registrano localmente. Il profilo viene creato/aggiornato automaticamente al primo login tramite l'API del registro scolastico.

---

## Architettura del sistema

```
          Utente (Studente / Professore)
                      |
                 Browser Web
                      |
               Frontend Web
            (HTML / CSS / JS)
            login.html
                      |
               Backend / API
                   (PHP )
                      |
          ┌───────────┴───────────┐
          │                       │
   Servizio Scolastico     Database MySQL
   Terzo (registro                |
   elettronico)                   │
   ClasseViva / Axios     ┌───────┴──────────┬────────────┬──────────┐
   Nuvola / altro…      users           sessioni   laboratori  prenotazioni
                        (cache profilo)  (token)              comments
```

Il **backend PHP** fa da adapter: riceve le credenziali dal browser, le inoltra all'API del registro scolastico dell'istituto, normalizza la risposta e crea la sessione locale.

---

## Schema del database

```
users                              sessioni
──────────────────────────         ──────────────────────────
id_utente      (PK)                id_sessione    (PK)
id_esterno     (UNIQUE)            id_utente      (FK)
nome                               token_locale   (UNIQUE)
cognome                            token_servizio
email          (UNIQUE)            ip
ruolo                              user_agent
classe                             creata_il
attivo                             scade_il
creato_il                          revocata
aggiornato_il

laboratori                         prenotazioni
──────────────────────────         ──────────────────────────
id_lab              (PK)           id_prenotazione (PK)
nome                               id_utente       (FK)
descrizione                        id_lab          (FK)
capienza                           data
posizione                          ora_inizio
attivo                             ora_fine
max_ore_giorno                     stato
max_ore_settimana                  note
anticipo_min_giorni                creata_il

comments
──────────────────────────
id_commento    (PK)
id_prenotazione (FK)
id_autore       (FK)
id_destinatario (FK, nullable)
contenuto
data_messaggio
```

### Nota: nessuna password locale

La tabella `users` non contiene `password_hash`. Le credenziali non transitano mai nel nostro database — restano tra il browser e il registro scolastico. Il campo `id_esterno` è l'identificativo restituito dall'API terza (può essere un codice fiscale, un UUID, ecc. — dipende dal provider).

---

## Flusso di autenticazione

```
Utente
  |
  | inserisce username + password (del registro scolastico)
  v
Frontend (login.html)
  |
  | POST /api/auth/login  { username, password }
  v
Backend PHP (adapter)
  |
  | chiama l'API del registro scolastico dell'istituto
  | (endpoint diverso per ogni scuola: ClasseViva, Axios, ecc.)
  |
  | [credenziali errate] → { ok: false, errore: "..." }
  |
  | [credenziali corrette]
  | ↳ risposta normalizzata:
  |   {
  |     ok: true,
  |     token:    "...",          ← token del servizio scolastico
  |     scade_il: "2026-...",
  |     utente: {
  |       id_esterno: "RSSMRC85A01F205X",
  |       nome:       "Marco",
  |       cognome:    "Rossi",
  |       email:      "marco.rossi@scuola.it",
  |       ruolo:      "professore",   ← "studente" | "professore"
  |       classe:     null,
  |       materie:    ["Informatica", "Sistemi e Reti"]
  |     }
  |   }
  |
  | Upsert su tabella users (crea o aggiorna il profilo locale)
  | INSERT INTO sessioni (token_locale, token_servizio, scade_il…)
  | Restituisce token_locale al browser (cookie HttpOnly)
  v
Dashboard (routing in base a utente.ruolo)
  ↳ ruolo "professore" → dashboard-prof.html
  ↳ ruolo "studente"   → dashboard-studente.html
```

### Perché un adapter PHP?

Ogni istituto usa un registro diverso con API proprietarie e formati incompatibili. Il backend PHP centralizza la logica di integrazione: basta cambiare l'implementazione interna dell'adapter senza toccare il frontend o il database.

---

## Diagramma di sequenza — Login

```
Utente        login.html        PHP Backend     Servizio Scolastico     MySQL
  |               |                  |                  |                  |
  | username +    |                  |                  |                  |
  | password      |                  |                  |                  |
  |-------------->|                  |                  |                  |
  |               | POST /api/auth/  |                  |                  |
  |               | login            |                  |                  |
  |               |----------------->|                  |                  |
  |               |                  | POST credenziali |                  |
  |               |                  |----------------->|                  |
  |               |                  | profilo + token  |                  |
  |               |                  |<-----------------|                  |
  |               |                  |                  |                  |
  |               |   [errore auth]  |                  |                  |
  |  "Credenziali |<-----------------|                  |                  |
  |   non valide" |                  |                  |                  |
  |               |                  |                  |                  |
  |               |   [ok]           |                  |                  |
  |               |                  | UPSERT users     |                  |
  |               |                  | INSERT sessioni  |                  |
  |               |                  |---------------------------------------->|
  |               |                  |<----------------------------------------|
  |               | token_locale     |                  |                  |
  |               | (cookie HttpOnly)|                  |                  |
  |<--------------|                  |                  |                  |
  | redirect      |                  |                  |                  |
  | dashboard     |                  |                  |                  |
```

---

## Diagramma di sequenza — Messaggio del professore

```
Professore      Frontend         PHP Backend        MySQL
    |               |                  |               |
    | scrive        |                  |               |
    | messaggio     |                  |               |
    |-------------->|                  |               |
    |               | POST             |               |
    |               | /commenti.php    |               |
    |               |----------------->|               |
    |               |                  | INSERT        |
    |               |                  | comments      |
    |               |                  |-------------->|
    |               |                  | OK            |
    |               |                  |<--------------|
    | messaggio     |                  |               |
    | inviato       |<-----------------|               |
    |<--------------|                  |               |
```

---

## Flusso funzionale del sistema

```
Login (tramite registro scolastico)
  |
  v
Caricamento orari laboratori (API)
  |
  v
Scelta laboratorio
  |
  v
Calendario con disponibilita' (da API)
  |
  v
Selezione data e orario
  |
  v
Prenotazione salvata (stato = attiva)
  |
  v
Messaggi professore ↔ studente
```

---

**Credenziali di test (simulazione servizio scolastico):**

```
  marco.rossi    Temp1234!   professore
  luca.ferrari   Temp5678!   studente
  sara.esposito  Temp9012!   studente
```

---

*PCTO 2025/26 — Classe 5FI — Istituto Terranova*
