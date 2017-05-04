# Dodavanje service discovery-a (Consul)

U prethodni primjer je dodan Consul koji se pokreće sa fiksno definiranom konfiguracijom. Svakom servisu su fiksno definirani adresa i port na kojem se nalaze.

Svakom servisu (sensor, worker, app) je dodan /health_check HTTP interface kako bi mogao javljati Consulu svoje stanje.

U produkciji ne adrese servisa ne definiraju fiksno nego se svaki servis kod pokretanja registrira na Consul pomoću [consul-registratora](https://github.com/gliderlabs/registrator) koji zna očitati promjene na docker hostu i dojavljivati ih Consulu.

U ovom primjeru se više ne spajamo direktno na nsqd nego se prvo resolve-a njegova adresa putem Consula.

S pokretanjem aplikacije (goreman) pokrenut će se i Consul koji će otvoriti svoj api:

- Consul UI <http://localhost:8500/>
- lista registriranih servisa <http://localhost:8500/v1/catalog/services>
- informacije o pojedinom servisu <http://localhost:8500/v1/catalog/service/nsqd>
- informacije o zdravlju servisa <http://localhost:8500/v1/health/checks/sensor>
