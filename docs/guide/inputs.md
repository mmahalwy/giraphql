---
name: Input Types
menu: Guide
---

# Input Objects

## Creating Input objects

Input objects can be created using `builder.inputType`.

```typescript
const GiraffeInput = builder.inputType('GiraffeInput', {
  fields: (t) => ({
    name: t.string({ required: true }),
    birthdate: t.string({ required: true }),
    height: t.float({ required: true }),
  }),
});

builder.mutationType({
  fields: (t) => ({
    createGiraffe: t.field({
      type: Giraffe,
      args: {
        input: t.arg({ type: GiraffeInput, required: true }),
      },
      resolve: (root, args) =>
        new Giraffe(args.input.name, new Date(args.input.birthdate), args.input.height),
    }),
  }),
});
```

## Recursive inputs

Types for recursive inputs get a slightly more complicated to implement because their types can't
easily be inferred. Referencing other input types works without any additional logic, as long as
there is no circular reference to the original type.

To build input types with recursive references you can use `builder.inputRef` along with a type or
interface that describes the fields of your input object. The builder will still ensure all the
types are correct, but needs to type definitions to help infer the correct values.

```typescript
interface RecursiveGiraffeInputShape {
  name: string;
  birthdate: string;
  height: number;
  friends?: RecursiveGiraffeInputShape[];
}

const RecursiveGiraffeInput = builder
  .inputRef<RecursiveGiraffeInputShape>('RecursiveGiraffeInput')
  .implement({
    fields: (t) => ({
      name: t.string({ required: true }),
      birthdate: t.string({ required: true }),
      height: t.float({ required: true }),
      friends: t.field({
        type: [RecursiveGiraffeInput],
      }),
    }),
  });

builder.mutationType({
  fields: (t) => ({
    createGiraffeWithFriends: t.field({
      type: [Giraffe],
      args: {
        input: t.arg({ type: RecursiveGiraffeInput, required: true }),
      },
      resolve: (root, args) => {
        const friends = (args.input.friends || []).map(
          (friend) =>
            new Giraffe(args.input.name, new Date(args.input.birthdate), args.input.height),
        );

        return [
          new Giraffe(args.input.name, new Date(args.input.birthdate), args.input.height),
          ...friends,
        ];
      },
    }),
  }),
});
```
