
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


```mermaid
erDiagram
    direction LR

    UTENTI ||--o{ SESSIONI : possiede
    UTENTI ||--o{ PRENOTAZIONI : effettua
    LABORATORI ||--o{ PRENOTAZIONI : ospita

    PRENOTAZIONI ||--o{ COMMENTI : contiene

    UTENTI ||--o{ COMMENTI : autore
    UTENTI ||--o{ COMMENTI : destinatario

    UTENTI {
        int id_utente PK
        string nome
        string cognome
        string email
        string ruolo
    }

    SESSIONI {
        int id_sessione PK
        string token
        datetime scadenza
        int id_utente FK
    }

    LABORATORI {
        int id_lab PK
        string nome
        int capienza
    }

    PRENOTAZIONI {
        int id_prenotazione PK
        date data
        time ora_inizio
        time ora_fine
        int id_utente FK
        int id_lab FK
    }

    COMMENTI {
        int id_commento PK
        text contenuto
        int id_prenotazione FK
        int id_autore FK
        int id_destinatario FK
    }
