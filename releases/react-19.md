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
