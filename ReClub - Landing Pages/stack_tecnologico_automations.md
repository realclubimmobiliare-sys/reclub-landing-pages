# ECOSISTEMA TECNOLOGICO E WORKFLOW DI AUTOMAZIONE RECLUB
### *Architettura No-Code/Low-Code per la Qualificazione e Vendita di Dati Commerciali*

Il presente documento descrive in dettaglio l'infrastruttura tecnologica, le integrazioni software e i flussi di automazione che governano la piattaforma **ReClub**. L'architettura è progettata per operare in modalità completamente asincrona, riducendo al minimo la gestione manuale dei dati e garantendo la massima trasparenza e conformità legale in ogni fase.

---

## 📌 1. ARCHITETTURA DI SISTEMA E SOFTWARE STACK

L'infrastruttura si basa su un ecosistema integrato di strumenti SaaS leader di mercato, connessi tramite l'automation engine **Make.com**:

```
[ Frontend: SOFTR ] <---> [ Automation: MAKE.COM ] <---> [ Database: AIRTABLE ]
                                  |
            +---------------------+---------------------+
            |                     |                     |
     [ Brevo: CRM ]       [ DocuSign: Mandati ]   [ Wise/Runa: Payouts ]
```

1. **Airtable (Database Centrale)**: Funge da "Single Source of Truth" (unica fonte di verità). Conserva i dati degli Ambassador, delle Agenzie Partner, dei Lead e lo storico delle transazioni commerciali.
2. **Softr (Frontend Web App / PWA)**: Interfaccia utente personalizzata e responsive. Gestisce due portali ad accesso riservato:
   - **Portale Ambassador (B2C/B2B)**: Registrazione, tracciamento delle segnalazioni inviate, stato dell'attività di scouting, monitoraggio e statistiche di payout.
   - **Portale Agenzie Partner (B2B)**: Visualizzazione dei Lead Qualificati assegnati, caricamento dei mandati di esclusiva firmati e rendicontazione dei corrispettivi.
3. **Make.com (Motore di Integrazione)**: Gestisce tutti i flussi di lavoro, la sincronizzazione dei dati in tempo reale e i trigger di pagamento.
4. **Brevo (CRM & Transazionale)**: Gestione delle campagne email e invio automatico di notifiche transazionali (SMS ed Email) a segnalatori e agenzie.
5. **DocuSign**: Integrazione per la firma digitale e il caricamento dei contratti partner e dei mandati di vendita in esclusiva.
6. **Wise API & Runa**: Gestione automatizzata dei payout. Runa eroga istantaneamente i Buoni Amazon (Premio Start), Wise dispone i bonifici SEPA per il Saldo dell'attività di scouting.

---

## ⚙️ 2. I FLUSSI DI AUTOMAZIONE PRINCIPALI (WORKFLOW)

### WORKFLOW 1: Ricezione e Qualificazione della Notizia Commerciale
1. **Trigger**: L'utente (invitato dall'Ambassador tramite link di referral) inserisce i propri dati e quelli dell'immobile nel modulo pubblico Softr.
2. **Azione 1**: Softr invia i dati ad Airtable tramite Make.com. Viene creato un record nella tabella `Notizie Commerciali` con lo stato `Screening Pendente`. Il record viene associato automaticamente all'ID dell'Ambassador referente.
3. **Azione 2**: Make.com invia una notifica istantanea su Slack/WhatsApp all'operatore ReClub per segnalare la presenza di una nuova notizia da verificare.
4. **Azione 3**: L'operatore contatta il proprietario ed esegue lo screening (SOP 1). Se i requisiti sono soddisfatti, imposta lo stato su `Qualificato`.

### WORKFLOW 2: Assegnazione e Vendita del Lead all'Agenzia Partner
1. **Trigger**: Lo stato del lead su Airtable passa a `Qualificato`.
2. **Azione 1**: Make.com seleziona l'Agenzia Partner di riferimento per territorio e fascia e le assegna il lead in esclusiva nel portale Softr.
3. **Azione 2**: Viene inviata un'email transazionale all'Agenzia con i dettagli preliminari del lead (indirizzo e caratteristiche dell'immobile) e la richiesta di confermare l'acquisizione effettuando il pagamento dell'Acconto di Screening.
4. **Azione 3**: L'agenzia effettua il pagamento dell'Acconto di €250,00 + IVA. Make.com rileva la transazione (via Stripe webhook) e sblocca i dati di contatto completi (nome e telefono del venditore) all'agenzia.

### WORKFLOW 3: Acquisizione Mandato e Liquidazione Corrispettivo
L'acquisizione del mandato in esclusiva sblocca il corrispettivo per ReClub e il saldo principale per l'Ambassador. Il flusso gestisce in sicurezza i controlli fiscali prima del bonifico.

```
[ Agenzia carica Mandato ] ---> [ Make: Verifica Firme ] ---> [ Generazione Buono Amazon €100 ]
                                                                      |
                                                              [ Airtable: Stato "Esclusiva Firmata" ]
                                                                      |
                                                              [ Fatturazione B2B a Agenzia ]
```

1. **Trigger 1 (Acquisizione Mandato)**: L'Agenzia comunica l'acquisizione del mandato di vendita in esclusiva caricando il file firmato sul portale Softr.
2. **Azione 1 (Fatturazione ReClub)**: Airtable passa la pratica a `In attesa di Corrispettivo`. Make.com invia in automatico la fattura elettronica per il corrispettivo a scaglioni (Art. 4 del contratto B2B, detratto l'acconto di €250) all'agenzia partner.
3. **Azione 2 (Premio Start)**: Make.com interroga l'API di Runa e genera un Buono Regalo Amazon da €100. Il buono viene inviato all'Ambassador tramite email.
4. **Azione 3 (Saldo Attività)**: Make.com crea un record in `Attività di Scouting Completate` e calcola l'importo dovuto all'Ambassador.
   - Per i privati (Canale A): Genera in automatico il modulo di ricevuta per prestazione d'opera occasionale (con ritenuta al 20%) e lo invia per la firma.
   - Per le P.IVA (Canale B): Invia richiesta di emissione fattura di co-marketing.
5. **Azione 4 (Erogazione Saldo)**: Una volta ricevuta la documentazione fiscale corretta, l'operatore imposta lo stato su `Approvato per Payout`. Make.com invia l'istruzione all'API di Wise che esegue il bonifico sul conto corrente dell'Ambassador.

---

## 🛡️ 3. PROTOCOLLI DI SICUREZZA E CONVERSIONE DEL DATO

* **Finestra temporale di 30 giorni**: Il sistema controlla quotidianamente la data di assegnazione del lead. Se entro 30 giorni l'agenzia non ha caricato un mandato in esclusiva firmato, Make.com imposta lo stato del lead su `Scaduto`, rimuovendo l'esclusiva visiva dal portale dell'agenzia e rendendo il dato nuovamente disponibile per la riassegnazione a partner concorrenti.
* **Controllo Anti-Scavalcamento**: Il database Airtable è integrato con script di monitoraggio periodico. Qualora venga rilevato un passaggio di proprietà catastale su un indirizzo registrato senza che sia stato registrato il relativo corrispettivo contrattuale, il sistema notifica automaticamente il team legale per l'applicazione della penale del triplo del valore della tariffa.
