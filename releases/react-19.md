## Nowości React 19

### Akcje

Akcje z konwencji to funkcje, które wykorzystują asynchroniczne przejście.

Przykładową akcją jaka została dodana jest nowy hook `useTransition`, który pozwala na obsłużenie stanu oczekującego

```tsx
// Using pending state from Actions
function UpdateName({}) {
  const [name, setName] = useState('');
  const [error, setError] = useState(null);
  const [isPending, startTransition] = useTransition();

  const handleSubmit = () => {
    startTransition(async () => {
      const error = await updateName(name);
      if (error) {
        setError(error);
        return;
      }
      redirect('/path');
    });
  };

  return (
    <div>
      <input
        value={name}
        onChange={(event) => setName(event.target.value)}
      />
      <button
        onClick={handleSubmit}
        disabled={isPending}>
        Update
      </button>
      {error && <p>{error}</p>}
    </div>
  );
}
```

To asynchroniczne przejście funkcji automatycznie ustawi stan `isPending` na stan `true` przekształcając rządania na rządania asynchroniczne, a po wykonaniu jakiegolwiek nowego przejścia typu submit lub innego rządania stan `isPending` zostanie automatycznie ustawiony na `false`.
Takie działanie pozwala na zachowanie aktualnego stanu UI responsywnego i interaktywnego podczas zmian danych.

Akcje automatycznie wykorzystują wysyłane dane:

- **Pending state**: akcje dostarczają status oczekujący na początku nowego zapytania, a resetuje się atutomatycznie podczas wysłania ostatecznego stanu.
- **Optimistic uppdates**: akcje wspierają nowy hook `useOptimistic` dzięki czemu użytkownik może zostać od razu poinformowany podczas wysyłania nowego rządania.
- **Error handling**: akcje dostarczają obsługę błędów, dzięki którym można wyświetlić informacje na temat błędów gdy zapytanie skończy się błędem, automatycznie zostanie obsłużona akcja cofnięcia `optimistic updates` do orginalnej wartości.
- **Form**: `<form>` elementy teraz wspierają przekazywanie funkcji do `action` oraz `formAction` prop. Przekazywanie funkcji do `action` prop używa akcji domyślnie oraz resetuje formularz atutomatycznie po akcji `submit`.

### Hook useActionState

Hook UseActionState powstał po to aby usprawnić prace na powszechnie wykorzystywanych operacjach `Akcji`

Dla przykładu:

```tsx
const [error, submitAction, isPending] = useActionState(
  async (previousState, newName) => {
    const error = await updateName(newName);
    if (error) {
      // You can return any result of the action.
      // Here, we return only the error.
      return error;
    }

    // handle success
    return null;
  },
  null
);
```

Hook `useActionState` przyjmuje jako argument funkcję (`Akcję`) opakowuję podaną funkcję i zwraca ją z możliwością wykonania.
Mechanizm działa ze względu na kompozycję akcji.

Działanie: opakowana `Akcja` jak zostanie wywołana to hook `useActionState` zwróci ostatnią wartość `akcji` jako dane, a gdy `akcja` będzie nadal w fazie oczekiwania to hook zwróci również wartość `pending`.

### React DOM: `<form>` Actions

Formularze w wersji React 19 wspierają bezpośrednio możliwość przekazywania funkcji jako `akcji` jako prop dla elementów `<form>`, `<input>` i `<button>` aby skorzystać z wysyłania formularza razem z akcją możemy zapisać formularz w następujący sposób:

```tsx
<form action={actionFunction}>
```

Działanie:
W momencie kiedy `Akcja` formularza zostanie wykonana w poprawny sposób to React automatycznie zresetuje formularz dla komponentów które nie są przez niego kontrolowane, ale oczekuję od niego wyniku jakiś danych.

Możliwe jest zresetowanie formularza manualnie za pomocą `requestFormReset`.

### React DOM: Hook useFormStatus

Hook `useFormStatus` jest providerem pozwalającym za subskrybować jego context. Działa podobnie do hooka `useContext` tyle, że `useFormStatus` został zaprojektowany specjalnie do wykorzystania wraz z formularzami.

```tsx
import { useFormStatus } from 'react-dom';

function DesignButton() {
  const { pending } = useFormStatus();
  return (
    <button
      type='submit'
      disabled={pending}
    />
  );
}
```

Działanie:
`useFormStatus` czyta aktualny status rodzica elementu `<form>`

### Hook useOptimistic

Aby usprawnić obsłużenie powtarzającego się wzorca UI, który polega na przedstawieniu już zmienionych danych podczas gdy odpowiedź z API/Serwera jeszcze nie została przetworzona.
Możemy stworzyć makietę danych za pomocą nowego hooka `useOptimistic`

```tsx
function ChangeName({ currentName, onUpdateName }) {
  const [optimisticName, setOptimisticName] = useOptimistic(currentName);

  const submitAction = async (formData) => {
    const newName = formData.get('name');
    setOptimisticName(newName);
    const updatedName = await updateName(newName);
    onUpdateName(updatedName);
  };

  return (
    <form action={submitAction}>
      <p>Your name is: {optimisticName}</p>
      <p>
        <label>Change Name:</label>
        <input
          type='text'
          name='name'
          disabled={currentName !== optimisticName}
        />
      </p>
    </form>
  );
}
```

Działanie:
W momencie gdy zapytanie `updateName` jest w procesie hook `useOptimistic` przejmuje dane jakie zostały przekazane przez użytkownika i aktualizuje formularz, pokazując makietę jak będzie wyglądać ostateczny wynik.

Jeżeli wystąpi jakiś błąd zostanie przywrócona poprzednia wartość.

### API: `use`

W wydaniu React 19 można skorzystać z nowego API `use` aby mieć dostęp do odczytu danych podczas renderu.

Przykładem może być odczytywanie obietnic (promise) za pomocą `use`. React w takim wypadku zatrzyma proces renderowania aż do momentu rozwiązania obietnicy.

```tsx
import { use } from 'react';

function Comments({ commentsPromise }) {
  // `use` will suspend until the promise resolves.
  const comments = use(commentsPromise);
  return comments.map((comment) => <p key={comment.id}>{comment}</p>);
}

function Page({ commentsPromise }) {
  // When `use` suspends in Comments,
  // this Suspense boundary will be shown.
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <Comments commentsPromise={commentsPromise} />
    </Suspense>
  );
}
```

### <span style="color:red">**Uwaga**</span>

`use` nie wspiera obietnic które zostały wytworzene podczas fazy render
Przy próbie dostania się do obietnicy która została wytworzona w fazie `render`, React zwróci nam taki warning w konsoli:

> A component was suspended by an uncached promise. Creating promises inside a Client Component or hook is not yet supported, except via a Suspense-compatible library or framework.

Aby naprawić taki błąd należy przekazać obietnicę z biblioteki/framoworka opartego na `suspense` do takiego, który obsługuje buforowanie obietnic.

> W przyszłości przerzucnie się pomiędzy bibliotekami ma zostać zastąpione specjalnie do tego przystosowanym rozwiązaniem

---

`Use` może zostać również wykorzystany do czytania `contextu`, pozwala nam to odczytać `Context` warunkowo

```tsx
import { use } from 'react';
import ThemeContext from './ThemeContext';

function Heading({ children }) {
  if (children == null) {
    return null;
  }

  // This would not work with useContext
  // because of the early return.
  const theme = use(ThemeContext);
  return <h1 style={{ color: theme.color }}>{children}</h1>;
}
```

API `use` może być tylko wywołane w fazie renderu, podobnie jak inne hooki.
Jednak róznicą `use` od hooków jest to, że może być wywołane warunkowo.

> W przyszłości mają powstać kolejne warunki umożliwiające dostęp danych w fazie render

### React DOM Static APIs

Do Reacta zostały dodane dwa nowe API to `react-dom/static` dla statycznego generowania strony

- `prerender`
- `prerenderToNodeStream`

Oba wymienione powyżej API poprawiają `renderToString` poprzez czekanie na dane do załadowania w statycznym HTML.
API zostało zaprojketowane aby działać wraz z środowiskiem jak `Node.js Streams` oraz `Web Streams`

```tsx
import { prerender } from 'react-dom/static';

async function handler(request) {
  const { prelude } = await prerender(<App />, {
    bootstrapScripts: ['/main.js'],
  });
  return new Response(prelude, {
    headers: { 'content-type': 'text/html' },
  });
}
```

`Prerender` API wstępnego renderowania będą czekać na załadowanie wszystkich danych przed zwróceniem statycznego HTML.
Strumienie można konwertować na ciągi znaków lub wysyłać z odpowiedzią strumieniową.
Nie obsługują strumieniowego przesyłania zawartości podczas jej ładowania, co jest obsługiwane przez istniejące interfejsy API renderujące serwera React DOM.

### React Server Components

React teraz ma wsparcie natywne dla Componentów serwerowych, co pozwala na renderowanie komponentów za wczasu, przed budowaniem w środowisku.

Ponieważ jak to serorwe komponentu są budowane od razu na serwerze, przed wysłaniem ich do klienta.

### Server Actions

Akcje serwerowe pozwalają komponentonm po stronie klienta wywoływać asynchroniczne funkcje, które są wykonywane po stronie serwera.

Kiedy akcje serwerowa jest zdefiniowana z użyciem dyrektywy `"use server"` framework automatycznie towrzy referencję do akcji serwerowej i przekaże ją do komponentu klienckiego.
W momencie wywołania funkcji, React wysyła zapytanie do serwera aby serwer uruchomił daną funkjcę oraz zwrócił jej wynik.

### <span style="color:red">**Uwaga**</span>

**Nie istnieje specjalna dyrektywa aby component był po stronie serwera `use server` jest wykorzystywane do tworzenia akcji serwerowych** - React.js

W framework Next.js każdy komponent, który nie posiada żadnej dyretkywy zdefiniowanej na samej górze komponentu automatycznie tworzy taki komponent jako komponent po stronie serwerowej.

Akcje serwerowe mogą być tylko zrobione po stronie komponentu serwerowego i mogą zostać przekazane, za pomocą `props` do kompoentnu klienckiego.

---

## Usprawnienia w React 19

### ref jako prop

W React 19 teraz jest możliwość do dostania się do `ref` jako prop z funkcji komponentu

```tsx
function MyInput({ placeholder, ref }) {
  return (
    <input
      placeholder={placeholder}
      ref={ref}
    />
  );
}

//...

<MyInput ref={ref} />;
```

Dzięki temu zastosowaniu nowe komponentu, nie będę już potrzebowały `forwardRef`.

Aktualnie wykorzystanie `ref` jako `prop` automtycznie zaktualizuje komponenty aby korzystały z aktualnego stanu `ref`.

W przyszłości `forawardRef` będzie poddany wycofaniu.

### `<Context>` jako provider

W wersji React 19 możemy generować `<Context>` jako provider bezpośrednio zamiast wykortzystywać właściwość obiektu `<Context.Provider>`

```tsx
const ThemeContext = createContext('');

function App({ children }) {
  return <ThemeContext value='dark'>{children}</ThemeContext>;
}
```

### Funckja czyszcząca dla ref

Teraz element `ref` wspiera funkcję czyszczącą

```tsx
<input
  ref={(ref) => {
    // ref created

    // NEW: return a cleanup function to reset
    // the ref when element is removed from DOM.
    return () => {
      // ref cleanup
    };
  }}
/>
```

W momencie kiedy komponent jest ususuwany z drzewa React, funkcja czyszcząca zostanie zwrócona.
Działa to dla DOM refs, refs w klasach komponentów oraz `useImperativeHandle`.

Poprzez zaimplementowanie funkcji czysczących wewnątrz `ref` funkcja callback teraz będzie odrzucana poprzez TypeScript.
Aby rozwiązać ten błąd należy stosować taką składnię:

```tsx
<div
  ref={(current) => {
    instance = current;
  }}
/>
```

### Wartość domyślna dla `useDeferredValue`

Została dodana możliwość dodania warości domyślanej do `useDeferredValue`

```tsx
function Search({ deferredValue }) {
  // On initial render the value is ''.
  // Then a re-render is scheduled with the deferredValue.
  const value = useDeferredValue(deferredValue, '');

  return <Results query={value} />;
}
```

W momencie kiedy wartość domyślna jest przekazana, `useDeferredValue` zwróci ją jako wartość jako wartość domyślna dla renderu komponentu i
planuje kolejny re-render w tle z `defferedValue`.

### Wsparcie dla Metadanych zawartych w dokumencie

W kodzie `HTML` znaczniki oznaczające metadane takie jak `<title></title>`, `<link>` i `<meta>` są zarezerwowane aby były umieszczane wewnątrz sekcji `<head>` dokumentu.
W Reactcie komponenty które potrzebują takich znaczników mogą być umieszczone bardzo daleko od sekcji `<head>` lub aplikacja sama w sobie nie posiada takiej sekcji.

Dlatego w wersji 19 została dodana obsługa renderowania znaczników dla metadanych natycwnie:

```tsx
function BlogPost({ post }) {
  return (
    <article>
      <h1>{post.title}</h1>
      <title>{post.title}</title>
      <meta
        name='author'
        content='Josh'
      />
      <link
        rel='author'
        href='https://twitter.com/joshcstory/'
      />
      <meta
        name='keywords'
        content={post.keywords}
      />
      <p>Eee equals em-see-squared...</p>
    </article>
  );
}
```

### Wsparcie dla stylesheet
