# GY9

### Kulcsok

> mappa: gy5
> fájl: `uzenet.c`

Ezzel a kulccsal tudunk egy folyamathoz hozzáférni, ezzel tuduink sorrendiséget beállítani, ill azonosítója lez az üzeneteknek.

Kulcs létrehozása:
````C
key_t kulcs;
kulcs = ftok(argv[0],1);  //ftok(létező állomány , a kulcs verziószám ami meghat hanyadik verzió /*lehet több verzió is*/ ez egy random szám); ezzel vférünk hozzá az üzenetsorhoz
````

Adatcsatorna létrehozása:
````C
uzenetsor = msgget( kulcs, 0600 | IPC_CREAT );
````

Ha megáll a program akkor le kell zárni!
````C
status = msgctl( uzenetsor, IPC_RMID, NULL ); //msgctl( uzenetsor neve, lecsatlakozás, mindig NULL a lecsatlakozáskor);
````

Küldés:
````C
int kuld( int uzenetsor )
{
     const struct uzenet uz = { 5, "Hajra Fradi!" };
     int status;
     
     status = msgsnd( uzenetsor, &uz, strlen ( uz.mtext ) + 1 , 0 ); //üzenet küldése //msgsnd( uzenetsor neve, &amibe, memóriaszelet mérete /*fix méret kell!*/, IPC_NOWAIT constans /*nincs várakozás*/ );
         // a 3. param ilyen is lehet: sizeof(uz.mtext)
         // a 4. parameter gyakran IPC_NOWAIT, ez a 0-val azonos
     if ( status < 0 )
          perror("msgsnd");
     return 0;
}
````

Fogadás:
````C
int fogad( int uzenetsor )
{
     struct uzenet uz;
     int status;
     // az utolso parameter(0) az uzenet azonositoszama
    // ha az 0, akkor a sor elso uzenetet vesszuk ki
    // ha >0 (5), akkor az 5-os uzenetekbol a kovetkezot
    // vesszuk ki a sorbol
     status = msgrcv(uzenetsor, &uz, 1024, 5, 0 ); //msgrcv(mit, &hova, memóriaszelet, melyik típusú üzenetet az üzenetsorból /*3-mtype mező*/, 0. elem );
                                                                                                                                                                                         X-ha nincs X típusú üzenet akkor addig vár amíg kap

     if ( status < 0 )
          perror("msgsnd");
     else
          printf( "A kapott uzenet kodja: %ld, szovege:  %s\n", uz.mtype, uz.mtext );
     return 0;
}
````

### Osztott memória

> fájl: `osztmem.c`
Hogyan tudunk létrehozni egy olyan változót amitz két foylamat párhuzamosan tud írni, olvasni, ez egy megosztott területen lévő változó

Szükséges `include`ok:
````C
#include <sys/ipc.h>
#include <sys/shm.h>
#include <sys/types.h>
#include <sys/stat.h>
````

PL: A gyermek folyamat olvasni fogja a szülő pedig írja
`````C
key_t kulcs;
kulcs=ftok(argv[0],1);
char *s; //ezt hozzuk létre
oszt_mem_id=shmget(kulcs,500,IPC_CREAT|S_IRUSR|S_IWUSR); //shmget(kulcs,megosztott terület mérete,kapcsoltó /*ha nem létezik létrehozzuk, jogosultságok etc*/);
 s = shmat(oszt_mem_id,NULL,0);  //megosztott memórai terület címe
`````


Szülő:
````C
strcpy(s,buffer); // beír a megosztottra
shmdt(s); //törli a meegosztottat


shmctl(oszt_mem_id,IPC_RMID,NULL); //shmctl(osztott memória terület cím, mit csináljunk vele, NULL); //jelen esetben törli
`````

Gyerek:
````C
 printf("A gyerek ezt olvasta az osztott memoriabol: %s",s); //kiírja amit elolvasott
 shmdt(s);  // gyerek is elengedi a megosztott területet
`````
