# Functional typescript

## State

The state of this library is purely experimental. I don't know if typescript
will support the vision I have - so this library is where I perform the
experiments.

## Vision

This is an attempt to implement functional design patterns in typescript,
particularly monadic binding.

Other libraries exist already, notably [fp-ts](https://github.com/gcanti/fp-ts),
It seems many of these libraries try to replicate the features of ML of Haskell
(e.g. data and functions are separated), but typescript applies some limitations
to this approach.

For example, you cannot have the compiler determine which function to call at
compile time. This means you have to be explicit about which version of `bind`,
`apply`, etc. to use.

So this library depends on classes to allow dynamic dispatch (i.e. the actual
function is evaluated at run-time).

It might also take advantage of the fact, that you can create new classes at
run-time. E.g. if a monad is implemented in a class, a monad transformer could
be a function that takes a monad class as input, and return a new class.

An example of the type of code I'd like to be able to write

```typescript
// Validates a field - may change the value, e.g. trim a string
const validateField = <
  T,
  TKey extends keyof T,
  E>(key: TKey, validator: (v: T[TKey]) => Either<E, T[TKey]) =>
  (o: T): Either<E, T> =>
  v(o[key]).map(newVal => ({ ...o, [key]: newVal }))

const validateNotEmpty = (s: string) => {
  const trimmed = s.trim();
  return trimmed ? Either.right(trimmed) : Either.left('EMPTY')
}

const validate = (input: Input) =>
  Either
    .right(input)
    .bind(validateField('firstName', validateNotEmpty))
    .bind(validateField('lastName', validateNotEmpty))

// Given an input, and a user, returns an
// updated user along with any relevant domain events
const updateUser = (input: Input) => (user: User) =>
  validate(input)
    .bind(i => ({ ...user, ...input }))
    .bind(x => Writer.of(x).tell({ type: "NAME_CHANGED" }))

const runUserUseCase = (id, useCase) => {
  // Imperative code, load the user with the id, call
  // the use case. If left - return the error - if
  // right, save the updated user, and publish any
  // domain events
  ...
}

// API layer
router.put("/users/:id", (req, res) => {
  const userId = req.params.id;
  const input = parseBodyTypeSafely(req.body)
  // The awaited data is the Either
  (await runUserUseCase(req.params.id, updateUser(input)))
    .ifLeft(err => res.status(400).send(generateResponse(err)))
    .ifRight(body => res.json(body)
})
```
