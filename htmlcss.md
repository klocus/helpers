HTML I CSS
### Workflow
+ Podczas pisania (S)CSS staraj się korzystać z metodologii [BEM](https://seesparkbox.com/foundry/bem_by_example) ([BEM Cheat Sheet](https://9elements.com/bem-cheat-sheet/)), czyli Block Element Modifier oraz z [ITCSS](https://www.xfive.co/blog/itcss-scalable-maintainable-css-architecture/).

+ Przy pisaniu w SCSS zwracaj uwagę na to, by liczba zagnieżdżeń selektorów nie przekraczała 4 poziomów *([Zasada Incepcji](http://thesassway.com/beginner/the-inception-rule))*.

+ Staraj się też unikać frameworków CSS. Wyjątkiem jest [Tailwind CSS](https://tailwindcss.com/) (konfiguracja VSC z Tailwindem: [KLIK](https://stackoverflow.com/a/62254613)). 

+ Wcięcia w plikach CSS oraz HTML twórz przy pomocy 2 spacji.

+ CSS pisz w oparciu o podejście [Mobile-First CSS](https://medium.com/@mrmrs_/mobile-first-css-48bc4cc3f60f).

+ [Kompletny przewodnik po media queries w CSS](https://polypane.app/blog/the-complete-guide-to-css-media-queries/)

+ Dokumenty HTML twórz w oparciu o semantycznie poprawny kod!

+ [Kiedy korzystać z Flexa, a kiedy z Grida?](https://ishadeed.com/article/grid-layout-flexbox-components/)

+ W dokumentach HTML z wieloma znacznikami stosuj komentarze pozwalające się odnaleźć w kodzie: `<!-- / name-of-class-or-id -->`
```html
<ol class="accessibility-nav">
  <li><a href="#navigation">Skip to navigation</a></li>
  <li><a href="#content">Skip to content</a></li>
  <li><a href="#sidebar">Skip to sidebar</a></li>
</ol>
<!-- / accessibility-nav -->
 
<p>
  <a href="#" title="Go to homepage"><em>Home</em></a>
</p>
<!-- / breadcrumb -->
```

### Font jak z projektu graficznego

Bardzo często grubość fontu w projekcie graficznym oraz naszym kodzie wydaje się być inna. I tak może być. Wystarczy zastosować jednak *antialiasing* w CSS, by font przypominał ten z projektu.

```css
* {
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}
```

### Kolejność właściwości CSS

Polecamy sortować właściwości zgodnie z typem. Możesz ten proces zautomatyzować: [webstorm](https://www.jetbrains.com/help/webstorm/settings-code-style-css.html#ws_css_settings_editor_code_style_arrangement), [vscode](https://marketplace.visualstudio.com/items?itemName=mrmlnc.vscode-postcss-sorting).

```css
.selector {
  /* Positioning */
  position: absolute;
  z-index: 10;
  top: 0;
  right: 0;

  /* Display & Box Model */
  display: inline-block;
  overflow: hidden;
  box-sizing: border-box;
  width: 100px;
  height: 100px;
  padding: 10px;
  border: 10px solid #333;
  margin: 10px;

  /* Color */
  background: #000;
  color: #fff
  
  /* Text */
  font-family: sans-serif;
  font-size: 16px;
  line-height: 1.4;
  text-align: right;

  /* Other */
  cursor: pointer;
}
```

### Lista 6 często popełnianych błędów przy budowie frontendu
M.in. źle zaimplementowane placeholdery, kiepsko osadzone ikony, blokowanie funkcji "resize" na textarea i inne.

https://dev.to/melnik909/the-6-most-common-mistakes-developers-when-writing-html-and-css-f92

### Jak nie tworzyć przycisków?

https://blog.benmyers.dev/clickable-divs/

### Lista 10 *supermocy*, które daje HTML5
A których prawdopodobnie nie używasz lub nie znasz :-)

https://dev.to/shadowwarior5/10-superpowers-that-html5-gives-you-and-you-are-not-using-4ph1

### Unikaj 100vh na stronach mobilnych
Unikaj korzystania z wartości `100vh` dla urządzeń mobilnych. Zamiast tego, można użyć `html, body { height: 100%; }` dla tego samego efektu na różnych urządzeniach bez kłopotów z opisywanym w artykule problemem.

https://chanind.github.io/javascript/2019/09/28/avoid-100vh-on-mobile-web.html

### 100vh / 100% i skoki na przeglądarkach mobilnych
Często zdarza się tak, że np. nasze intro posiada wysokość równą `100vh` lub `100%` dostępnej wysokści okna przeglądarki.
Na niektórych mniej popularnych przeglądarkach występuje efekt skoku po przescrollowaniu strony w dół.
Aby uniknąć tego efektu na przeglądarkach innych niż Chrome i Safari, należy wykonać poniższe kroki:

1) Definicja zmiennej CSS:
```css
:root {
   --vh: 100vh;
}
```
2) Definicja wysokości elementu w CSS:
```css
.intro {
  height: var(--vh);
}
```
3) Nadpisanie zmiennej przez JavaScript:
```javascript
document.body.style.setProperty('--vh', `${window.innerHeight}px`);
```

### Obsługa *dark mode*
W CSS można ustawić osobne właściwości w zależności od trybu urządzenia (dark mode / light mode).

https://tombrow.com/dark-mode-website-css

### Tworzenie mailingu
+ *Can I Use*, ale przydatny przy projektowaniu mailingów.
  https://www.caniemail.com/ 

+ Framework, który przyspiesza tworzenie mailingu i jest kompatybilny z wszystkimi popularnymi klientami poczty.
  https://mjml.io/

### Solved by Flexbox
Lista problemów z layoutem na stronie (np. z przypiętą stopką), które kiedyś wymagały CSS-wej gimnastyki, a dziś dzięki flexboxowi można je rozwiązać w kilku linijkach.

https://philipwalton.github.io/solved-by-flexbox/

JS

### Style Guide

Jak pisać w JS, żeby kod był ładny: https://github.com/airbnb/javascript

PAMIĘTAJ O WCIĘCIACH PRZY POMOCY **2 SPACJI**

### Co to jest ten CORS?
Wyjaśnienie przyczyny Twoich problemów w JS przy odpytywaniu innych serwerów. Przystępnie przedstawiony temat “Cross-Origin Resource Sharing”. Czym jest origin? Kiedy podpadasz pod CORS?

https://dev.to/g33konaut/understanding-cors-aaf

### Angular, NGRX i obiekt
**Paweł:** *Posługując się NGRX nie wiedziałem dlaczego nie dostaję zmienionej wartości obiektu. Okazuje się, że aby NGRX wysłał informację, że jedna z wartości obiektu została zmieniona, musi zostać "odświeżony" cały obiekt!*

>When updating a property of an object on the state, you must ALWAYS create a new containing object, because NGRX uses simple object equality to determine if something has changed and fire the relevant observables. If you update a property on an object, the object itself still has the same reference as its previous version, so NGRX will assume it is unchanged.

### Przeglądarka zwraca JSON zamiast strony

**Paweł:** *Po cofnięciu się przeglądarką do poprzedniej strony, która pobierała dane JSON, otrzymywałem sam JSON, zamiast reszty strony, tj. nagłówek, menu, tabelkę z danymi itd.

**Darek:** *Nie, to wina przeglądarki, która cachuje ostatnie requesty. Spróbuj w zapytaniach z frontu do backendu dodawać jakiś unikalny parametr np. `_t`, który będzie wynosił `new Date().getTime()`.*

### Domknięcia w JS
Bardzo fajny artykuł o domknięciach w JavaScript. Wyjaśnienie do czego służą w języku JavaScript domknięcia i jak je stosować.

http://blog.nebula.us/13-javascript-closures-czyli-zrozumiec-i-wykorzystac-domkniecia

### Modyfikacja adresu URL bez odświeżania strony
`URL()` pozwala w łatwy sposób dodawać i modyfikować istniejące parametry GET w URL. `window.history.pushState` modyfikować adres w przeglądarce bez odświeżania strony.

https://developer.mozilla.org/en-US/docs/Web/API/URL/URL

### Preloader treści we Vue
Prosty sposób na ukrycie niezaładowanej do końca treści aplikacji Vue.js

https://medium.com/vuejs-tips/v-cloak-45a05da28dc4