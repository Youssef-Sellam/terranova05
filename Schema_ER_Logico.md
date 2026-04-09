
__**SCHEMA_LOGICO**__


```
UTENTI(**id_utente**, id_esterno, nome, cognome, email, ruolo, classe, attivo, creato_il, aggiornato_il)

SESSIONI(**id_sessione**, token_locale, token_servizio, ip, user_agent, creata_il, scade_il, revocata, id_utente)

LABORATORI(**id_lab**, nome, descrizione, capienza, posizione, attivo, max_ore_giorno, max_ore_settimana, anticipo_min_giorni)

PRENOTAZIONI(**id_prenotazione**, data, ora_inizio, ora_fine, stato, note, creata_il, id_utente, id_lab)

COMMENTS(**id_commento**, contenuto, data_messaggio, id_prenotazione, id_autore, id_destinatario)
```
----



**DIAGRAMMA E/R**

### Schema E/R - Sistema di Prenotazione Laboratori (Notazione a Corvo)

# Schema E/R - Sistema di Prenotazione Laboratori

```mermaid
erDiagram
    direction LR

    UTENTI ||--o{ SESSIONI : possiede
    UTENTI ||--o{ PRENOTAZIONI : effettua
    LABORATORI ||--o{ PRENOTAZIONI : viene_prenotato_da
    PRENOTAZIONI ||--o{ COMMENTI : riceve
    UTENTI ||--o{ COMMENTI : scrive
    UTENTI ||--o{ COMMENTI : riceve

    UTENTI {
        int id_utente PK
        string id_esterno
        string nome
        string cognome
        string email
        string ruolo
        string classe
        boolean attivo
        datetime creato_il
        datetime aggiornato_il
    }

    SESSIONI {
        int id_sessione PK
        string token_locale
        string token_servizio
        string ip
        string user_agent
        datetime creata_il
        datetime scade_il
        boolean revocata
        int id_utente FK
    }

    LABORATORI {
        int id_lab PK
        string nome
        string descrizione
        int capienza
        string posizione
        boolean attivo
        int max_ore_giorno
        int max_ore_settimana
        int anticipo_min_giorni
    }

    PRENOTAZIONI {
        int id_prenotazione PK
        date data
        time ora_inizio
        time ora_fine
        string stato
        string note
        datetime creata_il
        int id_utente FK
        int id_lab FK
    }

    COMMENTI {
        int id_commento PK
        text contenuto
        datetime data_messaggio
        int id_prenotazione FK
        int id_autore FK
        int id_destinatario FK
    }