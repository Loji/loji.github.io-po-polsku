---
layout: post
title:  "Event Loop - kolejkowanie zdarzeń"
date:   2017-09-03 16:00:00 +0200
categories: trochę-o-js
---

Mając kontakt z JavaScriptem prędzej czy później natkniesz się na termin “Event Loop” wypowiadany z powagą, często działający jak magiczna inkantacja taka jak “algorytm” gdy autor nie ma pomysłu jak ubrać coś w słowa. Postaram się wyjaśnić dzisiaj o co tutaj chodzi. 

## Czym jest Event Loop?

Event loop jest częścią większego systemu, który pozwala by JavaScript - język który sam w sobie jest jednowątkowy, imperatywny, synchroniczny - mógł działać w sposób, który nie blokuje przeglądarki i robi pozornie bardzo dużo rzeczy na raz. System ten to całe środowisko urchomieniowe JS włącznie z API przeglądarki czy bindami z C++ w przypadku Node.

Poniżej mały diagram informacyjny. 

<div class="block-image" markdown="1">
![Diagram pomocniczy]({{ site.baseurl }}/images/callstack-diagram.png)
</div>

**Centralnym punktem wykonywania się JS jest Call Stack** gdzie jest cały przebieg programu. Działa on podobnie jak w innych językach. Nie będę wyjaśniał tutaj zasady jego działania. 

Z tego poziomu nasz kod może odwoływać się do dostępnego API, które w zależności od swojego działania dodaje funkcje do wywołania na **Callback queue**. Tak na przykład będzie działać callback `response` w bibliotece request, czy zdarzenie `onClick` dla przycisku. 

Co dzieje się z takim callbackiem gdy trafia do kolejki? W momencie gdy **Call Stack jest zwolniony** (kod został wykonany) **Event Loop ściąga ostatniego Callbacka z kolejki** i wykonuje. Tak wywołany callback też może skorzystać z API przeglądarki, które znów wrzuci coś do kolejki, a do tego w międzyczasie akcje użytkownika mogą sprawić że do kolejki zostanie wrzucone coś więcej (np. Kliknięcie przycisku).

## Czas na przykłady

Wpływ kolejki na działanie Twojego kodu może być zauważalny mimo tego że wykonywanie się JS jest dość szybkie. Na przykład gdy obsługujesz dużo asynchronicznego kodu takiego jak pobranie listy zadań, a potem wykonanie dla każdego dodatkowego zapytania o szczegóły czy relacje (nie wchodząc w to czy taki system jest dobry i jak można zrobić go lepiej). Możesz zauważyć wtedy że kod obsługujący zakończony request nie wykona się dopóki wcześniejszy się nie skończy. Jest to tym bardziej problematyczne gdy nasz kod ma dużo pętli, odwołań do DOM, czy dużo skomplikowanych obliczeń. 

Przykład takiego kodu:

<iframe width="100%" height="500" src="//jsfiddle.net/Loji/Lc9203e6/4/embedded/js,html,result/dark/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

`setTimeout` jako pierwszy argument przyjmuje funkcję, która wykona się przynajniej po liczbie milisekund z drugiego argumentu. Warto zwrócić uwagę na to jak zmienia się czas wykonania funkcji gdy wykomentujemy lub zakomentujemy `iterateWithNoRealPurpose`.

Nie wydaje się takie złe, prawda? Niestety w praktyce jest. W powyższym przykładzie od naciśnięcia przycisku “run” do pokazania wyniku z perspektywy użytkownia nie da się nic zrobić - przeglądarka czeka na zakończenie tego kodu, a dopiero potem może obsługiwać cokolwiek innego, począwszy od innych zapytań, aż po akcje użytkownika. Przez blokujący kod mamy czasem wrażenie że strona “zmula”. 

Przykład takiej sytuacji:

<iframe width="100%" height="500" src="//jsfiddle.net/Loji/bs78sbrg/7/embedded/js,html,result/dark/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

Zauważ że po kliknięciu “Start test” przycisk nie zmienił swojego wyglądu (ciągle wygląda jakby był wciśnięty) oraz to że klikanie na przycisk “Increment” jest zauważalne dopiero po tym jak pojawi się wynik wcześniejszego przykładu, który odpala w środku kosztowną pętlę. 
