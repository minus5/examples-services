# Primjer sustava koji komunicira HTTP-om

Dummy sustav koji se sastoji od tri servisa:

- [sensor](./sensor) koji generira nizove random vrijednosti i poslužuje ih na HTTP interfaceu
- [worker](./worker) koji zna zbrojiti niz brojeva (također exposea HTTP interface)
- [app](./app) je aplikacija koja svake sekunde pročita podatke sa sensora, pošalje ih na workera i pohrani rezultat u log

Pokretanje primjera:

```
./build.rb binary_all
goreman start
```
