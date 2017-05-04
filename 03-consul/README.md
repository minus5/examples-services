# Dodavanje service discovery-a (Consul)

U prethodni primjer je dodan Consul koji se pokreće sa fiksno definiranom konfiguracijom. U konfiguraciji su fiksno definirane adresa i port svakog pokrenutog servisa.

U produkciji se svaki servis registrira sam na Consul pomoću [consul-registratora](https://github.com/gliderlabs/registrator) koji zna reagirati na sve promjene na docker hostu i dojavljivati ih Consulu.

U ovom primjeru se više ne spajamo direktno na nsqd nego se prvo resolve-a njegova adresa putem Consula.

S pokretanjem aplikacije (goreman) pokrenut će se i Consul koji će otvoriti api:

- Consul UI <http://localhost:8500/>
- lista registriranih servisa <http://localhost:8500/v1/catalog/services>
- informacije o pojedinom servisu <http://localhost:8500/v1/catalog/service/nsqd>
- informacije o zdravlju servisa <http://localhost:8500/v1/health/checks/sensor>
