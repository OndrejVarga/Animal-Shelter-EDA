# Exploračná analýza dat

# Popis Notebooku
* Nasledujúci notebook vznikol ako domáci projekt pre zimný predmet **Vizualizace dát (2023)** na ČVUT.     
* Obsahuje exploračnú analýzu datasetov
    * [intakes](https://data.austintexas.gov/Health-and-Community-Services/Austin-Animal-Center-Intakes/wter-evkm), ktorý obsahuje informácie o  zvieratách prijatých do útulku od 1. 10. 2013 do 27.4 2022
    *  [outcomes](https://data.austintexas.gov/Health-and-Community-Services/Austin-Animal-Center-Outcomes/9t4d-g238), ktorý obsahuje údaje o zvieratách, které útulok opustili.

* Notebook je zložený z 3 častí
    1. Príprava dát
    2. Deskriptívne štatistiky príznakov
        * univariačná a multivariačná analýza jednotlivých príznakov
    3. Vizualizácia odpovedí na rôzne otázky pomocou dát.

# 1. Príprava dát
Načítanie potrebných knižníc.


```python
import pandas as pd
from pandas.api.types import CategoricalDtype
import numpy as np
import seaborn as sns
import matplotlib.pyplot as plt
import matplotlib.patches as mpatches
import missingno as msno
from datetime import date
sns.set_theme(style='whitegrid', palette="pastel")   
```

Načítanie potrebných datasetov.


```python
intakes = pd.read_csv('intakes.csv')
outcomes = pd.read_csv('outcomes.csv')
```

## Datasety
Prípravu dát začneme získaním základných informácii o datasete. Zaujímať nás bude jeho veľkosť, počet záznamov (existencia duplicít) a počet príznakov. 

Pre každý príznak budeme zisťovať jeho dátový typ a počet chýbajúcich údajov.


```python
print(f'Dataset intakes obsahuje {intakes.shape[0]} záznamov a {intakes.shape[1]} príznakov')
print(f'Dataset outcomes obsahuje {outcomes.shape[0]} záznamov a {outcomes.shape[1]} príznakov')
```

    Dataset intakes obsahuje 138585 záznamov a 12 príznakov
    Dataset outcomes obsahuje 138769 záznamov a 12 príznakov



```python
print('Duplikáty')
print(f'intakes: {intakes[intakes.duplicated()].shape}')
print(f'outcomes: {outcomes[outcomes.duplicated()].shape}')
```

    Duplikáty
    intakes: (20, 12)
    outcomes: (17, 12)


Oba datasety obsahujú duplicitné záznamy, ktoré odstránime. 


```python
intakes = intakes.drop_duplicates()
outcomes = outcomes.drop_duplicates()
```

Zistíme dátové typy jednotlivých príznakov a počet chýbajúcich hodnôt. 

Vytvoríme si funkciu, ktorá vypíše všetky príznaky v datasete, ktorým chýba aspoň jedna hodnota. Chýbajúce hodnoty v celom datasete vizualizujeme.


```python
def missing_vals(dataset :pd.DataFrame) -> None:
    """Print the features containing any missing values."""
    data = dataset.isna().any()

    # Get all columns containing missing values
    features = []
    for index, val in enumerate(data):
        if val:
            features.append(data.index[index])
    print(features)
```


```python
print('Intakes')
intakes.info()
```

    Intakes
    <class 'pandas.core.frame.DataFrame'>
    Index: 138565 entries, 0 to 138584
    Data columns (total 12 columns):
     #   Column            Non-Null Count   Dtype 
    ---  ------            --------------   ----- 
     0   Animal ID         138565 non-null  object
     1   Name              97300 non-null   object
     2   DateTime          138565 non-null  object
     3   MonthYear         138565 non-null  object
     4   Found Location    138565 non-null  object
     5   Intake Type       138565 non-null  object
     6   Intake Condition  138565 non-null  object
     7   Animal Type       138565 non-null  object
     8   Sex upon Intake   138564 non-null  object
     9   Age upon Intake   138565 non-null  object
     10  Breed             138565 non-null  object
     11  Color             138565 non-null  object
    dtypes: object(12)
    memory usage: 13.7+ MB



```python
print('Chýbajúce hodnoty:')
missing_vals(intakes)
```

    Chýbajúce hodnoty:
    ['Name', 'Sex upon Intake']



```python
#https://stackoverflow.com/questions/63869715/how-to-show-legend-in-missingno-matrix
ax = msno.matrix(intakes, labels=True, sparkline=False)
ax.set_title('Nullity matrix of the Intakes Dataset',{'fontsize':25,})
ax.legend(handles=[mpatches.Patch(color='gray', label='Present Data'), 
                    mpatches.Patch(color='white', label='Missing Data')], 
                    bbox_to_anchor=(1,1), fontsize=10)
ax.set_ylabel('Row Index',{'fontsize':18})
ax.set_xlabel('Column',{'fontsize':18})
```




    Text(0.5, 0, 'Column')




    
![png](./README-files/output_15_1.png)
    


Z predchádzajúcich buniek sme zistili, že dataset obsahuje 12 príznakov. Z grafu je možné vyčítať, že počet chýbajúcich dát je nízky v `Sex upon Intake` a naopak vysoký v `Name`. Ostatné príznaky neobsahujú chýbajúce hodnoty. 

Rovnako budeme postupovať aj v datasete outcomes.


```python
print('Outcomes')
outcomes.info()
```

    Outcomes
    <class 'pandas.core.frame.DataFrame'>
    Index: 138752 entries, 0 to 138768
    Data columns (total 12 columns):
     #   Column            Non-Null Count   Dtype 
    ---  ------            --------------   ----- 
     0   Animal ID         138752 non-null  object
     1   Name              97502 non-null   object
     2   DateTime          138752 non-null  object
     3   MonthYear         138752 non-null  object
     4   Date of Birth     138752 non-null  object
     5   Outcome Type      138729 non-null  object
     6   Outcome Subtype   63427 non-null   object
     7   Animal Type       138752 non-null  object
     8   Sex upon Outcome  138751 non-null  object
     9   Age upon Outcome  138747 non-null  object
     10  Breed             138752 non-null  object
     11  Color             138752 non-null  object
    dtypes: object(12)
    memory usage: 13.8+ MB



```python
print('Chýbajúce hodnoty:')
missing_vals(outcomes)
```

    Chýbajúce hodnoty:
    ['Name', 'Outcome Type', 'Outcome Subtype', 'Sex upon Outcome', 'Age upon Outcome']



```python
ax = msno.matrix(outcomes, sparkline=False)
ax.set_title('Nullity matrix of the Outcomes Dataset',{'fontsize':25})
ax.legend(handles=[mpatches.Patch(color='gray', label='Present Data'), 
                    mpatches.Patch(color='white', label='Missing Data')], 
                    bbox_to_anchor=(1,1), fontsize=10)
ax.set_ylabel('Row Index',{'fontsize':18})
ax.set_xlabel('Column',{'fontsize':18})
```




    Text(0.5, 0, 'Column')




    
![png](./README-files/output_19_1.png)
    


Dataset outcomes obsahuje 12 príznakov. Všetky sú reprezentované ako reťazce. Chýbajú hodnoty príznakov `Name`, `Outcome Type`, `Outcome Subtype`, `Sex upon Outcome`, `Age upon Outcome`. Z grafu je možné vyčítať, že okrem príznakov `Name` a `Outcome Subtype` je počet chýbajúcich hodnôt minimálny (počet chýbajúcich hodnôt je dostatočne malý na to, aby v grafe nebol pozorovateľný).

## Príznaky
Každému príznaku priradíme jeho typ. Ten zistíme výpisom unikátnych hodnôt. Vyšetríme, či hodnoty zodpovedajú očakávanému rozsahu a či je dodržaná integrita a konzistencia dát.


```python
def print_feature(dataset: pd.DataFrame, n=5) -> None:
    """Helper function that prints the number of unique values and lists first n unique values"""
    for col in dataset.columns:
        unique_vals = dataset[col].unique()
        # Column name
        print(col)
        print(f'Unikátne hodnoty: {unique_vals.size}')
        print(unique_vals[:n])
        print(30*'-')
```


```python
intakes.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Animal ID</th>
      <th>Name</th>
      <th>DateTime</th>
      <th>MonthYear</th>
      <th>Found Location</th>
      <th>Intake Type</th>
      <th>Intake Condition</th>
      <th>Animal Type</th>
      <th>Sex upon Intake</th>
      <th>Age upon Intake</th>
      <th>Breed</th>
      <th>Color</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>A786884</td>
      <td>*Brock</td>
      <td>01/03/2019 04:19:00 PM</td>
      <td>January 2019</td>
      <td>2501 Magin Meadow Dr in Austin (TX)</td>
      <td>Stray</td>
      <td>Normal</td>
      <td>Dog</td>
      <td>Neutered Male</td>
      <td>2 years</td>
      <td>Beagle Mix</td>
      <td>Tricolor</td>
    </tr>
    <tr>
      <th>1</th>
      <td>A706918</td>
      <td>Belle</td>
      <td>07/05/2015 12:59:00 PM</td>
      <td>July 2015</td>
      <td>9409 Bluegrass Dr in Austin (TX)</td>
      <td>Stray</td>
      <td>Normal</td>
      <td>Dog</td>
      <td>Spayed Female</td>
      <td>8 years</td>
      <td>English Springer Spaniel</td>
      <td>White/Liver</td>
    </tr>
    <tr>
      <th>2</th>
      <td>A724273</td>
      <td>Runster</td>
      <td>04/14/2016 06:43:00 PM</td>
      <td>April 2016</td>
      <td>2818 Palomino Trail in Austin (TX)</td>
      <td>Stray</td>
      <td>Normal</td>
      <td>Dog</td>
      <td>Intact Male</td>
      <td>11 months</td>
      <td>Basenji Mix</td>
      <td>Sable/White</td>
    </tr>
    <tr>
      <th>3</th>
      <td>A665644</td>
      <td>NaN</td>
      <td>10/21/2013 07:59:00 AM</td>
      <td>October 2013</td>
      <td>Austin (TX)</td>
      <td>Stray</td>
      <td>Sick</td>
      <td>Cat</td>
      <td>Intact Female</td>
      <td>4 weeks</td>
      <td>Domestic Shorthair Mix</td>
      <td>Calico</td>
    </tr>
    <tr>
      <th>4</th>
      <td>A682524</td>
      <td>Rio</td>
      <td>06/29/2014 10:38:00 AM</td>
      <td>June 2014</td>
      <td>800 Grove Blvd in Austin (TX)</td>
      <td>Stray</td>
      <td>Normal</td>
      <td>Dog</td>
      <td>Neutered Male</td>
      <td>4 years</td>
      <td>Doberman Pinsch/Australian Cattle Dog</td>
      <td>Tan/Gray</td>
    </tr>
  </tbody>
</table>
</div>




```python
outcomes.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Animal ID</th>
      <th>Name</th>
      <th>DateTime</th>
      <th>MonthYear</th>
      <th>Date of Birth</th>
      <th>Outcome Type</th>
      <th>Outcome Subtype</th>
      <th>Animal Type</th>
      <th>Sex upon Outcome</th>
      <th>Age upon Outcome</th>
      <th>Breed</th>
      <th>Color</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>A794011</td>
      <td>Chunk</td>
      <td>05/08/2019 06:20:00 PM</td>
      <td>May 2019</td>
      <td>05/02/2017</td>
      <td>Rto-Adopt</td>
      <td>NaN</td>
      <td>Cat</td>
      <td>Neutered Male</td>
      <td>2 years</td>
      <td>Domestic Shorthair Mix</td>
      <td>Brown Tabby/White</td>
    </tr>
    <tr>
      <th>1</th>
      <td>A776359</td>
      <td>Gizmo</td>
      <td>07/18/2018 04:02:00 PM</td>
      <td>Jul 2018</td>
      <td>07/12/2017</td>
      <td>Adoption</td>
      <td>NaN</td>
      <td>Dog</td>
      <td>Neutered Male</td>
      <td>1 year</td>
      <td>Chihuahua Shorthair Mix</td>
      <td>White/Brown</td>
    </tr>
    <tr>
      <th>2</th>
      <td>A821648</td>
      <td>NaN</td>
      <td>08/16/2020 11:38:00 AM</td>
      <td>Aug 2020</td>
      <td>08/16/2019</td>
      <td>Euthanasia</td>
      <td>NaN</td>
      <td>Other</td>
      <td>Unknown</td>
      <td>1 year</td>
      <td>Raccoon</td>
      <td>Gray</td>
    </tr>
    <tr>
      <th>3</th>
      <td>A720371</td>
      <td>Moose</td>
      <td>02/13/2016 05:59:00 PM</td>
      <td>Feb 2016</td>
      <td>10/08/2015</td>
      <td>Adoption</td>
      <td>NaN</td>
      <td>Dog</td>
      <td>Neutered Male</td>
      <td>4 months</td>
      <td>Anatol Shepherd/Labrador Retriever</td>
      <td>Buff</td>
    </tr>
    <tr>
      <th>4</th>
      <td>A674754</td>
      <td>NaN</td>
      <td>03/18/2014 11:47:00 AM</td>
      <td>Mar 2014</td>
      <td>03/12/2014</td>
      <td>Transfer</td>
      <td>Partner</td>
      <td>Cat</td>
      <td>Intact Male</td>
      <td>6 days</td>
      <td>Domestic Shorthair Mix</td>
      <td>Orange Tabby</td>
    </tr>
  </tbody>
</table>
</div>




```python
print_feature(intakes)
```

    Animal ID
    Unikátne hodnoty: 123890
    ['A786884' 'A706918' 'A724273' 'A665644' 'A682524']
    ------------------------------
    Name
    Unikátne hodnoty: 23545
    ['*Brock' 'Belle' 'Runster' nan 'Rio']
    ------------------------------
    DateTime
    Unikátne hodnoty: 97442
    ['01/03/2019 04:19:00 PM' '07/05/2015 12:59:00 PM'
     '04/14/2016 06:43:00 PM' '10/21/2013 07:59:00 AM'
     '06/29/2014 10:38:00 AM']
    ------------------------------
    MonthYear
    Unikátne hodnoty: 103
    ['January 2019' 'July 2015' 'April 2016' 'October 2013' 'June 2014']
    ------------------------------
    Found Location
    Unikátne hodnoty: 58367
    ['2501 Magin Meadow Dr in Austin (TX)' '9409 Bluegrass Dr in Austin (TX)'
     '2818 Palomino Trail in Austin (TX)' 'Austin (TX)'
     '800 Grove Blvd in Austin (TX)']
    ------------------------------
    Intake Type
    Unikátne hodnoty: 6
    ['Stray' 'Owner Surrender' 'Public Assist' 'Wildlife' 'Euthanasia Request']
    ------------------------------
    Intake Condition
    Unikátne hodnoty: 15
    ['Normal' 'Sick' 'Injured' 'Pregnant' 'Nursing']
    ------------------------------
    Animal Type
    Unikátne hodnoty: 5
    ['Dog' 'Cat' 'Other' 'Bird' 'Livestock']
    ------------------------------
    Sex upon Intake
    Unikátne hodnoty: 6
    ['Neutered Male' 'Spayed Female' 'Intact Male' 'Intact Female' 'Unknown']
    ------------------------------
    Age upon Intake
    Unikátne hodnoty: 54
    ['2 years' '8 years' '11 months' '4 weeks' '4 years']
    ------------------------------
    Breed
    Unikátne hodnoty: 2741
    ['Beagle Mix' 'English Springer Spaniel' 'Basenji Mix'
     'Domestic Shorthair Mix' 'Doberman Pinsch/Australian Cattle Dog']
    ------------------------------
    Color
    Unikátne hodnoty: 616
    ['Tricolor' 'White/Liver' 'Sable/White' 'Calico' 'Tan/Gray']
    ------------------------------


#### Rozdelenie príznakov datasetu Intake
##### Kvalitatívne ordinálne príznaky
- Animal ID
##### Kvalitatívne nominálne príznaky
- Name
- Found Location 
- Intake Type 
- Intake Condition
- Animal Type 
- Sex upon Intake
- Breed 
- Color

##### Kvantitatívne intervalové príznaky
- DateTime
- MonthYear
##### Kvantitatívne diskrétne príznaky
- Age upon Intake


```python
print_feature(outcomes)
```

    Animal ID
    Unikátne hodnoty: 124068
    ['A794011' 'A776359' 'A821648' 'A720371' 'A674754']
    ------------------------------
    Name
    Unikátne hodnoty: 23426
    ['Chunk' 'Gizmo' nan 'Moose' 'Princess']
    ------------------------------
    DateTime
    Unikátne hodnoty: 115364
    ['05/08/2019 06:20:00 PM' '07/18/2018 04:02:00 PM'
     '08/16/2020 11:38:00 AM' '02/13/2016 05:59:00 PM'
     '03/18/2014 11:47:00 AM']
    ------------------------------
    MonthYear
    Unikátne hodnoty: 103
    ['May 2019' 'Jul 2018' 'Aug 2020' 'Feb 2016' 'Mar 2014']
    ------------------------------
    Date of Birth
    Unikátne hodnoty: 7576
    ['05/02/2017' '07/12/2017' '08/16/2019' '10/08/2015' '03/12/2014']
    ------------------------------
    Outcome Type
    Unikátne hodnoty: 10
    ['Rto-Adopt' 'Adoption' 'Euthanasia' 'Transfer' 'Return to Owner']
    ------------------------------
    Outcome Subtype
    Unikátne hodnoty: 27
    [nan 'Partner' 'Foster' 'SCRP' 'Out State']
    ------------------------------
    Animal Type
    Unikátne hodnoty: 5
    ['Cat' 'Dog' 'Other' 'Bird' 'Livestock']
    ------------------------------
    Sex upon Outcome
    Unikátne hodnoty: 6
    ['Neutered Male' 'Unknown' 'Intact Male' 'Spayed Female' 'Intact Female']
    ------------------------------
    Age upon Outcome
    Unikátne hodnoty: 55
    ['2 years' '1 year' '4 months' '6 days' '7 years']
    ------------------------------
    Breed
    Unikátne hodnoty: 2749
    ['Domestic Shorthair Mix' 'Chihuahua Shorthair Mix' 'Raccoon'
     'Anatol Shepherd/Labrador Retriever'
     'American Foxhound/Labrador Retriever']
    ------------------------------
    Color
    Unikátne hodnoty: 619
    ['Brown Tabby/White' 'White/Brown' 'Gray' 'Buff' 'Orange Tabby']
    ------------------------------


#### Rozdelenie príznakov datasetu Outcome
##### Kvalitatívne ordinálne príznaky
- Animal ID 
##### Kvalitatívne nominálne príznaky
- Name
- Outcome Type
- Outcome Subtype
- Animal Type 
- Sex upon Outcome
- Breed 
- Color

##### Kvantitatívne intervalové príznaky
- DateTime
- MonthYear
- Date of Birth
##### Kvantitatívne diskrétne príznaky
- Age upon Outcome

Každú kategóriu spracujeme samostatne. 

Pre každý príznak budeme
- pretypovávať príznak na vhodný dátový typ
- kontrolovať rozsah hodnôt (pomocou ktorého by som mohol odhaliť chyby v dátach)
- overovať, či hodnoty používajú jednotný formát
- overovať dátovú integritu v prípade, ak príznaky majú vzťah s inými príznakmi
- riešiť problém chýbajúcich hodnôt

### Kvantitatívne intervalové príznaky
Príznak `DateTime` je intervalový priznak. Výpisom prvých pár hodnôt zistíme formát, v ktorom sú údaje uložené a ten použijeme pri konvertovaní dát na *datetime*.


```python
display(intakes[['DateTime']].head())
outcomes[['DateTime']].head()
```


<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>DateTime</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>01/03/2019 04:19:00 PM</td>
    </tr>
    <tr>
      <th>1</th>
      <td>07/05/2015 12:59:00 PM</td>
    </tr>
    <tr>
      <th>2</th>
      <td>04/14/2016 06:43:00 PM</td>
    </tr>
    <tr>
      <th>3</th>
      <td>10/21/2013 07:59:00 AM</td>
    </tr>
    <tr>
      <th>4</th>
      <td>06/29/2014 10:38:00 AM</td>
    </tr>
  </tbody>
</table>
</div>





<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>DateTime</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>05/08/2019 06:20:00 PM</td>
    </tr>
    <tr>
      <th>1</th>
      <td>07/18/2018 04:02:00 PM</td>
    </tr>
    <tr>
      <th>2</th>
      <td>08/16/2020 11:38:00 AM</td>
    </tr>
    <tr>
      <th>3</th>
      <td>02/13/2016 05:59:00 PM</td>
    </tr>
    <tr>
      <th>4</th>
      <td>03/18/2014 11:47:00 AM</td>
    </tr>
  </tbody>
</table>
</div>




```python
intakes['DateTime'] = pd.to_datetime(intakes['DateTime'], format="%m/%d/%Y %I:%M:%S %p")
outcomes['DateTime'] = pd.to_datetime(outcomes['DateTime'], format="%m/%d/%Y %I:%M:%S %p")
```


```python
def column_range(data: pd.DataFrame):
    """Print Min, Max values of series"""
    print(f'Min: {data.min()}')
    print(f'Max: {data.max()}')
```


```python
column_range(intakes['DateTime'])
```

    Min: 2013-10-01 07:51:00
    Max: 2022-04-27 07:54:00



```python
column_range(outcomes['DateTime'])
```

    Min: 2013-10-01 09:31:00
    Max: 2022-04-26 18:41:00


Príznak `DateTime` udáva dátum a čas vytvorenia záznamu. Neobsahuje žiadne chýbajúce hodnoty, má správny dátový typ (datetime), ktorý zároveň zabezpečuje dátovú konzistenciu. Rozsah neodhalil chybné medzné hodnoty.  

Nasleduje príznak `MonthYear`.  Príznak pretypujeme na *datetime*.


```python
intakes['MonthYear'] = pd.to_datetime(intakes['MonthYear'], format='%B %Y')
outcomes['MonthYear'] = pd.to_datetime(outcomes['MonthYear'], format='%b %Y')
```

Z predchádzajúcich výpisov sme vypozorovali, že príznak obsahuje mesiac a rok, ktorý sa nachádza v `DateTime`. Overíme si, či tento vzťah je platný pre všetky záznamy (budeme hľadať záznamy, v ktorých tento vzťah neplatí).


```python
print('Neočakávané dáta v intakes:', intakes[
    (intakes['MonthYear'].dt.month != intakes['DateTime'].dt.month) & 
    (intakes['MonthYear'].dt.year != intakes['DateTime'].dt.year)].shape[0])

print('Neočakávané dáta v outcomes:',outcomes[
    (outcomes['MonthYear'].dt.month != outcomes['DateTime'].dt.month) & 
    (outcomes['MonthYear'].dt.year != outcomes['DateTime'].dt.year)].shape[0])
```

    Neočakávané dáta v intakes: 0
    Neočakávané dáta v outcomes: 0


`MonthYear` obsahuje redundantné informácie, ktoré sú obsiahnuté v `DateTime`, preto stĺpec z datasetu odstránime.


```python
intakes = intakes.drop('MonthYear', axis=1)
outcomes = outcomes.drop('MonthYear', axis=1)
```

Posledným intervalovým príznakom je `Date of Birth`, ktorý prekonvertujeme na *datetime*.


```python
outcomes[['Date of Birth']].head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Date of Birth</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>05/02/2017</td>
    </tr>
    <tr>
      <th>1</th>
      <td>07/12/2017</td>
    </tr>
    <tr>
      <th>2</th>
      <td>08/16/2019</td>
    </tr>
    <tr>
      <th>3</th>
      <td>10/08/2015</td>
    </tr>
    <tr>
      <th>4</th>
      <td>03/12/2014</td>
    </tr>
  </tbody>
</table>
</div>




```python
outcomes['Date of Birth'] = pd.to_datetime(outcomes['Date of Birth'], format='%m/%d/%Y')
column_range(outcomes['Date of Birth'])
```

    Min: 1991-09-22 00:00:00
    Max: 2022-04-24 00:00:00


`Date of Birth` je príznak, ktorý zachytáva dátum narodenia zvieraťa. Rozsah neodhalil chybné medzné hodnoty. 

`Date of Birth` je vo vzťahu s príznakom `DateTime`. Každá hodnota `Date of Birth` <=  `DateTime`, pretože zviera nemohlo byť opustiť útulok pred narodením (rovnosť pripúštam v prípade, ak sa zviera narodilo v útulku).


```python
outcomes[outcomes['Date of Birth'] > outcomes['DateTime']]
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Animal ID</th>
      <th>Name</th>
      <th>DateTime</th>
      <th>Date of Birth</th>
      <th>Outcome Type</th>
      <th>Outcome Subtype</th>
      <th>Animal Type</th>
      <th>Sex upon Outcome</th>
      <th>Age upon Outcome</th>
      <th>Breed</th>
      <th>Color</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>647</th>
      <td>A788866</td>
      <td>NaN</td>
      <td>2019-02-15 12:02:00</td>
      <td>2019-12-06</td>
      <td>Adoption</td>
      <td>NaN</td>
      <td>Dog</td>
      <td>Neutered Male</td>
      <td>0 years</td>
      <td>German Shepherd/Catahoula</td>
      <td>Black Brindle</td>
    </tr>
    <tr>
      <th>1696</th>
      <td>A737397</td>
      <td>Jellybean</td>
      <td>2016-11-05 18:16:00</td>
      <td>2016-11-15</td>
      <td>Adoption</td>
      <td>NaN</td>
      <td>Cat</td>
      <td>Intact Female</td>
      <td>0 years</td>
      <td>Domestic Shorthair Mix</td>
      <td>White/Orange</td>
    </tr>
    <tr>
      <th>4235</th>
      <td>A804197</td>
      <td>NaN</td>
      <td>2019-09-11 18:24:00</td>
      <td>2019-09-12</td>
      <td>Transfer</td>
      <td>Partner</td>
      <td>Cat</td>
      <td>Intact Female</td>
      <td>0 years</td>
      <td>Domestic Shorthair</td>
      <td>Black/White</td>
    </tr>
    <tr>
      <th>9362</th>
      <td>A757376</td>
      <td>Gorda</td>
      <td>2017-09-05 19:25:00</td>
      <td>2019-11-05</td>
      <td>Rto-Adopt</td>
      <td>NaN</td>
      <td>Dog</td>
      <td>Spayed Female</td>
      <td>-2 years</td>
      <td>Miniature Schnauzer Mix</td>
      <td>White</td>
    </tr>
    <tr>
      <th>23563</th>
      <td>A745085</td>
      <td>Keira</td>
      <td>2017-03-13 18:11:00</td>
      <td>2017-10-11</td>
      <td>Adoption</td>
      <td>NaN</td>
      <td>Dog</td>
      <td>Spayed Female</td>
      <td>0 years</td>
      <td>Australian Cattle Dog Mix</td>
      <td>White</td>
    </tr>
    <tr>
      <th>25207</th>
      <td>A834123</td>
      <td>Colt</td>
      <td>2021-05-14 13:09:00</td>
      <td>2021-07-04</td>
      <td>Transfer</td>
      <td>Partner</td>
      <td>Dog</td>
      <td>Neutered Male</td>
      <td>0 years</td>
      <td>Great Pyrenees</td>
      <td>White/Tan</td>
    </tr>
    <tr>
      <th>34746</th>
      <td>A802049</td>
      <td>NaN</td>
      <td>2019-07-16 00:00:00</td>
      <td>2019-07-17</td>
      <td>Euthanasia</td>
      <td>At Vet</td>
      <td>Cat</td>
      <td>Intact Male</td>
      <td>0 years</td>
      <td>Domestic Shorthair</td>
      <td>White</td>
    </tr>
    <tr>
      <th>52867</th>
      <td>A751749</td>
      <td>Juan</td>
      <td>2014-09-10 17:29:00</td>
      <td>2014-12-12</td>
      <td>Adoption</td>
      <td>NaN</td>
      <td>Dog</td>
      <td>Neutered Male</td>
      <td>0 years</td>
      <td>Border Collie Mix</td>
      <td>Black/White</td>
    </tr>
    <tr>
      <th>60898</th>
      <td>A757376</td>
      <td>Gorda</td>
      <td>2018-10-21 19:01:00</td>
      <td>2019-11-05</td>
      <td>Return to Owner</td>
      <td>NaN</td>
      <td>Dog</td>
      <td>Spayed Female</td>
      <td>-1 years</td>
      <td>Miniature Schnauzer Mix</td>
      <td>White</td>
    </tr>
    <tr>
      <th>63725</th>
      <td>A736114</td>
      <td>NaN</td>
      <td>2016-10-04 15:13:00</td>
      <td>2016-10-28</td>
      <td>Transfer</td>
      <td>Partner</td>
      <td>Cat</td>
      <td>Intact Male</td>
      <td>0 years</td>
      <td>Domestic Shorthair Mix</td>
      <td>Orange Tabby</td>
    </tr>
    <tr>
      <th>69747</th>
      <td>A754280</td>
      <td>Chewbacca</td>
      <td>2016-04-11 00:00:00</td>
      <td>2016-07-12</td>
      <td>Transfer</td>
      <td>Partner</td>
      <td>Dog</td>
      <td>Intact Male</td>
      <td>0 years</td>
      <td>Australian Cattle Dog</td>
      <td>Black/Brown</td>
    </tr>
    <tr>
      <th>74596</th>
      <td>A687107</td>
      <td>Montopolis</td>
      <td>2018-02-28 11:18:00</td>
      <td>2019-03-17</td>
      <td>Return to Owner</td>
      <td>NaN</td>
      <td>Dog</td>
      <td>Neutered Male</td>
      <td>-1 years</td>
      <td>Rhod Ridgeback</td>
      <td>Red/Brown</td>
    </tr>
    <tr>
      <th>76016</th>
      <td>A797495</td>
      <td>Ace</td>
      <td>2019-06-15 12:44:00</td>
      <td>2020-12-16</td>
      <td>Return to Owner</td>
      <td>NaN</td>
      <td>Dog</td>
      <td>Neutered Male</td>
      <td>-1 years</td>
      <td>Cairn Terrier</td>
      <td>Black/Tan</td>
    </tr>
    <tr>
      <th>83186</th>
      <td>A703416</td>
      <td>Neko</td>
      <td>2015-05-26 16:58:00</td>
      <td>2015-05-29</td>
      <td>Return to Owner</td>
      <td>NaN</td>
      <td>Dog</td>
      <td>Neutered Male</td>
      <td>0 years</td>
      <td>Labrador Retriever Mix</td>
      <td>Black</td>
    </tr>
    <tr>
      <th>86714</th>
      <td>A702326</td>
      <td>Penelope</td>
      <td>2015-05-24 17:01:00</td>
      <td>2015-08-29</td>
      <td>Adoption</td>
      <td>NaN</td>
      <td>Cat</td>
      <td>Spayed Female</td>
      <td>0 years</td>
      <td>Domestic Shorthair Mix</td>
      <td>Black</td>
    </tr>
    <tr>
      <th>94184</th>
      <td>A687107</td>
      <td>Montopolis</td>
      <td>2017-06-25 12:20:00</td>
      <td>2019-03-17</td>
      <td>Rto-Adopt</td>
      <td>NaN</td>
      <td>Dog</td>
      <td>Neutered Male</td>
      <td>-1 years</td>
      <td>Rhod Ridgeback</td>
      <td>Red/Brown</td>
    </tr>
    <tr>
      <th>94555</th>
      <td>A687107</td>
      <td>Montopolis</td>
      <td>2017-10-04 17:57:00</td>
      <td>2019-03-17</td>
      <td>Return to Owner</td>
      <td>NaN</td>
      <td>Dog</td>
      <td>Neutered Male</td>
      <td>-1 years</td>
      <td>Rhod Ridgeback</td>
      <td>Red/Brown</td>
    </tr>
    <tr>
      <th>96636</th>
      <td>A660928</td>
      <td>Sadie</td>
      <td>2013-12-01 13:19:00</td>
      <td>2014-04-03</td>
      <td>Transfer</td>
      <td>Partner</td>
      <td>Dog</td>
      <td>Spayed Female</td>
      <td>0 years</td>
      <td>Labrador Retriever Mix</td>
      <td>Black/White</td>
    </tr>
    <tr>
      <th>99418</th>
      <td>A706929</td>
      <td>NaN</td>
      <td>2015-07-05 14:46:00</td>
      <td>2015-07-06</td>
      <td>Transfer</td>
      <td>Partner</td>
      <td>Cat</td>
      <td>Unknown</td>
      <td>0 years</td>
      <td>Domestic Shorthair Mix</td>
      <td>Tortie</td>
    </tr>
    <tr>
      <th>100975</th>
      <td>A687107</td>
      <td>Montopolis</td>
      <td>2016-02-25 18:04:00</td>
      <td>2019-03-17</td>
      <td>Return to Owner</td>
      <td>NaN</td>
      <td>Dog</td>
      <td>Neutered Male</td>
      <td>-3 years</td>
      <td>Rhod Ridgeback</td>
      <td>Red/Brown</td>
    </tr>
    <tr>
      <th>102241</th>
      <td>A788874</td>
      <td>NaN</td>
      <td>2019-02-14 17:13:00</td>
      <td>2019-12-06</td>
      <td>Adoption</td>
      <td>NaN</td>
      <td>Dog</td>
      <td>Spayed Female</td>
      <td>0 years</td>
      <td>Labrador Retriever Mix</td>
      <td>White/Tricolor</td>
    </tr>
    <tr>
      <th>102295</th>
      <td>A749253</td>
      <td>Orange</td>
      <td>2017-05-12 16:43:00</td>
      <td>2017-07-01</td>
      <td>Euthanasia</td>
      <td>Suffering</td>
      <td>Cat</td>
      <td>Intact Female</td>
      <td>0 years</td>
      <td>Domestic Shorthair Mix</td>
      <td>Orange Tabby</td>
    </tr>
    <tr>
      <th>116372</th>
      <td>A753893</td>
      <td>Chato</td>
      <td>2015-07-02 11:06:00</td>
      <td>2016-07-12</td>
      <td>Transfer</td>
      <td>Partner</td>
      <td>Dog</td>
      <td>Intact Male</td>
      <td>-1 years</td>
      <td>American Bulldog Mix</td>
      <td>White/Brown</td>
    </tr>
    <tr>
      <th>132700</th>
      <td>A825065</td>
      <td>NaN</td>
      <td>2020-10-31 11:05:00</td>
      <td>2021-10-01</td>
      <td>Died</td>
      <td>In Kennel</td>
      <td>Cat</td>
      <td>Intact Female</td>
      <td>0 years</td>
      <td>Domestic Shorthair</td>
      <td>Brown Tabby</td>
    </tr>
    <tr>
      <th>137755</th>
      <td>A853991</td>
      <td>NaN</td>
      <td>2021-06-15 16:07:00</td>
      <td>2021-09-28</td>
      <td>Adoption</td>
      <td>NaN</td>
      <td>Cat</td>
      <td>Neutered Male</td>
      <td>0 years</td>
      <td>Domestic Shorthair</td>
      <td>Brown Tabby/White</td>
    </tr>
  </tbody>
</table>
</div>



V datasete existujú záznamy zvierat, ktoré boli odovzdané ešte pred samotným narodením a majú záporný vek. Narušujú dátovú integritu, a preto ich z datasetu odstránime.


```python
outcomes = outcomes.drop(outcomes[outcomes['Date of Birth'] > outcomes['DateTime']].index)
```

### Kvantitatívne diskrétne príznaky
Príznaky `Age upon Outcome` a `Age upon Intake` prekonvertujeme na numerickú hodnotu. Z prvotnej analýzy datasetu vieme, že `Age upon Outcome` obsahuje chýbajúce hodnoty. Tie dokážeme nahradiť pomocou: (`DateTime` - `Date of Birth`).


```python
#https://stackoverflow.com/questions/29177498/replace-nan-in-one-column-with-value-from-corresponding-row-of-second-column
indexes = outcomes[outcomes['Age upon Outcome'].isna()]
indexes = indexes[['Animal ID']]
outcomes.loc[outcomes['Age upon Outcome'].isna(), 'Age upon Outcome'] = (outcomes['DateTime'] - outcomes['Date of Birth']).dt.days / 365
outcomes[outcomes['Animal ID'].isin(indexes['Animal ID'].unique())]
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Animal ID</th>
      <th>Name</th>
      <th>DateTime</th>
      <th>Date of Birth</th>
      <th>Outcome Type</th>
      <th>Outcome Subtype</th>
      <th>Animal Type</th>
      <th>Sex upon Outcome</th>
      <th>Age upon Outcome</th>
      <th>Breed</th>
      <th>Color</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>138063</th>
      <td>A854611</td>
      <td>NaN</td>
      <td>2022-04-06 17:16:00</td>
      <td>2020-04-06</td>
      <td>Euthanasia</td>
      <td>Rabies Risk</td>
      <td>Other</td>
      <td>Unknown</td>
      <td>2.0</td>
      <td>Raccoon</td>
      <td>Gray/Black</td>
    </tr>
    <tr>
      <th>138296</th>
      <td>A853907</td>
      <td>Oscar</td>
      <td>2022-04-13 13:48:00</td>
      <td>2006-03-27</td>
      <td>Euthanasia</td>
      <td>Suffering</td>
      <td>Cat</td>
      <td>Neutered Male</td>
      <td>16.057534</td>
      <td>Domestic Shorthair</td>
      <td>Blue/White</td>
    </tr>
    <tr>
      <th>138359</th>
      <td>A855204</td>
      <td>Bat</td>
      <td>2022-04-15 07:43:00</td>
      <td>2021-04-14</td>
      <td>Euthanasia</td>
      <td>Rabies Risk</td>
      <td>Other</td>
      <td>Unknown</td>
      <td>1.00274</td>
      <td>Bat</td>
      <td>Brown</td>
    </tr>
    <tr>
      <th>138675</th>
      <td>A855808</td>
      <td>NaN</td>
      <td>2022-04-23 16:04:00</td>
      <td>2020-04-23</td>
      <td>Euthanasia</td>
      <td>Rabies Risk</td>
      <td>Other</td>
      <td>Unknown</td>
      <td>2.0</td>
      <td>Bat</td>
      <td>Brown</td>
    </tr>
    <tr>
      <th>138708</th>
      <td>A855822</td>
      <td>A855822</td>
      <td>2022-04-25 16:14:00</td>
      <td>2009-04-25</td>
      <td>Euthanasia</td>
      <td>Suffering</td>
      <td>Dog</td>
      <td>Neutered Male</td>
      <td>13.008219</td>
      <td>Chow Chow Mix</td>
      <td>Tan</td>
    </tr>
  </tbody>
</table>
</div>



`Age upon Outcome` je potrebné prekonvertovať na numerickú hodnotu. Rozhodol som sa vek prepočítať na počet rokov. Keďže dáta nemajú jednotný formát, pretypovávať ich budem vlastnou funkciou. Pre správne spracovanie záznamov potrebujem zistiť, aké nečíselné reťazce sa v záznamoch nachádzajú.


```python
# get unique values from Age upon Outcome and filter out numbers
print(set([item for n in set(outcomes['Age upon Outcome'].unique()) for item in str(n).split(" ") if not item.isnumeric()]))
```

    {'month', '1.0027397260273974', 'year', 'weeks', '13.008219178082191', '2.0', '16.057534246575344', 'days', 'day', 'years', 'months', 'week'}



```python
def normalize_age_upon(data: pd.DataFrame) -> None:
    """Return the age in days"""
    # data from 'Date of Birth'
    if(isinstance(data, int) or isinstance(data, float)):
        return data
        
    # original data
    number,text = data.split(" ")
    text = text.lower()
    number_of_years = 0
    if text in 'years':
        number_of_years =  float(number)
    elif text in 'months':
        number_of_years = float(number) / 12
    elif text in 'weeks':
        number_of_years = float(number) / 52
    elif text in 'days':
        number_of_years = float(number) / 365
    return number_of_years
```


```python
outcomes['Age upon Outcome'] = outcomes['Age upon Outcome'].apply(normalize_age_upon)
```


```python
column_range(outcomes['Age upon Outcome'])
```

    Min: 0.0
    Max: 30.0


Rovnakým spôsobom prekonvertujeme aj `Age upon Intake`. Predtým si overíme, či neobsahuje textové reťazce, ktoré sa nenachádzali v outcomes (potom by bolo potrebné upraviť normalize_age_upon)


```python
# get all unique string values from Age upon Intake and filter out numbers
print(set([item for n in set(intakes['Age upon Intake'].unique()) for item in str(n).split(" ") if not item.isnumeric()]))
```

    {'month', '-3', 'year', 'weeks', 'days', 'day', '-2', 'years', 'months', '-1', 'week'}



```python
intakes[intakes['Age upon Intake'].str.startswith('-')]
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Animal ID</th>
      <th>Name</th>
      <th>DateTime</th>
      <th>Found Location</th>
      <th>Intake Type</th>
      <th>Intake Condition</th>
      <th>Animal Type</th>
      <th>Sex upon Intake</th>
      <th>Age upon Intake</th>
      <th>Breed</th>
      <th>Color</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>12599</th>
      <td>A687107</td>
      <td>Montopolis</td>
      <td>2017-10-04 10:22:00</td>
      <td>East Riverside Drive And Montopolis Drive in A...</td>
      <td>Stray</td>
      <td>Normal</td>
      <td>Dog</td>
      <td>Neutered Male</td>
      <td>-1 years</td>
      <td>Rhod Ridgeback</td>
      <td>Red/Brown</td>
    </tr>
    <tr>
      <th>49643</th>
      <td>A797495</td>
      <td>Ace</td>
      <td>2019-06-14 11:34:00</td>
      <td>6814 East Riverside Drive in Austin (TX)</td>
      <td>Stray</td>
      <td>Normal</td>
      <td>Dog</td>
      <td>Neutered Male</td>
      <td>-1 years</td>
      <td>Cairn Terrier</td>
      <td>Black/Tan</td>
    </tr>
    <tr>
      <th>59806</th>
      <td>A687107</td>
      <td>Montopolis</td>
      <td>2017-06-19 13:26:00</td>
      <td>7140 E Ben White in Austin (TX)</td>
      <td>Stray</td>
      <td>Normal</td>
      <td>Dog</td>
      <td>Neutered Male</td>
      <td>-1 years</td>
      <td>Rhod Ridgeback</td>
      <td>Red/Brown</td>
    </tr>
    <tr>
      <th>77603</th>
      <td>A757376</td>
      <td>Gorda</td>
      <td>2017-09-01 16:49:00</td>
      <td>6807 Blue Dawn Trl in Austin (TX)</td>
      <td>Stray</td>
      <td>Normal</td>
      <td>Dog</td>
      <td>Intact Female</td>
      <td>-2 years</td>
      <td>Miniature Schnauzer Mix</td>
      <td>White</td>
    </tr>
    <tr>
      <th>81295</th>
      <td>A753893</td>
      <td>Chato</td>
      <td>2015-06-26 16:30:00</td>
      <td>6709 Ponca Street in Austin (TX)</td>
      <td>Stray</td>
      <td>Normal</td>
      <td>Dog</td>
      <td>Intact Male</td>
      <td>-1 years</td>
      <td>American Bulldog Mix</td>
      <td>White/Brown</td>
    </tr>
    <tr>
      <th>93575</th>
      <td>A687107</td>
      <td>Montopolis</td>
      <td>2018-02-27 17:00:00</td>
      <td>400 West Parson in Manor (TX)</td>
      <td>Stray</td>
      <td>Normal</td>
      <td>Dog</td>
      <td>Neutered Male</td>
      <td>-1 years</td>
      <td>Rhod Ridgeback</td>
      <td>Red/Brown</td>
    </tr>
    <tr>
      <th>102222</th>
      <td>A687107</td>
      <td>Montopolis</td>
      <td>2016-02-12 00:58:00</td>
      <td>1005 Valdez St in Austin (TX)</td>
      <td>Stray</td>
      <td>Normal</td>
      <td>Dog</td>
      <td>Neutered Male</td>
      <td>-3 years</td>
      <td>Rhod Ridgeback</td>
      <td>Red/Brown</td>
    </tr>
    <tr>
      <th>123269</th>
      <td>A757376</td>
      <td>Gorda</td>
      <td>2018-10-13 13:42:00</td>
      <td>Austin (TX)</td>
      <td>Public Assist</td>
      <td>Normal</td>
      <td>Dog</td>
      <td>Spayed Female</td>
      <td>-1 years</td>
      <td>Miniature Schnauzer Mix</td>
      <td>White</td>
    </tr>
  </tbody>
</table>
</div>



Z predchádzajúcej bunky sme zistili, že existujú záznamy s negatívnym `Age upon Intake`. To narúša dátovú integritu. Chýbajúce hodnoty nahradíme hodnotami z `Date of Birth`.


```python
indexes = intakes[intakes['Age upon Intake'].str.startswith('-')]['Animal ID'].unique()
print('Chýba:', set(indexes) - set(intakes[intakes['Animal ID'].isin(outcomes['Animal ID'].unique())]['Animal ID'].unique()))
```

    Chýba: {'A757376'}


Zviera s Animal ID 'A757376' sa nenachádza v outcomes. Jeho vek nie je možné získať, preto ho odstránime.


```python
intakes = intakes.drop(intakes[intakes['Animal ID'] == 'A757376'].index)
```


```python
# Replace missing values in intakes with values from Date of Birth
indexes = intakes[intakes['Age upon Intake'].str.startswith('-')]['Animal ID'].unique()
values = outcomes[outcomes['Animal ID'].isin(indexes)][['Animal ID', 'Date of Birth']]
intakes = pd.merge(intakes, values, on='Animal ID', how='left')
```


```python
#Inspired by: https://stackoverflow.com/questions/29177498/replace-nan-in-one-column-with-value-from-corresponding-row-of-second-column
intakes.loc[intakes['Animal ID'].isin(indexes), 'Age upon Intake'] = (intakes['DateTime'] - intakes['Date of Birth']).dt.days / 365
intakes=intakes.drop(['Date of Birth'], axis=1)
```


```python
intakes['Age upon Intake'] = intakes['Age upon Intake'].apply(normalize_age_upon).apply(lambda x: round(x,3))
column_range(intakes['Age upon Intake'])
```

    Min: -3.093
    Max: 30.0


Z rozsahu hodnôt je možné vyčítať, že existujú záznamy s negatívnym vekom.


```python
intakes[intakes['Age upon Intake'] < 0]
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Animal ID</th>
      <th>Name</th>
      <th>DateTime</th>
      <th>Found Location</th>
      <th>Intake Type</th>
      <th>Intake Condition</th>
      <th>Animal Type</th>
      <th>Sex upon Intake</th>
      <th>Age upon Intake</th>
      <th>Breed</th>
      <th>Color</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>12595</th>
      <td>A687107</td>
      <td>Montopolis</td>
      <td>2017-10-04 10:22:00</td>
      <td>East Riverside Drive And Montopolis Drive in A...</td>
      <td>Stray</td>
      <td>Normal</td>
      <td>Dog</td>
      <td>Neutered Male</td>
      <td>-1.449</td>
      <td>Rhod Ridgeback</td>
      <td>Red/Brown</td>
    </tr>
    <tr>
      <th>12596</th>
      <td>A687107</td>
      <td>Montopolis</td>
      <td>2017-10-04 10:22:00</td>
      <td>East Riverside Drive And Montopolis Drive in A...</td>
      <td>Stray</td>
      <td>Normal</td>
      <td>Dog</td>
      <td>Neutered Male</td>
      <td>-1.449</td>
      <td>Rhod Ridgeback</td>
      <td>Red/Brown</td>
    </tr>
    <tr>
      <th>49637</th>
      <td>A797495</td>
      <td>Ace</td>
      <td>2019-06-14 11:34:00</td>
      <td>6814 East Riverside Drive in Austin (TX)</td>
      <td>Stray</td>
      <td>Normal</td>
      <td>Dog</td>
      <td>Neutered Male</td>
      <td>-1.510</td>
      <td>Cairn Terrier</td>
      <td>Black/Tan</td>
    </tr>
    <tr>
      <th>59799</th>
      <td>A687107</td>
      <td>Montopolis</td>
      <td>2017-06-19 13:26:00</td>
      <td>7140 E Ben White in Austin (TX)</td>
      <td>Stray</td>
      <td>Normal</td>
      <td>Dog</td>
      <td>Neutered Male</td>
      <td>-1.742</td>
      <td>Rhod Ridgeback</td>
      <td>Red/Brown</td>
    </tr>
    <tr>
      <th>59800</th>
      <td>A687107</td>
      <td>Montopolis</td>
      <td>2017-06-19 13:26:00</td>
      <td>7140 E Ben White in Austin (TX)</td>
      <td>Stray</td>
      <td>Normal</td>
      <td>Dog</td>
      <td>Neutered Male</td>
      <td>-1.742</td>
      <td>Rhod Ridgeback</td>
      <td>Red/Brown</td>
    </tr>
    <tr>
      <th>81281</th>
      <td>A753893</td>
      <td>Chato</td>
      <td>2015-06-26 16:30:00</td>
      <td>6709 Ponca Street in Austin (TX)</td>
      <td>Stray</td>
      <td>Normal</td>
      <td>Dog</td>
      <td>Intact Male</td>
      <td>-1.047</td>
      <td>American Bulldog Mix</td>
      <td>White/Brown</td>
    </tr>
    <tr>
      <th>81282</th>
      <td>A753893</td>
      <td>Chato</td>
      <td>2015-06-26 16:30:00</td>
      <td>6709 Ponca Street in Austin (TX)</td>
      <td>Stray</td>
      <td>Normal</td>
      <td>Dog</td>
      <td>Intact Male</td>
      <td>-1.047</td>
      <td>American Bulldog Mix</td>
      <td>White/Brown</td>
    </tr>
    <tr>
      <th>93563</th>
      <td>A687107</td>
      <td>Montopolis</td>
      <td>2018-02-27 17:00:00</td>
      <td>400 West Parson in Manor (TX)</td>
      <td>Stray</td>
      <td>Normal</td>
      <td>Dog</td>
      <td>Neutered Male</td>
      <td>-1.049</td>
      <td>Rhod Ridgeback</td>
      <td>Red/Brown</td>
    </tr>
    <tr>
      <th>93564</th>
      <td>A687107</td>
      <td>Montopolis</td>
      <td>2018-02-27 17:00:00</td>
      <td>400 West Parson in Manor (TX)</td>
      <td>Stray</td>
      <td>Normal</td>
      <td>Dog</td>
      <td>Neutered Male</td>
      <td>-1.049</td>
      <td>Rhod Ridgeback</td>
      <td>Red/Brown</td>
    </tr>
    <tr>
      <th>102211</th>
      <td>A687107</td>
      <td>Montopolis</td>
      <td>2016-02-12 00:58:00</td>
      <td>1005 Valdez St in Austin (TX)</td>
      <td>Stray</td>
      <td>Normal</td>
      <td>Dog</td>
      <td>Neutered Male</td>
      <td>-3.093</td>
      <td>Rhod Ridgeback</td>
      <td>Red/Brown</td>
    </tr>
    <tr>
      <th>102212</th>
      <td>A687107</td>
      <td>Montopolis</td>
      <td>2016-02-12 00:58:00</td>
      <td>1005 Valdez St in Austin (TX)</td>
      <td>Stray</td>
      <td>Normal</td>
      <td>Dog</td>
      <td>Neutered Male</td>
      <td>-3.093</td>
      <td>Rhod Ridgeback</td>
      <td>Red/Brown</td>
    </tr>
  </tbody>
</table>
</div>



Problematické záznamy majú zvieratá, ktorých vek bol nahradený hodnotou z `Date of Birth`. Skutočný vek nedokážeme z dát získať, preto tieto záznamy odstránime.


```python
intakes = intakes.drop(intakes[intakes['Age upon Intake'] < 0].index)
column_range(intakes['Age upon Intake'])
```

    Min: 0.0
    Max: 30.0


### Kategorické nominálne príznaky
Zvyšné príznaky sú kategorické. Tie pretypujeme na dátový typ "categorical". Zároveň odstránime chýbajúce hodnoty a overíme jednotný formát dát. 

Keďže časť príznakov je rovnaká pre oba datasety, vytvoríme jeden dátový typ pre oba príznaky. Aby bolo zaručené, že príznaky nadobúdajú rovnaké/podobné hodnoty, zistíme množinové rozdiely unikátnych hodnôt príznakov. 

Pre zjednodušenie procesu si príznaky rozdelíme podľa toho, či sa nachádzajú v oboch datasetoch.


```python
# Classify columns
intakes_unique = ['Found Location', 'Intake Type', 'Intake Condition']
outcomes_unique = ['Outcome Type', 'Outcome Subtype']
intakes_same = ['Name', 'Animal Type', 'Sex upon Intake', 'Breed', 'Color']
outcomes_same = ['Name', 'Animal Type', 'Sex upon Outcome', 'Breed', 'Color']
```


```python
def convert_to_cat(datasets:pd.DataFrame, cols:pd.Series) -> None:
    """Create categorical datatype from the columns and convert them""" 
    all_cat = []
    
    # names are the same
    if isinstance(cols, str):
        cols = [cols, cols]
        
    # create list of unique values 
    for i, dataset in enumerate(datasets):
        all_cat += set(dataset[cols[i]].unique())
    
    # combine values
    all_cat = set(all_cat)
    
    # create categorical data type
    cat = CategoricalDtype(categories=all_cat)

    # convert columns to categorical data type
    for i, dataset in enumerate(datasets):
        dataset[cols[i]] = dataset[cols[i]].astype(cat)
```


```python
def print_cat_info(dataset:pd.DataFrame, column:pd.Series, length=20)->None:
    """Print information about unique values"""
    n_unique = dataset[column].nunique()
    n_missing = dataset[dataset[column].isna()].shape[0]
    print(column)
    print(f'Počet unikátnych hodnôt: {n_unique}')
    print(f'Počet chýbajúcich hodnôt: {n_missing} ({n_missing / dataset[[column]].shape[0]}%)')
    print(f'Unikátne hodnty: {dataset[column].unique().tolist()[:min(length, n_unique)]}')
    print(30*'-')
```


```python
def cmp_cat(dataset1:pd.DataFrame, col1:pd.Series, dataset2:pd.DataFrame, col2:pd.Series):
    """Compare unique values"""
    print("Počet rozdielnych hodnôt:", len((set(dataset1[col1].unique()) - set(dataset2[col2].unique()))) + len((set(dataset2[col2].unique()) - set(dataset1[col1].unique()))))
```

Začneme s unikátnymi kategorickými príznakmi.


```python
# check unique values
for col in intakes_unique:
    print_cat_info(intakes, col)   
```

    Found Location
    Počet unikátnych hodnôt: 58364
    Počet chýbajúcich hodnôt: 0 (0.0%)
    Unikátne hodnty: ['2501 Magin Meadow Dr in Austin (TX)', '9409 Bluegrass Dr in Austin (TX)', '2818 Palomino Trail in Austin (TX)', 'Austin (TX)', '800 Grove Blvd in Austin (TX)', '415 East Mary Street in Austin (TX)', '2112 East William Cannon Drive in Austin (TX)', 'Braker Lane And Metric in Travis (TX)', '6600 Elm Creek in Austin (TX)', '8800 South First Street in Austin (TX)', 'Galilee Court And Damita Jo Dr in Manor (TX)', '9705 Thaxton in Austin (TX)', '4424 S Mopac Expwy in Austin (TX)', '208 Beaver St in Austin (TX)', '3110 Guadalupe Street in Austin (TX)', '6118 Fairway in Austin (TX)', 'South First And Stassney in Austin (TX)', '1501 S Fm 973 in Austin (TX)', '1801 Westridge in Austin (TX)', '6111 Softwood Drive in Austin (TX)']
    ------------------------------
    Intake Type
    Počet unikátnych hodnôt: 6
    Počet chýbajúcich hodnôt: 0 (0.0%)
    Unikátne hodnty: ['Stray', 'Owner Surrender', 'Public Assist', 'Wildlife', 'Euthanasia Request', 'Abandoned']
    ------------------------------
    Intake Condition
    Počet unikátnych hodnôt: 15
    Počet chýbajúcich hodnôt: 0 (0.0%)
    Unikátne hodnty: ['Normal', 'Sick', 'Injured', 'Pregnant', 'Nursing', 'Aged', 'Medical', 'Other', 'Neonatal', 'Feral', 'Behavior', 'Med Urgent', 'Space', 'Med Attn', 'Panleuk']
    ------------------------------



```python
# Convert to categorical data type
for col in intakes_unique:
    convert_to_cat([intakes], col)   
```

Príznak `Intake Condition` nadobúda hodnoty "Med Attn", "Med Urgent", "Panleuk", ktoré označujú to, čo "Medical". Tieto kategórie zjednotíme a nahradíme za "Medical".


```python
intakes['Intake Condition'] = intakes['Intake Condition'].replace('Med Urgent', 'Medical')
intakes['Intake Condition'] = intakes['Intake Condition'].replace('Med Attn', 'Medical')
intakes['Intake Condition'] = intakes['Intake Condition'].replace('Panleuk', 'Medical')
```


```python
for col in outcomes_unique:
    print_cat_info(outcomes, col)   
```

    Outcome Type
    Počet unikátnych hodnôt: 9
    Počet chýbajúcich hodnôt: 23 (0.0001657932486105805%)
    Unikátne hodnty: ['Rto-Adopt', 'Adoption', 'Euthanasia', 'Transfer', 'Return to Owner', 'Died', 'Disposal', 'Missing', 'Relocate']
    ------------------------------
    Outcome Subtype
    Počet unikátnych hodnôt: 26
    Počet chýbajúcich hodnôt: 75310 (0.5428647631679485%)
    Unikátne hodnty: [nan, 'Partner', 'Foster', 'SCRP', 'Out State', 'Suffering', 'Underage', 'Snr', 'Rabies Risk', 'In Kennel', 'Offsite', 'Aggressive', 'Enroute', 'At Vet', 'In Foster', 'Behavior', 'Medical', 'Field', 'Possible Theft', 'Barn']
    ------------------------------


Záznamy s chýbajúcimi hodnotami z `Outcome type` odstránime. Kategórie "Transfer" a "Relocate" označujú to isté, preto kategórie zjednotíme a nahradíme za "Transfer".

Príznaku `Outcome Subtype` chýba viac ako polovica záznamov, preto je pri ďalšej analýze tento príznak málo použiteľný.

Nájdené chyby opravíme a príznaky prekonvertujeme.


```python
outcomes['Outcome Type'] = outcomes['Outcome Type'].replace("Relocate", "Transfer")
```


```python
outcomes = outcomes.dropna(subset=['Outcome Type'])
outcomes['Outcome Subtype'] = outcomes['Outcome Subtype'].astype('category') 
outcomes['Outcome Type'] = outcomes['Outcome Type'].astype('category') 
```

Kategorické príznaky, ktoré sa nachádzadzajú v oboch datasetoch:


```python
for i in range(len(intakes_same)):
    print_cat_info(intakes, intakes_same[i])
    print_cat_info(outcomes, outcomes_same[i])
    cmp_cat(intakes, intakes_same[i], outcomes, outcomes_same[i])
```

    Name
    Počet unikátnych hodnôt: 23544
    Počet chýbajúcich hodnôt: 41265 (0.2978110723796739%)
    Unikátne hodnty: ['*Brock', 'Belle', 'Runster', nan, 'Rio', 'Odin', 'Beowulf', '*Ella', 'Mumble', '*Casey', '*Candy Cane', '*Pearl', 'Ziggy', 'Tommy', 'Tulip', '*Mint', '*Twilight', 'Stumpy', 'Rheia', 'Baby']
    ------------------------------
    Name
    Počet unikátnych hodnôt: 23425
    Počet chýbajúcich hodnôt: 41232 (0.29726612065982233%)
    Unikátne hodnty: ['Chunk', 'Gizmo', nan, 'Moose', 'Princess', 'Quentin', '*Donatello', '*Zeus', 'Tulip', 'Artemis', 'Fiona', '*Mary', '*Birch', 'Luigi', '*Liza', 'Einstein', 'Star', 'Millie', 'Big Girl', 'Theodore Nubbins']
    ------------------------------
    Počet rozdielnych hodnôt: 311
    Animal Type
    Počet unikátnych hodnôt: 5
    Počet chýbajúcich hodnôt: 0 (0.0%)
    Unikátne hodnty: ['Dog', 'Cat', 'Other', 'Bird', 'Livestock']
    ------------------------------
    Animal Type
    Počet unikátnych hodnôt: 5
    Počet chýbajúcich hodnôt: 0 (0.0%)
    Unikátne hodnty: ['Cat', 'Dog', 'Other', 'Bird', 'Livestock']
    ------------------------------
    Počet rozdielnych hodnôt: 0
    Sex upon Intake
    Počet unikátnych hodnôt: 5
    Počet chýbajúcich hodnôt: 1 (7.217037983270906e-06%)
    Unikátne hodnty: ['Neutered Male', 'Spayed Female', 'Intact Male', 'Intact Female', 'Unknown']
    ------------------------------
    Sex upon Outcome
    Počet unikátnych hodnôt: 5
    Počet chýbajúcich hodnôt: 1 (7.209597416080286e-06%)
    Unikátne hodnty: ['Neutered Male', 'Unknown', 'Intact Male', 'Spayed Female', 'Intact Female']
    ------------------------------
    Počet rozdielnych hodnôt: 0
    Breed
    Počet unikátnych hodnôt: 2741
    Počet chýbajúcich hodnôt: 0 (0.0%)
    Unikátne hodnty: ['Beagle Mix', 'English Springer Spaniel', 'Basenji Mix', 'Domestic Shorthair Mix', 'Doberman Pinsch/Australian Cattle Dog', 'Labrador Retriever Mix', 'Great Dane Mix', 'Domestic Shorthair', 'Chihuahua Shorthair', 'Pit Bull', 'Australian Cattle Dog/Labrador Retriever', 'Parson Russell Terrier Mix', 'Norfolk Terrier', 'Bat', 'Yorkshire Terrier Mix', 'Maltese Mix', 'Rottweiler Mix', 'Dachshund Mix', 'Bat Mix', 'Boxer Mix']
    ------------------------------
    Breed
    Počet unikátnych hodnôt: 2749
    Počet chýbajúcich hodnôt: 0 (0.0%)
    Unikátne hodnty: ['Domestic Shorthair Mix', 'Chihuahua Shorthair Mix', 'Raccoon', 'Anatol Shepherd/Labrador Retriever', 'American Foxhound/Labrador Retriever', 'Border Collie/Cardigan Welsh Corgi', 'Pit Bull', 'Domestic Shorthair', 'Domestic Medium Hair Mix', 'Domestic Medium Hair', 'Opossum', 'Yorkshire Terrier Mix', 'Jack Russell Terrier/Chihuahua Shorthair', 'Great Pyrenees Mix', 'Bat Mix', 'Chihuahua Shorthair', 'Australian Cattle Dog Mix', 'Beagle Mix', 'Labrador Retriever Mix', 'Labrador Retriever/Staffordshire']
    ------------------------------
    Počet rozdielnych hodnôt: 8
    Color
    Počet unikátnych hodnôt: 616
    Počet chýbajúcich hodnôt: 0 (0.0%)
    Unikátne hodnty: ['Tricolor', 'White/Liver', 'Sable/White', 'Calico', 'Tan/Gray', 'Chocolate', 'Black', 'Brown Tabby', 'Black/White', 'Cream Tabby', 'White/Tan', 'Brown/White', 'Brown Tabby/White', 'Torbie', 'Tan/White', 'Tortie', 'Blue/White', 'Brown/Cream', 'Brown', 'White/Black']
    ------------------------------
    Color
    Počet unikátnych hodnôt: 619
    Počet chýbajúcich hodnôt: 0 (0.0%)
    Unikátne hodnty: ['Brown Tabby/White', 'White/Brown', 'Gray', 'Buff', 'Orange Tabby', 'Brown', 'Black', 'White/Orange Tabby', 'Black/White', 'Blue/White', 'White/Blue', 'Brown Tabby', 'Calico', 'Tricolor', 'Brown/Black', 'White/Tan', 'White', 'Brown/White', 'Brown Brindle/White', 'Black/Brown']
    ------------------------------
    Počet rozdielnych hodnôt: 3


Z výpisu pozorujeme, že príznaku `Name` chýba 30% hodnôt a tie, ktoré nechýbajú, môžu byť chybné (napríklad začínajú *). Meno zvieraťa nepredstavuje dôležitú informáciu, pretože každé zviera je jednoznačne identifikovateľné pomocou `Animal ID`. Preto príznaky odstránime z oboch datasetov. 

`Sex upon Intake` a `Sex upon Outcomes` obsahujú chýbajúce hodnoty ako np.nan a "Unknown". Pre zachovanie dátovej konzistencie nahradíme výskyty "Unknown" za np.nan. Záznamy s chýbajúcimi hodnotami odstránime. 


```python
# Remove Name
intakes = intakes.drop('Name', axis=1)
outcomes = outcomes.drop('Name', axis=1)
intakes_same.remove('Name')
outcomes_same.remove('Name')
```


```python
# Replace all Unknown data to np.nan
intakes['Sex upon Intake'] = intakes['Sex upon Intake'].replace('Unknown',np.nan)
outcomes['Sex upon Outcome'] = outcomes['Sex upon Outcome'].replace('Unknown',np.nan)

intakes = intakes.dropna(subset=['Sex upon Intake'])
outcomes = outcomes.dropna(subset=['Sex upon Outcome'])
```


```python
# Convert data to categorical
for i in range(len(intakes_same)):
    convert_to_cat([intakes, outcomes], [intakes_same[i], outcomes_same[i]])
```

### Kategorické ordinálne príznaky
Príznak `Animal ID` konvertujeme a zachováme jeho prirodzené usporiadanie.


```python
#Convert Animal ID to categorical 
ids = intakes['Animal ID'].unique().tolist()
ids += outcomes['Animal ID'].unique().tolist()
ids = list(set(ids))
ids.sort()
cat = CategoricalDtype(categories=ids, ordered=True)
intakes['Animal ID'] = intakes['Animal ID'].astype(cat)
outcomes['Animal ID'] = outcomes['Animal ID'].astype(cat)
```

## Zhrnutie čistenia dát


```python
intakes.info()
```

    <class 'pandas.core.frame.DataFrame'>
    Index: 127494 entries, 0 to 138571
    Data columns (total 10 columns):
     #   Column            Non-Null Count   Dtype         
    ---  ------            --------------   -----         
     0   Animal ID         127494 non-null  category      
     1   DateTime          127494 non-null  datetime64[ns]
     2   Found Location    127494 non-null  category      
     3   Intake Type       127494 non-null  category      
     4   Intake Condition  127494 non-null  category      
     5   Animal Type       127494 non-null  category      
     6   Sex upon Intake   127494 non-null  category      
     7   Age upon Intake   127494 non-null  float64       
     8   Breed             127494 non-null  category      
     9   Color             127494 non-null  category      
    dtypes: category(8), datetime64[ns](1), float64(1)
    memory usage: 12.3 MB



```python
outcomes.info()
```

    <class 'pandas.core.frame.DataFrame'>
    Index: 127659 entries, 0 to 138768
    Data columns (total 10 columns):
     #   Column            Non-Null Count   Dtype         
    ---  ------            --------------   -----         
     0   Animal ID         127659 non-null  category      
     1   DateTime          127659 non-null  datetime64[ns]
     2   Date of Birth     127659 non-null  datetime64[ns]
     3   Outcome Type      127659 non-null  category      
     4   Outcome Subtype   53526 non-null   category      
     5   Animal Type       127659 non-null  category      
     6   Sex upon Outcome  127659 non-null  category      
     7   Age upon Outcome  127659 non-null  float64       
     8   Breed             127659 non-null  category      
     9   Color             127659 non-null  category      
    dtypes: category(7), datetime64[ns](2), float64(1)
    memory usage: 10.4 MB



```python
intakes_AnimalID = set(intakes['Animal ID'].unique())
outcomes_AnimalId = set(outcomes['Animal ID'].unique())

print(f'Intakes: {len(intakes_AnimalID)}')
print(f'Outcomes: {len(outcomes_AnimalId)}')
print(f'Dokopy: {len(intakes_AnimalID.union(outcomes_AnimalId))}')
```

    Intakes: 112833
    Outcomes: 113010
    Dokopy: 113643


- Datasety obsahujú 138561 (intakes) a 138704 (outcomes) záznamov
- V datasete sa nachádza 124703 zvierat z toho 123889 zvierat v intakes a  124042 v outcomes
- chýbajúce dáta sa nachádzajú v príznakoch `Sex upon Intake`(7%) , `Outcome Subtype` (52%), `Sex upon Outcome` (7%)
- Dataset intakes obsahuje 9 príznakov
    - Kvantitatívne
        - DateTime: dátum vytvorenia záznamu 
        - Age upon Intake: vek zvieraťa v čase prijatia do útulku (v rokoch)
    - Kvalitatívne
        - Animal ID: identifikačné číslo zvieraťa
        - Found Location: miesto nájdenia zvieraťa
        - Intake Type: dôvod prijatia zvieraťa do útulku
        - Intake Condition: stav v čase prijatia do útulku
        - Animal Type: druh zvieraťa
        - Sex upon Intake: pohlavie zvieraťa a jeho sterilita
        - Breed: konkrétnejší druh zvieraťa
        - Color: farba zvieraťa
- Dataset outcomes obsahuje 9 príznakov
    - Kvantitatívne
        - DateTime: dátum vytvorenia záznamu 
        - Age upon Outcome: vek zvieraťa v čase odchodu z útulku (v rokoch)
        - Date of Birth: dátum narodenia zvieraťa
    - Kvalitatívne
        - Animal ID: identifikačné číslo zvieraťa
        - Outcome Type: dôvod prepustenia zvieraťa z útulku
        - Outcome Subtype : detailnejší popis dôvodu prepustenia      
        - Animal Type: druh zvieraťa
        - Sex upon Outcome: pohlavie zvieraťa
        - Breed: konkrétnejší druh zvieraťa
        - Color: farba zvieraťa
- Kvantitatívne príznaky majú primeraný rozsah
- je zachovaná dátova konzistencia
- Je zachovaná dátová integrita
- z pôvodných datasetov boli odstránené
    - duplicitné záznamy
    - príznaky `MonthYear`,`Name`, ktoré obsahovali duplicitné informácie
    - záznamy, ktoré obsahovali chýbajúce/chybné hodnoty.

# 2. Deskriptívne štatistiky

## 1. DateTime
`DateTime` je kvantitatívny príznak. Deskriptívne štatistiky budú pozostávať z mediánu, kvartilov, minimálnej hodnoty, maximálnej hodnoty, rozsahu hodnôt a rozloženia hodnôt. Keďže ide o intervalový príznak, ktorý nemá definované operácie *+* a */*, nebudeme zisťovať priemer, smerodajnú odchýlku, šikmosť ani špicatosť. Rozloženie hodnôt budeme vizualizovať pomocou grafov boxplot, violinplot a histogramu. 


```python
def numerical_stats(dataset, all_stats = True):
    """Perform descriptive statistics on numerical data"""
    df = pd.DataFrame({'Min': [ dataset.min()],
                        'Max':  [dataset.max()],
                        '25%':  [dataset.quantile(0.25)],
                        '50%':  [dataset.median()],
                        '75%':  [dataset.quantile(0.75)],
                        'IQR':  [dataset.quantile(0.75) - dataset.quantile(0.25)],
                        'Min - Max': [ dataset.max() - dataset.min()]})
    if all_stats:
        tmp_df = pd.DataFrame({'Priemer' :[dataset.mean()], 
                            'Rozptyl' :[dataset.var()], 
                            'Smerodajná odchýlka' :[dataset.std()], 
                            'Koef. šikmosti' :[dataset.skew()], 
                            'Koef. špičatosti' :[dataset.kurtosis()]})
        tmp_df = tmp_df.round()
        df = df.join(tmp_df)
    return df.melt(var_name = 'Metric', value_name =  'Data') 
```


```python
def numerical_plot(dataset, col, title, ticks, label, ticks_label=None, p_figsize=(6,6), violin=True):
    """Plot Boxplot and Violinplot"""

    # Create ticks
    if ticks_label == None:
        ticks_label = ticks

    fig,ax  = plt.subplots(1,2, tight_layout=True, figsize=p_figsize)
    fig.suptitle(title)

    # Boxplot
    sns.boxplot(data=dataset, y=col, ax=ax[0])
    ax[0].set_yticks(ticks)
    ax[0].set_yticklabels(ticks_label)
    ax[0].set_ylabel(label)
    sns.despine(ax=ax[0], left=True)
    
    # Violin plot
    if not violin:
        ax[1].set_visible(False)
        return
        
    sns.violinplot(data=dataset, y=col, ax=ax[1])
    ax[1].set_yticks(ticks)
    ax[1].set_yticklabels(ticks_label)
    ax[1].set_ylabel(label)
    #ax[1].grid(False)
    sns.despine(ax=ax[1], left=True)
    
```


```python
ticks = [date(i,1,1) for i in range(2013, 2024)]
# Boxplot ViolinPlot
numerical_plot(intakes,'DateTime', 'Distribution of Animal Intakes Over a Decade', ticks,'Years', [item.year for item in ticks])
```


    
![png](./README-files/output_101_0.png)
    



```python
# Histplot
fig,ax  = plt.subplots(1,1, constrained_layout=True, figsize=(11,4))
# Sturges rule,
bins = 1 + np.log2(intakes.shape[0])
sns.histplot(data=intakes, x='DateTime', ax=ax, bins=int(bins))
ax.set_ylabel('Number of Intakes')
ax.set_xlabel('Years')
ax.set_title('Distribution of Animal Intakes Over a Decade')
ax.grid(axis='x')
sns.despine(ax=ax, left=True)
```


    
![png](./README-files/output_102_0.png)
    



```python
numerical_stats(intakes['DateTime'], all_stats=False)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Metric</th>
      <th>Data</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Min</td>
      <td>2013-10-01 07:51:00</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Max</td>
      <td>2022-04-27 07:54:00</td>
    </tr>
    <tr>
      <th>2</th>
      <td>25%</td>
      <td>2015-08-16 15:52:15</td>
    </tr>
    <tr>
      <th>3</th>
      <td>50%</td>
      <td>2017-07-31 09:34:30</td>
    </tr>
    <tr>
      <th>4</th>
      <td>75%</td>
      <td>2019-07-13 06:10:00</td>
    </tr>
    <tr>
      <th>5</th>
      <td>IQR</td>
      <td>1426 days 14:17:45</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Min - Max</td>
      <td>3130 days 00:03:00</td>
    </tr>
  </tbody>
</table>
</div>



Dataset obsahuje záznamy od 3.10 2013 do 24.7. 2022. Polovica dát leží medzi augustom 2015 a júlom 2019. Z grafov je možné vyčítať, že väčšina dát pochádza z obdobia 2014-2019. Najvyšší počet zvierat útulok prijal v roku 2019 a najnižší v roku 2020.

## 2. Age upon Intake

`Age upon Intake` je disktrétny kvantitatívny príznak. Okrem vyššie spomenutých vlastností dát bude deskriptívna štatistika obsahovať mieru variability (priemer, smerodajnú odchýlku, šikmosť a špičatosť).


```python
numerical_plot(intakes, 'Age upon Intake', 'Age distribution of Animals upon Intake', [item for item in range(0, 35, 5)], 'Years', p_figsize=(5,6), violin=False) 

fig,ax  = plt.subplots()
n_bins = 1 + np.log2(len(intakes['Age upon Intake']))

ax.set_title('Age distribution of Animals upon Intake')
ax.set_ylabel('Number of Intakes')
ax.set_xlabel('Age (in years)')
ticks = [i for i in range(0,35, 5)]
ax.grid(axis='y')
ax.set_xticks(ticks)
sns.histplot(data=intakes, x='Age upon Intake', ax=ax, bins=int(n_bins))
sns.despine(ax=ax)
```


    
![png](./README-files/output_106_0.png)
    



    
![png](./README-files/output_106_1.png)
    


Z grafov je možné vypozorovať, že príznak obsahuje množstvo odľahlých hodnôt, ktoré znižujú čítateľnosť grafu. Preto odľahlé hodnoty odstránime.


```python
numerical_plot(intakes[intakes['Age upon Intake'] <= 5], 'Age upon Intake', 'Age Distribution of Intake Animals (Under 5 Years)', [item/10 for item in range(0, 50, 5)], 'Years', p_figsize=(5,6), violin=True) 
```


    
![png](./README-files/output_108_0.png)
    



```python
numerical_stats(intakes['Age upon Intake'])
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Metric</th>
      <th>Data</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Min</td>
      <td>0.000</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Max</td>
      <td>30.000</td>
    </tr>
    <tr>
      <th>2</th>
      <td>25%</td>
      <td>0.167</td>
    </tr>
    <tr>
      <th>3</th>
      <td>50%</td>
      <td>1.000</td>
    </tr>
    <tr>
      <th>4</th>
      <td>75%</td>
      <td>2.000</td>
    </tr>
    <tr>
      <th>5</th>
      <td>IQR</td>
      <td>1.833</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Min - Max</td>
      <td>30.000</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Priemer</td>
      <td>2.000</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Rozptyl</td>
      <td>9.000</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Smerodajná odchýlka</td>
      <td>3.000</td>
    </tr>
    <tr>
      <th>10</th>
      <td>Koef. šikmosti</td>
      <td>2.000</td>
    </tr>
    <tr>
      <th>11</th>
      <td>Koef. špičatosti</td>
      <td>5.000</td>
    </tr>
  </tbody>
</table>
</div>



Distribúcia hodnôt `Age upon Intake` je skosená vpravo. Väčšina prijatých zvierat má menej ako 5 rokov. Najstaršie zviera malo 30 rokov.
Priemerný vek zvieraťa sú 2 roky. Kvôli nerovnomernénu rozloženiu je však priemer skreslený, pretože dataset obsahuje množstvo odľahlých hodnôt
(najlepšie možno pozorovať v prvom grafe). Lepšia metrika bude medián, ktorý je 1 rok. Z histogramu je možné vypozorovať, že dáta nepokrývajú niektoré vekové kategórie (v datasete napríklad nie sú zastúpené zvieratá medzi 2.25-2.5 rokmi). 

## 3. Animal Type

`Animal Type` Je to nominálny kategorický príznak. Deskriptívna analýza bude pozostávať z početnosti, relatívnej početnosti a modusu hodnôt.


```python
def cat_desc(dataset, column, ylabel, pfigsize, pres = 3, title=None):
    """Plot Frequency and Relative Frequency Barplots for categorical data"""
    if title==None:
        title = column
    
    fig, ax = plt.subplots(2,1,tight_layout = True, figsize = pfigsize)

    # Generate relative frequency data
    c_data = dataset[column].value_counts(normalize=True).sort_values(ascending=False)
    c_order = [index for index in c_data.index if c_data[index] > 0] 
    sns.barplot(data = c_data,ax=ax[0], order = c_order)
    ax[0].set_title('Relative Frequency of ' + title)
    dec = .01 * pres
    ax[0].bar_label(ax[0].containers[0],fmt=lambda x: f'{x*100:{dec}f}%')
    ax[0].set_ylabel(f'Proportion of of {ylabel}')
    ax[0].grid(False)
    sns.despine(ax=ax[0], left=True)

    c_data = dataset[column].value_counts().sort_values(ascending=False)
    sns.barplot(data = c_data, ax=ax[1],order=c_order)
    ax[1].set_title('Frequency of ' + title)
    ax[1].bar_label(ax[1].containers[0])
    ax[1].set_ylabel(f'Number of {ylabel}')
    ax[1].grid(False)
    sns.despine(ax=ax[1], left=True)
```


```python
cat_desc(intakes, 'Animal Type', 'Animals', pfigsize=(9,9), pres=2, title='Animal Types')
```


    
![png](./README-files/output_113_0.png)
    



```python
freq_table = intakes[['Animal ID', 'Animal Type']].groupby('Animal Type', observed=False).count()
freq_table['Frequency'] = freq_table['Animal ID']
freq_table.drop('Animal ID', axis=1)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Frequency</th>
    </tr>
    <tr>
      <th>Animal Type</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Dog</th>
      <td>77608</td>
    </tr>
    <tr>
      <th>Other</th>
      <td>1306</td>
    </tr>
    <tr>
      <th>Livestock</th>
      <td>15</td>
    </tr>
    <tr>
      <th>Bird</th>
      <td>276</td>
    </tr>
    <tr>
      <th>Cat</th>
      <td>48289</td>
    </tr>
  </tbody>
</table>
</div>



Útulok eviduje 5 druhov zvierat. Najzastúpenejšie sú psy a mačky. Tie tvoria 94%  zvierat, ktoré sa do útulku dostanú. Treťou najpočetnejšou skupinou sú zvieratá, ktoré sú evidované ako "Other". Vtáky a hospodárske zvieratá  tvoria menej ako 1% všetkých prijatých zvierat. 

## 4. Intake Condition
`Intake Condition` je kategorický nominálny príznak. Analyzovať a vizualizovať ho budeme rovnako ako `Animal Type`.


```python
cat_desc(intakes,'Intake Condition', 'Animals', (12,11), title='Intake Conditions',)
```


    
![png](./README-files/output_117_0.png)
    


Keďže `Intake Condition` obsahuje hodnôty, ktoré tvoria väčšinu datasetu ('Normal', 'Injured', 'Sick', 'Nursing'), zvyšné hodnoty nie sú v grafe viditeľné. Vizualizáciu si rozdelíme na 2 časti. V prvej budeme porovnávať početnosti kategórií, ktoré majú relatívnu početnosť > 1% a zvyšné hodnoty nahradíme novou hodnotou "Other". V druhej časti budeme porovnávať len hodnoty, ktoré spadajú do kategórie "Other".


```python
# conditions with <1% in dataset
cond_count = intakes['Intake Condition'].value_counts(normalize=True)
cond = [condition for condition in cond_count.index if cond_count[condition] < 0.01]

# Create a dataframe with the most significant conditions.
major_cond = intakes[['Intake Condition']].copy()

for condition in cond:
    major_cond['Intake Condition'] = major_cond['Intake Condition'].replace(condition, 'Others')

# Create dataframe with the lest significant conditions
minor_cond = intakes[intakes['Intake Condition'].isin(cond)].value_counts('Intake Condition').sort_values(ascending=False)
major_cond = major_cond.value_counts('Intake Condition').sort_values(ascending=False)
```


```python
# Plot major
fig, ax = plt.subplots(2,1,tight_layout = True, figsize = (10,10))
major_order = [index for index in major_cond.index if major_cond[index] > 0] 
sns.barplot(data = major_cond, ax=ax[0], order=major_order)
ax[0].set_title('Frequency of Intake Conditions')
ax[0].bar_label(ax[0].containers[0])
ax[0].set_ylabel('Number of Animals')
ax[0].grid(False)
sns.despine(left=True, ax=ax[0])

# Plot minor
minor_order = [index for index in minor_cond.index if minor_cond[index] > 0] 
sns.barplot(data = minor_cond, ax=ax[1],order=minor_order)
ax[1].set_title('Frequency of Intake Conditions in "Others" category (<1%)')
ax[1].bar_label(ax[1].containers[0])
ax[1].set_ylabel('Number of Animals')
ax[1].grid(False)
sns.despine(left=True, ax=ax[1])
```


    
![png](./README-files/output_120_0.png)
    



```python
freq_table = intakes[['Animal ID', 'Intake Condition']].groupby('Intake Condition', observed=False).count()
freq_table['Frequency'] = freq_table['Animal ID']
freq_table.drop('Animal ID', axis=1)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Frequency</th>
    </tr>
    <tr>
      <th>Intake Condition</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Nursing</th>
      <td>3131</td>
    </tr>
    <tr>
      <th>Sick</th>
      <td>4217</td>
    </tr>
    <tr>
      <th>Other</th>
      <td>231</td>
    </tr>
    <tr>
      <th>Injured</th>
      <td>6549</td>
    </tr>
    <tr>
      <th>Pregnant</th>
      <td>96</td>
    </tr>
    <tr>
      <th>Behavior</th>
      <td>49</td>
    </tr>
    <tr>
      <th>Neonatal</th>
      <td>259</td>
    </tr>
    <tr>
      <th>Feral</th>
      <td>114</td>
    </tr>
    <tr>
      <th>Normal</th>
      <td>112216</td>
    </tr>
    <tr>
      <th>Medical</th>
      <td>176</td>
    </tr>
    <tr>
      <th>Aged</th>
      <td>452</td>
    </tr>
    <tr>
      <th>Space</th>
      <td>4</td>
    </tr>
  </tbody>
</table>
</div>



Väčšina zviera, ktoré útulok príjme sú v normálnom stave, zranené alebo choré.

## 3. Outcome Type
`Outcome Type` je kategorický nominálny príznak. Analyzovať ho budeme rovnako ako `Intake Condition` a `Animal Type`.


```python
cat_desc(outcomes, 'Outcome Type', ylabel= 'Animals', pfigsize=(11,11), pres=2, title='Outcome Types')
```


    
![png](./README-files/output_124_0.png)
    



```python
# outcome types with <1% in dataset
type_count = outcomes['Outcome Type'].value_counts(normalize=True)
types = [condition for condition in type_count.index if type_count[condition] < 0.01]

# Create dataframe with uncommon types
common_types = outcomes[['Outcome Type']].copy()

for type in types:
    common_types['Outcome Type'] = common_types['Outcome Type'].replace(type, 'Others')

# Create data with common types
uncommon_types = outcomes[outcomes['Outcome Type'].isin(types)].value_counts('Outcome Type').sort_values(ascending=False)
common_types = common_types.value_counts('Outcome Type').sort_values(ascending=False)
```


```python
# Plot graphs
#inspired by https://stackoverflow.com/questions/50449628/how-to-remove-0-from-pie-chart
fig, ax = plt.subplots(1,2, figsize=(14,7))
ax[0].pie(common_types, autopct=lambda x: f'{x:.1f}%' if x > 0 else '', pctdistance=1.15)
ax[0].set_title('Frequency of Outcome Types')
my_circle=plt.Circle( (0,0), 0.8, color='white')
ax[0].add_artist(my_circle)
ax[0].legend([index for index in common_types.index.tolist()], title='Outcome Type', bbox_to_anchor=(0.8,0.7)) 

ax[1].pie(uncommon_types, autopct=lambda x: f'{x:.1f}%' if x > 0 else '', pctdistance=1.15)
ax[1].set_title('Frequency of Types in category "Others" (< 1%)')
my_circle=plt.Circle( (0,0), 0.8, color='white')
ax[1].add_artist(my_circle)
ax[1].legend([index for index in uncommon_types.index.tolist() if uncommon_types[index] > 0], title='Ohters Outcome Type', bbox_to_anchor=(0.8,0.7)) 
```




    <matplotlib.legend.Legend at 0x7792b15daed0>




    
![png](./README-files/output_126_1.png)
    



```python
freq_table = intakes[['Animal ID', 'Intake Condition']].groupby('Intake Condition', observed=False).count()
freq_table['Frequency'] = freq_table['Animal ID']
freq_table.drop('Animal ID', axis=1)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Frequency</th>
    </tr>
    <tr>
      <th>Intake Condition</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Nursing</th>
      <td>3131</td>
    </tr>
    <tr>
      <th>Sick</th>
      <td>4217</td>
    </tr>
    <tr>
      <th>Other</th>
      <td>231</td>
    </tr>
    <tr>
      <th>Injured</th>
      <td>6549</td>
    </tr>
    <tr>
      <th>Pregnant</th>
      <td>96</td>
    </tr>
    <tr>
      <th>Behavior</th>
      <td>49</td>
    </tr>
    <tr>
      <th>Neonatal</th>
      <td>259</td>
    </tr>
    <tr>
      <th>Feral</th>
      <td>114</td>
    </tr>
    <tr>
      <th>Normal</th>
      <td>112216</td>
    </tr>
    <tr>
      <th>Medical</th>
      <td>176</td>
    </tr>
    <tr>
      <th>Aged</th>
      <td>452</td>
    </tr>
    <tr>
      <th>Space</th>
      <td>4</td>
    </tr>
  </tbody>
</table>
</div>



Väčšina zvierat, ktoré útulok opustilo, boli adoptované, presunuté do iného útulku, vrátené majiteľovi alebo podstúpili eutanáziu. Najmenej častým dôvodom je útek zvieraťa. Viac ako polovica hodnôt, ktoré príznak nadobúda má relatívnu početnosť <1%, preto som sa rozhodol kategórie opäť rozdeliť. 

## 4. Bivariačná štatistika medzi príznakmi `Outcome Type` a `Animal Type`
`Outcome Type` a `Animal Type` sú kategorické nominálne príznaky, preto bude bivariačná popisná štatistika pozostávať z kontingenčnej tabuľky, pomocou ktorej zistím vzťah medzi príznakmi. Tú budeme vizualizovať pomocou tabuľky relatívnej početnosti a stĺpcového grafu.


```python
display(pd.crosstab(outcomes['Outcome Type'], outcomes['Animal Type'], normalize='columns'))  
```


<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>Animal Type</th>
      <th>Dog</th>
      <th>Other</th>
      <th>Livestock</th>
      <th>Bird</th>
      <th>Cat</th>
    </tr>
    <tr>
      <th>Outcome Type</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Adoption</th>
      <td>0.483462</td>
      <td>0.470138</td>
      <td>0.3750</td>
      <td>0.485507</td>
      <td>0.516635</td>
    </tr>
    <tr>
      <th>Died</th>
      <td>0.003467</td>
      <td>0.013017</td>
      <td>0.0000</td>
      <td>0.010870</td>
      <td>0.013386</td>
    </tr>
    <tr>
      <th>Disposal</th>
      <td>0.000516</td>
      <td>0.006126</td>
      <td>0.0000</td>
      <td>0.061594</td>
      <td>0.000990</td>
    </tr>
    <tr>
      <th>Euthanasia</th>
      <td>0.023886</td>
      <td>0.112557</td>
      <td>0.0625</td>
      <td>0.090580</td>
      <td>0.036879</td>
    </tr>
    <tr>
      <th>Missing</th>
      <td>0.000412</td>
      <td>0.002297</td>
      <td>0.0000</td>
      <td>0.000000</td>
      <td>0.000681</td>
    </tr>
    <tr>
      <th>Return to Owner</th>
      <td>0.263206</td>
      <td>0.027565</td>
      <td>0.2500</td>
      <td>0.050725</td>
      <td>0.048574</td>
    </tr>
    <tr>
      <th>Rto-Adopt</th>
      <td>0.008894</td>
      <td>0.001531</td>
      <td>0.0000</td>
      <td>0.000000</td>
      <td>0.003713</td>
    </tr>
    <tr>
      <th>Transfer</th>
      <td>0.216157</td>
      <td>0.366769</td>
      <td>0.3125</td>
      <td>0.300725</td>
      <td>0.379143</td>
    </tr>
  </tbody>
</table>
</div>



```python
crosstab = pd.crosstab(outcomes['Outcome Type'], outcomes['Animal Type'], normalize='columns')
fig, ax = plt.subplots()
sns.heatmap(crosstab, ax=ax, cmap='Blues')
fig.suptitle('Relative frequency between Animal Types and Outcome Types')
ax.set_title('Each column represents the 100% of animals with one type', fontdict={'fontsize' : 8})
```




    Text(0.5, 1.0, 'Each column represents the 100% of animals with one type')




    
![png](./README-files/output_131_1.png)
    



```python
colors = ['#A5DEF1','#1E1548','#5BE7A9','#2E99B0','#D83F31']
crosstab = pd.crosstab(outcomes['Outcome Type'], outcomes['Animal Type'], normalize='index')
crosstab = crosstab.reset_index()
ax = crosstab.plot(kind='bar', stacked=True,  x='Outcome Type', ylabel='Proportion of Animals', title='Animal Types Proportions in Outcome Types',figsize=(12,8),grid=False, color=colors)
sns.despine(ax=ax, left=True)
```


    
![png](./README-files/output_132_0.png)
    


Z prvého grafu je možné pozorovať, že zvieratá, ktoré útulok opustia boli najčastejšie adoptované alebo presunuté. Najčastejšími vrátenými zvieratami boli hospodárske zvieratá a psy. Druhý graf ukazuje, že hodnotu *Disposal* často nadobúdajú vtáky. 

# 3. Zadané otázky

## 1. Závisí typ odchodu zvierata z útulku (Outcome Type) na typu príchodu (Intake Type)?

Datasety si spojíme do jedného. Z **intakes** a **outcomes** získame len tie zvieratá, ktoré sa nachádzajú v oboch datasetoch (teda existujú informácie o tom kedy zviera prišlo a odišlo). Z týchto zvierat vyberieme tie, ktorých počet prijatí je rovnaký ako počet odchodov. Datasety usporiadame vzostupne podľa `Animal ID` a `DateTime` (získame tak datasety, ktoré majú na rovnakých riadkoch informácie o príchode a odchode toho istého zvieraťa). Datasety spojíme podľa príznaku `Animal ID`.

`Outcome Type` a `Intake Type` sú kategorické príznaky, preto budeme odpoveď hľadať v kontigenčnej tabuľke, ktorú budeme vizualizovať pomocou tabuľky relatívnej početnosti.


```python
# find Animal IDs, that are in intakes and outcomes
only_intakes = intakes[~intakes['Animal ID'].isin(outcomes['Animal ID'].unique())].shape[0]
only_outcomes = outcomes[~outcomes['Animal ID'].isin(intakes['Animal ID'].unique())].shape[0]
print(f"Prienik intakes a outcomes: {intakes[intakes['Animal ID'].isin(outcomes['Animal ID'].unique())].shape[0]}")
print(f'Len v intakes  {only_intakes} ({(only_intakes / intakes.shape[0])*100}%)')
print(f'Len v outcomes  {only_outcomes} ({(only_outcomes / outcomes.shape[0])*100}%)')
```

    Prienik intakes a outcomes: 126861
    Len v intakes  633 (0.4964939526565956%)
    Len v outcomes  810 (0.6345028552628487%)


Vyberieme si podmnožinu zvierat, ktoré sa nachádzajú v oboch datasetoch.


```python
intakes_a = intakes[intakes['Animal ID'].isin(outcomes['Animal ID'].unique())]
outcomes_a = outcomes[outcomes['Animal ID'].isin(intakes['Animal ID'].unique())]
```


```python
# Filter out data that 
data_int = intakes_a.value_counts('Animal ID')
data_out = outcomes_a.value_counts('Animal ID')
valid_ids = [id for id in intakes_a['Animal ID'].unique() if data_int[id] == data_out[id]] 
intakes_a = intakes[intakes['Animal ID'].isin(valid_ids)]
outcomes_a = outcomes[outcomes['Animal ID'].isin(valid_ids)]
```


```python
# Reset indexes and prepare datasets for combining
intakes_a = intakes_a.sort_values(by=['Animal ID', 'DateTime'])
intakes_a = intakes_a.reset_index(drop=True)
intakes_a['DateTime_intake'] = intakes_a['DateTime']
intakes_a = intakes_a.drop(['DateTime'], axis=1)
```


```python
outcomes_a = outcomes_a.sort_values(by=['Animal ID', 'DateTime'])
outcomes_a = outcomes_a.reset_index(drop=True)
outcomes_a['DateTime_outcome'] = outcomes_a['DateTime']
outcomes_a = outcomes_a.drop(['DateTime'], axis=1)
```


```python
# Join datasets and filter out unusable data
combined = intakes_a.join(outcomes_a, rsuffix='_out')
combined = combined.drop(combined[combined['DateTime_intake'].dt.date > combined['DateTime_outcome'].dt.date].index)
```


```python
# Plot the data
data = pd.crosstab(combined['Intake Type'], combined['Outcome Type'], normalize='index')
fig, ax = plt.subplots()
sns.heatmap(data, ax=ax, cmap='Blues')
fig.suptitle("Crosstabulation between Outcome Type and Intake Type")
ax.set_title('Each Row represents 100% of animals with one type', fontdict={'fontsize' : 10})
```




    Text(0.5, 1.0, 'Each Row represents 100% of animals with one type')




    
![png](./README-files/output_144_1.png)
    


Medzi typom príchodu a odchodu existuje vzťah. 
Z grafu je možné vyčítať, že ak zviera malo dôvod prijatia:
- Public Assist - zvieratá sú najčastejšie vrátené majiteľovi
- Stray - zvieratá sú adoptované alebo presunuté
- Owner surrender - zvieratá sú adoptované alebo presunuté
- Euthanasia request - zvieratá podstúpia eutanáziu 
- Wildslife - zvieratá podstúpia eutanáziu
- Abandoned - zvieratá sú adoptované alebo presunuté do iného útulku

## 2. Je príjem v rámci roku konstantný alebo existujú obdobia s vyššou/nižšou záťažou?

Z príznaku `DateTime` vytvoríme nové príznaky roku a mesiaca. Vypočítame počet prijatých zvierat v jednotlivých mesiacoch. Tieto hodnoty vizualizujeme pomocou stĺpcových grafov. Vytvoríme graf pre každý rok s cieľom overiť či graf za celé obdobie zodpovedá realite.


```python
# Extract parts of datetime
dt_stats = intakes[['DateTime']].copy()
dt_stats['Year'] = dt_stats['DateTime'].dt.year
dt_stats['Month'] = dt_stats['DateTime'].dt.month
dt_stats['Day'] = dt_stats['DateTime'].dt.day
```

Roky 2013 a 2022 do grafu nezahrnieme, pretože dataset neobsahuje záznamy z celého roku.


```python
years = dt_stats['Year'].unique().tolist()
years.sort()
years.remove(2013)
```


```python
# All years
index = 0
ticks = [i for i in range(0, 2001, 250)]

fig, ax  = plt.subplots(3,3, tight_layout=True, figsize=(11,11))
fig.suptitle('Monthly Animal Intakes between 2014-2021')
for i in range(3):
    for j in range(3):
        sns.barplot(data=dt_stats[dt_stats['Year'] == years[index]].groupby('Month').count(), x='Month', y='DateTime', ax=ax[i][j])
        ax[i][j].set_title(years[index])
        ax[i][j].set_ylabel('Number of Animals')
        ax[i][j].set_yticks(ticks)
        ax[i][j].grid(False)
        index+=1
ax[2][2].set_visible(False)
```


    
![png](./README-files/output_151_0.png)
    



```python
fig, ax = plt.subplots()
sns.barplot(data=dt_stats.groupby('Month').count(), x='Month', y='DateTime', ax=ax)
ax.set_ylabel('Number of Animals')
ax.set_title('Combined Monthly Animal Intakes between 2014-2021')
ax.grid(False)
sns.despine(ax=ax)
```


    
![png](./README-files/output_152_0.png)
    


Grafy zobrazujú počet prijatých zvierat počas roka za obdobie 2014-2021. V grafoch je vidieť, že najviac zvierat útulok prijal v letných mesiacoch (Máj, Jún, Júl) a v Októbri. Najmenej zvierat útulok prijal počas zimných mesiacov (December, Január, Február). 

Takéto rozloženie prijatých zvierat je možné (s menšími odchýlkami) pozorovať počas všetkých rokov okrem roku 2020. Počet prijatých zvierat bol na začiatku roka porovnateľný s predchádzajúcimi, no po Februári počet prijatých zvierat neobvykle klesol.

## 3. Hraje vek zvieraťa rolu pri adopcii?

Z datasetu **combined**, ktorý sme použili v prvej otázke, získame tú časť zvierat, ktorá bola adoptovaná. Vek týchto zvierat budeme vizualizovať pomocou histogramu veku. Keďže s rastúcim vekom klesá počet zviera v útulku ako takých, počet adoptovaných zvierat porovnám s celkovým počtom zvierat, ktoré útulok opustili.  


```python
# convert dataframe to long format
tmp = combined[['Outcome Type', 'Age upon Outcome']].copy()
for type in combined['Outcome Type'].unique():
    if type == 'Adoption':
        continue
    tmp['Outcome Type'] = tmp['Outcome Type'].replace(type, 'Other')
    
tmp = tmp.melt(id_vars='Outcome Type')
```


```python
fig, ax = plt.subplots(figsize=(12,8))
sns.histplot(data=tmp, x='value' ,hue='Outcome Type', bins=20, ax=ax)
ax.set_ylabel('Number of Animals')
ax.set_xlabel('Animal Age (years)')
ax.set_title('Age Distribution of Animals upon Outcome')
ax.set_xticks([i for i in range(0,30, 5)])
ax.grid(False)
sns.despine(ax=ax)
```


    
![png](./README-files/output_157_0.png)
    


V priloženom histograme je možné vidieť, že čím staršie sú zvieratá, tým väčší je relatívny rozdiel medzi adoptovanými zvieratami a zvieratami, ktoré adoptované neboli. Väčšina adoptovaných zvierat má menej ako 2 roky.

# 4. Vlastné otázky

## 1.Ktorá časť dňa je pre útulok najvyťaženejšia

Na [stránkach](https://www.austintexas.gov/austin-animal-center) útulku je možné nájsť otváracie hodiny útulku: od 7:00 do 19:00. Otázkou je, ktorá časť pracovného dňa je najvyťaženejšia. Na otázku odpovieme pomocou 2 vizualizácií. 

1. Distribúciu záznamov počas jedného dňa vizualizujem pomocou Violin grafu.

2. Deň si rozdelíme na časti **Morning**, **Afternoon**, **Evening** a **Night** a vizualizujeme ich početnosť pomocou prstencového grafu (donut chart).


```python
def checkTime(data):
    """Assign the part of a day"""
    if data.hour >= 5 and data.hour <= 12:
        return 'Morning'
    elif data.hour > 12 and data.hour <= 17:
        return 'Afternoon'
    elif data.hour > 17 and data.hour <= 21:
        return 'Evening'
    else:
        return 'Night'
```


```python
# Extract the time from the combined dataframe  
dt_stats = combined[['DateTime_intake', 'DateTime_outcome']].copy()
dt_stats = dt_stats.melt()
dt_stats['Day'] = dt_stats['value'].dt.day
dt_stats['Time'] = dt_stats['value'].dt.time.apply(lambda x: (x.hour + x.minute / 60%24))
dt_stats['DayPart'] = dt_stats['value'].copy().apply(checkTime)


days = dt_stats['DayPart'].value_counts(normalize=True)
days = np.array([days['Morning'], days['Afternoon'], days['Evening'], days['Night']])
labels = np.array(['Morning 5:00-12:00', 'Afternoon 12:00-17:00', 'Evening 17:00-21:00', 'Night 21:00-5:00'])
```


```python
fig, ax = plt.subplots(figsize=(7,7))
ax.set_title('Distribution of Intakes/Outcomes Throughout a Day"')
ax.set_xlabel('Number of Intakes/Outcomes')
ax.set_ylabel('Hour')
ax.axis(ymax=24, ymin=0)
ax.set_yticks([i for i in range(24)])
ax.set_yticklabels([i for i in range(24)])
sns.violinplot(dt_stats, y='Time', ax=ax)
```




    <Axes: title={'center': 'Distribution of Intakes/Outcomes Throughout a Day"'}, xlabel='Number of Intakes/Outcomes', ylabel='Hour'>




    
![png](./README-files/output_163_1.png)
    



```python
# Donut chart inspired by: https://python-graph-gallery.com/donut-plot/
fig, ax = plt.subplots(figsize=(7,7), constrained_layout=True)
ax.pie(days, autopct='%.01f%%',pctdistance=1.1)
ax.set_title('Relative Frequency of Intakes/Outcomes Throughout a Day')
my_circle=plt.Circle( (0,0), 0.8, color='white')
ax.add_artist(my_circle)
ax.legend(labels, title='Part of a Day', bbox_to_anchor = [0.8,0.7])
```




    <matplotlib.legend.Legend at 0x7792af0d3950>




    
![png](./README-files/output_164_1.png)
    


Z grafov vidieť, že útulok vykoná väčšinu príjmov a výdajov medzi 11:00 a 18:00 hodinou. Najvyťaženejšie časy sú 12:00 a 17:00. Počet prijatých a odovzdaných zvierat potom klesá, avšak nie do nuly. V grafoch je možné pozorovať, že v datasete má časť dát nesprávne priradený čas alebo útulok vytváral záznamy aj mimo otváracích hodín.

## 2. Ako sa menila zaplnenosť útulku a pomer zvierat v priebehu rokov.
V súčasnosti útulok prijíma zvieratá v urgentných prípadoch kvôli [nedostatočným kapacitám ](https://www.austintexas.gov/lost-found-pets). Otázka je preto, či zaplnenosť útulku závisí na časti roku. Zaujímať nás budú zvieratá, ktoré do útulku prišli, a ktoré útulok opustili. Využijeme spojený dataset *combined*. Dáta si usporiadame podľa príznaku `DateTime`. Pre každý záznam budeme počítať aktuálnu zaplnenosť. Výsledok vizualizujem pomocou čiarového grafu.


```python
# Extract the DateTime and Animal Type from a dataframe
data = combined[['DateTime_intake', 'DateTime_outcome', 'Animal Type']].melt(id_vars=['Animal Type']).sort_values('value')
data['value'] = data['value'].dt.date
data = data.sort_values(['value']).reset_index(drop=True)
```


```python
# Count the current capacity for each row in dataset
n_animals_t = {'Dog':0, 'Cat':0, 'Bird':0, 'Livestock':0, 'Other':0}
n_animals_arr = {'Dog':[], 'Cat':[], 'Bird':[], 'Livestock':[], 'Other':[]} 
n_animals = []
count = 0

# Calculate capacity
for index in data.index:
    n_animals_t[data.iloc[index]['Animal Type']] += 1 if data.iloc[index]['variable'] == 'DateTime_intake' else -1
    count += 1 if data.iloc[index]['variable'] == 'DateTime_intake' else -1
    for i in n_animals_t.keys():
        n_animals_arr[i].append(n_animals_t[i])
    n_animals.append(count)
```


```python
# combine datasets
for animal in n_animals_arr.keys():
    data[animal] = n_animals_arr[animal]
data['count'] = n_animals
```


```python
# Converta data to a corrent date type
data['value'] = pd.to_datetime(data['value'])
```

Graf tvorený všetkými hodnotami by okrem vyššej výpočtovej záťaže nepriniesol žiadne prínosné poznatky, ktoré by pomohli odpovedať na zadanú otázku, preto si z hodnôt vyberieme len podmnožinu, ktorá bude obsahovať záznamy na začiatku mesiaca. 


```python
indexes = []
for year in range(2014,2022):
    for month in range(1,13):
        # find the earliest record within a given month
        indexes.append(data[(data['value'].dt.year == year) & (data['value'].dt.month == month)].sort_values(by='value').head(1).index.tolist()[0])
subset = data.iloc[indexes]
```


```python
#Plot
fig, ax = plt.subplots(figsize=(12,5))
#ax.grid(axis='y')
ax.grid(False)
ax.stackplot(subset['value'], subset['Dog'], subset['Cat'],subset['Bird'], subset['Livestock'], subset['Other'],
              labels=['Dog', 'Cat', 'Bird', 'Livestock', 'Other'])
ax.legend(loc='upper left')
ax.set_xlim(date(2014,1,1),date(2021,12,1))
ax.set_title('The occupancy at the Austin Animal shelter from 2014 to 2021')
ax.set_ylabel('Number of Animals')
ax.set_xlabel('Year')
ax.spines[:].set_visible(False)
```


    
![png](./README-files/output_173_0.png)
    


Z grafu je možné pozorovať, že zaplnenosť útulku bola počas obdobia 2014-2019 stabilná. Rástla počas letných mesiacov a klesala ku koncu roka. Taktiež je možné pozorovať, že počet psov sa počas celého obdobia menil menej drasticky ako počet mačiek. 

## 3. Aké sú najčastejšie typy túlavých psov a mačiek
Na zodpovedanie tejto otázky využijeme dataset intakes, z ktorého získam túlavé mačky a psy ("Stray"). 
Zistíme početnosť jednotlivých plemien a farieb, ktoré vizualizujeme pomocou histogramu. Relatívnu početnosť pohlaví vizualizujeme pomocou koláčového grafu.


```python
# Filter out animals from the dataframe
dogs = intakes[(intakes['Animal Type'] == 'Dog') & (intakes['Intake Type'] =='Stray')]['Breed'].value_counts()
cats = intakes[(intakes['Animal Type'] == 'Cat') & (intakes['Intake Type'] =='Stray')]['Breed'].value_counts()
dogs_order = [i for i in dogs.index if dogs[i] > 0]
cats_order = [i for i in cats.index if cats[i] > 0]

def typ_animal(animal):
    """Plot the characteristics of averate stray animal"""
    breed = intakes[(intakes['Animal Type'] == animal) & (intakes['Intake Type'] =='Stray')]['Breed'].value_counts()
    breed_order = [i for i in breed.index if breed[i] > 0]

    fig, ax = plt.subplots(figsize=(15,6))
    fig.suptitle(f'Characteristics of Average Stray {animal} found in Austin TX')

    # Breed
    sns.barplot(data=breed, order=breed_order[:5], ax=ax)
    ax.set_title(f'The most common breeds of stray {animal}')
    ax.set_ylabel('Number of stray animals')
    ax.set_xlabel('Breed')
    ax.bar_label(ax.containers[0])
    ax.grid(False)
    sns.despine(ax=ax)
    
    animal_data = intakes[(intakes['Animal Type'] == animal ) &  (intakes['Intake Type'] =='Stray')]
    animal_color_data = animal_data.value_counts('Color')
    animal_color_order = [i for i in animal_color_data.index if animal_color_data[i] > 0]

    
    fig, ax = plt.subplots(1,2, figsize=(15,5), tight_layout=True)
    sui = animal_data.value_counts('Sex upon Intake', normalize=True)

    # Sex upon Intake
    ax[0].pie(sui, labels=sui.index, autopct='%1.1f%%')
    ax[0].set_title('Sex and Sterility distribution of stray animals') 
        
    # Color
    sns.barplot(data=animal_color_data, order=animal_color_order[:5], ax=ax[1])
    ax[1].set_title(f'The most common colors of stray {animal}')
    ax[1].set_ylabel('Number of Intake Animals')
    ax[1].set_xlabel('Color')
    ax[1].set_xlabel('Breed')
    sns.despine(ax=ax[1])
    ax[1].bar_label(ax[1].containers[0])
    ax[1].grid(False)
    
```


```python
typ_animal('Dog')
```


    
![png](./README-files/output_177_0.png)
    



    
![png](./README-files/output_177_1.png)
    


V útulku je možné najčastejšie nájsť čierno-bielych a hnedo-bielych pitbullov, labradorov, čivavy a ovčiakov. Počet feniek a psov je takmer rovnaký, no väčšina nie je sterilizovaná. 


```python
typ_animal('Cat')
```


    
![png](./README-files/output_179_0.png)
    



    
![png](./README-files/output_179_1.png)
    


V útulku je možné najčastejšie nájsť čierne a hnedé domáce krátkosrsté a dlhosrsté mačky. Počet mačiek a kocúrov je takmer rovnaký, no väčšina nie je sterilizovaná. 
