---
layout: post
title: Javarsovia 2009 - wrażenia z konferencji
description: (In Polish) Wrażenia z największej bezpłatnej konferencji o Javie w Polsce
date: 2009-07-07
author: tomek
image: '/images/javarsovia-2009-cover.jpg'
tags: [In Polish, Conferences, Java]
tags_color: '#aeb85f'
redirect_from:
  - /2009/07/javarsovia-2009-wrazenia/
---

Właśnie wróciłem z konferencji Javarsovia 2009. Ogólne wrażenia jak najbardziej pozytywne, dokładniejsza relacja poniżej. 
Najbardziej żałuję, że nie udało mi się wyciągnąć tylu znajomych, ilu chciałem, no i że nie wygrałem kubka 😉



### Godz. 9:00 – Intro
Na miejscu byliśmy wcześnie, bo tuż przed 9, dlatego spore było nasze zdziwienie, gdy okazało się, że przy rejestracji 
ustawiły się już trzy wężyki z uczestnikami. Na szczęście wszystko przebiegło sprawnie, każdy otrzymał 
trochę materiałów spamowo-informacyjnych, smycz, długopis, notatnik i identyfikator na szyję z rozkładem jazdy
konferencji na rewersie (super pomysł!). Potem nastąpiło krótkie rozpoczęcie i każdy mógł udać się na wykład lub warsztat, który go najbardziej interesował.

Ja z dostępnych prezentacji wybrałem następujące:

### 9:45 – Testy wspaniałe zdobywają Warszawę by Szczepan Faber
Wykład fajnie przeprowadzony i ciekawy. Ogólnie o testach, jak je pisać, żeby się później nie wstydzić 🙂 
Szczepan zaprezentował szablon testów, który w dużym uproszczeniu wygląda następująco:

```java
@Test
public void shouldDoSomethingWeWantToTest() throws Exception {
 // given
 // tutaj przygotowujemy dane do wykonania metody, którą chcemy przetestować
 
 // when
 // tutaj wykonujemy metodę, którą chcemy przetestować
 
 // then
 // tutaj sprawdzamy czy wynik jest zgodny z oczekiwaniami
 
}
```

Dzięki zastosowaniu takiego szablonu każda metoda testująca ma określony cel i jest łatwa 
do zrozumienia nawet po pobieżnym spojrzeniu w kod.

Kluczowe są cztery elementy każdej metody:

* **słowo “should” w nazwie metody testującej** Dzięki niemu wiemy, co dokładnie ma testować metoda, nawet
bez zaglądania do jej wnętrza. Jest to tym bardziej istotne, im więcej mamy testów w aplikacji.
Jeśli uruchamiamy testy i zobaczymy informację, że shouldLoadOldestUser() nie wykonał się poprawnie,
to od razu wiemy, co w aplikacji nie zadziałało jak powinno. Dodatkowo słowo “should” sprawia, że metoda musi 
mieć konkretny cel swojego istnienia. Nie może testować kilkunastu rzeczy, bo wtedy nazwa metody byłaby bardzo dziwna np. 
`shouldLoadOldestUserAndShouldSendOldestUserEmailWithInfo()`. 

* **słowo given** W tym miejscu przygotowujemy dane do testów. Powinno znajdować się tam jak najmniej kodu, czyli 
ustawiamy tam tylko te właściwości i te dane, które są nam rzeczywiście potrzebne do wykonania tego właśnie testu.
* **słowo when** Tutaj wykonujemy metodę, którą chcemy przetestować.
* **słowo then** Tutaj sprawdzamy, czy wynik wykonanej metody jest zgodny z oczekiwanym.

Schemat jest bardzo prosty w swej budowie i chyba właśnie w tej prostocie kryje się jego moc. Przyznaję, że moje testy nie
są szczytem elegancji i czytelności, więc przy najbliższej okazji pisania mojej magisterki wypróbuję w boju
zalecenia Szczepana, bo wydaje mi się, że dzięki temu znacznie zyskają na jakości.

W dalszej części prezentacji Szczepan zaprezentował kilka pojęć i bibliotek, z którymi wcześniej się nie zetknąłem. 
Poniżej lista:

* płynne interfejsy (fluent interfaces), więcej tutaj na blogu Martina Fowlera
* biblioteka Hamcrest do budowania matcherów. Używana najczęściej w testach, walidacjach i przy mockowaniu obiektów. Zapowiada się ciekawie.
* FEST, biblioteka upraszczająca tworzenie testów jednostkowych
* Mockito, biblioteka do mockowania (imitowania) obiektów podczas testowania kodu.
* plugin do Eclipse’a (nazwa wyleciała mi niestety z głowy), który odpala testy w tle, gdy my równolegle kodujemy. UPDATE: To narzędzie to Infinitest, dziękuję Tomkowi Nurkiewiczowi za podpowiedź 🙂

Podsumowując, bardzo dobry wykład. Nie oczekiwałem po nim niczego super ciekawego, ale na szczęście pozytywnie mnie zaskoczył 🙂 Szczepan poprowadził całość w dość młodzieżowym stylu i wyszło mu to bardzo fajnie. Dodatkowo pokazał, jak można naprawdę szybko pisać kod, wykorzystując dostępne w IDE skróty klawiszowe 🙂 Mój numer jeden tej konferencji.

### 11:00 – Ciągła Integracja w procesie wytwarzania oprogramowania by Kuba Kubryński
Ciekawy wykład o ciekawej tematyce, posłuchałem sporo o rzeczach, z którymi nie miałem nigdy styczności: continuous integration, Hudson, itd.

Z rzeczy, które wydały mi się najbardziej ciekawe i potencjalnie warte bliższego zapoznania:

* narzędzia do badania jakości tworzonego kodu, znajdowania miejsc potencjalnie zawierających błędy: PMD, Checkstyle, FindBugs, JavaNCSS
* JUnitPerf – biblioteka rozszerzająca możliwości JUnita o testowanie szybkości i skalowalności istniejących już testów jednostkowych.
* Sonar, najciekawsze z narzędzi, które pojawiły się na prezentacji. Prawdziwy kombajn do zarządzania jakością tworzonego kodu. Analizuje kod, wykrywa potencjalne błędy (używając wymienionych narzędzi PMD, Checkstyle, a także Cobertura), pokazuje zmiany parametrów w czasie rozwoju projektu. A wszystko to w przejrzystej graficznej formie. Naprawdę zapowiada się super!

Ten wykład to także pozytywne zaskoczenie. Styl prowadzenia inny niż u Szczepana, mniej luzacki, ale nie przeszkadzało to w odbiorze treści. To było dobrze spędzone 60 minut.

### 12:15 – Apache Wicket by Paweł Szulc
To był mój faworyt na najciekawszy wykład tej konferencji. Wicket to mój ulubiony framework frontendowy i ciekawiło mnie, co też nowego dowiem się z tej prezentacji. Okazało się jednak, że nie za wiele, bo wykład był bardziej skierowany do osób, które tylko o Wickecie słyszały lub ewentualnie bawiły się tylko w jakieś proste przykłady. Oczywiście nie mogę uznać tego za wadę, bo Paweł ciekawie i nieszablonowo zaprezentował zalety i cechy tego frameworka. Na tyle ciekawie, że mojego znajomego, który wcześniej kodował w JSF, zachęcił do wypróbowania Wicketa. Na koniec pojawiła się też rzecz dla mnie nowa. Otóż Paweł zaprezentował szablon mavenowy, który tworzy prototyp aplikacji w Wicket. W odróżnieniu od quickstarta ze strony frameworku jest on jednak dużo bardziej rozbudowany (o DAO, service layer, autoryzację) i pokazuje pewne wzorce architektury, których warto przestrzegać już od samego początku tworzenia projektu w Wicket. Bardzo fajna sprawa i jak tylko Paweł udostępni szablon na swoim blogu, na pewno się z nim dokładnie zapoznam.

Paweł Szulc ciekawie i niestandardowo pokazał cechy Wicketa. Ci, którzy znali ten framework, skorzystali mniej, ale dzięki takiemu, a nie innemu poziomowi zaawansowania omawianych zagadnień, grono osób, które przychylnie spojrzą na pomysł wyboru Wicketa do rzeczywistego projektu, znacznie się powiększyło. I o to chodziło! 🙂

### 13:15 – Break
Po wykładzie o Wicket nastąpiła przerwa obiadowa. Organizatorzy spisali się na medal i każdy chętny mógł spokojnie zjeść smaczny, ciepły i darmowy posiłek. Nie zauważyłem, żeby dla kogoś zabrakło 😉 Przy okazji można było zakręcić się koło rejestracji i zdobyć koszulkę z konferencji 🙂

Po posileniu się, pełny nowych sił wyruszyłem na kolejny wykład.

### 14:15 – Android z perspektywy programisty Java by Jacek Zadrąg
Wykład zapowiadał się ciekawie, bo tworzenie aplikacji na systemy mobilne jest coraz popularniejsze. Prowadzący pokazał w skrócie, jak w łatwy sposób można stworzyć aplikację do szybkiego wybierania numerów na telefon od Google. Niestety kilka razy zgubił się przy implementacji i sporo czasu stracił na tworzenie kodu. Zamiast tego można było spróbować pokazać przyrostowo na slajdach, jak wygląda dodawanie kolejnych elementów do aplikacji, a na koniec pokazać działającą aplikację i efekt byłby moim zdaniem lepszy. Ogólnie wykład ciekawy, ale można było go poprowadzić lepiej.

### 15:30 – Praca z odziedziczonym kodem by Jakub Dziwisz
Prowadzący pokazał, jak programista rzucony do projektu, w którym pojęcie testów jednostkowych nie istnieje, może próbować się odnaleźć. Zaproponował sekwencję kroków, którą warto wykonać, tak aby zabezpieczyć się przed zepsuciem działającego rozwiązania, ale żeby możliwe było także jak najbardziej bezbolesne zmodyfikowanie kodu zgodnie z żądaniami klienta.

Z rzeczy, które pojawiły się podczas prezentacji warto wymienić:

* AgileTuning, gdzie można znaleźć podcasty o lekkich metodykach programowania.
* narzędzia do automatycznego generowania testów: płatne JTest, AgitarOne oraz darmowy TestGen4J.
* JDepend, narzędzie do generowania metryk kodu i badania zależności pomiędzy klasami.
* http://www.objectmentor.com/ – strona z poradami, kursami o programowaniu obiektowym, Extreme Programming, TDD, Agile, itp. Wszystko zebrane w jednym miejscu.
* Refactoring To Patterns (Joshua Kerievsky), książka warta przeczytania w kontekście omawianych na wykładzie zagadnień. Widzę, że nawet jest wydanie polskie (http://helion.pl/ksiazki/refawp.htm), niestety aktualnie niedostępne.

Wykład ciekawy, trochę mniej techniczny, bardziej koncepcyjny, ale i tak warto było posłuchać, jak ktoś może mieć prze****ne, gdy trafi do projektu, w którym nic nie było robione zgodnie ze sztuką 😉

### 16:45 – Google Web Toolkit i JBoss Seam, czyli piękny wygląd z bogatym wnętrzem by Łukasz Kobyliński
Ten wykład wybrałem głównie po to, żeby posłuchać o GWT w kontekście użycia go w mojej pracy magisterskiej. Część Seamowa interesowała mnie już mniej 🙂 Całość została przeprowadzona na dobrym poziomie, ale w odróżnieniu od prezentacji o Wicket, tutaj zastosowano bardziej standardowy plan prezentacji. Było to, co w większości prezentacji o frameworkach: opis ogólny, cechy, filozofia, zastosowania czy wady i zalety. Dla mnie jako osoby, która przyszła dowiedzieć się, czym w ogóle jest GWT bez żadnej wcześniejszej z nim styczności, taki sposób zaprezentowania biblioteki zdał egzamin. Dostałem dawkę informacji, której oczekiwałem 🙂

### Podsumowanie
**Plusy:**

* wysoki poziom wykładów
* 4 ścieżki, każdy znalazł coś dla siebie
* dobra organizacja: informacja, minimalne poślizgi, dobre jedzenie 🙂
* wygodne aule i sale wykładowe.

**Minusy:**

* szkoda, że jednodniowa. Fajnie by było spędzić na takim evencie cały weekend.
* catering zwinął się ponad 1h przed zakończeniem i musiałem “o suchym pysku” iść na ostatni wykład 🙂
* w paru wypadkach zabrakło miejsc siedzących dla wszystkich chętnych.

Ogólnie plusy są mocne, a minusy to raczej detale, które nie wpływały na sumaryczny odbiór konferencji. Tak po prostu, żeby organizatorzy mieli świadomość, że nie było idealnie 🙂