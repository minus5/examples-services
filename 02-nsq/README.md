# Primjer sustava koji komunicira NSQ-om

Servisi iz prethodnog primjera su prera]eni da komuniciraju putem NSQ-a. 

Pokretanje primjera:

```
./build.rb binary_all
goreman start
```

Nakon pokretanja nsqadmin sučelje je dostupno na <http://localhost:4171/>.

Poruke sa nekog topica se mogu pratiti nsq_tail alatom:

```
nsq_tail -lookupd-http-address=127.0.0.1:4161 -topic sensor_values
```

Dobar primjer distribucije loada se može vidjeti pokretanjem dodatnog workera.

```
./worker/worker
```