# Legenda:
- ELF : Executable and Linkable Format → formato finale che la CPU può caricare ed eseguire (direttamente o tramite il loader del sistema operativo)
	- gcc -o main main.c → main diventa un elf file.
	-  -g aggiunge debug informations al file 

2 shell: 

1* uriscv-cli config_machine.json --gdb --debug ---> lancia server debug
2* dentro cartella progetto gdb-multiarch build/MultiPandOS
	- target remote: 8080
	ora sei connesso al server
## GDB comandi
- Lay next → layout next dovrebbe mostrare codice c e assembly insieme
- break / b → dove si ferma l'esecuzione
- continue/c → prosegue l'esecuzione veloce
- next → uno step di esecuzione in c
- nexti → uno step di esecuzione assembly
- stepi → entriamo nella funzione chiamata
- ref → refresh screen
- x/i  $pc→ examine the instruction
- info registers → panoramica dei registri durante una specifica istruzione
- bt/backtrace → stack delle chiamate di funzione
- p/print \[nome variabile] → print della variabile in quel punto !!! 
- info locals → mostra tutte variabili locali
- x/4x \[indirizzo memoria] → mostra 4 parole  in esadec.
- info functions/ info symbol \[indirizzo memoria]-> mostra simboli(?)

## Registri importanti
- pc: registro che contiene prossima istruzione da eseguire
- sp: stack pointer contiene:
- ra: return address → dove tornare dopo funzione
## comandi asm risc-v
Generalmente: istruzione destinazione, src1,src2
Parola in pandos è 4 byte(32-bit)
- **li** load immediate: carica numero → li a0, 5 carica 5 in registro a0
- mv move: copia registro → mv a0, a1 sposta a a0 il contenuto di a1
- add: somma di registri → add a0,a1,a2 :a0 = a1+a2
- sub: sottrazione → sub a0,a1,a2 : a0 = a1-a2
- lw: load word legge da memoria → lw a0, 0(sp) → legge memoria a sp :
	  lw registro_dest, offset(registro_base)
- sw: store word sw a0, 0(sp)
- j: jump \[label]|indirizzo
- call: chiamata funzione
- ret: return
- beq: branch if equal beq a0, a1 LABEL
- bne: brach if not equal


## Debug pazzesco
- metti una funzione db(); fino al punto in cui vuoi controllare.
- poi fai break \[nomefile].c:\[numero riga]
- poi c e vedi se runna fino a lì.