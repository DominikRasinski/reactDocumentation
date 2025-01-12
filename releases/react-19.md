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
  null,
);
```

Hook `useActionState` przyjmuje jako argument funkcję (`Akcję`) opakowuję podaną funkcję i zwraca ją z możliwością wykonania.
Mechanizm działa ze względu na kompozycję akcji.

Działanie: opakowana `Akcja` jak zostanie wywołana to hook `useActionState` zwróci ostatnią wartość `akcji` jako dane, a gdy `akcja` będzie nadal w fazie oczekiwania to hook zwróci również wartość `pending`.

## React DOM: `<form>` Actions

Formularze w wersji React 19 wspierają bezpośrednio możliwość przekazywania funkcji jako `akcji` jako prop dla elementów `<form>`, `<input>` i `<button>` aby skorzystać z wysyłania formularza razem z akcją możemy zapisać formularz w następujący sposób:

```tsx
<form action={actionFunction}>
```

Działanie:
W momencie kiedy `Akcja` formularza zostanie wykonana w poprawny sposób to React automatycznie zresetuje formularz dla komponentów które nie są przez niego kontrolowane, ale oczekuję od niego wyniku jakiś danych.

Możliwe jest zresetowanie formularza manualnie za pomocą `requestFormReset`.

## React DOM: Hook useFormStatus

Hook `useFormStatus` jest providerem pozwalającym za subskrybować jego context. Działa podobnie do hooka `useContext` tyle, że `useFormStatus` został zaprojektowany specjalnie do wykorzystania wraz z formularzami.

```tsx
import {useFormStatus} from 'react-dom';

function DesignButton() {
  const {pending} = useFormStatus();
  return <button type="submit" disabled={pending} />
}
```

Działanie:
`useFormStatus` czyta aktualny status rodzica elementu `<form>`

## Hook useOptimistic

Aby usprawnić obsłużenie powtarzającego się wzorca UI, który polega na przedstawieniu już zmienionych danych podczas gdy odpowiedź z API/Serwera jeszcze nie została przetworzona.
Możemy stworzyć makietę danych za pomocą nowego hooka `useOptimistic`

```tsx
function ChangeName({currentName, onUpdateName}) {
  const [optimisticName, setOptimisticName] = useOptimistic(currentName);

  const submitAction = async formData => {
    const newName = formData.get("name");
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
          type="text"
          name="name"
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

## API: `use`