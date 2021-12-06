Programowanie równolegle i rozproszone - projekt 2 labolatorium 7 "Gra w Życie"("Conway's Game of Life") w środowisku MPI.
Jest to przykład automatu komórkowego. Każda z komórek może przyjąć jeden ze stanow , przy czym liczba stanów jest skończona. Stan komórki zależy od jej obecnego stanu i stanu jej sąsiadów.

Import bibliotek ( w tym biblioteki MPI umożliwiającej pisanie programów wykonywanych równolegle):

    #include <stdio.h>
    #include <stdlib.h>
    #include <string.h>
    #include "mpi.h"
    
Definicja stałych:

    #define DOMYSLNE_ITERACJE 64
    #define SIATKA 256
    #define WYMIAR 16
    
Definicja funkcji modulo(zwracającej resztę  z dzielenia):

    int mod(int a, int b)
    {
        int r = a % b;
        return r < 0 ? r + b : r;
    }

W mainie realizacja założeń gry w życie z wykorzystaniem biblioteki MPI, a najpierw zdefiniowanie domyślengo wzoru siatki i deklaracja zmiennych oraz domyślnych stanów startowych komórek:

    int main(int argc, char **argv)
    {
        int wzor_siatki[256] =
            {0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0,
             0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0,
             0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0,
             0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0,
             0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0,
             0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0,
             0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0,
             0, 0, 0, 1, 1, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0,
             0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0,
             0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0,
             0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0,
             0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0,
             0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 0, 1, 0, 0,
             0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0,
             0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0,
             0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0};

        int suma_procesow;
        int suma_iter;
        int ID; 
        int j;
        int iter = 0;

        if (argc == 1)
        {
            suma_iter = DOMYSLNE_ITERACJE;
        }
        else if (argc == 2)
        {
            suma_iter = atoi(argv[1]);
        }
        else
        {
            exit(1);
        }
        MPI_Init(&argc, &argv);  //inicjalizacja środowiska MPI
        MPI_Status status;
        MPI_Comm_size(MPI_COMM_WORLD, &suma_procesow);  // okresla liczbę uruchomionych procesow tworzących sieć
        MPI_Comm_rank(MPI_COMM_WORLD, &ID);             // okresla numer aktualnego procesu

Alokacja głównej tablicy w pamięci:

        int *tablica = (int*)malloc(WYMIAR*((WYMIAR/suma_procesow)+2)*sizeof(int));  

Główna pętla:  

        for (iter = 0; iter < suma_iter; iter++)
        {
      ...
        }

Uwolnienie tablicy z pamięci. 
Na koniec funkcja, która kończy pracę w trybie MPI:

        free(tablica);
        MPI_Finalize();
    }
