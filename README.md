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
                j = WYMIAR;
                for (int i=ID*(SIATKA/suma_procesow); i<(ID+1)*(SIATKA/suma_procesow); i++)
                {
                    tablica[j]=wzor_siatki[i];
                    j++;
                }

                if (suma_procesow != 1)
                {
                    int odbior1[WYMIAR];
                    int odbior2[WYMIAR];
                    int wyslij1[WYMIAR];
                    int wyslij2[WYMIAR];
                    if (ID%2 == 0)
                    {
                        for (int i=0; i<WYMIAR; i++)
                        {
                            wyslij1[i] = tablica[i + WYMIAR];
                        }
                        MPI_Send(&wyslij1, WYMIAR, MPI_INT, mod(ID - 1, suma_procesow), 1, MPI_COMM_WORLD);

                        for (int i = 0; i < WYMIAR; i++)
                        {
                            wyslij2[i] = tablica[(WYMIAR * (WYMIAR / suma_procesow)) + i];
                        }
                        MPI_Send(&wyslij2, WYMIAR, MPI_INT, mod(ID + 1, suma_procesow), 1, MPI_COMM_WORLD);
                    }
                    else
                    {
                        MPI_Recv(&odbior2, WYMIAR, MPI_INT, mod(ID + 1, suma_procesow), 1, MPI_COMM_WORLD, &status);
                        MPI_Recv(&odbior1, WYMIAR, MPI_INT, mod(ID - 1, suma_procesow), 1, MPI_COMM_WORLD, &status);
                    }
                    if (ID%2==0)
                    {
                        MPI_Recv(&odbior2, WYMIAR, MPI_INT, mod(ID + 1, suma_procesow), 1, MPI_COMM_WORLD, &status);
                        MPI_Recv(&odbior1, WYMIAR, MPI_INT, mod(ID - 1, suma_procesow), 1, MPI_COMM_WORLD, &status);
                    }
                    else
                    {
                        for (int i=0; i<WYMIAR; i++)
                        {
                            wyslij1[i] = tablica[i+WYMIAR];
                        }
                        MPI_Send(&wyslij1, WYMIAR, MPI_INT, mod(ID - 1, suma_procesow), 1, MPI_COMM_WORLD);

                        for (int i=0; i<WYMIAR; i++)
                        {
                            wyslij2[i] = tablica[(WYMIAR * (WYMIAR / suma_procesow)) + i];
                        }
                        MPI_Send(&wyslij2, WYMIAR, MPI_INT, mod(ID + 1, suma_procesow), 1, MPI_COMM_WORLD);
                    }
                    for (int i=0; i<WYMIAR; i++)
                    {
                        tablica[i] = odbior1[i];
                        tablica[(WYMIAR * ((WYMIAR / suma_procesow) + 1)) + i] = odbior2[i];
                    }
                }
                else
                {
                    for (int i=0; i<WYMIAR; i++)
                    {
                        tablica[i + SIATKA + WYMIAR] = wzor_siatki[i];
                    }
                    for (int i = SIATKA; i < SIATKA + WYMIAR; i++)
                    {
                        tablica[i - SIATKA] = wzor_siatki[i - WYMIAR];
                    }
                }
                int *wynik = (int*)malloc(WYMIAR*((WYMIAR/suma_procesow))*sizeof(int));

                for (int k=WYMIAR; k<WYMIAR*((WYMIAR/suma_procesow)+1); k++)
                {
                    int wszystkie_wiersze = WYMIAR * (WYMIAR / suma_procesow) + 2;
                    int wiersz = k / WYMIAR;
                    int kolumna = k % WYMIAR;
                    int poprzedni_wiersz = mod(wiersz - 1, wszystkie_wiersze);
                    int poprzednia_kolumna = mod(kolumna - 1, WYMIAR);
                    int nastepny_wiersz = mod(wiersz + 1, wszystkie_wiersze);
                    int nastepna_kolumna = mod(kolumna + 1, WYMIAR);

                    int policz = tablica[poprzedni_wiersz * WYMIAR + poprzednia_kolumna] + tablica[poprzedni_wiersz * WYMIAR + kolumna] + tablica[poprzedni_wiersz * WYMIAR + nastepna_kolumna] + tablica[wiersz * WYMIAR + poprzednia_kolumna] + tablica[wiersz * WYMIAR + nastepna_kolumna] + tablica[nastepny_wiersz * WYMIAR + poprzednia_kolumna] + tablica[nastepny_wiersz * WYMIAR + kolumna] + tablica[nastepny_wiersz * WYMIAR + nastepna_kolumna];
                    if (tablica[k] == 1)
                    {
                        if (policz < 2)
                            wynik[k - WYMIAR] = 0;
                        else if (policz > 3)
                            wynik[k - WYMIAR] = 0;
                        else
                            wynik[k - WYMIAR] = 1;
                    }
                    else
                    {
                        if (policz == 3)
                            wynik[k - WYMIAR] = 1;
                        else
                            wynik[k - WYMIAR] = 0;
                    }
                }

                j = 0;
                for (int i=ID*(SIATKA/suma_procesow); i<(ID+1)*(SIATKA/suma_procesow); i++)
                {
                    wzor_siatki[i] = wynik[j];
                    j++;
                }
                MPI_Gather(wynik, WYMIAR * (WYMIAR / suma_procesow), MPI_INT, &wzor_siatki, WYMIAR * (WYMIAR / suma_procesow), MPI_INT, 0, MPI_COMM_WORLD);

                if (ID == 0)
                {
                    printf("\nIteracja %d: Siatka:\n", iter);
                    for (j = 0; j < SIATKA; j++)
                    {
                        if (j % WYMIAR == 0)
                        {
                            printf("\n");
                        }
                        printf("%d  ", wzor_siatki[j]);
                    }
                    printf("\n");
                }
            }

Uwolnienie tablicy z pamięci. 
Na koniec funkcja, która kończy pracę w trybie MPI:

        free(tablica);
        MPI_Finalize();
    }
