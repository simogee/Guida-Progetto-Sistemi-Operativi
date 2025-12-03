
- ## Introduzione
	In questa fase c'è poco codice da scrivere. E' molto importante capire le strutture dati presenti in types.h e le funzioni delle liste contenute in listx.h poichè le funzioni in quest'ultimo header saranno utilizzate quasi in tutte le funzioni da implementare.
- ## Strutture dati:
	- types.h
		- Pcb_t → struttura del pcb che contiene numerosi campi. Quelli usati in questa fase sono:
			- p_list → gancio per aggiungere il pcb alle queue.
			- p_child → lista dei figli del processo pcb in cui ci troviamo
			- p_sib → lista di altri processi alla stessa profondità del pcb in cui ci troviamo.
			- p_semAdd → l'indirizzo associato alla coda di semafori in cui il pcb si trova.
			- p_prio → valore di priorità del processo corrente.(utile nelle code dei pcbs).
		- semd_t → struttura dei semafori 
			- \*s_key ⇒ è il valore dell'indirizzo a cui è associato il nostro semaforo(il suo id)
			- s_procq ⇒ coda dei processi associati al semaforo
			- s_link ⇒ lista dei semafori (ordine crescente per PandOs specification).
		- ### Liste in stile linux
			- Siamo abituati a vedere strutture con un campo \*next associato alla prossima struttura. In questo caso la situazione è diversa:
				- I campi associati alle strutture sono essi stessi strutture con campi prev e next.
				- Quasi nessuna funzione agisce sulle pcb direttamente: molto spesso ci tocca utilizzare la funzione container_of() per estrarre dal nostro struct list_head la struttura ad esso associata. Quindi quello che succede spesso è che iteriamo lungo un campo list_head e facciamo container_of per ottenere le informazioni associate al punto della lista in cui siamo arrivati.
		- ## Semafori
			- Abbiamo due liste nel file Asl.c:
				- semdFree_h → lista da inizializzare per avere la nostra pool di semafori disponibili
				- semd_h → lista dei semafori attivi.
			- InitASL → le slide dicono di inizializzare solo semdFree. In realtà va inizializzata anche la lista semd_h.
			- Quando gestiamo i semafori dobbiamo pensare che avremo un solo semd_t per ogni coda di processi. in questo semd_t abbiamo la chiave associata al semaforo, un list_head associato ai semafori vicini e un list_head per la coda di processi bloccati in quel semaforo.
			- Momento ASCII art:
			```
			--------
			-semd_t-
			-header-
			--------	
		      |                                                    to sentinel
		      | s_link            |                         ...  <----------------|
		      v                   v              pcb_t A                          |
		  --------  s_procq  ----------      --------------       ------------   |
		  -semd_t- --------> -sentinel- ---->- p_list     - ----->- p_list   -  -|
		  -s_key -           -s_procq -      - p_semAdd   -       - p_semAdd -   
		  --------           ----------      --------------       ------------
		     ...
		     
		     back to the header
		     
		     
			```
		- ## PCBs
			- abbiamo una sola lista da gestire: pcbFree_h → pool dei pcb a disposizione.
			- i pcb hanno un valore di priorità da gestire. 
			- #### Alberi
				- abbiamo tre campi del pcb relativi a questa sezione:
					- pcb * p_parent → puntatore al pcb del padre 
					- list_head p_child → lista che inizia da valore sentinella e da inizio alla lista dei processi figli. Il valore sentinella è  il campo p_child dentro il pcb del padre.
					- list_head p_sib → lista dei processi fratelli.
				- ASCII ART scarsa rappresentativa della relazione child e sib.:
			- ```
					-----PCB Genitore----- (sentinella)
					-   -p_child-        -               (l'ultimo p_sib)
					-   - next  -        -  <---------------------------------
					-	- prev  -        -          (e' collegato a prev)      | 
				   ---------|------------          (di p_child del genitore)
							|                                                  |
							| next al p_sib                                    |
							v                                                  |
					-----PCB A-------                 -----PCB_N------         |
					-   --p_sib--   -   next a sib    -   --p_sib--  -         |
					- 	--next--    - ---->     ...   -   -- next -- -  -------
					-	--prev---   - <----     ...   -   -- prev -- - 
					-----------------                 ----------------
				```
- ## Funzioni/macro Listx.h
	- cointainer_of(ptr, type, member) ⇒ ptr: è un puntatore list_head che punta ad un campo del nostro target.
	  //todo esempio
	- offsetof(TYPE, MEMBER) ((size_tt)(&((TYPE *)0)->MEMBER))
	- struct list_head{ struct list_head \*next, \*prev;}
	- static inline void INIT_LIST_HEAD(struct list_head \*list)
	- static inline void \_\_list_add(struct list_head \*new, struct list_head \*prev, struct list_head \*next)
		- static inline void list_add(struct list_head \*new, struct list_head \*head)
		- static inline void list_add_tail(struct list_head \*new, struct list_head \*head)
	- static inline void \_\_list_del(struct list_head \*prev, struct list_head \*next)
		- static inline void list_del(struct list_head \*entry)
	- static inline int list_is_last(const struct list_head \*list, const struct list_head \*head)
	- static inline int list_empty(const struct list_head \*head)
	- list_for_each(pos, head) for (pos = (head)->next; pos != (head); pos = pos->next)