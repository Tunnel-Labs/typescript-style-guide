# Tunnel's TypeScript Style Guide

## Rules

1. Top-level interfaces and types must use `PascalCase` identifiers.
2. Generic type parameters and temporary types must use `PascalCase` identifiers and start with a dollar sign (`$`).
3. If a complex type is split across multiple lines, each of these lines be one of the following:
   - A line starting with an identifier
   - A line containing an entire subtype
   - A line consisting of only spaces and/or symbols (i.e. non alpha-numeric characters)
4. The indentation of an `extends` conditional branch should follow the format similar to a switch statement.

## Examples

### Simple `extends`

```typescript
export type SelectionItem<$Selection> =
  NonNullable<$Selection> extends Array<infer $Item> ?
    NonNullable<$Item> :
  NonNullable<$Selection>;
```

```typescript
// prettier-ignore
export type TunnelErrorDefinition<
  $Message extends string,
  $SubstitutionSchemas extends Record<string, ZodSchema>
> =
  (
    IsEmptyObject<z.infer<ZodObject<$SubstitutionSchemas>>> extends true ?
      () => TunnelError<ErrorSchema<$Message, $SubstitutionSchemas>> :
    (substitutions: z.infer<ZodObject<$SubstitutionSchemas>>) => TunnelError<ErrorSchema<$Message, $SubstitutionSchemas>>
  ) &
  { key: string };
```

### Mapped Types

```typescript
import type { ClientRef } from '@-/client-ref';

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

```typescript
import type { ClientRef } from '@-/client-ref';
import type { SelectionItem } from '~/types/item.ts';

/**
  Allows a type to accept a `ClientRef` instead of a nested selection object
*/
// prettier-ignore
export type DeepRefable<$Collections, $Selection> =
  {
    [
      $Key in keyof $Selection as
        $Key extends '__type__' ?
          never :
        $Key
    ]:
      NonNullable<$Selection[$Key]> extends { __type__?: any } ?
        DeepRefable<$Collections, $Selection[$Key]> |
        ClientRef<
          $Collections,
          NonNullable<
            NonNullable<NonNullable<$Selection[$Key]>['__type__']>['__name__']
          >
        > |
        (null extends $Selection[$Key] ? null : never) :
      $Selection[$Key] extends { __type__?: any }[] ?
        Array<
          DeepRefable<$Collections, SelectionItem<$Selection[$Key]>> |
          ClientRef<
            $Collections,
            NonNullable<
              NonNullable<
                NonNullable<$Selection[$Key][number]>['__type__']
              >['__name__']
            >
          > |
          (null extends $Selection[$Key][number] ? null : never)
        > :
      $Selection[$Key];
  } &
  { __selection__?: $Selection };
```
