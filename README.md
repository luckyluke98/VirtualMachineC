# VirtualMachineC

Progetto per il corso di Programmazione, Corso di Laurea triennale in Informatica.
Il progetto consiste nella realizzazione di una maccchina virtuale con registri e stack
che esegua dei programmi specificati in codice macchina. La macchina virtuale può
produrre un output a console utilizzando delle apposite istruzioni per il debugging.

## Stack

La macchina virtuale utilizza un'area di memoria (array di interi) di 64KB come stack (accesso LIFO). Il registro
speciale SP (unsigned int) indica la posizione (indice) della prossima posizione dello stack da scrivere (0 stack vuoto).
L'istruzione PUSH inserisce un intero nello stack (ed incrementa SP), l'istruzione POP preleva un elemento dallo
stack (dopo aver decrementato SP). In caso di stack over
ow o stack under
ow il programma termina immediatamente segnalando un errore.

## Instruction pointer, jump, chiamate a subroutine

La macchina virtuale, dopo il bootstrap durante il quale vengono inizializzate le strutture dati ed i registri, ripetutamente
legge le istruzioni del programma in codice macchina (fetch), ne prepara gli operandi e le esegue (execute).
Il registro speciale instruction pointer (IP) contiene l'indice (unsigned int) della prossima istruzione da eseguire.
Dopo il fetch di un'istruzione viene incrementato in modo tale da indicare l'istruzione successiva (dipende dal numero
di argomenti dell'istruzione, v. sezione instruction set). L'istruzione jump (JMP) sostituisce il valore corrente
di IP con il valore indicato mentre le istruzioni jump condizionate eseguono il salto solo se le rispettive condizione
sono vere (v. instruction set). La chiamata a subroutine (CALL) inserisce nello stack il valore corrente di IP
(gia aggiornato alla posizione successiva a CALL) e lo sostituisce con il valore indicato. RET permette di rientrare
da una chiamata a subroutine sostituendo ad IP il valore prelevato dallo stack.

## Registri

La macchina virtuale puo utilizzare 32 registri interi R0-R31. Ciascun registro puo essere utilizzato come operando
o contenere il risultato di operazioni.

## Utilizzo Macchina virtuale

La virtual machine accede da linea di comando due parametri: operazione ('esegui' o 'stampa') e nome del file
di testo contenente il programma. Ciascuna linea delfile di testo contiene un numero intero corrispondente
ad un elemento del codice macchina del programma (una posizione dell'array di interi) ed eventuali commenti
preceduto dal carattere ';'. La prima linea contiene il numero di linee non vuote del file (esclusa la prima), uguale
alla lunghezza dell'array di interi che conterrà il programma da eseguire.
Nel caso l'operazione richiesta sia 'stampa' la virtual machine:

* verificherà che il file esista (altrimenti terminerà con un errore)
* aprirà il file e stamparà le istruzioni del programma corrispondente secondo il formato:
[posizione] istruzione P1 P2
dove posizione è la posizione nell'array, istruzione il codice mnemonico, P1 e P2 sono gli eventuali parametri.
Nel caso uno del parametri sia un registro stampare la rappresentazione mnemonica corrispondente (es: R3
anzichè 3)

Nel caso l'operazione richiesta sia 'esegui' VM :
* verificherà che il file esista (altrimenti termina con un errore)
* aprirà il file, allocarà il blocco di memoria di dimensione adeguata (array di interi, dimensione dalla prima linea
del file) e leggere le istruzioni dal le scrivendo gli interi corrispondenti in memoria
* inizializzarà i registri e lo stack
* inizializzarà IP a zero
* ripeterà fetch-execute fino a che l'istruzione è diversa da HALT

## Programmi dimostrativi

Alcuni programmi dimostrativi sono presenti nella Repository.
Codice macchina nel formato richiesto per il file di input,

codice scritto in c:
```c
#include <stdio.h>

int fattoriale (int n) {
  int i, risultato = 1;
  for (i = 2; i <= n; i++)
    risultato *= i;
  return risultato;
}

int main() {
  int r = fattoriale(5);
  printf("%d", r);
}
```

codice c convertito in codice macchina accettato dalla virtual machine:
```
35 ; numero linee(posizione massima)
12 ; [ 0 ] MOV R0 5
0  ; [ 1 ]
5  ; [ 2 ]
20 ; [ 3 ] CALL fattoriale
8  ; [ 4 ]
1  ; [ 5 ] DISPLAY R30
30 ; [ 6 ]
0  ; [ 7 ] HALT
; fattoriale:
12 ; [ 8 ] MOV R1 2
1  ; [ 9 ]
2  ; [ 10 ]
12 ; [ 11 ] MOV R20 1
20 ; [ 12 ]
1  ; [ 13 ]
12 ; [ 14 ] MOV R30 1
30 ; [ 15 ]
1  ; [ 16 ]
; inizio:
31 ; [ 17 ] SUB R0 R1
0  ; [ 18 ]
1  ; [ 19 ]
25 ; [ 20 ] JNEG fine
34 ; [ 21 ]
32 ; [ 22 ] MUL R30 R1
30 ; [ 23 ]
1  ; [ 24 ]
11 ; [ 25 ] POP R30
30 ; [ 26 ]
30 ; [ 27 ] ADD R1 R20
1  ; [ 28 ]
20 ; [ 29 ]
11 ; [ 30 ] POP R1
1  ; [ 31 ]
22 ; [ 32 ] JMP inizio
17 ; [ 33 ]
; fine:
21 ; [ 34 ] RET
```

## Comandi per utilizzare la VirtualMachine

Comando "esegui":

```bash
./VM "esegui" "nome_del_file"
```
Comando "stampa"

```bash
./VM "stampa" "nome_del_file"
```
Un'output di esempio del comando stampa di un file scritto in codice macchina sarà come segue:

```
[ 0 ] MOV R0 5
[ 3 ] CALL 8
[ 5 ] DISPLAY R30
[ 7 ] HALT
[ 8 ] MOV R1 2
[ 11 ] MOV R20 1
[ 14 ] MOV R30 1
[ 17 ] SUB R0 R1
[ 20 ] JNEG 34
[ 22 ] MUL R30 R1
[ 25 ] POP R30
[ 27 ] ADD R1 R20
[ 30 ] POP R1
[ 32 ] JMP 17
[ 34 ] RET
```
