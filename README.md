![MikroE](http://www.mikroe.com/img/designs/beta/logo_small.png)

---

# Cloud demo for ECG_3 Click

---

- **CIC Prefix**  : G2C_ECG3
- **Author**      : Nemanja Medakovic
- **Verison**     : 1.0.0
- **Date**        : Jan 2019.

---

- **Libstock** : https://libstock.mikroe.com/projects/view/2740/ecg-3-cloud-demo

---
#### Package link
---

- **ECG_3 package ** : https://libstock.mikroe.com/projects/view/2599/ecg-3-click
- **G2C package**    : https://libstock.mikroe.com/projects/view/2628/g2c-click

---

**Examples Description**

The application is composed of three sections :

- System Initialization - Initializes peripherals and pins.
  MIKROBUS1 is reserved for G2C click and MIKROBUS2 is reserved for ECG 3 click.
- Application Initialization - Initializes UART and SPI interface used for the communication with the G2C click
  and ECG 3 click, respectively.
  Commands are sent to prepare the module for communication with the cloud.
- Application Task - Performs in parallel a core state machine process and a interrupt timer state check.
  When the interrupt timer reaches the desired value, heart rate and R-R interval value will be read and both values 
  will be sent to the cloud.
  After that the interrupt timer will be started from the beggining.


```.c
void applicationTask()
{
    char demoBuffer[ 50 ];
    uint16_t rr_interval;
    uint16_t heart_rate;

// CORE STATE MACHINE
    g2c_process();

// SENSOR DEMO
    if (taskTime > 500)
    {
        taskTime = 0;

        ecg3_getRTOR( &heart_rate, &rr_interval );
        
        if (heart_rate <= 300)
        {
            WordToStr( heart_rate, &demoBuffer[0] );
            Ltrim( &demoBuffer[0] );
            g2c_packCmd( &_AT_DSET[0], &ECG_3_hr_ref[0], &demoBuffer[0]);
            g2c_cmdSingle( &_AT_PUB[0] );

            WordToStr( rr_interval, &demoBuffer[0] );
            Ltrim( &demoBuffer[0] );
            g2c_packCmd( &_AT_DSET[0], &ECG_3_rr_ref[0], &demoBuffer[0]);
            g2c_cmdSingle( &_AT_PUB[0] );
        }
    }
}
```

### Upustvo za projekat ECG 3 - Cloud demo


**Potreban HW**

- Razvojni sistem po zelji sa MCU karticom (npr. Clicker 2 for STM32)
- G2C click
- ECG 3 click sa ECG sondama


**Potreban SW**

- Demo paket ECG 3 - Cloud projekta  (https://libstock.mikroe.com/projects/view/2740/ecg-3-cloud-demo)
- Source paket za G2C click  (https://libstock.mikroe.com/projects/view/2628/g2c-click)
- Source paket za ECG 3 click  (https://libstock.mikroe.com/projects/view/2599/ecg-3-click)
- MikroSDK paket za odabrani kompajler  (https://libstock.mikroe.com/projects/view/2249/mikrosdk)
- Cloud web aplikacija  (https://app.clickcloud.io/#/login)


**HW priprema za rad**

Odabrati zeljeni razvojni sistem (u nasem primeru koriscen je Clicker 2 for STM32).
Jedan mikrobus ce biti iskoriscen za G2C click, za koji je potreban UART interfejs i odredjeni pinovi.
Drugi mikrobus ce biti iskoriscen za ECG 3 click, za koji je potreban SPI interfejs i odredjeni pinovi.
Za ispravno merenje vrednosti Heart Rate signala potrebno je prisustvo odgovarajucih ECG sondi na ECG 3 kliku, 
koje moraju biti propisno postavljene na ljudsko telo.


**SW priprema za rad**

Instalirati sve gore navedene source i demo pakete za odabrani razvojni sistem.
Paketi i biblioteke ce biti vidljivi i selektovani u odabranom kompajleru (library manager) prilikom pokretanja demo projekta.
Svi navedeni paketi su neophodni za pravilan rad i ispravnost projekta.
Takodje kod za demo primer ce biti prikazan korisniku, gde se mogu pravilno iskonfigurisati oba pomenuta klika i potrebne MCU periferije i pinovi.
U nasem default-nom primeru MIKROBUS1 je kosriscen za G2C click, a MIKROBUS2 za ECG 3 click.
U samom demo kodu je takodje izvrsena default-na konfiguracija oba klika, kao i konekcija sa cloud web aplikacijom putem zeljene WiFi mreze, 
cije parametre je potrebno uneti u demo kod (network name i password).
Logovanje rezultata bice izvrseno na cloud web aplikaciji, ali je prethodno potrebno napraviti zeljeni device u samoj aplikaciji.


**Priprema Cloud web aplikacije**

*Kreiranje novog device-a na cloud-u*

Izvrsiti logovanje na cloud web aplikaciju (na gore postavljen link).
Izvrsiti selekciju Devices taska.
Izabrati opciju Add device i zatim opciju Create device u iskacucem prozoru.
Dobija se novi prozor koji se sastoji iz dva koraka.
U prvom koraku izvrsiti selekciju device manifesta, u nasem slucaju izabrati ECG 3 click device manifest.
U drugom koraku uneti zeljeni naziv device-a i izabrati opciju Save.
Nakon opcije Save dobija se iskacuci prozor sa izgenerisanim parametrima za novi device, koje je potrebno uneti u demo kod (device key i password).
Izgenerisane parametre za novi device je pozeljno i sacuvati.
Ako su prethodni koraci svi ispostovani navedenim redosledom, novi device je uspesno kreiran i trebao bi biti vidljiv u tasku Devices.
Sada je potrebno kreirati dashboard za dobijeni device, gde ce heart rate vrednost i ostali rezultati biti logovani.

*Kreiranje dashboard-a za dobijeni device*

Izvrsiti selekciju Dashboards taska.
Izabrati opciju Add new dashboard, dati mu naziv i izabrati opciju Save.
Nakon toga pojavljuje se prazan prozor za kreirani dashboard, koji je moguce popuniti odgovarajucim zeljenim widget-ima izborom opcije Add new widget.
Svakom widget-u je moguce dodeliti opciju koju ce ponudjenu vrednost da prikazuje za selektovani device.
Widget moze da bude kartica sa vrednostima merenja, grafik, aktuator...
U nasem projektu zelimo da prikazemo rezultate merenja Heart Rate signala i R-R intervala sa ECG 3 klika (device-a).
Kada je dashboard pravilno uredjen, sada se mozemo vratiti na samo logovanje rezultata merenja na cloud.


**Logovanje rezultata ECG 3 senzora na cloud**

Posto su HW, SW i cloud aplikacija pripremljeni za rad, sada je moguce pokrenuti kod i logovanje rezultata na pomenutu cloud web aplikaciju.
Oba modula ce biti resetovana i bice poslata komanda za uspostavljanje konekcije G2C klika sa selektovanom mrezom.
Demo kod radi tako sto za tacno odredjeno vreme tajmerskog interapta pokrece citanje Heart Rate vrednosti u bpm (beats per minute) i vrednosti R-R intervala u ms (milliseconds).
Ovo citanje se vrsi putem SPI interfejsa sa ECG 3 klika na mikrobasu 2.
Zatim se iscitane vrednosti salju pomocu G2C klika putem UART interfejsa na mikrobasu 1 i unesene WiFi mreze na cloud aplikaciju.
Poslate vrednosti rezultata trebale bi da se menjaju sada na widget-ima cloud aplikacije, sto znaci da je logovanje merenja heart rate senzora na cloud uspesno izvrseno.


**Project demo test**

Primer testiranja ovakvog jednog projekta moze biti izvrseno, kako je i gore napomenuto, na clickeru 2 for STM32.
Na pomenuti razvojni sistem je potrebno spustiti demo kod sa unesenim parametrima za network i prethodno kreirani device.
Da bi ceo sistem bio mobilan predlaze se stavljanje baterije kao izvor napajanja sistema.
Prisustvo signala unesene WiFi mreze je takodje neophodno, da bi rezultati mogli da se salju na web aplikaciju.
G2C klik potrebno je postaviti na jedan mikrobus, a ECG 3 klik senzor na drugi, onako kako je u samom kodu selektovano.
Potrebno je takodje i izvrsiti pravilan raspored sondi na ljudskom telu, radi sto bolje ispravnosti merenja heart rate signala.
Kada se uspostavi konekcija G2C klika sa mrezom (zuta - CONN ledovka na kliku), posle odredjenog (zadatog) vremenskog perioda pocinje slanje rezultata merenja sa senzora na cloud 
u tacno odredjenim vremenskim intervalima.
Ceo sistem je sada umrezen sa cloud web aplikacijom i mobilan, a rezultati ce svakog trenutka gde god se sistem zatekne biti vidljivi na widget-ima cloud aplikacije.
Ovakav demo projekat omogucuje posmatranje rezultata heart rate signala odredjene osobe na web aplikaciji, bilo da se ta osoba krece, ili miruje, gde god da se zatekne, 
pod uslovom da su gore navedeni uslovi za pravilan rad projekta svi redom ispostovani.

#### Cloud

| Sensor | Actuator | Reference | Measurement unit | Range - min  | Range - max |
|:------:|:--------:|:-----:|:-----:|:------------:|:-----------:|
| x | x | ECG3_X | X | X | X | 
| x | x | ECG3_X | X | X | X | 
---
