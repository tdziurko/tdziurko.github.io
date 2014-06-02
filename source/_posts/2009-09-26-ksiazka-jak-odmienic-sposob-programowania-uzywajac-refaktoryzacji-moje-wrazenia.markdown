---
author: Tomasz Dziurko
comments: true
date: 2009-09-26 05:57:00+00:00
layout: post
slug: ksiazka-jak-odmienic-sposob-programowania-uzywajac-refaktoryzacji-moje-wrazenia
title: Książka "Jak odmienić sposób programowania używając refaktoryzacji" - moje
  wrażenia
keywords: Książka, Jak odmienić sposób programowania używając refaktoryzacji, refaktoryzacja, recenzja, opis, wrażenia
wordpress_id: 269
categories:
- Books
- In Polish
tags:
- Books
- In Polish
- review
---

Dear Reader
    This post is in my native language as it applies only to developers society in Poland. Please feel free to check my other posts which are mainly written in English.


Ku mojej radości rodzą się w Polsce inicjatywy, by pisać książki  informatyczne niosące sporą dawkę wiedzy w naszym ojczystym języku.  Cieszy to tym bardziej, że często czas czekania na polskie tłumaczenie  jest skandalicznie długi. Faktem jest, że i tak każdy z nas chcąc być na  bieżąco z nowinkami musi czytać książki, odwiedzać portale i blogi w  dużej części pisane po angielsku, ale z drugiej strony miło jest od  czasu do czasu zanurzyć się w świat Javy i programowania opisany "przez  naszych i po naszemu".

Jedną z takich pozycji jest e-book pod tytułem "Jak całkowicie odmienić sposób programowania używając refaktoryzacji" autorstwa [Mariusza Sieraczkiewicza](http://msieraczkiewicz.blogspot.com/).  Książka nie jest przesadnie potężna objętościowo, ale po jej  przeczytaniu mogę stwierdzić, że jako wprowadzenie w świat  refaktoryzacji i pokazanie najważniejszych i najbardziej użytecznych  technik spełnia swoją rolę bardzo dobrze.<!-- more -->

Autor na wstępnie  przedstawia myśl, która na początku nauki programowania nie była obca  chyba nikomu: "Obym nie musiał w tym kodzie nic zmieniać". Niestety  okazuje się, że kod aplikacji jest częściej czytany i modyfikowany po  wdrożeniu niż zdajemy sobie sprawę, dlatego tak duży nacisk powinniśmy  kłaść na jakość i przejrzystość tego co tworzymy. Nasze sprytne  rozwiązanie problemu, które wymyśliliśmy dzisiaj za pół roku może być  dla nas równie trudne do zrozumienia co kod napisany przez kogoś innego.

``` java
//
//When I wrote this, only God and I understood what I was doing
//Now, God only knows
//
```

Źródło: [StackOverflow](http://stackoverflow.com/questions/184618/what-is-the-best-comment-in-source-code-you-have-ever-encountered)

Dlatego właśnie tak ważne jest, aby pisać kod, który nie tylko dobrze  działa, ale i który będzie łatwo się czytało, analizowało i modyfikowało  w przyszłości. I właśnie o tym jest ta książka, na przykładzie programu  do tłumaczenia słów z wykorzystaniem internetowego słownika  polsko-angielskiego autor krok po kroku wyjaśnia co można zrobić, żeby  kod aplikacji był czytelniejszy i przygotowany do zmian w przyszłości.

Najważniejsze, ale i najbardziej podstawowe porady zawarte w książce to:
- twórz zmienne i metody, których nazwy opisują ich cel i sposób działania
- unikaj rozwlekłych klas i metod. Dobrym miernikiem długości metody  jest możliwość przeczytania jej w całości bez konieczności przewijania  ekranu.
- nie twórz bytów ponad miarę, anty-przykład: "wypychanie"  metod i zmiennych na siłę do klas abstrakcyjnych, z których potem i tak  dziedziczy tylko jedna klasa.
- unikaj niejawnego modyfikowania  zmiennych klasowych wewnątrz metod. Jeśli inicjalizujesz taką zmienną  klasy za pomocą metody, łatwiej będzie to odnotować, gdy metoda będzie  zwracała właśnie tą zmienną.

Po więcej ciekawych technik refaktoryzacji odsyłam do książki (**do kupienia** [tutaj](http://www.mistrzprogramowania.pl/sklep?page=shop.product_details&flypage=garden_flypage.tpl&product_id=1&category_id=1)),  cena wersji elektronicznej jest bardzo atrakcyjna (pro-studencka można  by rzec), bo za 25zł dostajemy ponad 90 stron przydatnej wiedzy. W  dodatku wspieramy rodzimych autorów, co miejmy nadzieję zachęci innych  do pójścia w ich ślady :)
