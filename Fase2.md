

- Nucleo
	- Inizializzazione → chiama scheduler → fa partire processi → o terminano o vengono interrotti da interrupt o eccezioni per poi richiamare scheduler
	- Scheduler
	- Gestione eccezioni(fa syscall)
	- Interrupt
	- Passing events to lv 4

- Si utilizzano strutture dati di fase 1.
- syscall exception handling
	- a1 
	- a2  priorità
	- a3 struttura supporto



# Processo
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
		- Ready → p_list in readyqueue
		- Running
		- Blocked → p_list nella s_procq di un semaforo
		Quando passa da Ready a Running → Scheduled: è stato scelto e assegnata CPU.
			1. Il processo viene rimosso dalla ready queue
			2. Il suo stato viene caricato nei registri CPU
			3. Il kernel esegue un context switch
			4. Il processo inizia a girare
		Vicecersa De-scheduled:
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