
L’endpoint `/search/results` recupera i dati tramite una chiamata Elasticsearch. Per costruire la query di Elasticsearch la funzione `buildQuery`  utilizza una configurazione. La libreria di base mette già una configurazione di default. Quest’ultima può essere sovrascritta per ottenere il risultato desiderato. Per costruire le proprie configurazioni è stata definita un’interfaccia che controlla la struttura del parametro in input alla  funzione `buildQuery`. Qui di seguito sono illustrate tutti i valori e le proprietà accettate in fase di definizione della configurazione.
I principali componenti dell’interfaccia sono i seguenti:

* `SearchStruct`: definisce una query di Elasticsearch. È possibile combinare dentro a questa interfaccia altri due componenti qui sotto elencati: `searchAggregation` e `SearchFilter`.
* `SearchAggregation`: definisce le aggregazioni che possono essere aggiunte dentro alla query di `SearchStruct`.
* `SearchFilter`: definisce i filtri che possono essere aggiunti dentro alla query di `SearchStruct`.
* `RangeAggregation`: un componente che può essere utilizzato dentro a Search Aggregation.
* `SearchResults`: insieme alla `SearchStruct` è un’altra configurazione accettata per la search. In questo caso è utilizzata per quali valori e label debbano essere utilizzate per ritornare i dati all’utente.

###  ConfigSearch

Questa interfaccia funge da principale punto di ingresso per la configurazione delle impostazioni di ricerca. Consente la specifica dinamica delle chiavi di ricerca e delle relative configurazioni.


| Proprietà     | Tipo                                | Opzionale | Descrizione                                                                                                                        |
|---------------|-------------------------------------|-----------|------------------------------------------------------------------------------------------------------------------------------------|
| `key: string` | `SearchStruct` \| `SearchResults[]` | No        | il valore è un `SearchStruct` o un array di `SearchResults`. La chiave speciale "results" configura le impostazioni dei risultati. |

###  SearchStruct

Definisce la struttura di una query di ricerca, inclusa la query base, l'ordinamento, le query specifiche per lingua, le opzioni per filtrare i risultati, le impostazioni di aggregazione per le faccette e i filtri da applicare alla query.


| Proprietà   | Tipo                                                                                                   | Opzionale | Descrizione                                                                                                                                      |
|-------------|--------------------------------------------------------------------------------------------------------|-----------|--------------------------------------------------------------------------------------------------------------------------------------------------|
| base_query  | `{`<br/> `field?: string;` <br/> `value?: string;` <br/> `}`                                           | Sì        | Configurazione della query base di Elasticsearch. Il `value` indica il valore del `field` sui cui fare un match all'interno dell'indice Elastic. |
| sort        | `string[]`                                                                                             | No        | Serve per indicare a Elasticsearch su quali campi fare un riordino dei risultati.                                                                |
| lang        | `{` <br/> `query: {` <br/> `type: string;` <br/> `field: string;` <br/> `};}`                          | No        | ...                                                                                                                                              |
| options     | `{` <br/> `exclude?: string[];` <br/> `include?: string[];` <br/> `}`                                  | Sì        | Impostazioni opzionali per escludere o includere campi specifici nel risultato.                                                                  |
| facets-aggs | `{` <br/> `type: 'obj';` <br/> `aggregations: {` <br/> `[key: string]:` <br/>  `SearchAggregation;};}` | No        | Serve per definire come Elasticsearch deve gestire le aggregazione dei dati ritornati.                                                           |
| filters     | `{` <br/> `[key: string]: SearchFilter;` <br/> `}`                                                     | No        | Filtri da applicare alla query Elasticsearch.                                                                                                    |




####  SearchAggregation

| Proprietà        | Tipo                                                           | Opzionale | Descrizione                                                                                                       | Esempi |
|------------------|----------------------------------------------------------------|-----------|-------------------------------------------------------------------------------------------------------------------|--------|
| nested           | `boolean`                                                      | No        | Indica se l'aggregazione è fatta su un campo nidificato.                                                          |        |
| nestedFields     | `string[]`                                                     | Sì        | Elenca i campi genitore di un campo nidificato, applicabile solo se nested è true.                                |        |
| search           | `string`                                                       | No        | Il campo da utilizzare per l'aggregazione.                                                                        |        |
| title            | `string`                                                       | No        | Il campo da visualizzare come etichetta per l'aggregazione.                                                       |        |
| sort             | `term`                                                         | Sì        | Ordinamento opzionale per i termini dell'aggregazione, impostato su 'term' per un ordinamento alfabetico.         |        |
| sortValues       | `string[]`                                                     | Sì        | Ordinamento manuale per le faccette, specificato come un array di stringhe.                                       |        |
| innerFilterField | `string[]`                                                     | Sì        | Campi utilizzati per la ricerca all'interno di una faccetta, applicabile per filtraggi di faccette più complessi. |        |
| generalFilter    | `{` <br/> `fields: string[];` <br/> `value: string;` <br/> `}` | Sì        | Filtro generale applicato all'aggregazione, specificando i campi su cui filtrare e il valore del filtro.          |        |
| extra            | `{` <br/> `[key: string]:` <br/> `string;` <br/>  `}`          | Sì        | Oggetto contenente proprietà extra da includere nella risposta, definite come coppie chiave-valore.               |        |
| ranges           | `RangeAggregation[]`                                           | Sì        | Definisce un insieme di intervalli per le aggregazioni, ognuno specificato da un valore di inizio e fine.         |        |
| global           | `boolean`                                                      | Sì        | Indica se l'aggregazione è globale, non basata sulla query corrente.                                              |        |

####  SearchFilter

| Proprietà    | Tipo                                        | Opzionale | Descrizione                                                                                                                                 |
|--------------|---------------------------------------------|-----------|---------------------------------------------------------------------------------------------------------------------------------------------|
| type         | `'fulltext'` \| `'multivalue'` \| `'range'` | No        | Tipo di filtro, può essere "fulltext" per una ricerca libera, "multivalue" per una faccetta o "range" per una ricerca basata su intervalli. |
| addStar      | `boolean`                                   | Sì        | Flag opzionale per aggiungere caratteri jolly all'inizio e alla fine del termine di ricerca.                                                |
| field        | `string` \| `string[]`                      | No        | Specifica il/i campo/i su cui effettuare la query.                                                                                          |
| operator     | `'OR'` \| `'AND'`                           | Sì        | Operatore logico da usare nel filtro, di default è "AND" se non specificato.                                                                |
| nested       | `boolean`                                   | Sì        | Indica se il filtro è su un campo nidificato, richiedendo configurazioni aggiuntive per i campi nidificati.                                 |
| nestedFields | `string[]`                                  | Sì        | Elenca i campi genitore di un campo nidificato, applicabile solo se nested è true.                                                          |

####  RangeAggregation

| Proprietà | Tipo     | Opzionale | Descrizione                                             |
|-----------|----------|-----------|---------------------------------------------------------|
| from      | `number` | No        | Il valore di inizio dell'intervallo per l'aggregazione. |
| to        | `number` | No        | Il valore di fine dell'intervallo per l'aggregazione.   |

###  SearchResults

| Proprietà | Tipo                                                                                                | Opzionale | Descrizione                                                                                                                         |
|-----------|-----------------------------------------------------------------------------------------------------|-----------|-------------------------------------------------------------------------------------------------------------------------------------|
| label     | `title` \| `text` \| `metadata` \| `highlights` \| `id` \| `link` \| `image` \| `routeId` \| `slug` | No        | Specifica il tipo di etichetta da mostrare nell'elenco dei risultati, come 'titolo', 'testo', 'metadati', ecc.                      |
| field     | `string` \| `string[]`                                                                              | No        | Specifica il/i campo/i i cui valori dovrebbero essere restituiti, che possono variare in base al tipo di etichetta.                 |
| max-char  | `number`                                                                                            | Sì        | Proprietà opzionale che specifica il numero massimo di caratteri da restituire per il valore del campo, applicabile in alcuni casi. |

