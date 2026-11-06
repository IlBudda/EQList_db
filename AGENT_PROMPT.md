# Prompt per l'agente autonomo: completamento catalogo EQList

Copia tutto quello che segue (dalla riga "MISSIONE" in poi) come primo messaggio di una nuova sessione Claude Code collegata al repo `IlBudda/EQList_db`, con il connettore Google Sheets attivo.

---

## MISSIONE
Sei l'orchestratore. Lavori in autonomia finché non hai completato **tutti i brand** presenti nei fogli **LG / Pending LG**, **Camera**, **Pending Ottiche / Ottiche**, **Pending Grip / Grip** del Google Sheet `1hfBYQhWBHzwMcpVTuc2Tg8pKrB8mgC25ENayFKyOlgw`. Ogni brand va trattato con lo stesso procedimento già applicato ad ARRI, Zeiss e Cooke:
1. Analisi.
2. Unione dei doppioni e id unici.
3. Dati tecnici e descrizioni dal sito ufficiale.
4. Collegamenti nei due versi.
5. Verifica cella per cella.

Prima di iniziare leggi `HANDOFF.md` nel repo: contiene sheetId, regole, metodi e stato del lavoro.

## RUOLI (obbligatorio)
- **Tu (modello principale)**: inventario, piani di unione, regole di parsing, controlli, verifiche, tutte le operazioni distruttive (eliminare righe, rinominare, cambiare colonne) e i report.
- **Agenti economici** (`model: "sonnet"`, `subagent_type: "general-purpose"`): raccolta dei dati ufficiali dai siti dei produttori e caricamenti meccanici (update_values da file JSON già preparati, copyPaste da file di richieste già pronti). Ogni agente riceve un perimetro preciso:
  - file di input (lista di id e famiglie);
  - file di output JSON con le chiavi esatte delle colonne;
  - fonti ammesse, cioè solo il dominio ufficiale del brand e i PDF che ospita;
  - il divieto di toccare fogli e repo, salvo le scritture meccaniche esplicite;
  - un report breve (massimo 15 righe).
- Lancia in parallelo e in background gli agenti di raccolta dati. Tutto ciò che un agente scrive va riverificato da te prima di procedere.

## REGOLE DEI DATI (non negoziabili)
- Tutto in inglese, nel formato delle colonne esistenti. Metrico prima, imperiale dopo la barra.
- Fonte unica: sito ufficiale del produttore e suoi PDF. Mai rental house, cinelenses, forum o Wikipedia. Se non c'è fonte ufficiale, la cella resta com'è e lo annoti.
- Celle già piene: non sovrascrivere. Fanno eccezione `websiteUrl` (va sulla pagina ufficiale), l'apertura massima se l'ufficiale è diversa e la descrizione se è vuota, non in inglese o presa da un sito non ufficiale.
- id `brand_famiglia_focale` minuscolo con underscore, unico (per esempio `zeiss_supreme_prime_15mm`). Le varianti diverse (uncoated, SF, aperture diverse, kit) hanno id distinti. Nomi nel formato "FAMIGLIA 25mm T2.1"; brand come scritto dal produttore, in maiuscolo.
- Doppioni: unisci solo se brand, famiglia, focale e variante coincidono. Le righe con intestazioni di listino sbagliate vanno ricondotte alla famiglia giusta solo se le prove sono forti (stessa sequenza di focali e aperture del listino ufficiale). Nella riga che resta tieni la descrizione più completa e riempi i campi vuoti con quelli delle righe eliminate.
- Collegamenti: aggiorna ogni riferimento all'id vecchio in **tutti** i fogli. Ottiche ↔ camere solo per attacco nativo (gli adattatori non contano); accessori ↔ prodotti nei due versi.

## REGOLE TECNICHE (dalle sessioni precedenti)
- Locale italiano: `;` nelle formule. Gli id in LG, Camera e Pending Camera sono formule calcolate dal nome: per rinominare si cambia il nome. In Pending Ottiche gli id sono valori.
- findReplace regex in lotti di al massimo 25-30 richieste; un timeout significa che non è stato applicato. Pattern: `(^|\s)OLD(,|$)` → `$1NEW$2`.
- Valori che iniziano con "+": anteponi l'apostrofo.
- Scritture sparse, sempre con questo metodo:
  1. Crea il foglio temporaneo `_map_tmp_<brand>`.
  2. Carica i valori nuovi ("" = invariato) e scrivi la formula `=IF(B1="";INDEX('Foglio'!$A:$AF;$A1;COLUMN(B1)-1);B1)`.
  3. Controlla ISNUMBER/ISERROR/COUNTA.
  4. Fai la copyPaste PASTE_VALUES.
  5. Rileggi **tutto** il foglio di destinazione e confrontalo cella per cella con l'atteso. Deve risultare 0 differenze impreviste.
  6. Elimina i doppioni, dal basso verso l'alto.
  7. Elimina il foglio temporaneo.
- Collegamenti per attacco: raggruppa per combinazione di attacchi, poi usa una tabella chiave→stringa e una formula LET, infine un'unica copyPaste della colonna G; verifica con un diff.
- Prima di ogni eliminazione di righe, crea una copia della scheda (`duplicateSheet`) chiamata `_bak_<scheda>_<brand>`. Eliminala solo dopo che la verifica finale è passata.
- Non pushare su main. Lavora sul branch `claude/vibrant-gauss-7w8kgw`; le immagini ufficiali vanno nel repo immagini secondo `HANDOFF.md`.

## PROCEDIMENTO
**Fase 0 – Inventario.** Leggi tutte le schede indicate e produci in `PROGRESS.md` (nel repo) una tabella con: foglio, brand, numero di righe, numero di id duplicati, famiglie, presenza di fonti ufficiali e stato (da fare / fatto / bloccato). Segna come già fatti: ARRI (tutte le categorie), Zeiss e Cooke (ottiche). Ordine di lavoro: per ogni foglio, prima i brand con più righe.

**Per ogni brand, in quest'ordine:**
1. Analisi delle famiglie e piano dei doppioni (script locale), più l'elenco degli id nuovi senza collisioni.
2. Agenti di raccolta dati sulle famiglie, divise in 2 o 3 gruppi. Controlla a campione i risultati e adatta il piano se gli agenti trovano nuovi doppioni o prodotti inesistenti.
3. Costruzione delle righe finali, compreso lo scarto dei dati attribuiti alla variante sbagliata.
4. Caricamento tramite agente, verifica del caricamento, formule, incolla tramite agente.
5. Verifica completa del foglio, eliminazione dei doppioni e aggiornamento dei riferimenti.
6. Collegamenti: ottiche e camere per attacco nativo; accessori e prodotti nei due versi.
7. Verifica dei collegamenti (nessun id orfano, nessun id doppio, nessuna voce ripetuta), poi eliminazione di backup e fogli temporanei.
8. Aggiornamento di `PROGRESS.md` (riga del brand: cosa è stato fatto, numeri, cosa manca e perché), poi commit e push.

**Per categoria:**
- **Luci (LG/Pending LG):** colonne tecniche già presenti (lampheadType … certifications). Collega accessori ↔ luci.
- **Camere (Camera):** schema completo già presente. Per gli accessori usa accessoryType … powerSource. Collega camere ↔ accessori ↔ ottiche.
- **Ottiche (Pending Ottiche):** colonne I:AF. Collega ottiche ↔ camere per attacco.
- **Grip (Pending Grip/Grip):** colonne gripType … storageTemp. Collega ai prodotti compatibili se il produttore lo indica.
- Le righe generiche dei listini noleggio senza un modello preciso (per esempio "SUPER 35 LENSES - X") non vanno inventate: ricevono solo id puliti e unici.

## QUANDO FERMARSI E CHIEDERE
- Cambi strutturali (colonne nuove) in una scheda non ancora progettata.
- Un'unione che richiede un'ipotesi non supportata dai listini ufficiali.
- Un errore di permessi o un'operazione negata: non aggirarla, annotala e prosegui con il brand successivo.
- In tutti gli altri casi prosegui in autonomia fino alla fine dell'elenco.

## FINE LAVORO
Hai finito quando ogni brand in `PROGRESS.md` è "fatto" o "bloccato" con un motivo. A quel punto:
- esegui un controllo globale su tutte le schede: id unici, riferimenti risolti, nessun foglio temporaneo o di backup rimasto, nessuna descrizione non in inglese;
- aggiorna `HANDOFF.md`, fai commit e push;
- consegna un report finale in italiano con totali per foglio e brand, decisioni prese, casi bloccati e domande aperte.
