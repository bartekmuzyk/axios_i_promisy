# Promise'y

## Zadanie 4

### Zacznijmy od `Promise.all`
`Promise.all` jest przydatną funkcją, która przyjmuje jeden argument - **listę Promise'ów**.

Działanie tej funkcji wygląda następująco:
- Weź wszystkie Promise'y z listy podanej jako argument i sam zwróć **nowy Promise** (nazwijmy ten Promise literą `p` na potrzeby wyjaśnienia).
- Poczekaj, aż wszystkie Promise'y z listy osiągną stan `fulfilled` (czyli wszystkie wywołają u siebie `resolve`).
- Kiedy tak się stanie, wypełnij Promise `p` listą wartości, które zwróciły Promise'y z listy (w takiej samej kolejności, w której zostały podane)

<hr/>

#### ⚠️ Ważna sprawa!
Jeżeli wywołujemy `Promise.all` w taki sposób:
```js
Promise.all(promisesTab);
```
i mamy do dyspozycji następujące funkcje asynchroniczne (tzn. takie, które zwracają `Promise`):
```js
function a() {
    return new Promise(resolve => /* ... */);
}

function b() {
    return new Promise(resolve => /* ... */);
}

function c() {
    return new Promise(resolve => /* ... */);
}
```

to lista `promisesTab` musi wyglądać tak:
```js
const promisesTab = [a(), b(), c()];  // ✅ Dobrze
```
a nie tak:
```js
const promisesTab = [a, b, c];  // ❌ Źle
```

Jest tak, ponieważ funkcja `Promise.all` przyjmuje listę ***samych Promise'ów***, a nie *funkcji, które zwracają Promise'y*. Ponieważ funkcje `a`, `b` i `c` *zwracają* Promise'y, to żeby uzyskać te Promise'y należy te funkcje **wywołać**.

<hr/>

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
