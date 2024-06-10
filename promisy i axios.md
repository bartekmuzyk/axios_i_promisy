# Spis treści
- Promise'y
  - [Zadanie 4](#zadanie-4)
  - [Zadanie 5](#zadanie-5)
- Axios
  - [Zadanie 1.1](#zadanie-11)
  - [Zadanie 1.2](#zadanie-12)
  - Zadanie 2 **(chyba jutro, w sensie 11 czerwca)**

# Promise'y

## Zadanie 4

### Zacznijmy od `Promise.all`
`Promise.all` jest przydatną funkcją, która przyjmuje jeden argument - **listę Promise'ów**.

Działanie tej funkcji wygląda następująco:
- Weź wszystkie Promise'y z listy podanej jako argument i sam zwróć **nowy Promise** (nazwijmy ten Promise literą `p` na potrzeby wyjaśnienia).
- Poczekaj, aż wszystkie Promise'y z listy osiągną stan `fulfilled` (czyli wszystkie wywołają u siebie `resolve`).
- Kiedy tak się stanie, wypełnij Promise `p` listą wartości, które zwróciły Promise'y z listy (w takiej samej kolejności, w której zostały podane)

> [!IMPORTANT]
> Jeżeli wywołujemy `Promise.all` w taki sposób:
> ```js
> Promise.all(promisesTab);
> ```
> i mamy do dyspozycji następujące funkcje asynchroniczne (tzn. takie, które zwracają `Promise`):
> ```js
> function a() {
>     return new Promise(resolve => /* ... */);
> }
> 
> function b() {
>     return new Promise(resolve => /* ... */);
> }
> 
> function c() {
>     return new Promise(resolve => /* ... */);
> }
> ```
> 
> to lista `promisesTab` musi wyglądać tak:
> ```js
> const promisesTab = [a(), b(), c()];  // ✅ Dobrze
> ```
> a nie tak:
> ```js
> const promisesTab = [a, b, c];  // ❌ Źle
> ```
> 
> Jest tak, ponieważ funkcja `Promise.all` przyjmuje listę ***samych Promise'ów***, a nie *funkcji, które zwracają Promise'y*. Ponieważ funkcje `a`, `b` i `c` *zwracają* Promise'y, to żeby uzyskać te Promise'y należy te funkcje **wywołać**.

#### Przykład użycia `Promise.all`
```js
const a = new Promise((resolve) => {
    setTimeout(() => resolve("a"), 3000);
});

const b = new Promise((resolve) => {
    setTimeout(() => resolve("b"), 2000);
});

const c = new Promise((resolve) => {
    setTimeout(() => resolve("c"), 4000);
});

Promise.all([a, b, c])
    .then(literki => {
        console.log(literki); // Wyświetli na ekranie ["a", "b", "c"] po 4 sekundach (bo tyle trwało zwrócenie wartości przez Promise'a, który wykonywał się najdłużej).
    });
```

#### Przykład użycia `Promise.all` na funkcjach
```js
function a() {
    return new Promise((resolve) => {
        setTimeout(() => resolve("a"), 3000);
    });
}

function b() {
    return new Promise((resolve) => {
        setTimeout(() => resolve("b"), 2000);
    });
}

function c() {
    return new Promise((resolve) => {
        setTimeout(() => resolve("c"), 4000);
    });
}

// Lista wygląda tak samo, tylko teraz - zamiast przekazywać samo a, b i c - przekazujemy wywołania a, b i c
Promise.all([a(), b(), c()])
    .then(literki => {
        console.log(literki); // Wyświetli na ekranie ["a", "b", "c"] po 4 sekundach (bo tyle trwało zwrócenie wartości przez Promise'a, który wykonywał się najdłużej).
    });
```

### `Promise.all` można zagnieżdżać
Ponieważ `Promise.all` zwraca pełnoprawnego Promise'a, który jest tak dobry jak każdy inny, to nic nas nie zatrzymuje przed napisaniem czegoś takiego:
```js
// Zmienne p1, p2, p3 i p4 to Promise'y, które zwracają kolejne liczby
const p1 = new Promise(resolve => resolve(1));
const p2 = new Promise(resolve => resolve(2));
const p3 = new Promise(resolve => resolve(3));
const p4 = new Promise(resolve => resolve(4));

// Tworzymy sobie *dwie* listy z Promise'ami - lista1 zawiera pierwsze dwa Promise'y, lista2 zawiera kolejne dwa Promise'y
const lista1 = [p1, p2];
const lista2 = [p3, p4];

// Tworzymy sobie dwa Promise'y, które owiną Promise'y z tych dwóch list
const pierwszaCzesc = Promise.all(lista1);
const drugaCzesc = Promise.all(lista2);

// Tworzymy jeden wielki Promise z tych dwóch części
const listaCalosc = [pierwszaCzesc, drugaCzesc];
const calosc = Promise.all(listaCalosc);
```
Skrótowo można to zapisać tak:
```js
// Zmienne p1, p2, p3 i p4 to Promise'y, które zwracają kolejne liczby
const p1 = new Promise(resolve => resolve(1));
const p2 = new Promise(resolve => resolve(2));
const p3 = new Promise(resolve => resolve(3));
const p4 = new Promise(resolve => resolve(4));

// Tworzymy jeden wielki Promise, na który składają się dwa pod-Promise'y
const calosc = Promise.all([
    Promise.all([p1, p2]),
    Promise.all([p3, p4])
]);
```
Gdyby teraz wykonać
```js
calosc.then(wyniki => {
    console.log(wyniki);
});
```
To w obu przypadkach dostalibyśmy na ekranie coś takiego
```js
[
    [1, 2],
    [3, 4]
]
```
a więc efektywnie podzieliliśmy Promise'y `p1`, `p2`, `p3` i `p4` na dwie części, a wtedy ich wyniki też dostaliśmy w dwóch częściach **dokładnie w takiej samej kolejności w jakiej zostały podane do końcowego `Promise.all`**.

Całość działa, ponieważ wykonując `Promise.all` dostajemy nowy Promise, który *"owija"* Promise'y podane w liście argumentów.

### To jak z samym zadaniem?
Naszym zadaniem jest napisać funkcję `grupuj(funTab1, funTab2, cb)`, która:
- Wykona wszystkie funkcje funTab1 **oraz** funTab2 **równolegle** (tzn. **w tym samym momencie**)
- Poczeka, aż wszystkie funkcje z **obu** tablic zwrócą wyniki
- Po otrzymaniu **wszystkich** wyników pogrupuje je w następujący sposób:
```js
[
    [funTab1_funkcja1, funTab2_funkcja1],
    [funTab1_funkcja2, funTab2_funkcja2],
    [funTab1_funkcja3, funTab2_funkcja3],
    ...
]
```
Można tą postać wyników wyrazić za pomocą tabelki o szerokości **2** i wysokości **równej ilości funkcji w dłuższej liście**:
|            |**funTab1**             |**funTab2**             |
|------------|:-----------------------|-----------------------:|
|**funkcja1**|funTab**1**_funkcja**1**|funTab**2**_funkcja**1**|
|**funkcja2**|funTab**1**_funkcja**2**|funTab**2**_funkcja**2**|
|**funkcja3**|funTab**1**_funkcja**3**|funTab**2**_funkcja**3**|

gdzie poziomo zmienia się **numer tablicy** z funkcjami, a pionowo zmienia się **numer funkcji** z danej tablicy.

W zadaniu znajduje się również taka wzmianka:

> Jeżeli liczba funkcji w obu tablicach nie jest równa, to funkcja grupuj powinna uzupełniać brakujące wyniki wartością 0. 

Chodzi o to, że jeżeli np. `funTab1` zawiera 5 funkcji, a `funTab2` tylko 3, to wynikowa lista będzie wyglądała tak:
|            |**funTab1**             |**funTab2**             |
|------------|:-----------------------|-----------------------:|
|**funkcja1**|funTab**1**_funkcja**1**|funTab**2**_funkcja**1**|
|**funkcja2**|funTab**1**_funkcja**2**|funTab**2**_funkcja**2**|
|**funkcja3**|funTab**1**_funkcja**3**|funTab**2**_funkcja**3**|
|**funkcja4**|funTab**1**_funkcja**3**|0                       |
|**funkcja5**|funTab**1**_funkcja**3**|0                       |

czyli dosłownie:
```js
[
    [funTab1_funkcja1, funTab2_funkcja1],
    [funTab1_funkcja2, funTab2_funkcja2],
    [funTab1_funkcja3, funTab2_funkcja3],
    [funTab1_funkcja4, 0],
    [funTab1_funkcja5, 0]
]
```

#### Zacznijmy implementację!
Zaczynamy od skopiowania definicji funkcji z polecenia zadania:
```js
const grupuj = (funTab1, funTab2, cb) => {
    // do zrobienia :3
};
```

Musimy teraz zrobić wszystkie te kroki, które wypisałem wcześniej:
- Wykona wszystkie funkcje funTab1 **oraz** funTab2 **równolegle** (tzn. **w tym samym momencie**)
- Poczeka, aż wszystkie funkcje z **obu** tablic zwrócą wyniki
- Po otrzymaniu **wszystkich** wyników pogrupuje je w następujący sposób:

Zacznijmy od pierwszego:

#### Wykonanie wszystkich funkcji z `funTab1` i `funTab2` równolegle
Sama idea wykonywania funkcji asynchronicznych równolegle nie jest zbyt skomplikowana.

Funkcje asynchroniczne mają ten benefit, że one zamiast **najpierw robić rzeczy** a **potem zwracać**, **najpierw zwracają** (konkretniej zwracają Promise'a), a **potem robią rzeczy**. Dlatego aby zwrócić wynik z funkcji asynchronicznej, należy użyć `resolve()` - `return` został już użyty w wyrażeniu
```js
return new Promise(/* ... */);
```
więc musimy użyć alternatywnej metody przekazywania wyniku do miejsca, w którym została wywołana funkcja asynchroniczna (np. za pomocą `.then()`).

Ponieważ funkcje asynchroniczne **od razu zwracają**, to wystarczy wykonać je jedna po drugiej aby wykonały się równolegle.

Jeżeli posiadamy funkcje asynchroniczne o nazwach `a`, `b` oraz `c`, to aby wykonać je równolegle możemy napisać coś takiego:
```js
a();
b();
c();
```
Każda z nich od razu zwraca, więc po `a()` od razu wywoła się `b()`, a po `b()` od razu wywoła się `c()`, dlatego że Promise odkłada właściwe wykonanie funkcji na później.

Natomiast istnieje inna funkcja, która może posłużyć do równoległego wykonania dowolnej liczby funkcji asynchronicznych równolegle. Mówiliśmy o niej przed chwilą.

Jeśli chcielibyśmy wykonać równolegle funkcje `a`, `b` i `c`, następnie przechwycić ich wyniki oraz dowiedzieć się kiedy wszystkie z tych funkcji skończą się wykonywać, wymagałoby to napisania takiego kodu:
```js
function cb(wyniki) {
    console.log("wszystkie funkcje skończyły działanie!");
    console.log("oto wyniki, które te funkcje wyprodukowały:");
    console.log(wyniki);
}

const wyniki = Array(3).fill(null);  // Tworzymy listę [null, null, null], bo mamy 3 funkcje do wykonania.
a().then(wynikA => {
    wyniki[0] = wynikA;

    // Sprawdzamy, czy już nie ma żadnego null'a w liście z wynikami.
    if (wyniki.indexOf(null) === -1) {
        cb(wyniki);
    }
});

b().then(wynikB => {
    wyniki[1] = wynikB;

    // Sprawdzamy, czy już nie ma żadnego null'a w liście z wynikami... Znowu...
    if (wyniki.indexOf(null) === -1) {
        cb(wyniki);
    }
});

c().then(wynikC => {
    wyniki[2] = wynikC;

    // Sprawdzamy, czy już nie ma żadnego null'a w liście z wynikami... I jeszcze raz...
    if (wyniki.indexOf(null) === -1) {
        cb(wyniki);
    }
});
```
Jest to rozwiązanie beznadziejne, ponieważ zawiera wiele potwórzonego kodu oraz utworzenia listy wynikowej, którą trzeba ogarniać samemu. Dodatkowo, trzeba pilnować indeksu dla każdej funkcji.

Na szczęście wszystko to automatyzuje nam `Promise.all`, za pomocą którego możemy zrobić to co wyżej w sensowniejszej postaci:
```js
Promise.all([a(), b(), c()])
    .then(wyniki => {
        console.log("wszystkie funkcje skończyły działanie!");
        console.log("oto wyniki, które te funkcje wyprodukowały:");
        console.log(wyniki);
    });
```
Nie ma żadnej potrzeby na dodatkowe listy ani pilnowanie indeksów, ponieważ `Promise.all` robi to sam.

`Promise.all` również wykonuje wszystkie funkcje **równolegle**, ponieważ gdyby wykonywał je po kolei, to potencjalnie mógłby stracić mnóstwo czasu. Gdyby funkcja `b` wykonywała się **10** sekund, a funkcja `c` **4** sekundy, to `Promise.all` może wywołać `b` i `c` na raz i otrzymać wyniki obu funkcji po **10 sekundach**. Gdyby `Promise.all` wykonywał funkcje po kolei, na wyniki obu funkcji czekałby **14 sekund**.

Możemy ten mechanizm wykorzystać w tym zadaniu:
```js
const grupuj = (funTab1, funTab2, cb) => {
    const pierwszaCzesc = Promise.all(funTab1);
    const drugaCzesc = Promise.all(funTab2);
};
```

Pamiętajmy natomiast, że `Promise.all` przyjmuje jako argument listę **Promise'ów**, a nie **funkcji zwracających Promise'y**, więc musimy każdą funkcję z obu tablic, zamienić na jej wywołanie używając `.map()`:
```js
const grupuj = (funTab1, funTab2, cb) => {
    const promisyZFunTab1 = funTab1.map(funkcja => funkcja());
    const pierwszaCzesc = Promise.all(promisyZFunTab1);

    const promisyZFunTab2 = funTab2.map(funkcja => funkcja());
    const drugaCzesc = Promise.all(promisyZFunTab2);
};
```

Natomiast mamy teraz jeszcze jeden problem. Promise `pierwszaCzesc` czeka tylko na ukończenie funkcji z `funTab1`, a Promise `drugaCzesc` czeka tylko na ukończenie funkcji z `funTab2`. Ponieważ `Promise.all` od razu zwraca nowy Promise (jak inne funkcje asynchroniczne), to "czekanie" na oba zestawy funkcji rozpocznie się w tym samym czasie.

To oznacza, że najpierw może skończyć wykonywać się `pierwszaCzesc` albo `drugaCzesc` i nie wiemy, która z nich skończy się wykonywać jako pierwsza. Aby temu zapobiec, możemy po prostu **poczekać na obie części** korzystając z tej samej funkcji.

```js
const grupuj = (funTab1, funTab2, cb) => {
    const promisyZFunTab1 = funTab1.map(funkcja => funkcja());
    const pierwszaCzesc = Promise.all(promisyZFunTab1);

    const promisyZFunTab2 = funTab2.map(funkcja => funkcja());
    const drugaCzesc = Promise.all(promisyZFunTab2);

    const obieCzesci = Promise.all([pierwszaCzesc, drugaCzesc]);
};
```

W efekcie dostajemy Promise'a `obieCzesci`, który skończy się wykonywać dopiero kiedy wszystkie funkcje z `funTab1` **oraz** wszystkie funkcje z `funTab2` skończą się wykonywać. Zacznijmy więc nasłuchiwać na zakończenie działania Promise'a `obieCzesci`.

```js
const grupuj = (funTab1, funTab2, cb) => {
    const promisyZFunTab1 = funTab1.map(funkcja => funkcja());
    const pierwszaCzesc = Promise.all(promisyZFunTab1);

    const promisyZFunTab2 = funTab2.map(funkcja => funkcja());
    const drugaCzesc = Promise.all(promisyZFunTab2);

    const obieCzesci = Promise.all([pierwszaCzesc, drugaCzesc]);

    obieCzesci.then(wynikiZObuTablic => {
        console.log(wynikiZObuTablic);
    });
};
```

Nie jest to natomiast koniec zadania, ponieważ jak możesz zauważyć, nasze `wynikiZObuTablic` będą w odrobinę innym formacie:
```js
[
    [funTab1_funkcja1, funTab1_funkcja2, funTab1_funkcja3, ...],
    [funTab2_funkcja1, funTab2_funkcja2, funTab2_funkcja3, ...]
]
```
W postaci tabelki można pokazać to tak:

|            |**funkcja1**            |**funkcja2**            |**funkcja3**             |
|-----------:|:-----------------------|:-----------------------|:------------------------|
|**funTab1** |funTab**1**_funkcja**1**|funTab**1**_funkcja**2**|funTab**1**_funkcja**3** |
|**funTab2** |funTab**2**_funkcja**1**|funTab**2**_funkcja**2**|funTab**2**_funkcja**3** |

To co chcemy otrzymać, to to co mamy, ale z **odwróconymi osiami poziomymi i pionowymi** (numer tablicy funkcji powinien znajdować się w poziomie, a numer samej funkcji w pionie - teraz jest na odwrót).

Niestety JavaScript nie udostępnia żadnej wbudowanej funkcji, która pozwala takie odwrócenie osi zrobić automatycznie. Musimy też jeszcze wziąć pod uwagę fakt, że `funTab1` i `funTab2` mogą być różnej długości. Taki mechanizm musimy napisać samemu:

```js
const grupuj = (funTab1, funTab2, cb) => {
    const promisyZFunTab1 = funTab1.map(funkcja => funkcja());
    const pierwszaCzesc = Promise.all(promisyZFunTab1);

    const promisyZFunTab2 = funTab2.map(funkcja => funkcja());
    const drugaCzesc = Promise.all(promisyZFunTab2);

    const obieCzesci = Promise.all([pierwszaCzesc, drugaCzesc]);

    obieCzesci.then(wynikiZObuTablic => {
        // Rozdzielmy sobie całościowe wyniki na ich "składowe"
        const wynikiFunTab1 = wynikiZObuTablic[0];  // wyniki z "pierwszaCzesc"
        const wynikiFunTab2 = wynikiZObuTablic[1];  // wyniki z "drugaCzesc"

        // Znajdujemy długość dłuższej z dwóch list
        const maxLength = Math.max(wynikiZFunTab1.length, wynikiZFunTab2.length);

        // Tworzymy pustą listę długości maxLength i wypełniamy ją null'ami
        // Listę musimy wypełnić od razu jakimiś danymi, ponieważ inaczej nie da się bo niej iterować
        const podstawka = Array(maxLength).fill(null);

        // Zamieniamy każdego takiego null'a, na i-te wyniki z obu tablic (wynikiFunTab1 i wynikiFunTab2)
        const poprawnieFormatowaneWyniki = podstawka.map((_, i) => {
            return [
                wynikiFunTab1[i] ?? 0, // Pierwsza pozycja = wynik i-tej funkcji z funTab1 (lub zero jeśli takowego nie ma)
                wynikiFunTab2[i] ?? 0  //    Druga pozycja = wynik i-tej funkcji z funTab2 (lub zero jeśli takowego nie ma)
            ]; // Zwracamy dwuelementową listę (ponieważ mamy dwie kolumny do utworzenia, jedna na każdego funTab'a)
        });

        cb(poprawnieFormatowaneWyniki);  // Dajemy już poprawnie sformatowane wyniki do callbacka przyjętego przy wywołaniu grupuj() przez użytkownika
    });
};
```

# Zadanie 5
Naszym zadaniem jest napisać funkcję `connect(funTab, fun)`, która:
- Wykona wszystkie funkcje z `funTab` **równolegle** (tzn. **w tym samym momencie**)
- Poczeka, aż wszystkie funkcje z tablicy `funTab` zwrócą wyniki
- Po otrzymaniu **wszystkich** wyników funkcji z `funTab` wykona dla każdego z nich `fun(wynik)` **równolegle** (gdzie `wynik` to pojedynczy wynik którejś z funkcji z `funTab`)
- Po otrzymaniu **wszystkich** wyników od wywołań `fun(wynik)`, pogrupować je w takim formacie:
```js
[
    [w1, fun(w1)],
    [w2, fun(w2)],
    [w3, fun(w3)],
    [w4, fun(w4)],
    [w5, fun(w5)],
    // ....
]
```
Można tą postać wyników wyrazić za pomocą tabelki o szerokości **2** i wysokości **równej ilości funkcji w liście `funTab`**:
|     |**funTab**    |**fun**    |
|-----|:-------------|----------:|
|**1**|w**1**        |fun(w**1**)|
|**2**|w**2**        |fun(w**2**)|
|**3**|w**3**        |fun(w**3**)|

Zróbmy wszystko po kolei:

### Kopiujemy definicję funkcji z polecenia
```js
const connect = (funTab, fun) => {
    // do zrobienia :33333
};
```
Od tej pory nowe fragmenty kodu będą oddzielone blokami od kodu, który istniał krok wcześniej.

### Wykonanie wszystkich funkcji z `funTab` równolegle
```js
const connect = (funTab, fun) => {
```
```js
    const promisy = funTab.map(funkcja => funkcja()); // Wydobywamy Promise'y z każdej funkcji z funTab'a
    Promise.all(promisy) // Tworzymy Promise'a, który się spełni jak wszystkie Promise'y z listy "promisy" się spełnią
```
```js
};
```

### Czekamy, aż wszystkie funkcje z `funTab` zwrócą wyniki
```js
const connect = (funTab, fun) => {
    const promisy = funTab.map(funkcja => funkcja());
    Promise.all(promisy)
```
```js
        .then(wynikiZFunTaba => { // Tu nasłuchujemy, aż Promise zwrócony przez Promise.all się spełni
            // "wynikiZFunTaba" zawiera listę wartości zwróconych przez wszystkie funkcje z "funTab"
        });
```
```js
};
```

### Po otrzymaniu wszystkich wyników funkcji z `funTab` wykonujemy dla każdego z nich `fun(wynik)` **równolegle**
```js
const connect = (funTab, fun) => {
    const promisy = funTab.map(funkcja => funkcja());
    Promise.all(promisy)
        .then(wynikiZFunTaba => {
```
```js
            // Zamieniamy każdy wynik na wywołanie "fun" z tym wynikiem podanym jako argument
            const promisy = wynikiZFunTaba.map(wynik => fun(wynik));
            Promise.all(promisy)
                .then(wynikiZFun => {
                    // "wynikiZFun" zawiera listę wartości zwróconych przez wywołania "fun" dla każdej wartości z "wynikiZFunTaba"
                });
```
```js
        });
};
```

### Po otrzymaniu wszystkich wyników od wywołań `fun(wynik)`, pogrupować je w odpowiednim formacie
```js
const connect = (funTab, fun) => {
    const promisy = funTab.map(funkcja => funkcja());
    Promise.all(promisy)
        .then(wynikiZFunTaba => {
            const promisy = wynikiZFunTaba.map(wynik => fun(wynik));
            Promise.all(promisy)
                .then(wynikiZFun => {
```
```js
                    const pogrupowane = wynikiZFunTaba.map(
                        (wynikZFunTaba, indeks) => {
                            return [
                                wynikiZFunTaba,
                                wynikiZFun[indeks]
                            ];
                        }
                    );
```
```js
                });
        });
};
```

Zamieniamy każdą wartość z listy "wynikiZFunTaba" na dwuelementową listę, w której:
- **Indeks 0** = wynik którejś funkcji z listy "funTab"
- **Indeks 1** = wynik fun() dla tego wyniku funkcji z listy "funTab"

Ponieważ Promise.all zachowuje kolejność wyników taką, jaka była kolejność funkcji, które te wyniki zwracają, to możemy być pewni, że odwołanie po indeksie poprawnie pogrupuje wszystkie wartości.

### Pozostało zwrócić `pogrupowane`... Ale jak?
Ponieważ korzystamy w naszej funkcji `connect` z innych funkcji asynchronicznych, to funkcja `connect` też musi być asynchroniczna.

Na razie taka nie jest, ponieważ:
- Nie ma nigdzie słówka `async` w definicji funkcji `connect`
- Funkcja `connect` nie przyjmuje *callback'a* (aka nie ma argumentu `cb` albo `callback` albo coś podobnego)
- Funkcja `connect` nie zwraca Promise'a

Aby zmienić funkcję `connect` w funkcję asynchroniczną, użyjemy trzeciego rozwiązania ponieważ:
- Nie wolno nam używać `async`/`await`
- Nie możemy modyfikować definicji funkcji (więc nie możemy do listy argumentów dodać *callback'a*)
- Wolno nam jedynie modyfikować treść funkcji, a tylko tego wymaga zwrócenie Promise'a

```js
const connect = (funTab, fun) => {
```
```js
    return new Promise((resolve) => {
```
```js
        // Przesuwamy wszystko o jedno wcięcie
        const promisy = funTab.map(funkcja => funkcja());
        Promise.all(promisy)
            .then(wynikiZFunTaba => {
                const promisy = wynikiZFunTaba.map(wynik => fun(wynik));
                Promise.all(promisy)
                    .then(wynikiZFun => {
                        const pogrupowane = wynikiZFunTaba.map(
                            (wynikZFunTaba, indeks) => {
                                return [
                                    wynikiZFunTaba,
                                    wynikiZFun[indeks]
                                ];
                            }
                        );
                    });
            });
```
```js
    });
};
```

Teraz aby zwrócić `pogrupowane`, możemy użyć `resolve`, który pojawił się dzięki "owinięciu" wszystkiego w `return new Promise()`:
```js
const connect = (funTab, fun) => {
    return new Promise((resolve) => {
        const promisy = funTab.map(funkcja => funkcja());
        Promise.all(promisy)
            .then(wynikiZFunTaba => {
                const promisy = wynikiZFunTaba.map(wynik => fun(wynik));
                Promise.all(promisy)
                    .then(wynikiZFun => {
                        const pogrupowane = wynikiZFunTaba.map(
                            (wynikZFunTaba, indeks) => {
                                return [
                                    wynikiZFunTaba,
                                    wynikiZFun[indeks]
                                ];
                            }
                        );

```
```js
                        resolve(pogrupowane);
```
```js
                    });
            });
    });
};
```

### Końcowy program

#### Na "czysto"
```js
const connect = (funTab, fun) => {
    return new Promise((resolve) => {
        const promisy = funTab.map(funkcja => funkcja());

        Promise.all(promisy)
            .then(wynikiZFunTaba => {
                const promisy = wynikiZFunTaba.map(wynik => fun(wynik));

                Promise.all(promisy)
                    .then(wynikiZFun => {
                        const pogrupowane = wynikiZFunTaba.map(
                            (wynikZFunTaba, indeks) => {
                                return [
                                    wynikiZFunTaba,
                                    wynikiZFun[indeks]
                                ];
                            }
                        );

                        resolve(pogrupowane);
                    });
            });
    });
};
```

#### Z komentarzami
```js
const connect = (funTab, fun) => {
    return new Promise((resolve) => {
        const promisy = funTab.map(funkcja => funkcja()); // Wydobywamy Promise'y z każdej funkcji z funTab'a

        Promise.all(promisy) // Tworzymy Promise'a, który się spełni jak wszystkie Promise'y z listy "promisy" się spełnią
            .then(wynikiZFunTaba => { // Tu nasłuchujemy, aż Promise zwrócony przez Promise.all się spełni
                // "wynikiZFunTaba" zawiera listę wartości zwróconych przez wszystkie funkcje z "funTab"

                // Zamieniamy każdy wynik na wywołanie "fun" z tym wynikiem podanym jako argument
                const promisy = wynikiZFunTaba.map(wynik => fun(wynik));

                Promise.all(promisy)
                    .then(wynikiZFun => {
                        // "wynikiZFun" zawiera listę wartości zwróconych przez wywołania "fun" dla każdej wartości z "wynikiZFunTaba"

                        
                        /* Zamieniamy każdą wartość z listy "wynikiZFunTaba" na dwuelementową listę, w której:

                            - Indeks 0 = wynik którejś funkcji z listy "funTab"
                            - Indeks 1 = wynik fun() dla tego wyniku funkcji z listy "funTab"

                        Ponieważ Promise.all zachowuje kolejność wyników taką, jaka była kolejność funkcji,
                        które te wyniki zwracają, to możemy być pewni, że odwołanie po indeksie poprawnie
                        pogrupuje wszystkie wartości. */
                        const pogrupowane = wynikiZFunTaba.map(
                            (wynikZFunTaba, indeks) => {
                                return [
                                    wynikiZFunTaba,
                                    wynikiZFun[indeks]
                                ];
                            }
                        );

                        resolve(pogrupowane);
                    });
            });
    });
};
```

#### Z oznaczonymi wcięciami
```js
const connect = (funTab, fun) => {
    return new Promise((resolve) => {
    ┆   const promisy = funTab.map(funkcja => funkcja());
    ┆
    ┆   Promise.all(promisy)
    ┆       .then(wynikiZFunTaba => {
    ┆       ┆   const promisy = wynikiZFunTaba.map(wynik => fun(wynik));
    ┆       ┆
    ┆       ┆   Promise.all(promisy)
    ┆       ┆       .then(wynikiZFun => {
    ┆       ┆       ┆   const pogrupowane = wynikiZFunTaba.map(
    ┆       ┆       ┆       (wynikZFunTaba, indeks) => {
    ┆       ┆       ┆       ┆   return [
    ┆       ┆       ┆       ┆       wynikiZFunTaba,
    ┆       ┆       ┆       ┆       wynikiZFun[indeks]
    ┆       ┆       ┆       ┆   ];
    ┆       ┆       ┆       }
    ┆       ┆       ┆   );
    ┆       ┆       ┆
    ┆       ┆       ┆   resolve(pogrupowane);
    ┆       ┆       });
    ┆       });
    });
};
```

# Axios

## Zadanie 1.1

>Stwórz projekt i dołącz do niego bibliotekę axios.
>
>Następnie wykonaj zapytanie metodą GET za pomocą dodanej biblioteki pod następujący url: https://jsonplaceholder.typicode.com/posts
>
>Jako pierwszy callback - sprawdź, czy response jest poprawny (status równy 200). 
>
>Jeśli tak - zwróć response, w przeciwnym wypadku - wypisz błąd w konsoli.
>
>Jako następny callback - użyj destrukcji obiektów, aby wypisać w konsoli zmienną 'data' i 'headers'.

### Co to *destrukcja obiektów*?
*Destrukcja obiektów* - a dokładniej *destruk**turyza**cja obiektów* (z ang. *Object Destructuring*) - polega na wzięciu obiektu i rozbiciu go na wybrane części pierwsze.

Dla przykładu, przypuśćmy obiekt:
```js
const obj = {
    foo: 2,
    bar: "test",
    uwu: "fifonż"
};
```
Teraz powiedzmy, że chcemy z niego otrzymać tylko następujące pozycje:
```js
{
    foo: 2,
    bar: "test"
}
```
Możemy użyć do tego klasycznego przypisania do zmiennych po kluczach:
```js
const foo = a["foo"]; // lub a.foo
const bar = a["bar"]; // lub a.bar
```
Istnieje natomiast skrótowy zapis takiego rozbijania obiekty na wybrane części:
```js
const {foo, bar} = a;

console.log(foo === a["foo"]);  // true
console.log(bar === a["bar"]);  // true
```
Składnia destrukturyzacji obiektów wygląda tak:
```js
const {[lista nazw kluczy, których wartości chcemy otrzymać...]} = [obiekt z którego chcemy otrzymać wspomniane klucze];
```
Dodatkowo, ta składnia powoduje, że nazwy kluczy podane w klamrach (między `{` a `}`) są traktowane jako normalne nazwy zmiennych, co oznacza, że rozbijając obiekt `obj` na części `foo` oraz `bar`, ta składnia pozwala na jednoczesne pobranie wartości dla tych kluczy i przypisanie ich do zmiennych o tych samych nazwach.

Przykład użycia w praktyce pojawi się podczas rozwiązywania zadania

### Przejdźmy do samego zadania
No to zacznijmy po kolei

#### Stwórz projekt i dołącz do niego bibliotekę axios.
Jeśli potrzebne jest przypomnienie, to tak to się robi:
- Utworzenie projektu
  - `npm init -y`
- Zainstalowanie biblioteki axios
  - `npm install axios`

Tworzymy plik (np. `index.js`) i wbijamy do niego tę linijkę:
```js
const axios = require("axios");
```
aby zaciągnąć axios'a do projektu i móc z niego korzystać.

> [!NOTE]
> Taki `require()` działa niejako jak `import` z Pythona:
> ```python
> import axios
> ```

#### Następnie wykonaj zapytanie metodą GET za pomocą dodanej biblioteki pod następujący url
URL (czyli adres internetowy), pod który mamy wykonać zapytanie (czyli żądanie) to:
```
https://jsonplaceholder.typicode.com/posts
```

Sam [jsonplaceholder.typicode.com](https://jsonplaceholder.typicode.com) jest stroną, która przechowuje przykładowe dane. Nie ma to znaczenia czym one są, ważne jest w tej stronie to, że dane te dostępne są *przez internet*, i to jedyne co się liczy. Strona ta służy głównie do nauki wykonywania właśnie żądań lub testowania istniejącego kodu bez konieczności zmiany *samego sposobu* pobierania danych.

Każde zapytanie/żądanie (od tej pory stosować będę słowo *zapytanie*) ma swoją *metodę*, która określa niejako cel takiego zapytania. Oto przykłady kilku z nich:
- **`GET`** pobiera dane z serwera
- **`POST`** wrzuca nowe dane do serwera
- **`PUT`** wrzuca lub aktualizuje dane na serwerze
- **`UPDATE`** aktualizuje dane na serwerze
- **`DELETE`** usuwa dane z serwera

Metody zapytań rozpoznawane są w taki sam sposób przez dowolny serwer i wszędzie nazywają się tak samo, ponieważ są **standardem komunikacji internetowej**. Jest ich więcej niż te, które podałem powyżej, natomiast najpopularniejsze są dwa z nich: `GET` i `POST`.

To dlatego, że reprezentują najczęstsze czynności w serwisach internetowych:
- Pobierasz swój feed na Instagramie? `GET`
- Pobierasz zdjęcie posta? `GET`
- Wrzucasz nowe story? `POST`
- Wchodzisz na stronę internetową i musisz pobrać jej stronę główną? `GET`
- Publikujesz post na ~~Twitterze~~ X? `POST`

Innymi popularnymi są `DELETE` i `PUT`, które stosowalibyśmy kolejno do np. usunięcia zdjęcia ze swojego profilu albo edytowania posta na Facebook'u, natomiast `GET` i `POST` to jedyne, które nam się przydadzą.

**Teraz kiedy już wiemy czym są metody, możemy zacząć wykonywać zapytania z nimi za pomocą Axios'a!**

Aby wykonać zapytanie z wybraną metodą, używamy takiej składni:
```js
axios.[metoda]([adres]);
```
Np. jeśli chcielibyśmy wykonać zapytanie `GET` pod adres `abc`, możemy napisać:
```js
axios.get("abc");
```
Zapytanie `POST` pod adres `uwu` wygląda za to tak:
```js
axios.post("uwu");
```
> [!NOTE]
> Przykład z `.post()` jest trochę niekompletny. Ponieważ jest to jedna z metod, która **wysyła jakieś dane** do serwera, to możemy te dane podać w dodatkowym argumencie.
> 
> Wysyłania danych na podany adres URL nauczymy się w kolejnym zadaniu.

Wróćmy do oryginalnego polecenia, które kazało nam wysłać zapytanie metodą `GET` pod adres `https://jsonplaceholder.typicode.com/posts`. Robimy to w dokładnie taki sam sposób jak wyżej:
```js
const axios = require("axios");  // to jest super ważne, inaczej axiosa w ogóle nie ma w projekcie, mimo że jest zainstalowany

axios.get("https://jsonplaceholder.typicode.com/posts");
```

#### Jako pierwszy callback - sprawdź, czy response jest poprawny (status równy 200).
Każda *odpowiedź* na dane *zapytanie* (nie ważne jaką metodą) zawiera coś takiego jak **kod statusu**.

Kody statusu również są ustandaryzowane, a więc wszędzie są takie same i oznaczają to samo. Oto kilka najpopularniejszych:
- **`200`** *OK* (aka wszystko jest dobrze)
- **`404`** *Not Found* (serwer nie znalazł tego o co prosiliśmy)
- **`500`** *Internal Server Error* (coś poważnego się stało i kod na serwerze dostał wylewu)
- **`503`** *Service Unavailable* (serwis nie jest teraz dostępny z dowolnego powodu)

Aby było jaśniej, wszystkie kody zaczynają się od numerków, które określają ich kategorię:
- `1XX` są informacyjne
- `2XX` określają jakiś rodzaj sukcesu w wykonywaniu zapytania
- `3XX` mówią, że nasze zapytanie powinno zostać przekierowane na inny adres
- `4XX` opisują błędy od strony wykonawcy zapytania
- `5XX` opisują błędy od strony serwera (odbiory zapytania)

Dodajmy więc `.then()` do naszego wywołania, aby poczekać na odpowiedź na zapytanie:

```js
const axios = require("axios");

axios.get("https://jsonplaceholder.typicode.com/posts")
    .then(response => {
        // "response" będzie zawierało odpowiedź od serwera, kiedy ten takową nam wyśle

        if (response.status === 200) { // sprawdzamy czy kod statusu jest równy 200
            // tutaj coś będzie... pewnie...... chyba :33333333333 jestem trochę zmęczony
        }
    });
```

#### Jeśli status jest równy 200 - zwróć response, w przeciwnym wypadku - wypisz błąd w konsoli.
```js
const axios = require("axios");

axios.get("https://jsonplaceholder.typicode.com/posts")
    .then(response => {
        // "response" będzie zawierało odpowiedź od serwera, kiedy ten takową nam wyśle

        if (response.status === 200) {
            return response;  // zwracamy response, tak jak jest napisane w poleceniu - tak zwrócony może być przyjęty przez kolejny .then().
        } else {
            console.log(`Błąd! Odpowiedź ma kod statusu ${response.status} a nie 200.`);
        }
    });
```

#### Jako następny callback - użyj destrukcji obiektów, aby wypisać w konsoli zmienną 'data' i 'headers'.
Tutaj pojawia się nasza znajoma ~~destrukcja~~ destrukturyzacja obiektów!

```js
const axios = require("axios");

axios.get("https://jsonplaceholder.typicode.com/posts")
    .then(response => {
        // "response" będzie zawierało odpowiedź od serwera, kiedy ten takową nam wyśle

        if (response.status === 200) {
            return response;  // ponieważ tu zwracamy response, to możemy go przyjąć w kolejnym .then() jak poniżej.
        } else {
            console.log(`Błąd! Odpowiedź ma kod statusu ${response.status} a nie 200.`);
        }
    })
    .then(response => {
        // tutaj jest ten "następny callback"

        // rozdzielamy obiekt "response" na dwie części:
        //   - data (które przechowuje dane z odpowiedzi wysłane przez serwer)
        //   - headers (które zawiera tzw. nagłówki z odpowiedzi wysłanej przed serwer, nieistotne co to jest na razie)
        const {data, headers} = response;
        
        // wypisujemy obie rzeczy do konsoli :3333
        console.log(data);
        console.log(headers);
    });
```

### Całe zadanie bez komentarzy:
```js
const axios = require("axios");

axios.get("https://jsonplaceholder.typicode.com/posts")
    .then(response => {
        if (response.status === 200) {
            return response;
        } else {
            console.log(`Błąd! Odpowiedź ma kod statusu ${response.status} a nie 200.`);
        }
    })
    .then(response => {
        const {data, headers} = response;
        
        console.log(data);
        console.log(headers);
    });
```

## Zadanie 1.2
> Stwórz funkcje, która przyjmuje jako parametr obiekt w postaci:
>
> ```js
> {
>   idUser: (pole typu int)
>   title: (pole typu string)
>   completed: (pole typu boolean)
> }
> ```
>
> Następnie wysyła taki obiekt za pomocą funkcji POST z biblioteki axios pod url: https://jsonplaceholder.typicode.com/todos
> 
> Jeśli dodanie zakończy się sukcesem - wyświetl w konsoli komunikat 'Dodano' i id dodanego obiektu. W przeciwnym wypadku wypisz błąd.

#### Stwórz funkcję, która przyjmuje jako parametr obiekt
```js
const axios = require("axios");

function fun(obj) {

}
```

#### Wyślij obiekt za pomocą funkcji POST z biblioteki axios pod url: `https://jsonplaceholder.typicode.com/todos`
Aby wysłać jakieś dane do serwera, musimy wykonać zapytanie do tego serwera z metodą, która wspiera wysyłanie danych. Jedną z nich jest właśnie `POST`.

Aby powiedzieć axiosowi jakie dane konkretnie chcemy serwerowi wysłać pod dany adres, podajemy te dane po prostu jako drugi argument:
```js
const obj = {
    foo: 2,
    bar: "test",
    uwu: "fifonż"
};

// Takie wywołanie wyśle dane obiektu "obj" pod adres "abc" za pomocą metody POST
axios.post("abc", obj);
```

Jedyne co musimy więc zrobić w naszej funkcji, to:
```js
const axios = require("axios");

function fun(obj) {
    axios.post("https://jsonplaceholder.typicode.com/todos", obj)
        .then(response => {
            // czekamy na odpowiedź, która znajdzie się w "response"...
        });
}
```

#### Jeśli dodanie zakończy się sukcesem - wyświetl w konsoli komunikat 'Dodano' i id dodanego obiektu...
Jeżeli uruchomisz teraz taki kod:
```js
const axios = require("axios");

function fun(obj) {
    axios.post("https://jsonplaceholder.typicode.com/todos", obj)
        .then(response => {
            console.log(response.data);  // po otrzymaniu odpowiedzi, wyświetlimy zawarte w niej dane
        });
}

// testowe wywołanie, żeby zobaczyć co się stanie dla przykładowych, wymyślonych danych
fun({
    idUser: 1,
    title: "uwu",
    completed: false
});
```

To w konsoli powinno pojawić się coś w tym stylu:
```js
{ idUser: 1, title: 'uwu', completed: false, id: 201 }
```

Jak możesz zauważyć, w danych odpowiedzi, którą dostaliśmy, znajduje się cały obiekt "notatki", którą utworzyliśmy naszym zapytaniem. Oprócz danych, które przekazaliśmy sami (`idUser`, `title` oraz `completed`) znajduje się tam również pole `id`. Jest to identyfikator naszej "notatki", który nam się teraz przyda.

```js
const axios = require("axios");

function fun(obj) {
    axios.post("https://jsonplaceholder.typicode.com/todos", obj)
        .then(response => {
            const daneOdpowiedzi = response.data;  // daneOdpowiedzi to ten obiekt z wyżej

            console.log("Dodano");
            console.log(daneOdpowiedzi.id); // LUB console.log(daneOdpowiedzi["id"])
        });
}

// testowe wywołanie, żeby zobaczyć co się stanie dla przykładowych, wymyślonych danych
fun({
    idUser: 1,
    title: "uwu",
    completed: false
});
```

#### ..., w przeciwnym wypadku wypisz błąd
```js
const axios = require("axios");

function fun(obj) {
    axios.post("https://jsonplaceholder.typicode.com/todos", obj)
        .then(response => {
            const daneOdpowiedzi = response.data;

            console.log("Dodano");
            console.log(daneOdpowiedzi.id);
        })
        .catch(blad => {  // dodajemy nasłuchiwanie na błąd po prostu
            console.log(`błąd: ${blad}`);
        });
}

// testowe wywołanie, żeby zobaczyć co się stanie dla przykładowych, wymyślonych danych
fun({
    idUser: 1,
    title: "uwu",
    completed: false
});
```

I to całe zadanie 1.2! :33333333333333
