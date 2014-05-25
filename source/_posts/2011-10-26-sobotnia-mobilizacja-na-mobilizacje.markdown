---
author: Tomasz Dziurko
comments: true
date: 2011-10-26 20:34:32+00:00
layout: post
slug: sobotnia-mobilizacja-na-mobilizacje
title: Sobotnia mobilizacja na Mobilizację
wordpress_id: 1030
categories:
- Conferences
- In Polish
tags:
- Android
- Mobile
- Mobilization
- Łódź
---

Dear Reader
    This post is in my native language as it applies only to developers society in Poland. Please feel free to check my other posts which are mainly written in English.


W sobotę w Łodzi odbyła się pierwsza edycja konferencji [Mobilization](http://mobilization.pl/) organizowanej przez JUG z tego miasta. Ponieważ w tematyce technologii mobilnych czuję się mocno zielony i poza warsztatami z Androida na ostatniej [Warsjawie](https://github.com/warszawajug/warsjawa2011/wiki/Konferencja-warsjawa-2011) nie miałem z tą częścią programowania styczności, zdecydowałem się zainwestować sobotę na poszerzenie moich horyzontów w tym obszarze. Jak się okazało nie tylko mnie zainteresowała ta tematyka, bo na listę uczestników załapałem się tuż przed zamknięciem rejestracji, a, jak pisali organizatorzy, na liście rezerwowej czekało na uśmiech losu ponad 50 osób! Ale jeśli weźmiemy pod uwagę fakt, że była to pierwsza w Polsce konferencja całkowicie poświęcona technologiom mobilnym to nie ma się co dziwić tak dużemu zainteresowaniu.

[![](/images/blog/2011/10/mobilization.png)](/images/blog/2011/10/mobilization.png)

<!-- more -->


### Prelekcje


**Windows Phone Mango**

Konferencja rozpoczęła się od krótkiego przywitania i zaraz po tym mogliśmy wysłuchać prelekcji o programowaniu na [Windows Phone Mango](http://mobilization.pl/schedule/programowanie-windows-phone-mango/) poprowadzonej przez Barłomieja Zassa z Microsoftu. Bartek pokazał jak za pomocą Visual Studio i Microsoft Expression Blend można szybko stworzyć prostą aplikację na telefon z systemem WP. Jakoś nigdy nie byłem fanem języków od firmy z Redmond, ale prezentacja była poprowadzona na tak wysokim poziomie, że wielu niezdecydowanych na jaką platformę zacząć pisać aplikacje na pewno zostało zaciekawionych ofertą Microsoftu. Dodatkowo widać, że firma ta intensywnie próbuje zdobyć przychylność developerów, bo nie dość ze oba narzędzia są naprawdę dopracowane i oferują wiele możliwości to w dodatku są dostępne za darmo. Jedynym zgrzytem było wydłużenie prezentacji o 15 minut przez co już na starcie rozjechała się cała agenda konferencji.

**Aplikacje na system Android 3.0**

Kolejnym wykładem było [Pisanie aplikacji na Tablety z system Android 3.0](http://mobilization.pl/schedule/android-30/), podczas którego Krzysztof Janik pokazał kilka nowości z tej wersji systemu od Google, które powinny zainteresować osoby planujące tworzenie programów na tablety. Jednak już na początku pojawiło się kilka zgrzytów: tablet, który mieliśmy zobaczyć w działaniu nie chciał współpracować z komputerem, prowadzący na pierwszym slajdzie sam zauważył dwie literówki, a na jednym z kolejnych przyznał się, że wkleił zły fragment klasy :) Więc prezentacja nie powaliła, temat trochę ciężki, bo tablety w Polsce to jednak bardzo wąska nisza i projekt napisania aplikacji na takie urządzenie pojawia się równie często co aplikacja www, która wygląda tak samo pod Firefoxem i Internet Explorerem 6.0 ;-), a w dodatku prowadzący nie przyłożył się do kilku detali, co popsuło końcowe wrażenie. Sposób prezentowania był ok, temat trudny, slajdy kiepskie, no i jeszcze nie wypalił gwóźdź prezentacji, czyli tablet live.

**Aplikacje wielo-platformowe**

Trzecią prelekcją, na którą się wybrałem była [Multiplatform mobile apps](http://mobilization.pl/schedule/multiplatform-mobile-apps/). I okazało się, że była to najlepsza i najwartościowsza prezentacja tej konferencji. Widać było, że prowadzący Konrad Hołowiński ma na ten temat dużą wiedzę, którą w bardzo przemyślany sposób zebrał, uporządkował i przedstawił na swoim wystąpieniu. Prelegent skoncentrował się na omówieniu dwóch najpopularniejszych rozwiązań służących do tworzenia wieloplatformowych aplikacji mobilnych: [Appcelerator Titanium](http://www.appcelerator.com/) oraz [PhoneGap](http://www.phonegap.com/). Titanium używa języka JavaScript, oferuje wsparcie dla Androida, iOS i Blackberry (w wersji beta), ale dla każdej z tych platform zapewnia natywny wygląd komponentów. Z kolei PhoneGap wspiera aż 6 platform, a aplikacje piszemy w HTML5, CSS3 oraz JavaScripcie (jQuery Mobile lub Sencha Touch). Za wadę PhoneGapa niektórzy mogą uznać fakt, że aplikacje nie wyglądają jak natywne. Potencjał tej biblioteki dostrzegło Adobe i przejęło twórców PhoneGapa, firmę Nitobi.

**Obiad**

Po przerwie obiadowej kolejną prezentacją, na którą planowałem się wybrać była [Advanced layouts on the Android battlefield](http://mobilization.pl/schedule/layouts/), ale niestety przez kolejkę w chińczyku nie zdążyliśmy wejść na salę odpowiednio wcześniej i okazało się, że możemy jedynie stanąć na końcu wśród innych, dla których zabrakło miejsc siedzących na krzesełkach i leżących na podłodze. Dlatego z [Marcinem Stachniukiem](http://mstachniuk.blogspot.com/) zdecydowaliśmy się na odpuszczenie tego wykładu i podyskutowanie na tematy różne, między innymi o wadach i zaletach pracy kontraktowej i etatowej oraz wyższości [Yerba Mate](http://yerbamate.info.pl/) nad kawą i innymi dopalaczami :)

**Testowanie na Androidzie**

Tematem kolejnego wykładu miał być [Android and SmartTV](http://mobilization.pl/schedule/android-and-smart-tv/) ale niestety prelegent nie dojechał i zamiast tego mieliśmy prezentację, która miała się odbyć na końcu, czyli [Testowanie na Androidzie](http://mobilization.pl/schedule/testing-in-android/). Temat bardzo interesujący, bo każdemu spokojniej się pracuje i refaktoruje, gdy ma kod pokryty testami, więc ciekawiło mnie jak można taki spokój ducha osiągnąć tworząc aplikacje na Androida. Prowadzący Marek Defeciński pokazał (niestety tylko teoretycznie, bo Maven nie chciał współpracować ) kilka bardzo ciekawych narzędzi, na które warto zwrócić uwagę pisząc testy na system od Google: [Robotium](http://code.google.com/p/robotium/) (Selenium dla Androida) oraz [Robolectric](http://pivotal.github.com/robolectric/), czyli taki JUnit na tę platformę. Dodatkowo wspomniane zostały również [Android Mock](http://code.google.com/p/android-mock/) oraz [PowerMock](http://code.google.com/p/powermock/). Prezentacja dobra, pomimo widocznej tremy u prowadzącego oraz nie działających przykładów.

**Testowanie aplikacji na prototypie**

Później nadszedł czas na mniej techniczne wystąpienie, czyli [Testowanie aplikacji na prototypie](http://mobilization.pl/schedule/test-prototype/). Damian Szczepanik w przyjemny sposób opisał różne podejścia do testowania naszych aplikacji na telefony, których nie ma jeszcze na rynku, a są dostępne tylko u producenta. Przedstawiony został łańcuch komunikacji (deweloperem - inwestor - operator sieci komórkowej - producent - tester), przez który muszą przejść wszystkie zgłoszenia błędów. Tak długa lista osób, które są zaangażowane w cały proces wymusza jak najdokładniejsze logowanie błędów (unikalne kody wyjątków, itp.) tak, aby uniknąć wielokrotnego przechodzenia tym łańcuchem z dodatkowymi pytaniami i wyjaśnieniami, co każdorazowo trwa przynajmniej kilka dni. A jeśli ktoś akurat jest na urlopie albo zwolnieniu, to zostajemy z nieprawidłowo działającą aplikacją na jeszcze dłużej.

Prelegent nie silił się na nadętą techniczną nomenklaturę i wprowadził Product Ownera Wieśka oraz Zycha i kilka innych postaci o przyjemnych imionach. Dzięki temu prezentacja była ciekawa i łatwo przyswajalna.

**Scrumo-Waterfall**

Ostatnią prezentacją była przygotowana na szybko prelekcja o połączeniu Scruma z Waterfallem w firmie jednego z organizatorów konferencji. Wystąpienie było wymuszone absencją jednego z prelegentów (tego od Androida i SmartTV) i szybko zamieniło się w dyskusję na temat szacowania czasochłonności czy obchodzenia mało elastycznego podejścia szefostwa do metodyk zwinnych tak, aby programistom się dobrze pracowało, a przełożeni byli zadowoleni. Generalnie można było posłuchać i powymieniać poglądy na około-agilowe tematy.


### Podsumowanie


Same prelekcje były na w miarę dobrym poziomie. Jedna prezentacja była kiepska, jedną trudno ocenić, bo była przygotowywana awaryjnie, a pozostałe były albo dobre (testowanie Androida, testowanie na prototypie telefonu) albo bardzo dobre (Windows Phone oraz Aplikacje wielo-platformowe). Patrząc z perspektywy osoby mającej niewielkie doświadczenie w programowaniu na urządzenia mobilne konferencja była ciekawa i pozwoliła mi dowiedzieć się wielu interesujących i przydatnych rzeczy.

Organizacyjnie było w porządku, ale (jak to w przypadku konferencyjnego debiutu) nie obyło się bez kilku wpadek. Poniżej moja subiektywna lista.

**Plusy**

- sensownie działające Wi-fi (tylko na początku były z nim problemy)

- ładna koszulka, co na darmowej konferencji jest rzadkością

- dobre sale: wygodne krzesła z podkładką i notatnikiem

- kanapki i jogurcik w przerwach, świetny i niedrogi patent na głodomorów

- aplikacja na telefon z agendą, prelegentami, itp.

- dużo nagród i upominków od sponsorów

**Minusy**

- przesunięcia i zmiany w agendzie

- o niektórych konkursach dowiedziałem się dopiero na zakończeniu w momencie losowania nagród

- mówienie na rozpoczęciu "mam nadzieję, że nie zapomniałem o żadnym ze sponsorów" nie brzmiało najlepiej :)  Slajdy z logotypami i nazwami firm, dzięki którym konferencja się odbyła, pokazane na rozpoczęciu i zakończeniu rozwiązałyby problem

- brak oficjalnego taga na Twitterze, przez co trudno było śledzić co piszą ludzie

- streaming live chyba nie za bardzo działał, a nawet jeśli byłby ok, to na stronie konferencji nie widziałem wcześniej żadnej informacji, że coś takiego będzie w ogóle dostępne

- duża odległość od lokali gastronomicznych + kolejki  = trudno było się wyrobić z obiadem w trakcie przerwy

Jak widać konferencja była udana, ale jest kilka obszarów, które niewielkim kosztem można poprawić. Zainteresowanie imprezami o tematyce mobilnej jest duże, więc myślę, że w 2012 odbędzie się kolejna edycja Mobilization. Czego sobie i Wam życzę, bo w tym roku było dobrze, więc za rok powinno być jeszcze lepiej.
