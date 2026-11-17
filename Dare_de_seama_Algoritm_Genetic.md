# Darea de seamă – Lucrare de laborator: Algoritm genetic

**Disciplina:** Metode de optimizare / Inteligență artificială
**Tema:** Maximizarea unei funcții de o variabilă cu ajutorul algoritmului genetic
**Limbaj / mediu:** Java 11+ (Swing, Maven, IntelliJ IDEA)

---

## 1. Scopul lucrării

Însușirea principiilor de funcționare ale algoritmului genetic (AG) și implementarea lui pentru găsirea maximului funcției

$$
f(x) = \sin^2(4\pi x) + 0{,}3\,x, \qquad x \in [0, 1]
$$

cu reprezentare binară a soluțiilor, selecție proporțională (metoda ruletei), încrucișare într-un punct și mutație bit cu bit. Se studiază influența probabilităților de încrucișare ($p_c$) și de mutație ($p_m$) asupra convergenței.

## 2. Formularea problemei

| Parametru | Valoare |
|---|---|
| Funcția obiectiv | $f(x)=\sin^2(4\pi x)+0{,}3x$ |
| Domeniul | $x\in[0,1]$ |
| Lungimea cromozomului | $L = 4$ biți (fixă) |
| Mărimea populației | $N = 4$ |
| Numărul de generații | $G = 30$ |
| Generatorul aleator | `java.util.Random`, seed fix (42, 142, 242) pentru reproductibilitate |

**Decodificarea:** $x = \dfrac{b}{2^L-1} = \dfrac{b}{15}$, unde $b$ este valoarea zecimală a cromozomului.

**Funcția de adecvare (fitness):** coincide cu funcția obiectiv, $fitness(x)=f(x)$.

Deoarece spațiul de căutare este foarte mic (16 puncte), se pot calcula toate valorile și se poate afla optimul global exact:

| b | Cromozom | x | f(x) |
|--:|:--:|--:|--:|
| 0 | 0000 | 0.0000 | 0.0000 |
| 1 | 0001 | 0.0667 | 0.5723 |
| 2 | 0010 | 0.1333 | 1.0291 |
| 3 | 0011 | 0.2000 | 0.4055 |
| 4 | 0100 | 0.2667 | 0.1232 |
| 5 | 0101 | 0.3333 | 0.8500 |
| 6 | 0110 | 0.4000 | 1.0245 |
| 7 | 0111 | 0.4667 | 0.3054 |
| 8 | 1000 | 0.5333 | 0.3254 |
| 9 | 1001 | 0.6000 | 1.0845 |
| 10 | 1010 | 0.6667 | 0.9500 |
| 11 | 1011 | 0.7333 | 0.2632 |
| 12 | 1100 | 0.8000 | 0.5855 |
| **13** | **1101** | **0.8667** | **1.2491** |
| 14 | 1110 | 0.9333 | 0.8323 |
| 15 | 1111 | 1.0000 | 0.3000 |

**Optimul global pe grila de 4 biți:** $b^*=13$ (`1101`), $x^*\approx0{,}8667$, $f(x^*)\approx1{,}2491$.
Funcția are mai multe maxime locale (`0010` – 1.0291; `0110` – 1.0245; `1001` – 1.0845), deci este o problemă multimodală, potrivită pentru a ilustra riscul convergenței premature.

## 3. Descrierea algoritmului

1. **Inițializare** – se generează aleator $N$ cromozomi de $L$ biți.
2. **Evaluare** – se decodifică fiecare cromozom, se calculează $x$ și $fitness$; se determină min / medie / max pe generație.
3. **Selecție (ruleta)** – probabilitatea de alegere a individului $i$ este $P_i=\dfrac{f_i+\varepsilon}{\sum_j (f_j+\varepsilon)}$, cu $\varepsilon=10^{-6}$ (pentru ca un individ cu $f=0$ să aibă șanse nenule). Dacă ambii părinți coincid ca indice, al doilea se realege diferit.
4. **Încrucișare într-un punct** – cu probabilitatea $p_c$ (dacă $r<p_c$), punctul de tăiere se alege uniform din $[1, L-1]$ și părinții își schimbă cozile; altfel descendenții copiază părinții.
5. **Mutație** – fiecare bit se inversează independent cu probabilitatea $p_m$.
6. **Înlocuirea generației** – noua populație o înlocuiește integral pe cea veche (**fără elitism**). Pașii 2–6 se repetă $G$ generații.
7. Se reține cel mai bun individ găsit pe parcurs și generația în care a apărut prima dată.

## 4. Arhitectura programului

```
GeneticAlgorithmLab/
├── pom.xml
├── src/main/java/com/galab/
│   ├── model/    Individual, FitnessFunction, GenerationStats, ExperimentResult
│   ├── engine/   SelectionOperator / RouletteWheelSelection
│   │             CrossoverOperator / OnePointCrossover
│   │             MutationOperator  / BitFlipMutation
│   │             GeneticAlgorithmEngine
│   ├── ui/       GeneticAlgorithmLabApp (Swing), FitnessChartPanel (grafic)
│   └── util/     GaConstants (L=4, 2^L-1=15)
```

| Strat | Responsabilitate |
|---|---|
| `model` | Entități imuabile: individ (decodare + fitness calculate la creare), statistica generației, rezultatul experimentului |
| `engine` | Algoritmul și operatorii genetici, expuși prin **interfețe** (strategie), deci pot fi înlocuiți fără a modifica motorul |
| `ui` | Fereastră Swing: parametri, tabele ale populației, jurnal, tabel comparativ, grafic max fitness |
| `util` | Constante |

Fragment reprezentativ – decodarea și funcția obiectiv:

```java
// Individual.java
this.x = (double) decimalValue / GaConstants.MAX_DECIMAL_VALUE;   // b / (2^L - 1)
this.fitness = FitnessFunction.evaluate(this.x);

// FitnessFunction.java
double s = Math.sin(4.0 * Math.PI * x);
return s * s + 0.3 * x;
```

Fragment – încrucișarea într-un punct:

```java
double r = random.nextDouble();
if (r < pc) {
    int cut = 1 + random.nextInt(L - 1);
    child1 = p1.substring(0, cut) + p2.substring(cut);
    child2 = p2.substring(0, cut) + p1.substring(cut);
}
```

## 5. Modul de rulare

```bash
mvn compile exec:java            # prin Maven
./compile_and_run.sh             # doar cu JDK
```

La pornire aplicația rulează automat cele 3 experimente obligatorii și afișează: tabelul populației, jurnalul detaliat al fiecărei generații, tabelul comparativ și graficul evoluției fitness‑ului maxim.

## 6. Experimente

Toate experimentele folosesc $N=4$, $L=4$, $G=30$. Seed‑urile sunt 42, 142 și 242 (seed de bază + 100·i).

| Experiment | $p_c$ | $p_m$ | Caracterizare |
|---|---:|---:|---|
| 1 | 0.80 | 0.05 | parametri standard, echilibrați |
| 2 | 0.40 | 0.01 | încrucișare și mutație rare |
| 3 | 0.90 | 0.15 | încrucișare și mutație foarte frecvente |

### 6.1 Exemplu de jurnal (Experimentul 1, generația 1)

```text
Поколение: 1

Популяция:
1. Хромосома: 1010 | Десятичное значение: 10 | x: 0.6667 | fitness: 0.9500
2. Хромосома: 0101 | Десятичное значение: 5 | x: 0.3333 | fitness: 0.8500
3. Хромосома: 1010 | Десятичное значение: 10 | x: 0.6667 | fitness: 0.9500
4. Хромосома: 0001 | Десятичное значение: 1 | x: 0.0667 | fitness: 0.5723

Селекция:
- Пара 1: Родитель 1: особь №2 (0101) | Родитель 2: особь №3 (1010) [Пропорциональная селекция (рулетка)]
- Пара 2: Родитель 1: особь №2 (0101) | Родитель 2: особь №3 (1010) [Пропорциональная селекция (рулетка)]

Кроссовер:
- Пара 1 -> Кроссовер выполнен: нет (r=0.92 >= pc=0.80), потомки копируют родителей: 0101 и 1010
- Пара 2 -> Кроссовер выполнен: да (r=0.39 < pc=0.80)
  Точка кроссовера: 2
  Потомки: 0110 и 1001

Мутация:
- Мутация в особи №4, бит 1 изменен с 1 на 0

Статистика поколения:
- Максимальный fitness: 0.9500
- Минимальный fitness: 0.5723
- Средний fitness: 0.8306
--------------------------------------------------------------------------------
```

### 6.2 Evoluția pe generații

**Experimentul 1** ($p_c=0{,}80$; $p_m=0{,}05$)

| Gen. | min | medie | max |
|---:|---:|---:|---:|
| 1 | 0.5723 | 0.8306 | 0.9500 |
| 2 | 0.5723 | 0.8492 | 1.0245 |
| 3 | 0.5723 | 0.9195 | 1.2491 |
| 4 | 0.5723 | 0.9195 | 1.2491 |
| 5 | 0.3000 | 0.7865 | 1.2491 |
| 6 | 0.5723 | 0.8198 | 1.0245 |
| 7 | 0.5723 | 0.9676 | 1.2491 |
| 8 | 0.3054 | 0.9009 | 1.2491 |
| 9 | 0.8323 | 0.9901 | 1.2491 |
| 10 | 0.1232 | 0.2644 | 0.4055 |
| 11 | 0.1232 | 0.3766 | 0.5723 |
| 12 | 0.1232 | 0.3933 | 0.5723 |
| 13 | 0.3054 | 0.4388 | 0.5723 |
| 14 | 0.0000 | 0.1527 | 0.3054 |
| 15 | 0.3054 | 0.5777 | 0.8500 |
| 16 | 0.3054 | 0.7139 | 0.8500 |
| 17 | 0.3054 | 0.5777 | 0.8500 |
| 18 | 0.1232 | 0.5322 | 0.8500 |
| 19 | 0.3054 | 0.7139 | 0.8500 |
| 20 | 0.3054 | 0.7139 | 0.8500 |
| 21 | 0.8500 | 0.8500 | 0.8500 |
| 22 | 0.8500 | 0.8500 | 0.8500 |
| 23 | 0.8500 | 0.8500 | 0.8500 |
| 24 | 0.3054 | 0.7139 | 0.8500 |
| 25 | 0.3054 | 0.7139 | 0.8500 |
| 26 | 0.3054 | 0.6444 | 0.8500 |
| 27 | 0.8500 | 0.9934 | 1.2491 |
| 28 | 0.8323 | 0.9453 | 1.2491 |
| 29 | 0.8323 | 1.0887 | 1.2491 |
| 30 | 0.3000 | 0.8515 | 1.2491 |

**Experimentul 2** ($p_c=0{,}40$; $p_m=0{,}01$)

| Gen. | min | medie | max |
|---:|---:|---:|---:|
| 1 | 0.1232 | 0.4927 | 0.9500 |
| 2 | 0.5723 | 0.9090 | 1.0845 |
| 3 | 0.5723 | 0.7948 | 1.0845 |
| 4 | 0.5723 | 0.8145 | 1.0845 |
| 5 | 0.5723 | 0.7948 | 1.0845 |
| 6 | 0.2632 | 0.5614 | 1.0845 |
| 7 | 0.2632 | 0.7511 | 1.0845 |
| 8 | 0.2632 | 0.7511 | 1.0845 |
| 9 | 0.2632 | 0.6231 | 1.0845 |
| 10 | 0.5723 | 0.7003 | 1.0845 |
| 11 | 0.2632 | 0.6231 | 1.0845 |
| 12 | 0.5723 | 0.8284 | 1.0845 |
| 13 | 0.5723 | 0.8284 | 1.0845 |
| 14 | 0.5723 | 0.8284 | 1.0845 |
| 15 | 0.5723 | 0.9564 | 1.0845 |
| 16 | 1.0845 | 1.0845 | 1.0845 |
| 17 | 1.0845 | 1.0845 | 1.0845 |
| 18 | 1.0845 | 1.0845 | 1.0845 |
| 19 | 1.0845 | 1.0845 | 1.0845 |
| 20 | 1.0845 | 1.0845 | 1.0845 |
| 21 | 1.0845 | 1.0845 | 1.0845 |
| 22 | 1.0845 | 1.0845 | 1.0845 |
| 23 | 1.0845 | 1.0845 | 1.0845 |
| 24 | 1.0845 | 1.1256 | 1.2491 |
| 25 | 1.0845 | 1.1668 | 1.2491 |
| 26 | 1.0845 | 1.1668 | 1.2491 |
| 27 | 1.0845 | 1.1256 | 1.2491 |
| 28 | 1.0845 | 1.1256 | 1.2491 |
| 29 | 1.0845 | 1.0845 | 1.0845 |
| 30 | 1.0845 | 1.0845 | 1.0845 |

**Experimentul 3** ($p_c=0{,}90$; $p_m=0{,}15$)

| Gen. | min | medie | max |
|---:|---:|---:|---:|
| 1 | 0.0000 | 0.3539 | 1.0291 |
| 2 | 0.3254 | 0.5831 | 1.0291 |
| 3 | 0.0000 | 0.3102 | 0.5723 |
| 4 | 0.0000 | 0.4569 | 0.8500 |
| 5 | 0.0000 | 0.2502 | 0.5723 |
| 6 | 0.0000 | 0.2958 | 0.5723 |
| 7 | 0.3054 | 0.6881 | 1.0245 |
| 8 | 0.5723 | 0.8678 | 1.0245 |
| 9 | 0.8323 | 0.8892 | 1.0245 |
| 10 | 0.1232 | 0.5758 | 1.0245 |
| 11 | 0.1232 | 0.4408 | 1.0291 |
| 12 | 0.0000 | 0.7073 | 1.0291 |
| 13 | 0.0000 | 0.4936 | 1.0245 |
| 14 | 0.2632 | 0.6531 | 1.0245 |
| 15 | 0.2632 | 0.4502 | 0.8323 |
| 16 | 0.1232 | 0.5716 | 0.9500 |
| 17 | 0.2632 | 0.7981 | 1.0291 |
| 18 | 0.3254 | 0.6283 | 0.9500 |
| 19 | 0.2632 | 0.7825 | 1.0845 |
| 20 | 0.2632 | 0.7981 | 1.0291 |
| 21 | 0.5723 | 0.8742 | 1.0245 |
| 22 | 0.0000 | 0.3047 | 0.8323 |
| 23 | 0.0000 | 0.5403 | 1.0291 |
| 24 | 0.4055 | 0.8228 | 1.0291 |
| 25 | 0.8500 | 0.9832 | 1.0291 |
| 26 | 0.1232 | 0.6675 | 1.0245 |
| 27 | 0.3054 | 0.4421 | 0.8323 |
| 28 | 0.3054 | 0.7978 | 1.0291 |
| 29 | 0.0000 | 0.7017 | 1.0245 |
| 30 | 0.8323 | 0.9799 | 1.0291 |

### 6.3 Tabel comparativ

| Experiment | $p_c$ | $p_m$ | Cel mai bun cromozom | b | x* | f(x*) | Prima apariție (gen.) | Fitness mediu, gen. 30 | Media fitness-ului mediu (30 gen.) |
|---|---:|---:|:--:|--:|--:|--:|--:|--:|--:|
| 1 | 0.80 | 0.05 | 1101 | 13 | 0.8667 | **1.2491** | 3 | 0.8515 | 0.7313 |
| 2 | 0.40 | 0.01 | 1101 | 13 | 0.8667 | **1.2491** | 24 | 1.0845 | 0.9271 |
| 3 | 0.90 | 0.15 | 1001 | 9 | 0.6000 | 1.0845 | 19 | 0.9799 | 0.6236 |

Statistici suplimentare din jurnale (30 generații × 2 perechi = 60 încrucișări posibile, 4 indivizi × 4 biți × 30 gen. = 480 biți posibili de mutat):

| Experiment | Încrucișări efectuate | Încrucișări omise | Biți mutați | Generații fără mutații | Generații cu max = optim |
|---|--:|--:|--:|--:|--:|
| 1 | 42 | 18 | 24 | 13 | 10 |
| 2 | 22 | 38 | 3 | 27 | 5 |
| 3 | 56 | 4 | 72 | 0 | 0 |

### 6.4 Graficul fitness‑ului maxim pe generații (rezumat)

Graficul complet se află în interfața aplicației (`FitnessChartPanel`). Valorile esențiale:

- **Exp. 1:** 0.95 → **1.2491 la gen. 3**, se menține până la gen. 9, apoi scade (gen. 10: 0.4055) din cauza lipsei elitismului, revine la 0.85, apoi regăsește optimul la gen. 27–30.
- **Exp. 2:** 0.95 → 1.0845 (maxim local `1001`) din gen. 2; populația se uniformizează complet la gen. 16–23 (toți indivizii = `1001`); optimul `1101` apare abia la gen. 24 prin mutație/încrucișare, dar se pierde la gen. 29.
- **Exp. 3:** fitness‑ul maxim oscilează între 0.57 și 1.08; optimul nu este atins niciodată.

## 7. Analiza rezultatelor

**Experimentul 1 (pc = 0.8, pm = 0.05).** Echilibrul dintre explorare (încrucișare frecventă) și mutația moderată a permis găsirea rapidă a optimului global (generația 3) și revenirea la el după o pierdere temporară. Diversitatea populației rămâne suficientă (media variază între 0.15 și 1.09). Este cel mai bun rezultat ca viteză a convergenței.

**Experimentul 2 (pc = 0.4, pm = 0.01).** Cu doar 3 biți mutați în 30 de generații și 27 de generații fără nicio mutație, populația a „înghețat” rapid în maximul local `1001` (f = 1.0845): din gen. 16 toți indivizii sunt identici, iar încrucișarea între indivizi identici nu produce nimic nou. Este un exemplu clasic de **convergență prematură**. Optimul a fost găsit doar tardiv (gen. 24) printr-o mutație norocoasă. Media fitness‑ului este mare (0.93), dar pentru că populația este uniformă într-un optim local, nu din cauză că ar fi aproape de optimul global.

**Experimentul 3 (pc = 0.9, pm = 0.15).** Cu 72 de biți mutați (în medie 2,4 pe generație, adică ~15 % din biți) mutația distruge sistematic soluțiile bune: nu există nicio generație fără mutații, iar media fitness‑ului este cea mai mică (0.62). Algoritmul se comportă aproape ca o **căutare aleatoare**, nu reușește să stabilizeze optimul și se oprește la 1.0845 (`1001`), sub optimul global.

**Influența parametrilor – concluzii:**

| Parametru | Valoare mică | Valoare mare |
|---|---|---|
| $p_c$ | amestec redus de material genetic, progres lent | explorare intensă; în exces poate distruge scheme bune |
| $p_m$ | pierderea diversității, blocare în optim local | distruge soluțiile bune, căutare aleatoare |

Valorile intermediare ($p_c\approx0{,}8$; $p_m\approx0{,}05$) dau cel mai bun compromis explorare / exploatare.

**Observații privind implementarea și limitările:**

- Nu se folosește **elitism**: cel mai bun individ poate fi pierdut (Exp. 1, gen. 10; Exp. 2, gen. 29). Reținerea celui mai bun individ global în program compensează acest lucru în raportarea rezultatului, dar nu în populație.
- Cu $N=4$ și $L=4$ spațiul este minuscul (16 puncte), iar AG nu aduce avantaj față de enumerarea completă; scopul este didactic.
- Rezultatele depind de seed (o singură rulare per configurație); pentru concluzii statistice ar fi necesare multiple rulări independente cu medierea rezultatelor.
- Selecția cu ruleta cu populație de 4 indivizi are variație mare; turneul sau rang‑selecția ar fi alternative.

## 8. Concluzii

1. Algoritmul genetic implementat (ruletă + încrucișare într-un punct + mutație bit cu bit) găsește corect optimul global al grilei de 4 biți: `1101`, $x^*=0{,}8667$, $f(x^*)=1{,}2491$, confirmat prin enumerarea celor 16 valori.
2. Experimentele 1 și 2 au atins optimul global (în generația 3, respectiv 24), Experimentul 3 – doar un maxim local (1.0845).
3. Parametrii $p_c$ și $p_m$ influențează decisiv comportamentul: $p_m$ prea mic duce la convergență prematură, $p_m$ prea mare la o căutare haotică; configurația standard (0.8 / 0.05) este cea mai eficientă.
4. Arhitectura bazată pe interfețe permite extinderea ușoară (alți operatori de selecție, încrucișare, mutație, elitism) fără modificarea motorului.

## Anexă – fișiere sursă

`Individual.java`, `FitnessFunction.java`, `GenerationStats.java`, `ExperimentResult.java`, `GaConstants.java`, `SelectionOperator.java`, `RouletteWheelSelection.java`, `CrossoverOperator.java`, `OnePointCrossover.java`, `MutationOperator.java`, `BitFlipMutation.java`, `GeneticAlgorithmEngine.java`, `GeneticAlgorithmLabApp.java`, `FitnessChartPanel.java`.
