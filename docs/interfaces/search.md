
`#!typescript interface ConfigSearch` agisce come il contenitore principale che ha la capacità di ospitare configurazioni per vari tipi di ricerca, ù
noti come `#!typescript interface SearchStruct`, insieme alle impostazioni per la presentazione dei risultati, denominate `#!typescript interface SearchResults`. 
All'interno di questo quadro, `#!typescript interface SearchStruct` si avvale di `#!typescript interface SearchAggregation` e `#!typescript interface SearchFilter` per orchestrare la modalità con cui i risultati vengono aggregati e 
sottoposti a filtro. L'interazione tra SearchAggregation e `#!typescript interface SearchFilter` gioca un ruolo cruciale nella definizione di faccette e filtri, con l'importante 
implicazione che i filtri impiegati possono modulare i risultati dell'aggregazione. Inoltre, SearchResults dettaglia il metodo con cui i dati emersi dalla ricerca,
potenzialmente modulati dalle configurazioni di `#!typescript interface SearchStruct`, vengono esposti all'utente finale.


###  `#!typescript interface ConfigSearch`

Questa interfaccia funge da principale punto di ingresso per la configurazione delle impostazioni di ricerca. Consente la specifica dinamica delle chiavi di ricerca e delle relative configurazioni.


| Proprietà     | Tipo                                | Opzionale | Descrizione                                                                                                                                                                                                                           |
|---------------|-------------------------------------|-----------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `key: string` | `SearchStruct` \| `SearchResults[]` | No        | Coppie chiave-valore dinamiche dove la chiave è un identificatore delle impostazioni di ricerca, e il valore è un `SearchStruct` o un array di `SearchResults`. La chiave speciale "results" configura le impostazioni dei risultati. |

###  `#!typescript interface SearchStruct`

Definisce la struttura di una query di ricerca, inclusa la query base, l'ordinamento, le query specifiche per lingua, le opzioni per filtrare i risultati, le impostazioni di aggregazione per le faccette e i filtri da applicare alla query.


| Proprietà   | Tipo                                                                                                      | Opzionale | Descrizione                                                                                                           |
|-------------|-----------------------------------------------------------------------------------------------------------|-----------|-----------------------------------------------------------------------------------------------------------------------|
| base_query  | `{`<br/> `field?: string;` <br/> `value?: string;` <br/> `}`                                              | Sì        | Configurazione della query base, specificando il campo su cui effettuare la query e i valori da cercare.              |
| sort        | `string[]`                                                                                                | No        | Array di stringhe che specificano i campi per ordinare i risultati della ricerca.                                     |
| lang        | `{` <br/> `query: {` <br/> `type: string;` <br/> `field: string;` <br/> `};` <br/> `}`                    | No        | Impostazioni di query specifiche per lingua, inclusi il tipo di query e il campo.                                     |
| options     | `{` <br/> `exclude?: string[];` <br/> `include?: string[];` <br/> `}`                                     | Sì        | Impostazioni opzionali per escludere o includere campi specifici nei risultati.                                       |
| facets-aggs | `{` <br/> `type: 'obj';` <br/> `aggregations: {` <br/> `[key: string]: SearchAggregation;` `};` <br/> `}` | No        | Impostazioni di aggregazione per le faccette, con un tipo 'obj' e coppie chiave-valore dinamiche per le aggregazioni. |
| filters     | `{` <br/> `[key: string]: SearchFilter;` <br/> `}`                                                        | No        | Filtri da applicare alla query di ricerca, con coppie chiave-valore dinamiche.                                        |

###  `#!typescript interface SearchAggregation`

| Proprietà        | Tipo                                                           | Opzionale | Descrizione                                                                                                       |
|------------------|----------------------------------------------------------------|-----------|-------------------------------------------------------------------------------------------------------------------|
| nested           | `boolean`                                                      | No        | Indica se l'aggregazione è fatta su un campo nidificato.                                                          |
| nestedFields     | `string[]`                                                     | Sì        | Elenca i campi genitore di un campo nidificato, applicabile solo se nested è true.                                |
| search           | `string`                                                       | No        | Il campo da utilizzare per l'aggregazione.                                                                        |
| title            | `string`                                                       | No        | Il campo da visualizzare come etichetta per l'aggregazione.                                                       |
| sort             | `term`                                                         | Sì        | Ordinamento opzionale per i termini dell'aggregazione, impostato su 'term' per un ordinamento alfabetico.         |
| sortValues       | `string[]`                                                     | Sì        | Ordinamento manuale per le faccette, specificato come un array di stringhe.                                       |
| innerFilterField | `string[]`                                                     | Sì        | Campi utilizzati per la ricerca all'interno di una faccetta, applicabile per filtraggi di faccette più complessi. |
| generalFilter    | `{` <br/> `fields: string[];` <br/> `value: string;` <br/> `}` | Sì        | Filtro generale applicato all'aggregazione, specificando i campi su cui filtrare e il valore del filtro.          |
| extra            | `{` <br/> `[key: string]:` <br/> `string;` <br/>  `}`          | Sì        | Oggetto contenente proprietà extra da includere nella risposta, definite come coppie chiave-valore.               |
| ranges           | `RangeAggregation[]`                                           | Sì        | Definisce un insieme di intervalli per le aggregazioni, ognuno specificato da un valore di inizio e fine.         |
| global           | `boolean`                                                      | Sì        | Indica se l'aggregazione è globale, non basata sulla query corrente.                                              |

###  `#!typescript interface SearchFilter`

| Proprietà    | Tipo                                        | Opzionale | Descrizione                                                                                                                                 |
|--------------|---------------------------------------------|-----------|---------------------------------------------------------------------------------------------------------------------------------------------|
| type         | `'fulltext'` \| `'multivalue'` \| `'range'` | No        | Tipo di filtro, può essere "fulltext" per una ricerca libera, "multivalue" per una faccetta o "range" per una ricerca basata su intervalli. |
| addStar      | `boolean`                                   | Sì        | Flag opzionale per aggiungere caratteri jolly all'inizio e alla fine del termine di ricerca.                                                |
| field        | `string` \| `string[]`                      | No        | Specifica il/i campo/i su cui effettuare la query.                                                                                          |
| operator     | `'OR'` \| `'AND'`                           | Sì        | Operatore logico da usare nel filtro, di default è "AND" se non specificato.                                                                |
| nested       | `boolean`                                   | Sì        | Indica se il filtro è su un campo nidificato, richiedendo configurazioni aggiuntive per i campi nidificati.                                 |
| nestedFields | `string[]`                                  | Sì        | Elenca i campi genitore di un campo nidificato, applicabile solo se nested è true.                                                          |

###  `#!typescript interface RangeAggregation`

| Proprietà | Tipo     | Opzionale | Descrizione                                             |
|-----------|----------|-----------|---------------------------------------------------------|
| from      | `number` | No        | Il valore di inizio dell'intervallo per l'aggregazione. |
| to        | `number` | No        | Il valore di fine dell'intervallo per l'aggregazione.   |

###  `#!typescript interface SearchResults`

| Proprietà | Tipo                                                                                                | Opzionale | Descrizione                                                                                                                         |
|-----------|-----------------------------------------------------------------------------------------------------|-----------|-------------------------------------------------------------------------------------------------------------------------------------|
| label     | `title` \| `text` \| `metadata` \| `highlights` \| `id` \| `link` \| `image` \| `routeId` \| `slug` | No        | Specifica il tipo di etichetta da mostrare nell'elenco dei risultati, come 'titolo', 'testo', 'metadati', ecc.                      |
| field     | `string` \| `string[]`                                                                              | No        | Specifica il/i campo/i i cui valori dovrebbero essere restituiti, che possono variare in base al tipo di etichetta.                 |
| max-char  | `number`                                                                                            | Sì        | Proprietà opzionale che specifica il numero massimo di caratteri da restituire per il valore del campo, applicabile in alcuni casi. |

