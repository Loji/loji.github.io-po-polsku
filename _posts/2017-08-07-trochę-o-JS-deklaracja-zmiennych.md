---
layout: post
title:  "Deklaracja zmiennych w JS"
date:   2017-08-07 20:08:59 +0200
categories: trochę-o-js
---
Witaj w moim nowym cyklu o JavaScript. Zamierzam nauczyć Cię tutaj różnic między JS’em a innymi językami z rodziny C, tak żebyś mógł korzystać z jego potęgi i mógł świadomie unikać pułapek, jakie w nim się mogą kryć. 

## Moja motywacja

Jako że nie chcę pisać suchego zbioru instrukcji, który możesz przeczytać niczym dokumentację opiszę pokrótce dlaczego chcę napisać tą serię. Pierwszym powodem jest to że… mam ochotę napisać coś po polsku. Drugi powód to chęć pokazania, że JS może być fajnym narzędziem, którego specyficzne cechy są jego zaletą, a nie czymś z czym się walczy by cokolwiek zrobić. 

## Dla kogo jest ta seria

Seria jest skierowana do osób, które umieją programować w innych językach, najlepiej w językach z rodziny C. Nie chcę tworzyć kursu od podstaw, który wprowadzi osoby całkiem nowe w programowanie, chcę dać wiedzę komuś kto programować już umie. 

No to skoro wprowadzenie mamy za sobą to przejdźmy do tematu tego wpisu :) 

## Zmienne w JS - dlaczego tak a nie inaczej

Deklaracja zmiennych (i stałych) w JS w standardzie ES2015 (ES6) jest poprzedzona słowami kluczowymi `let` dla zmiennych oraz `const` dla stałych. We wcześniejszych wersjach było tylko słowo kluczowe `var` lub… całkowitym brakiem słowa kluczowego, o czym zaraz. 

### Co robi deklaracja przy pomocy let i const w JS?

Tutaj jest dość klasycznie. Deklaracja tworzy nam nową zmienną w miejscu, w którym to następuje. Zmienna nie jest dostępna wyżej, tylko w bloku kodu (`{ }`), w którym została zadeklarowana. Ponowna próba zadeklarowania skończy się wyrzuceniem wyjątku. 

Deklaracja wewnątrz bloku jest dość restrykcyjna, błędny będzie kod, który wygląda tak:

Błędny 
```
const a = 1; 
switch(a) {
    case 1: 
        let b = 1; 
        break; 
    case 2:
        let b = 3; 
        break; 
}
```
Poprawny
```
const a = 1; 
switch(a) {
    case 1: {
        let b = 1; 
        break; 
    }
    case 2: {
        let b = 3; 
        break; 
     }
}
```
Błędny 
```
function test() { 
    let a = 1;
    let a = 1; 
    console.log(a);
}
test();
```
Poprawny
```
function test() { 
    let a = 1;
    { let a = 1; }  
    console.log(a);
}
test();
```

### W jaki sposób const jest stałą?

`Const` deklaruje stałe, których wartość nie może się zmienić, ale mogą być zmutowane. Tablica lub obiekt zadeklarowany za pomocą const może być “zmodyfikowany”, nie może być natomiast nadpisany. 

Poprawny kod: 
```
const array = [];
array.push('nowa własność');
console.log(array);
```
Niepoprawny kod: 
```
const array = [];
array = ['nowa własność'];
console.log(array);
```

Nie ma tutaj nic dziwnego, prawda?

### Strict mode w przypadku zmiennych

Zanim opowiem o słowie kluczowym `var` chciałbym powiedzieć co w jego kontekście zmienia użycie strict mode.

Strict mode jest trybem, który sprawia też że pewne cechy JS przechodzące z cichym błędem powodują wyrzucenie wyjątku. Jest też trybem działania zwiększającym wydajność poprzez całkowite wyłączenie funkcjonalności trudnych do zoptymalizowania. Został wprowadzony wraz ze standardem ES5 z 2009 roku i jest wspierany obecnie przez praktycznie wszystko, nawet mądrą lodówkę. 

Strict mode jest włączany użyciem wyrażenia `'use strict';`, na początku funkcji. Jest to zwykły string, który jest ignorowany przez przeglądarki niewspierające ES5. 

W kontekście tego tekstu warto wiedzieć, że strict mode nie pozwoli na użycie niezadeklarowanej zmiennej. 

### Stara szkoła - czyli var

Kiedyś deklarowanie zmiennych za pomocą słowa kluczowego `var` potrafiło dawać dziwne, niespodziewane rezultaty, a to przez parę jego właściwości:
deklaracja zmiennej przez var może być powtórzona bez żadnego błędu 
gdziekolwiek zadeklarowany vary są dostępne w całej funkcji, także nad miejscem, w którym były zadeklarowane jak jeden - nie przechowują wartości ale pojedynczą referencję.

Poniższy kod jest w pełni poprawny:
```
function wtf() {
    'use strict';
    console.log(1, a);
    var a;
    console.log(2, a); 
    a = 'co tu się...'; 
    console.log(3, a);
}
wtf();
```
Niech nie zmyli Cię to, że `console.log` wyświetla wartość `undefined` - zmienna tak naprawdę jest zadeklarowana, ale nie jest zainicjalizowana, czyli nie jest ustawiona. Usunięcie powyższego `var a;` usunie deklarację zmiennej z funkcji i uniemożliwi jej wykonanie. 

#### Niemiłe konsekwencje używania var

`setTimeout` sprawia, że kod wykona się później, niż kiedy został napisany. Funkcja podana jako pierwszy argument wykona się po przynajmniej 10 milisekundach. O tym dlaczego przynajmniej w jednym z następnych wpisów ;)

Porównaj działanie obu poniższych kodów:

```
var a = ['1', '2', '3'];
for(var i = 0; i < a.length; i++) {
    setTimeout(function() {
        console.log(i, a[i]);
    }, 10);
}
```

```
var a = ['1', '2', '3'];
for(let i = 0; i < a.length; i++) {
    setTimeout(function() {
        console.log(i, a[i]);
    }, 10);
}
```

Drugi blok kodu działa poprawnie, zgodnie z oczekiwaniami, natomiast pierwszy zachowuje się... dziwnie. Dzieje się tak przez to, że `var` zadeklarowany nawet wewnątrz logiki operującą pętlą `for` jest tak naprawdę wyniesiony wyżej, co sprawia że wewnątrz każdego `setTimeout` nasz `console.log` używa jednej i tej samej referencji do zmiennej, która w momencie wykonania kodu zdążyła mieć przypisane 3 zakańczając wykonywanie pętli `for`.

## Słowem podsumowania 

Mam nadzieję że trochę przybliżyłem różnice między trzema sposobami deklarowania zmiennych dostępnych w JavaScripcie, dzięki czemu Twoje doświadczenia z nim częściej zakończą się miłym sukcesem.