# Funkcjonalna dokumentacja React.js

## Słownik

**Odwołanie** - w programowaniu pozwala nam na interakcje z wartościami przechowywanymi w pamięci komputera, mechanizm odwołania upraszcza składnie języka pozwalając nam na "przechowywanie" wartości w bardziej zrozumiałym dla człowieka języku. W innym przypadku musielibyśmy dostawać się do wartości poprzez podawanie jej adresu komórki pamięci.</br>
**Deklaracja** - deklaracja to tak naprawdę tworzenie "rezerwacji" identyfikatora dla jeszcze nie określonego typu danych.</br>
**Definicja** - to rezerwacja oraz dokładne opisanie danego identyfikatora. Każda definicja jest zarazem deklaracją ale **NIE** odwrotnie.</br>
**Inicjalizacja (inicjowanie)** - inicjacja polega na przypisaniu wartości do danej zmiennej w momencie jej deklaracji.

## Nowe zmiany w wydaniach biblioteki

- [React 19](/releases/react-19.md)

## Spis treści

### Często używane hooki

- [`useState`](#usestate)
- [`useEffect`](#useeffect)
- [`useRef`](#useref)
- [`useReducer`](#usereducer)
- [`useContext`](#usecontext)
- useMemo
- useCallback

### Wykorzystywane techniki

- [`Prop drilling`](#prop-drilling)
- Context API
- [`Component composition`](#component-composition)
- [`Derived state`](#derived-state)

### React Server Components

- [React Server Components](#react-server-components)
- [Hydracja](#hydracja)

### Problemy

- [`Stale State`](#stale-state)

## Często używane hooki

### useState

> Hook useState w React.js jest używany do zarządzania stanem komponentu.
>
> Jest to funkcja, która przyjmuje jako argument początkowy stan i zwraca tablicę z dwoma elementami: aktualnym stanem i funkcją do ustawiania nowego stanu.
>
> Ustawiony stan jest dostępny w komponencie tak długo jak komponent nie zostanie usunięty.

Przykład:

```tsx
import React, { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>Click me</button>
    </div>
  );
}
export default Counter;
```

- Montowanie: Kiedy komponent Counter jest po raz pierwszy renderowany, useState(0) inicjalizuje stan count z wartością 0.
- Aktualizacje: Kiedy użytkownik kliknie przycisk, setCount(count + 1) aktualizuje stan count, co powoduje ponowne renderowanie komponentu z nową wartością stanu.
- Odmontowanie: Kiedy komponent Counter jest usuwany z drzewa komponentów (np. przez usunięcie go z DOM), stan count jest również usuwany.

---

### Czas życia zapisanego stanu

Hook `useState` przechowuje stan w pamięci komponentu podczas jego cyklu życia. Oznacza to, że stan jest zachowywany tak długo, jak długo komponent jest zamontowany w drzewie komponentów. Inaczej mówiąc stan jest tak długo przetrzymywany w pamięci komponentu jak jest wyrenderowany, jeżeli komponent zostanie wyrenderowany ponownie, stan zostanie przywrócony do początkowego stanu.

---

### Cykl życia stanu w useState

1. **Montowanie komponentu - wywołanie funkcji komponentu**
   - Kiedy komponent jest po raz pierwszy montowany (renderowany) w drzewie komponentów, `useState` inicjalizuje stan z podaną wartością początkową.
   - Stan jest przechowywany w pamięci i jest dostępny dla komponentu podczas jego cyklu życia.
2. **Aktualizacja komponentu - wywołanie funkcji set w `useState`**
   - Kiedy stan jest aktualizowany za pomocą funkcji zwróconej przez `useState` (np. `setCount`), komponent jest ponownie renderowany z nową wartością stanu.
   - Nowy stan jest przechowywany w pamięci i jest dostępny dla komponentu podczas jego cyklu życia.
3. **Odmontowanie komponentu - usunięcie komponentu**
   - Kiedy komponent jest odmontowany (usuwany) z drzewa komponentów, stan przechowywany przez `useState` jest również usuwany.
   - Oznacza to, że stan nie jest już dostępny i nie jest przechowywany w pamięci.

---

Zapisany stan w hooku `useState` jest zapisywany w momencie renderowania komponentu. Jeśli chcesz, aby stan był zapisywany w innym miejscu, możesz użyć funkcji `useEffect `.

### useEffect

> useEffect pozwala zarządzać efektami ubocznymi w komponentach funkcyjnych.
>
> Efekt uruchamia się po każdym renderowaniu, chyba że podasz tablicę zależności.
>
> Możesz zwrócić funkcję czyszczącą, która uruchomi się przed kolejnym uruchomieniem efektu lub przy odmontowywaniu komponentu

UseEffect jest uruchamiany dopiero po renderowaniu komponentu.

UseEffect załącza się po wyświetleniu DOM w przeglądarce, czyli po etapie paint browser
Tablica zależności w jest bardzo ważnym elementem useEffect, ponieważ dzięki niej udaje się zapanować nad momentem, kiedy useEffect ma zostać odpalony, albo pozwala zaplanować, kiedy dane pobranie danych ma zostać wykonane.

```tsx
import React, { useState, useEffect } from 'react';

const UseEffectExample: React.FC = () => {
  const [count, setCount] = useState<number>(0);

  useEffect(() => {
    document.title = `You clicked ${count} times`;

    // funkcja czyszcząca (opcjonalna)
    return () => {
      document.title = 'React App';
    };
  }, [count]); // tablica zależności

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>Click me</button>
    </div>
  );
};

export default UseEffectExample;
```

Przykład pokazuje użycie hooka `useEffect` pozwalającego na aktulizację tytułu dokumentu za każdym razem gdy zmienia się wartość `count`

---

### useReducer

Hook useReducer jest bardzo podobny do hooka `useState`, ale umożliwia przeniesienie logiki aktualizacji stanu do pojedynczej funkcji poza komponentem.
Dzięki temu ułatwia zarządzanie bardziej skomplikowaną logiką stanu w porównaniu do prostych zmian stanu, które można łatwiej obsłużyć za pomocą `useState`

#### Przykładowe użycie `useRecuder`

```TSX
const [state, dispatch] = useReducer(reducer, initialState);
```

#### Opis

- `state` - reprezentuje aktualnie przekazaną wartość do `useReducer` przyjmuje wartość inicjacyjną (`initialState`) podczas inicjacyjnego renderu
- `dispatch` - jest to funkcja która odpowiada za logikę aktualizacji stanu oraz jest odpowiedzialna za wyzwolenie akcji `re-render` tak jak funkcja aktualizująca w `useState`
- `reducer` - jest to funkcja, która przechowuje całą logikę odpowiedzialną jak stan zostanie zaktualizowany. Przyjmuje jako argumenty `state` i `action` zwraca na ich podstawie kolejny stan.
- `initialState` - przechowuje inicjacyjną wartość może być dowolnego typu(\*)

(\*) dowolnego typu aplikuje się tylko dla aplikacji pisanych w JSX jeżeli będziemy pisać aplikację za pomocą TypeScript musimy ściśle o typować taką zmienną.

https://www.freecodecamp.org/news/react-usereducer-hook/

## Działanie hooka useReducer

Hook przyjmuje trzy argumenty:

1. `reducer`: funkcja, która zawiera całą logikę aktualizacji stanu. Przyjmuje bieżący stan i akcję jako argument i zwraca następny stan.
2. `initialState`: początkowa wartość stanu, może być dowolnego typu.
3. `init` (opcjonalnie): funkcja używana do leniwego inicjalizowania stanu, jeśli jest potrzebna

Zwraca tablicę składającą się z dwóch elementów:

1. `state`: reprezentuje bieżącą wartość stanu, ustawioną na wartość `initialState` podczas pierwszego renderowania
2. `dispatch`: funkcja, które aktualizuje wartość stanu i zawsze wywołuje ponowny render, podobnie jak funkcja aktulizująca w `useState`

```tsx
import React, { useReducer } from 'react';
// Definicja typów akcji
const ActionTypes = {
  INCREMENT: 'increment',
  DECREMENT: 'decrement',
  RESET: 'reset',
};

// Reducer
function reducer(state, action) {
  switch (action.type) {
    case ActionTypes.INCREMENT:
      return { count: state.count + 1 };
    case ActionTypes.DECREMENT:
      return { count: state.count - 1 };
    case ActionTypes.RESET:
      return { count: 0 };
    default:
      throw new Error();
  }
}

// Komponent Counter
function Counter() {
  const [state, dispatch] = useReducer(reducer, { count: 0 });

  return (
    <div>
      Licznik: {state.count}
      <button onClick={() => dispatch({ type: ActionTypes.INCREMENT })}>
        Zwiększ
      </button>
      <button onClick={() => dispatch({ type: ActionTypes.DECREMENT })}>
        Zmniejsz
      </button>
      <button onClick={() => dispatch({ type: ActionTypes.RESET })}>Resetuj</button>
    </div>
  );
}

export default Counter;
```

---

### useRef

`useRef` - jest hookiem który pozwala odwołać się do wartości która nie wymaga re-render'u całego komponentu w którym się znajduje.
Dane jakie nie wymagają re-render'u całego komponentu to dane które **NIE** wpływają na wyświetlaną treść komponentu jak na przykład `id` wciśniętego przycisku przez użytkownika to jest typ danych które nie są potrzebne dla użytkownika, ale mogą być wykorzystywane w dalszej części aplikacji.

```tsx
const ButtonWithId = () => {
  // definiowanie hook'a useRef wraz z wartością inicjowaną
  const lastClickedButtonId = useRef(null);

  // funkcja odpowiedzialna za obsługę kliknięcia przycisku
  const handleClick = (id) => {
    lastClickedButtonId.current = id;
    console.log(`Button with ID ${id} clicked`);
  };

  return (
    <div>
      <button onClick={() => handleClick(1)}>Button 1</button>
      <button onClick={() => handleClick(2)}>Button 2</button>
      <button onClick={() => handleClick(3)}>Button 3</button>
    </div>
  );
};
```

---

### useContext

`useContext` to hook w React, który pozwala na korzystanie z kontekstu w funkcjonalnych komponentach. Kontekst w React służy do przekazywania danych przez drzewo komponentów bez konieczności ręcznego przekazywania propsów na każdym poziomie.

Jak działa `useContext`:

1. Tworzenie kontekstu: Najpierw tworzysz kontekst za pomocą React.createContext.
1. Dostarczanie kontekstu: Następnie używasz komponentu Provider, aby dostarczyć wartość kontekstu do drzewa komponentów.
1. Korzystanie z kontekstu: W końcu używasz hooka useContext, aby uzyskać dostęp do wartości kontekstu w dowolnym komponencie.

Aby skorzystać z hooka `useContext` należy skorzystać `Providera` który umożliwy udostępnienie stanu jaki zostanie przypisany do hooka `useContext`.

Kroki jak zacząć korzystać z hooka `useContext`:

1. Towrzymyu provider który będzie przyjmować stan aktualnej wartości

```tsx
const NazwaProvidera = createContext(null); // <- ta zmienna będzie przetrzymywać aktualny stan jaki zostanie przekazany do kontekstu

// Przakazanie komponentu do providera aby mial dostep do aktualnego stanu kontekstu
<NazwaProvidera.Provider value='dowolna_wartość'>
  <ExampleComponent />
</NazwaProvidera.Provider>;
```

2. Wewnątrz każdego komponentu który ma korzystać z kontekstu musimy zdefiniować zmienną o dowolnej nazwiem, do której będzie przypisywany hook `useContext` dla przykładu:

```tsx
const exampleName = useContext(NazwaProvidera);
```

Zmienna `exampleName` powinna być zdefiniowana na samej górze każdego komponentu, który ma mieć dostęp stanu jaki przyjmie aktualny stan `Provider'a`.
Jeżeli rodzić komponentu zostanie wstrzyknięty do `context provider'a` to jego dzieci będzie mieć dostęp do stanu kontekstu, jeżeli będzie chciały skorzystać z jego stanu, będzie właśnie wymaganie zdefiniowanie zmiennej, która będzie przyjmować wartość `useContext(NazwaProvider'a)`

```
ContextProvider

  ParentComponent

    ChildComponent-1
    ChildComponent-2
      GrandChildComponent-1
    ChildComponent-3
    ...
    ChildComponent-n

  ParentComponent

ContextProvider
```

Każdy z komponentów zawartych w `ParentComponent` będzie posiadać dostęp do wartości udostępnianych poprzez kontekst, oczywiście rodzic również będzie posiadać dostęp.
Czyli dostęp do kontekstu będą posiadać:

- `ParentComponent`
- `ChildComponent-n`
- `GrandChildComponent-n`

---

## Wykorzystywane techniki

### prop-drilling

Prop driling jest to zarazem sposób na przekazywanie danych z rodzica do dziecka w react.js. I również jest to problem jakim możemy się zacząć borykać podczas tworzenia większej aplikacji, ponieważ przekazywanie z jednego komponentu do drugiego, jeżeli jest ich na przykład dwoje to jest prosty sposób. Ale jeżeli chcemy przekazać dane z rodzica do dziecka, a następnie do dziecka do dziecka i tak dalej to jest problem, ponieważ musimy śledzić wszystkie komponenty, które są "pośrednikami" przekazania propsów.

![prop drilig](./img/prop-driling-light.svg)

Przykładowy kod z prop drilling:

```js
import React, { useState } from 'react';

function ParentComponent() {
  const [data, setData] = useState('');
  const handleChange = (event) => {
    setData(event.target.value);
  };
  return (
    <div>
      <h1>Parent Component</h1>
      <ChildComponent
        data={data}
        onChange={handleChange}
      />
    </div>
  );
}

function ChildComponent({ data, onChange }) {
  return (
    <div>
      <h2>Child Component</h2>
      <input
        type='text'
        value={data}
        onChange={onChange}
      />
    </div>
  );
}
export default ParentComponent;
```

Zalecanym sposobem na przekazywanie danych pomiędzy większą ilością komponentów jest użycie kontekstu. Kontekst pozwala na przekazywanie danych pomiędzy wszystkimi komponentami bez potrzeby śledzenia ich hierarchii. Albo tak zwanego podejścia `component composition`

---

### component-composition

Kompozycja komponentów polega na tym, że nie przekazujemy kolejnych parametrów za pomocą `props` a zamiast tego robimy kompozycję z innego komponentu.\
Kompozycja polega na tym że przekazujemy jeden komponent do kolejnego jako dziecko używamy do tego `children` jako `ReactNode`

```tsx
const Button = ({ onClick, children }) => (
  <button onClick={onClick}>{children}</button>
);

const App = () => {
  const onClick = () => alert('Hey 👋');

  return <Button onClick={onClick}>Click me!</Button>;
};
```

Kompozycja rozwiązuje problem [prop driling](#prop-drilling) napotykany wtedy kiedy musimy w dużej ilości pod komponentów przekazać potrzebne im do działania props'y\
Również kompozycja jest częścią optymalizacji ze względu na to, że aktualizacja komponentu dziecka **NIE** uruchamia re-render'u komponentu rodzica pozbywamy się zbędnego re-render'u (Wasted Renders)

---

### Derived state

Technika odnośni się do tworzenia stanu komponentu, który jest tworzony na podstawie innych stanów lub właściwości pozyskanych z props. **Stan jest dynamicznie tworzony w trakcie renderowania komponentu**, nie wykorzystujemy do jego stworzenia hooka `useState`.

Przykładowy kod wykorzystujący `Delivered State` ustawiony za pomocą dostarczonych **Props'ów**

```tsx
import React from 'react';

function ItemList({ items }) {
  // Derived state: liczba elementów
  const itemCount = items.length;

  return (
    <div>
      <p>Number of items: {itemCount}</p>
      <ul>
        {items.map((item, index) => (
          <li key={index}>{item}</li>
        ))}
      </ul>
    </div>
  );
}

export default ItemList;
```

> **Delivered state** pozwala na obliczanie stanu na podstawie innych stanów lub props'ów, co pozwala nam na uproszczenie logiki komponentów i uniknięcie problemów z synchronizacją.
>
> Jednym z minusów takiego działania jest to, że takie obliczenia lub ustawianie danych stanów są kosztowne, aby zminimalizować wpływ takiego **Delivered state** na wydajność możemy wykorzystać to tego hook'a `useMemo`

---

## React Server Components

Ciekawa lektura na temat komponentu jest zawarta tutaj: https://www.freecodecamp.org/news/react-server-components-for-beginners/

W streszczeniu serwerowe komponenty w React pozwalają nam na pozbycie się problemu jakim jest "waterfall" wystepujacy w zapytaniach o dane komponentu rodzica oraz jego dzieci.

```tsx
const App = () => {
  return (
    <Wrapper>
      <ComponentA />
      <ComponentB />
    </Wrapper>
  );
};
```

Załóżmy, że komponent `Wrapper` odpytuje API o dane `wrapperData.json` oraz jego dzieci `ComponentA` i `ComponentB` następująco odpytują API od ane `componentAData.json`, `componentBData.json` w takim przypadku jeżeli komponent będzie renderowany na poziomie klienta nastąpi zjawisko `waterfall` czyli aby komponent `Wrapper` został w renderowany musimy poczekać na wszystkie zapytania idąc w dół czyli gdy zakończą odpytywać API wszystkie jego dzieci, w tym przykładzie `ComponentA` oraz `ComponentB`, zakładając że każde zapytanie zajmuje komponentowi 1 sekundę to kończymy z winkiem 3 sekund zanim ujrzymy komponent `Wrapper` wraz z jego dziećmi.

### Rozwiązania jakie posiadamy to:

1. Zaciąganie danych na wyższym poziomie i wstrzykiwanie ich do danych komponentów za pomocą wykorzystania `props`, zamiast zaciąganie danych wewnątrz każdego z komponentów
2. Wykorzystanie `React Server Component`

#### 1 wstrzykiwanie danych do komponentu jako props

```tsx
const App = () => {
  const data = fetchAllStuffs();

  return (
    <Wrapper data={data.wrapperData}>
      <ComponentA data={data.componentAData} />
      <ComponentB data={data.componentBData} />
    </Wrapper>
  );
};
```

Wstrzykiwanie danych co prawda rozwiązuje zjawisko `waterfall` ale ma to do siebie, że jest trudniejsze w utrzymaniu gdy aplikacja się rozrasta może ulec zmianie implementacja danych, backend może przestać wystawiać dane itp... to wszystko przekłada się na późniejsze sprzątanie kodu.

#### 2 Wykorzystanie React Server Component

```tsx
// Note.js - Server Component

import NoteEditor from 'NoteEditor';

async function Note(props) {
  const { note } = props;

  return (
    <div>
      <h1>{note.title}</h1>
      <section>{note.body}</section>
    </div>
  );

```

Komponent serwerowy ma taką przewagę nad klienckim komponentem, że ma dostęp do danych w serwerze bez potrzeby ich pobierania. Dlatego w komponencie serwerowym nie wykorzystujemy, żadnego fetchowana danych ani podobnego pobierania danych za pomocą API od razu mamy dostęp do całych zasobów przetrzymywanych na serwerze.

#### Rzeczy jakie możną robić w React Serwer

- używanie `async/await` z danymi w bazie danych, wewnętrznych serwisów, systemów plików itp.
- renderowanie innych komponentów serwerowych, natywnych elementów takich jak `div` i inne elementy `html` lub **kliencki komponent**

#### Rzeczy jakich się nie da zrobić w React Serwer

- Nie ma możliwości użycia hooków jakie są dostarczane dzięki Reactowi takich jak `useState` itd. jako komponent serwerowy renderowanie odbywa się po stronie serwera
- Nie ma możliwości używania API jako Local Storage dane pobierane za pomocą API można od razu uzupełnić za pomocą serwera
- Nie ma możliwości używania funkcjonalności które bazują na przeglądarce lub custom hooks które są oparte na useState lub useEffect

#### Zasada importowania komponentów RSC & RCC

**Komponent kliencki nie może importować komponentu serwerowego**

Za to można robić takie kombinacje:

1. **Dozwolone** jest importowanie komponentu klienckiego wewnątrz komponentu serwerowego
2. **NIE** dozwolone jest importowanie serwerowego komponentu wewnątrz komponentu klienckiego
3. **Dozwolone** jest przekazanie serwerowego komponentu jako dziecko do klienckiego komponentu wewnątrz komponentu serwerowego

##### Przykład trzeciego punktu

```tsx
const ServerComponentA = () => {
  return (
    <ClientComponent>
      <ServerComponentB />
    </ClientComponent>
  );
};
```

#### Zaletami korzystania z RSC są:

1. Automatyczne cięcie kodu, czyli serwerowy komponent wycina wszystkie nie potrzebne importy kodu, lub ładuje dodatkowe części kodu jako "lazy"
2. Pozbycie efektu waterfall

Źródło: https://www.freecodecamp.org/news/react-server-components-for-beginners/

## Hydracja

Hydracja jest to dosyć ważnym mechanizmem wykorzystania [React Server Component](#react-server-components) ponieważ wy renderowany element po stronie serwera nie posiada, żadnej interaktywności ze względu na brak kodu JS oraz tego, że każdy element wy renderowany po stronie serwera jest czystym szkieletem HTML. Nadanie interaktywności komponentu jaki został wy renderowany po stronie serwera nazywamy Hydracją co można przedstawić analogicznie jako hipotetyczne nawadnianie JS'em suchego szkieletu komponentu jakim jest HTML.

### Częste błędy hydracji:

Błąd hydracji może nastąpić w momencie gdy nasz element jaki renderujemy po stronie klienta nie zgadza się z elementem jaki został przekazany jako komponent serwerowy.

1. Niepoprawne zagnieżdżenie znaczników HTML
   1. `<p>` zagnieżdżone w kolejnym znaczniku `<p>`
   2. `<div>` zagnieżdżony w znaczniku `<p>`
   3. `<ul>` lub `<ol>` zagnieżdżone w znaczniku `<p>`
   4. Interaktywne elementy nie mogą być zagnieżdżone w znaczniku `<a>`, `<button>`
2. Używanie sprawdzania takiego jak `typeof window !== 'undefined'` w logice renderowania
3. Rozszerzania przeglądarki modyfikujące strukturę `HTML`

Więcej przykładów można znaleźć tutaj: https://nextjs.org/docs/messages/react-hydration-error

## Server Actions

Są to asynchroniczne funkcje które działają tylko i wyłącznie na serwerze, umożliwiające nam na edytowanie danych po stronie serwera.
Akcje serwerowe tworzy się za pomocą `use server` na samej górze funkcji albo całego modułu.

> `use server` jest specjalnie stworzoną dyrektywą, działająca tylko do tworzenia akcji serwerowych, jej działanie jest takie, że umożliwia klintowi na komunikację się z serwerem poprzez zdefioniowane akcje serwerowe.

Server Actions

- Można zdefiniować jako asynchroniczną funkcję działającą wewnątrz komponentu albo może zostać przekazana do klienckiego komponentu.
- W odzielnym pliku, poczym eksportowane funkcje będą zaimportowane do serwerowego komponentu działają jako akcję serwera.

Działanie:

Next.js tworzy API endpoint dla każdej zdefiniowanej akcji serwerowej. W każdym przypadku kiedy akcja jest wywoływana, to jest tworzone zapytanie za pomocą metody POST.
Funkcja odpowiadająca za daną akcję na serwerze nigdy nie trafia do klienta. Ponieważ w jej wywołaniu pośredniczy API obsłużone przez Next.js

## Problemy

### Stale state

`Stale State` odnosi się do sytuacji w której komponent używa przestarzałej wersji stanu, ponieważ stan nie został zaktualizowany w odpowiednim momencie lub w odpowiedni sposób. Co zazwyczaj prowadzi do nie oczekiwanym zachowań oraz błędów.

Przykład problemu `stale state`

```tsx
import React, { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);

  const handleClick = () => {
    setTimeout(() => {
      setCount(count + 1);
    }, 1000);
  };

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={handleClick}>Increment</button>
    </div>
  );
}

export default Counter;
```

Kliknięcie przycisku zwiększy wartość `count` o 1 po upływie 1 sekundy. Jednakże, jeśli klikniemy w przycisk kilka razy w krótkim odstępie czasu to zmienna `count` ulegnie rozsynchronizowaniu. Dzieje się tak ponieważ funkcja `setTimeout` używa przestarzałej wartości zmiennej `count` w momencie jej wywoływania.

Rozwiązaniem problemu jest wykorzystanie funkcji aktualizującej stan, która przyjmuje poprzedni stan i na jego podstawie zwraca nowy stan.

Przed:

```tsx
const handleClick = () => {
  setTimeout(() => {
    setCount(count + 1);
  }, 1000);
};
```

Po:

```tsx
const handleClick = () => {
  setTimeout(() => {
    setCount((prevCount) => prevCount + 1); // użycie funkcji aktualizującej
  }, 1000);
};
```
