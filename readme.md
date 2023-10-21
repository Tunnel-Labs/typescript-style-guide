# Tunnel's TypeScript Style Guide

## Rules

1. Top-level interfaces and types must use `PascalCase` identifiers.
2. Generic type parameters and temporary types must use `PascalCase` identifiers and start with a dollar sign (`$`).
3. Lines must either start with an identifier or only contain symbols.
4. The indentation of an `extends` conditional branch should follow the format similar to a switch statement.

## Examples

### Simple `extends`

```typescript
export type SelectionItem<$Selection> =
  NonNullable<$Selection> extends Array<infer $Item> ?
    NonNullable<$Item> :
  NonNullable<$Selection>;
```

### Mapped Type

```typescript
import type { Clientref } from '@-/client-ref';

export type DeepExpandSelection<$Collections, $FlattenedSelection> = {
  [
    $Key in keyof $FlattenedSelection as
      $Key extends | `${infer $Property}Ref` | `${infer $Property}Refs` ?
        $Property :
      $Key extends '__type__' ?
        never :
      $Key
  ]:
    NonNullable<$FlattenedSelection[$Key]> extends ClientRef<
      $Collections,
      infer $CollectionKey
    > ?
      DeepExpandSelection<
        $Collections,
        $Collections[$CollectionKey][string]
      > |
      (null extends $FlattenedSelection[$Key] ? null : never) :

    $FlattenedSelection[$Key] extends ClientRef<$Collections, infer $CollectionKey>[] ?
      DeepExpandSelection<
        $Collections,
        $Collections[$CollectionKey][string]
      >[] :

    $FlattenedSelection[$Key];
};
```
