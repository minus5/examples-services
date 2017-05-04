# Kontejnerizacija servisa pomoću Dockera

U ovom primjeru se za svaki pojedni servis (sensor, worker, app) napraviti novi Docker image. 

Za buildanje servisa se koristi source iz [prethodnog primjera](../03-consul). Da bi se servis mogao pokrenuti unutar linux conatinera aplikacija sa builda za linux arhitekturu.

Za nsq i consul servise se koriste image-i koji su dostupni na [docker hub-u](https://hub.docker.com/).

Primjer se pokreće pomoću docker-compose-a:

```
# bulida sve linux binaries
./build.rb binary_all

# builda sve Docker images
./build.rb image_all

cd datacenters/dev/host1
docker-compose up
```

Svi servisi pronađu Consul pomoću enviromnent varijable CONSUL_ADDR koja im se postavi u docker-compose.yml. Nakon toga pitaju Consul za adresu nsqd-a. Nakon toga se spajaju na nsqd i mogu početi sa slanjem/obradom poruka.
