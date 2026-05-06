## Intro
- Un kernel si costruisce su più strati: la stratificazione è uno degli argomenti più ricorrenti nel mondo dell'informatica(esempio lo stack di rete)
- Questa fase è la prima fase in cui dobbiamo effettivamente controllare programmi che concorrono per l'uso delle risorse.
- In pratica abbiamo costruito nella prima fase delle strutture dati che ora useremo per gestire le interazioni tra processi.
- Potrà essere richiesto di modificare alcune cose della fase 1.(io almeno ho fatto così, non so se è possibile evitarlo).

- Nucleo:
	- Inizializzazione → chiama scheduler → fa partire processi → o terminano o vengono interrotti da interrupt o eccezioni per poi richiamare scheduler
	- Noi dobbiamo implementare:
	- init
	- Scheduler
	- Gestione eccezioni(fa syscall)
	- Interrupt
	- Passing events to lv 4
	
- syscall exception handling(parte forse più difficile di questa fase)
	- a1 
	- a2  priorità
	- a3 struttura supporto
	- Possono cambiare ma in pratica le syscall hanno questi 3 registri che passano infomazioni necessarie alla loro corretta gestione.

### Ciclo normale di un kernel
- Abbiamo come componenti una ready_queue, uno scheduler  ed un processore.
- Normalmente i processi si dispongo in ready_queue, essendo il nostro scheduler round robin selezionerà dai processi in ready_queue un processo(il primo in ordine di prio) e lo carica sul processore. questo processo verrà eseguito per una finestra di tempo dettata dal PLT(processor local timer) che ne indica la durata. Una volta finita la sua finestra di tempo il processo termina e viene dispached nella readyqueue se non ha finito con le sue computazioni
 
# Processo(info buone da sapere per non partire allo sbaraglio)
- Un processo è un'istanza di un programma.
- Il sistema operativo permette la virtualizzazione della CPU → Time sharing: molti processi concorrenti. costi di performance dipende da quanti processi.
	- Per permettere la corretta virtualizzazione di cpu è necessario un mix di meccanismi di basso livello → eg. Context switch(permette di stoppare il processo corrente mantenendone lo stato) e di meccanismi di alto livello : Policies → algoritmi di gestione dei processi e che servono per compiere decisioni su quali processi eseguire,ecc..
- Un processo è definito dal suo machine state:
	- Cosa può leggere e scrivere il processo?
	- Quali parti di memoria sono necessarie affinchè il processo venga eseguito correttamente?
	La memoria è un aspetto fondamentale del processo:
	- Address space(spazio di memoria che il processo può modificare)
		- Stack
		- heap
	- Registri:
		- PC
		- Stack pointer
		- Frame pointer
- API base:
	- Create 
	- Destroy
	- Wait
	- Miscellaneous control → eg. suspend
	- Status → per avere info del processo
- #### Stati
	- Un processo si può trovare solo in tre stati:
		- Ready → p_list in ready_queue
		- Running
		- Blocked → p_list nella s_procq di un semaforo
		Quando passa da Ready a Running → Scheduled: è stato scelto e assegnata CPU.
			1. Il processo viene rimosso dalla ready queue
			2. Il suo stato viene caricato nei registri CPU
			3. Il kernel esegue un context switch
			4. Il processo inizia a girare
		Viceversa De-scheduled:
			- Il kernel salva lo stato nei campi del PCB (`p_s`)
			- Reinserisce il processo nella ready queue
			- Schedula qualcun altro
		In realtà esistono stati ready e terminated ulteriori.
## INIT
- si parte dall'inizializzazione delle variabili di sistema e l'inizializzazione delle componenti fondamentali del kernel.
- Cercare sui file type.h, const.h e anche in quelle installate con uriscv quindi in usr/include/uriscv/..(dipende dalla vostra installazione) i valori dei registri o delle macro da utilizzare e includere i file.h
- Importante distinzione : \#include<> e \#include"" si comportano in modo diverso:
	- <> cercherà nelle librerie di sistema /usr/include, usr/local/include … → uriscv/types.h
	- "" cercherà nella directory corrente del progetto → eg. asl.h e simili
- extern void : tutte le funzioni che usiamo direttamente e  che sono state definite in altri file.
- Una volta inizializzate le variabili specificate dalle slide, bisogna popolare il passup vector.
	- Pass-up Vector: vettore per la gestione delle exceptions.
	- Creiamo un puntatore all'indirizzo del passup-vector
	- Si passa l'indirizzo della funzione uTLB_RefillHandler(presente in p2test.c)al campo relativo del passupvector;
	- Si punta alla funzione exception_handler che andrà creata in seguito.
	- Bisogna settare gli stack delle due funzioni a KERNELSTACK
- Si inizializzano le strutture dati della fase 1
- Inizializziamo le variaibli dichiarate in questo file
- Tramite la macro LDIT(presente sempre nei file ursicv/...) carichiamo 100 ms 
- inizializziamo un processo(root) e seguiamo le istruzioni delle slide. di base non c'è nulla di complicato e per quanto riguarda i termini in maiuscolo solitamente sono costanti o definizioni che si trovano nei vari file da includere. quindi bisogna cercarli in giro.
Init viene eseguito una sola volta, dopo il resto lo farà lo scheduler.x
- N.B. più avanti nel progetto verrà scritto di tenere in considerazione il tempo dei vari processi. Riguardo questo aspetto abbiamo due diversi timer:
	- PLT timer → timer del tempo di esecuzione del processo all'interno del processore
	- Interval timer → timer generale che viene richiamato ogni tot e servirà suppongo nella prossima fase

### Scheduler
- Bisogna considerare gli stati in cui lo scheduler deve comportarsi in modo diverso.
- Stato di deployment base: ready_queue > 0
	- In questo caso dobbiamo:
		 - current_process= processo da rimuovere dalla coda
			  - SETTIMER(TIMESLICE) settiamo un intervallo di tempo perchè il processo sia eseguito: allo scadere scatta l'interrupt.
			  - LDST(&current_process->p_s) carichiamo lo stato del processo nel processore.
	- Caso  process_counter == 0 ovvero: non ci sono processi in ready_queue e non ci sono processi attivi e non ci sono processi bloccati. allora va in HALT
	- Caso process_counter > 0 e soft_block > 0  allora si deve attendere WAIT(sulle slide c'è un blocco di codice)
		- mie → machine interrupt-enable register che contiene bit che abilitano i singoli interrupt di livello machine.
			-  MTIE →  timer interrupt machine
			- MSIE →  software interrupt machine
			- MEIE → external interrupt machine 
		- MIE_ALL & ~MIE_MTIE_MASK →  setta a neg il timer interrupt immagino perchè andando in wait evita di attivare l'interrupt del timer.
		- status |= MSTATUS_MIE_MASK → accende i bit mie del status. “la CPU può accettare interrupt”
			 - `|=` = accendi
			 - `& ~MASK` = spegni
	- Caso process_counter > 0 e soft_block == 0 → siamo nel caso di deadlock e viene invocato PANIC
### Exception Handler
- Nel momento in cui viene generata un'eccezione, questa viene salvata nel BIOS DATAPAGE(una zona di memoria RAM usata da bios e kernel per lo scambio di informazioni).
	- Al suo interno avremmo lo stato minimale del processo al momento dell'eccezione e conterrà tutte le informazioni necessarie per la sua corretta gestione.
	- Le varie eccezioni generate passeranno determinati valori in registri specifici.
	- Per svolgere al meglio il progetto consiglio di guardare il file di test e implementare le syscall in ordine di comparsa.
	  N.B. può essere necessario dover implementare anche un minimo di interrupt handler per poter completare determinate operazioni.
	- Passando il BIOS DATAPAGE all'interno di una struttura state_t possiamo leggere correttamente i vari campi necessari.
	- ### Syscall handler
		- Gestione delle varie syscall con privilegio kernel.
		- Nelle syscall quando vengono chiamate vengono posti dei valori all'interno di un set di registri già passati con state_t dal BIOS Datapage.
		- ### Create Process
			- Abbastanza basic: con le primitive create in fase1 basta usare quelle e poi aggiungere il processo in coda.
		- ### Terminate Process
			- Questo è più tricky: bisogna terminare un processo e tutta la sua progenie. ma questo è un lavoro da fare ricorsivamente.
			- No spoilers eccessivi ma probabilmente necessita una funzione extra(subtree_killer) che dovete creare voi. 
			- I processi non si nascondono: si possono trovare: in ready_queue, semafori device e non device o il current process
			- Se eliminate un processo diverso da quello corrente tenete in considerazione che potrebbe essere un genitore di quello corrente quindi tenete in considerazione anche quello.
		- ### Passaren e Verhogen
			- Servizi offerti dal kernel
			- Potete farli senza pensare ai semafori mantenuti dal kernel tanto poi avrete modo di gestirli con altre syscall e interrupts(Spoiler)
		- ### DoIO
			- Call per i semafori dei device:
			- 
		- ### GetCPUTime
			- Controlla il tempo passato sulla cpu del processo corrente. non è nulla di piu complicato, è un check del tempo passato da quando è stato schedulato quel processo.
			- Ricordare che quando un processo viene sospeso il tempo del processo viene salvato nel registro apposito così quando verrà ripreso questo valore non sarà resettato.
		
		- ### WaitForClock
			- Onestamente non mi è chiara ma penso sarà più chiara nella prossima fase
		- ### GetSupportData
			- Non c'è moloto da dire sulle get
		- ### GetProcessID
			- """"
		- ### Yield
			- Abbastanza semplice: In pratica quando viene chiamata il processo corrente viene rimesso in coda.
			- Bisogna però ricordarsi di gestire tutti i casi 
	- ### PassupOrDIE
		- Qui bisogna salvare in determinate strutture dati i valori di supporto se presenti.
		  in caso contrario verrà chiamata terminate process.
- ### Interrupt Handler
	- #### PTL
	- #### Interval Timer
	- #### Device Interrupt
	- 

## TODO
- memcpy copiato da gcc per permettere a compilatore di fare copia di campi
- cpu_t slice_start; per permettere alla cpu di contare il tempo passato
- findByPid; funzione per phase1 da creare per facilitare ricerca processi
- casting ptr_exc->a0 in intero per leggere correttamente la variabile
- STCK() per salvare il tempo in quell'istante