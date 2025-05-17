# Tema Proiectarea Algoritmilor

## Descriere

Acest proiect scris în limbajul C implementează o aplicație modulară pentru gestionarea și procesarea unor date despre candidați. Se folosesc diverse structuri de date precum cozi, arbori binari, heap-uri, pentru a simula un proces de selecție și filtrare în mai multe etape.

Proiectul este împărțit pe pași succesivi, fiecare reprezentând o fază logică a prelucrării. 

Deși am explorat idei nice, cum ar fi criptarea traseelor în pasul 7, acestea au fost abandonate pentru a menține coerența și simplitatea. De asemenea, din considerente de logice, am ales să nu eliberez memoria între pași, desi am avut niste tenative de a o face. Din cauza reutilizarii structurilor, si a costului prea mare de rezolvare a memory leak-urilor imi asum penalizarea. Totodata, tin sa mentionez ca am ales solutii optime pentru fiecare task unde mai mereu am luat in considerare complexitatea algoritmilor raportati la datele de intrare.

---

## Structura Fișierelor

- `main.c` – punctul de intrare, apelează pașii în ordine
- `functions_1.c` – procesarea inițială a datelor din fișier CSV
- `functions_2.c` – separarea candidaților în două categorii
- `functions_3.c` – filtrare și prelucrare a Lorzilor
- `functions_4.c` – asocierea candidaților cu trasee in heap
- `functions_5.c` – calculul experientei totale si reasezarea heap-ului
- `functions_6.c` – afisare top 3
- `functions_7.c` – generarea traseelor
- `modells.h` – definirea structurilor de date
- `C_modules.h` – antete pentru funcțiile utilizate

---

## Cum se compilează

Asigură-te că ai un compilator C instalat (ex: `gcc`) și rulează comanda:

```bash
gcc main.c functions_*.c -o proiect
```

---

## Cum se rulează

```bash
./proiect
```

---

## Explicația pașilor
Pentru urmarirea atenta a pasilor propun visualizarea main.c de unde reiese contributia fiecarui pas la rezolvare;

	printf("-----------------PAS1------------------\n");
	coada *Q = pas1();

	printf("-----------------PAS2------------------\n");
	frunza *BFS_Lorzi = NULL;
	frunza *BFS_Aventurieri_Cavaleri = NULL;
	pas2(&Q, &BFS_Lorzi, &BFS_Aventurieri_Cavaleri);

	printf("-----------------PAS3------------------\n");
	pas3(&BFS_Lorzi);

	printf("-----------------PAS4------------------\n");
	heap *CopacMaxHeap = creare_HeapMax();
	pas4(&BFS_Lorzi, &BFS_Aventurieri_Cavaleri, &CopacMaxHeap);

	printf("-----------------PAS5------------------\n");
	pas5(&CopacMaxHeap);
	printf("-----------------PAS6------------------\n");
	pas6(&CopacMaxHeap);
	printf("-----------------PAS7------------------\n");
	SGraph g;
	g = creare_graf();


## 1. Functii importante, idei;
Fișier: `functions_1.c`


## funcția `pas1()`

- Deschide fișierul `./Pas_1/candidati.csv` pentru citire și fișierul `./Pas_1/test_1.csv` pentru scriere.
- Sare peste antetul CSV (`fgets()` inițial).
- Citește rând cu rând restul fișierului și parsează informațiile folosind manipulare directă cu pointeri.

---

### Procesare linie cu linie

Folosind `fgets()`, fișierul este citit rând cu rând într-un buffer de 1000 de caractere, ceea ce permite gestionarea rândurilor de lungime variabilă.

### Strategia de tokenizare

În loc de funcții clasice de tip `strtok()`, programul folosește funcții precum `strchr()` și pointer arithmetic pentru a localiza delimitatori specifici:

- `strchr(string, ' ')` → primul spațiu (delimitează statutul social)
- `strchr(primul_spatiu + 1, ';')` → primul punct și virgulă după spațiu
- `strchr(punctvirgula + 1, ';')` → al doilea punct și virgulă

Această abordare oferă un control mai fin asupra parsării și evită efectele secundare ale funcțiilor clasice de tokenizare.

---

## Funcții de extragere continut din buffer (string)

### `extragere_statut_social()`

- Extrage statutul social (de exemplu, "LORD", "CAVALER") din linie
- Convertește literele în majuscule pentru uniformizare:

```c
char *statut_social = malloc(11);
strncpy(statut_social, string, lungime_statut);
statut_social[lungime_statut] = '\0';
for (int i = 0; i < lungime_statut; i++) {
    statut_social[i] = toupper(statut_social[i]);
}
```

### `extragere_nume()`

- Extrage numele candidatului și îl formatează astfel:
  - Prima literă cu majusculă
  - Spațiile interne sunt înlocuite cu cratime
  - Literele după cratimă sunt și ele cu majusculă
  - Restul caracterelor → litere mici

### `extragere_xp()`

- Extrage valoarea experienței și o transformă într-un float:

```c
strncpy(xp, punctvirgula + 1, lungime_xp);
xp[lungime_xp] = '\0';
float xpF = atof(xp);
```

### `extragere_varsta()`

- Extrage câmpul de vârstă ca șir de caractere.

---

## Crearea structurii de date

- După parsare, fiecare candidat este adăugat într-o coadă folosind `adaugare_la_Coada()`.
- În cadrul acestei funcții, statutul social este convertit într-o valoare numerică (`enum`):

```c
if (strcmp(statut_social, "LORD") == 0)
    v.statut_socialint = LORD;
else if (strcmp(statut_social, "CAVALER") == 0)
    v.statut_socialint = CAVALER;
else
    v.statut_socialint = AVENTURIER;
```

---

## Observații 
- Parsarea este robustă și bine controlată.
- Transformările (cum ar fi conversia la majuscule sau transformarea numelor) sunt realizate direct în pasul de parsare, nu într-un pas ulterior.
- Această abordare de parsare, deși implică un efort manual mai mare, oferă o precizie și un control superior față de metodele standard, fiind potrivită pentru scenarii în care formatul de intrare este rigid, dar nu standardizat complet.

## 2. funcția `deletenod()`
Fișier: `functions_3.c`
```c
    frunza *deletenod(frunza *root, float key, int data)
```
### Descriere
Această funcție implementează un algoritm de ștergere a arborelui binar de căutare cu un pas suplimentar de verificare a datelor. Elimină un nod cu o valoare specifică a experienței (key) și o sumă de control a datelor din arbore, menținând în același timp proprietatea BST.

root: Pointer către nodul curent din BST.
key: Valoarea experienței (valoare_xp_float) de căutat și șters.
data: O valoare a sumei de control generată din numele, vârsta și starea nodului pentru a verifica dacă nodul corect este șters.

### Valoare returnată

Returnează un pointer către rădăcina BST după finalizarea ștergerii.

---
## pas7()
La pasul 7 am incercat sa gasesc o solutie care sa nu necesite o matrice pentru sortatea traseelor in ordine lexicografica. 
Prin intermediul structurii de mai jos imi propuneam sa stochez fiecare caracter cu dubla sa valoarea de integer/ascii. Strategia functioneaza pana la numere de maxim 2 caractere.
```c

typedef struct numascii
 {
     int decimal;
     int ascii;
 } ascii_obj;
```
---

## Autor

Alexandru Dinu – Proiect realizat în cadrul disciplinei PA, 2025.
## Bibliografie
1. Curs Proiectarea Algoritmilor
2. gitHub.com
3. stackoverflow.com

