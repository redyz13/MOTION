# UC-05 — Crosswalk candidato ACI → ISPRA → COPERT 5.9.2

| Campo | Valore |
|---|---|
| Versione del metodo | 0.1 |
| Revisione editoriale | 21/08/2026 |
| Stato | Allegato metodologico in Working Draft |
| Ambito | Costruzione offline dei profili `CAR`, `MOTORCYCLE`, `BUS` e `TRUCK` usati per derivare fattori emissivi medi di CO₂ |

## 1. Scopo e relazione con UC-05

Questo documento descrive una metodologia candidata per trasformare i dati
disponibili sul parco veicolare in profili compatibili con COPERT 5.9.2.
Il profilo risultante permette di aggregare i fattori emissivi delle singole
tecnologie in un fattore medio per ciascuna macro-categoria osservata da
FRONTIERE.

Il crosswalk è un'attività di supporto **offline** e non fa parte del flusso
runtime principale di UC-05. A runtime UC-05 riceve i transiti, associa i
fattori emissivi già disponibili e calcola la stima sul segmento e
sull'intervallo considerati.

La metodologia non è ancora approvata scientificamente. Offre una base
riproducibile che potrà essere sostituita o calibrata quando saranno
disponibili osservazioni locali più rappresentative.

## 2. Fonti e ruolo dei dati

| Fonte | Uso nel metodo | Limite principale |
|---|---|---|
| ACI Open Parco Veicoli, Salerno 2024 | Composizione locale dello stock per categoria, alimentazione e classe Euro. | Lo stock immatricolato non coincide necessariamente con i veicoli che transitano a Baronissi. |
| ISPRA, `DatiCopertTrasportoStrada1990-2024.xlsx`, basato su COPERT 5.8.1 | Quote nazionali di attività/VKT usate per disaggregare dimensioni non disponibili nei dati ACI. | È un proxy nazionale e non descrive direttamente il traffico locale. |
| COPERT 5.9.2 Tier 3 | Calcolo dei fattori di emissione hot tailpipe per tecnologia e velocità. | Richiede una flotta e condizioni di guida configurate; gli output devono essere validati per l'uso progettuale. |
| FRONTIERE | Conteggi di transiti classificati nelle macro-categorie osservabili. | Tassonomia effettiva, qualità del conteggio e trattamento delle classi ambigue sono ancora da confermare. |

I dati ISPRA 5.8.1 sono usati esclusivamente per comporre la flotta. Non viene
effettuata una conversione di fattori emissivi da COPERT 5.8.1 a 5.9.2: i
fattori sono ricalcolati direttamente con COPERT 5.9.2.

## 3. Tipi di corrispondenza

| Stato | Significato |
|---|---|
| `DIRECT` | La classificazione sorgente è compatibile con la famiglia COPERT necessaria. |
| `PROXY` | La corrispondenza usa un'approssimazione esplicita. |
| `CONDITIONAL` | La regola dipende dalla tassonomia che FRONTIERE renderà effettivamente disponibile. |
| `UNMAPPED` | Non esiste ancora una corrispondenza utilizzabile; la quota deve restare dichiarata. |

Lo stato si riferisce alla singola dimensione. Per esempio, una categoria può
essere diretta mentre cilindrata, massa o segmento COPERT sono ottenuti tramite
proxy.

## 4. Macro-categorie veicolari

| Categoria MOTION | ID COCO atteso | Sorgente ACI 2024 | Stock Salerno | Regola candidata |
|---|---:|---|---:|---|
| `CAR` | 2 | `AV` — Autovetture | 737.963 | Usare l'intera categoria AV. |
| `MOTORCYCLE` | 3 | `MC` — Motocicli | 138.710 | Usare MC; l'eventuale inclusione dei ciclomotori dipende dalla semantica FRONTIERE. |
| `BUS` | 5 | `AB` — Autobus | 2.233 | Usare l'intera categoria AB. |
| `TRUCK` | 7 | `AM` — Autocarri merci + `TS` — Trattori stradali | 94.162 | Creare un profilo aggregato LCV/HDT finché FRONTIERE espone una sola classe `truck`. |
| `SPECIAL_TBD` | — | `AS` — Autoveicoli speciali/specifici | 15.991 | Conservare separatamente; non includere automaticamente in `TRUCK`. |

### 4.1 Opzioni predisposte per FRONTIERE

- Se FRONTIERE espone soltanto `truck`, UC-05 usa il profilo aggregato
  `TRUCK`, costruito da AM e TS.
- Se FRONTIERE distingue commerciali leggeri e pesanti, possono essere creati
  profili separati `TRUCK_LCV` e `TRUCK_HDV`.
- Se FRONTIERE include i veicoli speciali in `truck`, l'eventuale integrazione
  di AS richiede una nuova versione del profilo.
- Se `motorcycle` comprende anche i ciclomotori, il profilo deve includere i
  segmenti moped usando una fonte integrativa, poiché il dataset ACI MC
  disponibile copre i motocicli oltre 50 cm³.

## 5. Alimentazioni

Le principali regole candidate sono riepilogate di seguito. Le
semplificazioni indicate come proxy restano visibili nel profilo.

| Codici ACI | Interpretazione nel profilo COPERT | Nota |
|---|---|---|
| `BE` | Petrol | Diretto per CAR, MOTORCYCLE e TRUCK; proxy per BUS. |
| `GA`, `GG` | Diesel | `GA` è diretto per CAR, BUS e TRUCK; gli altri casi sono proxy. |
| `EL` | Battery Electric | Contributo pari a zero nel perimetro **hot tailpipe CO₂**; non implica zero emissioni nel ciclo di vita. |
| `BG`, `IBG` | LPG Bifuel per CAR; Petrol negli altri profili | La componente ibrida non è rappresentata quando manca una tecnologia equivalente. |
| `BM`, `IBM`, `IM`, `ME` | CNG/CNG Bifuel dove disponibile | Il collasso della tecnologia è un proxy quando la granularità ACI non è esposta da COPERT. |
| `IB` | Petrol Hybrid per CAR; Petrol negli altri profili | Diretto soltanto per CAR. |
| `IG` | Diesel Hybrid per BUS; Diesel negli altri profili | Non viene interpretato automaticamente come Diesel PHEV. |
| `IGG`, `IGM` | Diesel | Proxy conservativo. |
| `AL` | Petrol per MOTORCYCLE; non mappato negli altri profili | Il mapping dei motocicli conserva la quota ma richiede validazione. |
| `ND` | Non mappato | Non viene trasformato in zero né scartato implicitamente. |

### 5.1 Copertura dell'alimentazione nello snapshot analizzato

| Categoria | Diretto | Proxy | Non mappato |
|---|---:|---:|---:|
| `CAR` | 99,297% | 0,699% | 0,004% |
| `MOTORCYCLE` | 93,752% | 6,245% | 0,002% |
| `BUS` | 99,060% | 0,940% | 0,000% |
| `TRUCK` | 96,473% | 3,524% | 0,003% |

Queste percentuali descrivono soltanto la corrispondenza delle alimentazioni;
non includono le approssimazioni necessarie per massa, cilindrata, tipo di bus
o segmento COPERT.

## 6. Classi Euro

COPERT raggruppa alcune fasi regolamentari con una granularità diversa da ACI.
Inoltre, la legenda pubblica ACI consultata non decodifica formalmente i
suffissi `5B` e `6A`–`6E`. Le relative corrispondenze restano quindi un working
mapping da confermare con ACI o ISPRA.

| Etichetta ACI | Passenger Cars / LCV | HDT / BUS | L-Category | Stato |
|---|---|---|---|---|
| `EURO 0` | Stadi pre-Euro compatibili / `Conventional` | `Conventional` | `Conventional` | Proxy per le autovetture; diretto verso `Conventional` negli altri casi. |
| `EURO 1`–`EURO 4` | `Euro 1`–`Euro 4` | `Euro I`–`Euro IV` | `Euro 1`–`Euro 4` | Diretto. |
| `EURO 5` | `Euro 5` | `Euro V` | `Euro 5` | Diretto alla granularità COPERT. |
| `EURO 5B` | `Euro 5` | `Euro V` | `Euro 5` | Raggruppato; suffisso ACI da confermare. |
| `EURO 6` | Ripartizione tra `Euro 6 a/b/c`, `6 d-temp`, `6 d/e` | Ripartizione tra `Euro VI A/B/C` e `VI D/E` | `Euro 5` | Ripartizione proxy mediante quote ISPRA. |
| `EURO 6A`–`EURO 6C` | `Euro 6 a/b/c` | `Euro VI A/B/C` | `Euro 5` | Working mapping; collasso proxy per L-Category. |
| `EURO 6D` | `Euro 6 d-temp` | `Euro VI D/E` | `Euro 5` | Working mapping; collasso proxy per L-Category. |
| `EURO 6E` | `Euro 6 d/e` | `Euro VI D/E` | `Euro 5` | Working mapping; collasso proxy per L-Category. |
| `NC` con `EL` | Zero hot tailpipe | Zero hot tailpipe | Zero hot tailpipe | Valido soltanto nel perimetro emissivo dichiarato. |
| `NC` non elettrico | — | — | — | Non mappato. |
| `ND` | Distribuzione Euro ISPRA condizionata | Distribuzione Euro ISPRA condizionata | Distribuzione Euro ISPRA condizionata | Proxy; se anche il carburante è ND, resta non mappato. |

## 7. Costruzione dei pesi di flotta

ACI non espone tutte le dimensioni richieste da COPERT. Per ciascuna cella
`categoria × alimentazione × Euro`, le quote di attività ISPRA sono usate per
ripartire lo stock tra i segmenti compatibili:

- `CAR`: Mini, Small, Medium, Large/SUV/Executive;
- `MOTORCYCLE`: tempi del motore e classi di cilindrata disponibili;
- `BUS`: urbano/coach e classi dimensionali;
- `AM`: LCV e HDV rigidi;
- `TS`: HDV articolati.

Per evitare una doppia allocazione, AM non alimenta i segmenti articolati e TS
non alimenta i segmenti rigidi.

Indicando con $S^{ACI}_{c,f,e}$ lo stock ACI e con
$p^{ISPRA}_{i\mid c,f,e}$ la quota di attività ISPRA della tecnologia $i$:

$$
q_{c,i}=S^{ACI}_{c,f,e}\,p^{ISPRA}_{i\mid c,f,e}
$$

$$
w_{c,i}=\frac{q_{c,i}}{\sum_i q_{c,i}}
$$

Per ogni macro-categoria deve risultare $\sum_i w_{c,i}=1$. I pesi sono
activity-oriented, ma restano un proxy perché combinano stock provinciale e
quote VKT nazionali.

Quando manca la combinazione ISPRA esatta, la proposta usa nell'ordine:

1. la stessa categoria, alimentazione e classe Euro;
2. la stessa categoria e alimentazione, con distribuzione Euro ISPRA 2024;
3. per i BEV, una tecnologia virtuale con contributo hot tailpipe CO₂ nullo;
4. in assenza di una corrispondenza utilizzabile, lo stato `UNMAPPED`.

Non è previsto un fallback implicito tra alimentazioni differenti.

## 8. Esito preliminare del mapping

L'applicazione delle regole precedenti allo snapshot Salerno 2024 ha prodotto
il seguente esito. I valori documentano il working design e non costituiscono
una validazione scientifica della rappresentatività della flotta.

| Categoria | Stock sorgente | Stock mappato | Diretto | Proxy | Condizionale | Non mappato | Tecnologie risultanti |
|---|---:|---:|---:|---:|---:|---:|---:|
| `CAR` | 737.963 | 737.937 | 54,997% | 38,938% | 6,062% | 0,004% | 130 |
| `MOTORCYCLE` | 138.710 | 138.707 | 93,675% | 6,323% | 0,000% | 0,002% | 25 |
| `BUS` | 2.233 | 2.233 | 78,818% | 10,255% | 10,927% | 0,000% | 49 |
| `TRUCK` | 94.162 | 94.156 | 70,111% | 19,504% | 10,378% | 0,006% | 138 |

Il mix principale ottenuto è:

| Categoria | Composizione del profilo |
|---|---|
| `CAR` | Diesel 49,018%; Petrol 35,939%; LPG Bifuel 9,357%; CNG Bifuel 3,059%; Petrol Hybrid 2,311%; Battery Electric 0,316%. |
| `MOTORCYCLE` | Petrol 99,765%; Battery Electric 0,235%. |
| `BUS` | Diesel 99,328%; CNG 0,493%; Battery Electric 0,179%. |
| `TRUCK` | Diesel 92,952%; Petrol 5,161%; CNG 1,724%; Battery Electric 0,164%; ripartizione LCV 55,006% e HDV 44,994%. |

La quota LCV/HDT è ottenuta dal proxy ISPRA nazionale: non è una misura del
traffico locale di Salerno o Baronissi.

## 9. Uso dei pesi per ottenere gli EF

Per ciascuna tecnologia COPERT $i$, il peso $w_{c,i}$ viene applicato al
relativo fattore hot tailpipe di CO₂. Il fattore medio della macro-categoria è:

$$
EF_c(v)=\sum_i w_{c,i}\,EF_i(v)
$$

dove $v$ è la velocità rappresentativa del segmento e dell'intervallo. I BEV
restano nel denominatore del profilo con contributo hot tailpipe pari a zero.

La metodologia candidata prevede di generare offline una matrice di EF su una
griglia di velocità e di usare a runtime il valore corrispondente, o
un'interpolazione tra nodi se approvata. Una modifica della composizione di
flotta, della tassonomia o delle condizioni COPERT richiede la rigenerazione
del profilo e degli EF; non richiede una chiamata COPERT per ogni intervallo di
15 minuti.

Il POC preliminare conservato nella repository è descritto in
[`artifacts/reference/copert-poc-592/README.md`](../artifacts/reference/copert-poc-592/README.md).
Il POC verifica il flusso di input/output COPERT, ma non rappresenta il profilo
ACI/ISPRA pesato descritto in questo documento.

## 10. Assunzioni e punti aperti

1. Confermare il data dictionary FRONTIERE, in particolare la semantica di
   `truck`, l'eventuale distinzione LCV/HDT e l'inclusione dei ciclomotori.
2. Confermare con ACI o ISPRA la decodifica di `EURO 5B` e
   `EURO 6A`–`EURO 6E`.
3. Valutare la rappresentatività locale del proxy stock ACI × VKT ISPRA e,
   quando possibile, calibrarlo con dati di traffico osservati a Baronissi.
4. Decidere se e come includere gli autoveicoli speciali nel profilo `TRUCK`.
5. Approvare versione COPERT, perimetro hot tailpipe, profili, griglia di
   velocità, interpolazione e gestione delle velocità fuori dominio.
6. Definire un protocollo di validazione e un'analisi di sensibilità prima di
   considerare i fattori idonei all'uso operativo.

## 11. Fonti

- ACI Open Parco Veicoli, snapshot Salerno 2024 per AV, MC, AB, AM, TS e AS.
- [ACI Open Parco Veicoli — legenda categorie e alimentazioni](https://opv.aci.it/WEBDMCircolante/legenda.html).
- [ACI Open Parco Veicoli — note metodologiche](https://opv.aci.it/WEBDMCircolante/noteOPV.html).
- ISPRA, `DatiCopertTrasportoStrada1990-2024.xlsx`, foglio `veickm`, dataset
  basato su COPERT 5.8.1.
- [EMEP/EEA Road Transport Guidebook 2024](https://copert.emisia.com/wp-content/uploads/2024/07/1.A.3.b.i-iv-Road-transport-2024.pdf).
- [Regolamento (UE) n. 459/2012](https://eur-lex.europa.eu/eli/reg/2012/459/oj/eng), fasi Euro 5a/5b e 6a/6b/6c.
- [Regolamento (UE) 2017/1347](https://eur-lex.europa.eu/eli/reg/2017/1347/oj/eng) e [testo consolidato 2017/1151](https://eur-lex.europa.eu/legal-content/EN/TXT/?uri=CELEX%3A02017R1151-20200125), fasi Euro 6d-TEMP e 6d.
- COPERT 5.9.2, installazione Windows usata per i test progettuali.
