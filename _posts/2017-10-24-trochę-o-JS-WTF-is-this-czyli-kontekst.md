---
layout: post
title:  "WTF is this?! Czyli o kontekście"
date:   2017-10-29 20:20:59 +0200
categories: trochę-o-js
---

This w języku JavaScript nie zachowuje się domyślnie tak jak ma to miejsce w innych językach. Co to znaczy? Głównie to, że może zmieniać się w zależności od tego gdzie wykonasz swoją metodę lub funkcję stworzoną przez słowo kluczowe `function`. 

Jako przykład: 

```
class Cat {
    constructor() {
        this.name = 'Tom';
        this.launchAsyncDownload();
    }

    showName() {
        console.log('name', this.name);
    }

    launchAsyncDownload() {
        pseudoAsyncDownload(this.showName);
    }
}

function pseudoAsyncDownload(callbackFn) {
    callbackFn();
}

const cat = new Cat();
```

Powyższy kod się nie wykona. Będzie to spowodowane tym, że funkcja `showName` która wykonuje się w hipotetycznym callbacku jakiegoś mocka serwisu nie będzie miała dostępu do `this`. 

W momencie przekazania referencji do tej funkcji i wykonania jej w innej funkcji kontekst `this` będzie prowadził do miejsca gdzie została wykonana czyli funkcji `pseudoAsyncDownload`. Żeby uniknąć takiej sytuacji możemy zrobić parę rzeczy. 

## bind i inne metody wbudowane

JavaScript posiada parę wbudowanych metod, które zmienią tablicę tak, by posiadała kontekst którego oczekujesz. Opiszę pokrótce metody dostępne dla `function` które pomogą ze zmianą kontekstu:

### bind

Możliwe że najpopularniejszą spotykaną metodą dla zmiany kontekstu jest `bind`. Jest on wywoływany na referencji do funkcji i zwraca nową, z kontekstem dla this który sami podamy w argumencie.

``` 
class City {
    // ...this is just a mock
}
const sparta = new City();

function iHaveNoThis() {
    console.log(this, 'is madness!');
}

const wellIHaveThis = iHaveNoThis.bind(sparta);
```

Wykonując powyższy kod sprawiamy że funkcja `iHaveNoThis` z kontekstem `sparta` będzie dostępna pod referencją `wellIHaveThis`. 

### call oraz apply 

`call` oraz `apply` działają podobnie do `bind`, również zmieniają kontekst na ten podany w argumencie, ale za to wywołują od razu funkcję. `apply` jako drugi argument przyjmuje tablicę z argumentami dla funkcji, natomiast `call` przyjmuje n+1 argumentów dla funkcji n-argumentowej (pierwszym argumentem jest nowy `this`).

```func.apply(newThis, [1, 2, 'test'])``` 
będzie równoznaczny dla kodu
```
const newFunc = func.bind(newThis);
newFunc(1, 2, 'test');
```

Analogicznie dla powyższego można zastosować `call` w sposób `func.call(newThis, 1, 2, 'test')`.

### call oraz apply jako "przechwytywanie funkcji"

Częściej znajdziesz w kodzie użycie `call` oraz `apply` na przykład w:

```
const mappedArrayLike = Array.prototype.map.call(arrayLikeObject, mapFunction)
```

Powyższy kod wykonuje `mapFunction` na każdym elemencie `arrayLikeObject` który przypomina `Array` ale jest innym, zwykłym obiektem który nie posiada swojej funkcji `map`. Taki kod najczęściej spotkasz w bibliotekach i wypełnieniach (polyfillach).

## funkcje strzałkowe 

Funkcje strzałkowe to nowość wprowadzona w ES2015 (ES6) do używania której gorąco zachęcam. Funkcja strzałkowa różni się od tych deklarowanych przez słowo kluczowe `function` paroma rzeczami: 

* nie ma własnego kontekstu, zawsze posługuje się kontekstem rodzica czyli miejsca gdzie została zadeklarowana
* jest nazwana (atrybut `name`) po nazwie zmiennej/atrybutu do którego została przypisana (`const a = () => 0` to nazwa 'a')
* nie ma prototypu - nie może służyć do tworzenia obiektów 
* są ograniczone zasięgiem zmiennych - nie są dostępne od razu w całej funkcji gdzie zostały zadeklarowane jak `function name() {}`

Jedynym minusem funkcji strzałkowych jaki przychodzi mi do głowy jest fakt że używając składni `class` nie dodasz ich poza konstruktorem do swojego obiektu.

# Wniosek 

Moja rada jest prosta - gdziekolwiek przekazujesz callbacki posługujące się kontekstem używaj funkcji strzałkowych. Nawet więcej, używaj ich gdziekolwiek możesz, ich zachowanie jest łatwiejsze do przewidzenia dla początkującego, a fakt że nie posiadają prototypu w czasach gdy mamy `class` nie powinno nikogo martwić. 
