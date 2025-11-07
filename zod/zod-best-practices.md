# Zod Best Practices

## Overview

Problem: How to leverage TypeScript, which is good for checking compile-time errors, to validate form input and 3rd-party API responses without having to write too much code?

Of course, devs can read docs but in the long run it will become too much of a hassle. They will need a library that can:

- Define the form/API structure by writing dead simple code.
- Cover runtime validation.
- Cover compile-time validation.

=> Zod.

## Cheatsheet

### Basic API

```ts
import { z } from "zod";

// convention: camelCase for schema, PascalCase for type
const playerSchema = z.object({
  username: z.string(),
  xp: z.number(),
});

// return a strongly-typed deep clone of the input
const validPlayer = Player.parse({ username: "billie", xp: 100 });
console.log(validPlayer);
// { username: "billie", xp: 100 }

// error handling
try {
  Player.parse({ username: 47, xp: "100" });
} catch (error) {
  if (error instanceof z.ZodError) {
    console.log(error.issues);
    /* [
      {
        expected: 'string',
        code: 'invalid_type',
        path: [ 'username' ],
        message: 'Invalid input: expected string'
      },
      {
        expected: 'number',
        code: 'invalid_type',
        path: [ 'xp' ],
        message: 'Invalid input: expected number'
      }
    ] */
  }
}

// list of zod issue code
export declare const ZodIssueCode: {
  invalid_type: "invalid_type";
  invalid_literal: "invalid_literal";
  custom: "custom";
  invalid_union: "invalid_union";
  invalid_union_discriminator: "invalid_union_discriminator";
  invalid_enum_value: "invalid_enum_value";
  unrecognized_keys: "unrecognized_keys";
  invalid_arguments: "invalid_arguments";
  invalid_return_type: "invalid_return_type";
  invalid_date: "invalid_date";
  invalid_string: "invalid_string";
  too_small: "too_small";
  too_big: "too_big";
  invalid_intersection_types: "invalid_intersection_types";
  not_multiple_of: "not_multiple_of";
  not_finite: "not_finite";
};

const result = Player.safeParse({ username: 47, xp: "100" });
// result type: Discrimination Union
if (!result.success) {
  console.log(result.error); // ZodError instance
} else {
  console.log(result.data); // { username: string, xp: number }
}

type Player = z.infer<typeof PlayerSchema>;
const player: Player = { username: "johnwick", xp: 1111111 };

// due to .transform() API, input and output type can diverge
const mySchema = z.string().transform((val) => val.length);
const MySchemaInput = z.input<typeof mySchema>;
const MySchemaOutput = z.output<typeof mySchema>;

// =============================================================================
// COMMON TYPES
// =============================================================================

z.string();
z.number();
z.bigint(); // API same as number()
z.boolean();
z.symbol();
z.undefined();
z.optional(/* ... */);
z.nullable(/* ... */);
z.null(); // alias
z.nullish(/* ... */); // both optional and nullable
z.unknown(); // type inferance
z.any(); // type inferance
z.never(); // type inferance
z.string().default("foo");
z.number().default(Math.random); // auto-generated default value
z.number().catch(47); // parse(4) => 4, parse("foobar") => 47
z.object({
  /* ... */
}).readonly();
z.json(); // validate JSON-encoded string

// =============================================================================
// OBJECT
// =============================================================================

const personSchema = z.object({
  name: z.string(),
  age: z.number(),
});
const animalSchema = z.object({
  name: z.string(),
  age: z.number().optional(),
});
console.log(animalSchema.shape.name);
console.log(animalSchema.shape.age);
const keySchema = animalSchema.keyof();
const dogSchema = animalSchema.extend({
  isNeutered: z.boolean(),
  // CAUTION: can override existing field
});

// Pro: avoid library-specific APIs and less extensive
const alterDogSchema = z.object({
  ...animalSchema.shape, // spread operator, a language-level feature
  isNeutered: z.boolean(),
});
const alterDogSchema2 = z.object({
  ...animalSchema.shape,
  ...mechaSchema.shape,
  isNeutered: z.boolean(), // Neutering a mecha dog?
});

// TS Pick/Omit utility type
const justTheName = personSchema.pick({ name: true });
const noAge = personSchema.omit({ age: true });

// TS Partial utility type
const partialPersonSchema = personSchema.partial();
// type: { name?: string; age?: number }
const ageOptionalPersonSchema = personSchema.partial({
  age: true,
});
// type: { name: string; age?: number }

// TS Required utility type
const requiredAllPersonSchema = personSchema.required();
const requiredNamePersonSchema = personSchema.required({
  name: true,
});

// =============================================================================
// ARRAY
// =============================================================================

z.array(z.string());
z.array(z.string()).min(1); // must contain 5 or more items
z.array(z.string()).max(10); // must contain 5 or fewer items
z.array(z.string()).length(5); // must contain 5 items exactly
```

### Advanced API

```ts
// =============================================================================
// TUPLE
// =============================================================================

const myTupleSchema = z.tuple([
  z.string(),
  z.number(),
  z.boolean()
]);

type MyTuple = z.infer<typeof MyTuple>;

// =============================================================================
// UNION
// =============================================================================

const strOrNum = z.union([z.string(), z.number()]); // string | number
strOrNum.parse("food"); // passed
strOrNum.parse(14);     // passed

// =============================================================================
// DISCRIMINATED UNION
// =============================================================================

type MyResult =
  | { status: "success"; data: string }
  | { status: "failed"; error: string }

function handleResult(result: MyResult) {
  if (result.status === "success") {
    console.log(result.data); // string
  } else {
    console.log(result.error); // string
  }
}

const myResultSchema = z.discriminatedUnion("status", [
  z.object({ status: z.literal("success"), data: z.string() }),
  z.object({ status: z.literal("failed"), error: z.string() }),
]);

myResultSchema.parse({
  status: "error",
  error: "ERROR_CODE"
});

// =============================================================================
// RECORD
// =============================================================================

// TS Record<string, number> utility type
const idCacheSchema = z.record(z.string(), z.string());
type IdCache = z.infer<typeof idCacheSchema>; // Record<string, string>

idCacheSchema.parse({
  foo: "bar",
  marco: "polo",
});

z.record(z.union([z.string(), z.number(), z.symbol()]), z.unknown());
// Record<string | number | symbol, unknown>

// create object schemas containing keys defined in an enum
// NOTE: less common
z.record(z.enum(["id", "name", "email"]), z.string());
// { id: string, name: string, email: string }

const numberSetSchema = z.set(z.number());
type NumberSet = z.infer<typeof numberSetSchema>;

// =============================================================================
// FILES
// =============================================================================

const fileSchema = z.file();
fileSchema.min(10_000); // minimum .size (bytes)
fileSchema.max(1_000_000); // maximum .size (bytes)
fileSchema.mime("image/png"); // MIME type
fileSchema.mime(["image/png", "image/jpeg"]); // multiple MIME types

// =============================================================================
// INSTANCEOF
// =============================================================================

class Test {
  name: string;
}
const TestSchema = z.instanceof(Test);
TestSchema.parse(new Test()); // passed

const blobSchema = z.instanceof(URL).check(
  z.property("protocol", z.literal("https:" as string, "Only HTTPS allowed"))
);

// =============================================================================
// COERCION
// =============================================================================

// It's a little like implicit transformation
z.coerce.string();  // String(input)
z.coerce.number();  // Number(input)
z.coerce.boolean(); // Boolean(input)
z.coerce.bigint();  // BigInt(input)

const schema = z.coerce.string();
schema.parse("johnwick"); // => "johnwick"
schema.parse("47");       // => "47"
schema.parse(true);       // => "true"
schema.parse(null):       // => "null"

const A = z.coerce.number();
const AInput = z.infer<typeof A>; // "unknown"

const B = z.coerce.number<number>();
const BInput = z.infer<typeof B>; // "number"

// =============================================================================
// LITERALS
// =============================================================================

const tunaSchema = z.literal("tuna");
const twelveSchema = z.literal(12);
const colors = z.literal(["red", "green", "blue"]);
console.log(colors.values);
colors.parse("green"); // OK
colors.parse("yellow"); // NOT OK

z.null();
z.undefined();
z.void(); // similar to z.undefined()

// =============================================================================
// STRING
// =============================================================================

z.string().max(10);
z.string().min(1);
z.string().length(5);
z.string().regex(/[a-z]+$/);
z.string().startsWith("aaa");
z.string().endsWith("zzz");
z.string().includes("---");
z.string().uppercase(); // "abcdef" => passed
z.string().lowercase(); // "ABCDEF" => passed

// transformation API
z.string().trim(); // remove whitespace
z.string().toLowerCase();
z.string().toUpperCase();
z.string().normalize(); // unicode character normalized to UTF-8

z.email();      // Gmail regex: /^(?!\.)(?!.*\.\.)([a-z0-9_'+\-\.]*)[a-z0-9_+-]@([a-z0-9][a-z0-9\-]*\.)+[a-z]{2,}$/i
z.email({ pattern: z.regexes.rfc5322Email });
z.email({ pattern: z.regexes.html5Email });
z.email({ pattern: z.regexes.unicodeEmail });

z.uuid();       // any UUID version
z.uuid({ version: "v4" /* v1,2,3,4,5,6,7,8 */ }); // specific UUID version
z.uuidv4();
z.uuidv6();
z.uuidv7();
z.guid();       // "g", not "u". validate any UUID-like identifier

z.url();        // WHATWG-compatible URL
z.url({ hostname: /^example\.com$/, protocol: /^https$/, normalize: true });  // custom hostname and protocol, enable normalization

z.iso.date();   // YYYY-MM-DD
z.iso.time();   // HH:mm[:SS[.s+]]
z.iso.time({ precision: -1 });      // HH:mm
z.iso.time({ precision: 0 });       // HH:mm:SS (second precision)
z.iso.time({ precision: 1 });       // HH:mm:SS.s (decisecond precision)
z.iso.time({ precision: 2 });       // HH:MM:SS.ss (centisecond precision)
z.iso.time({ precision: 3 });       // HH:MM:SS.sss (millisecond precision)
z.iso.datetime(); // YYYY-MM-DDTHH:mm:SS[.s+]Z (ISO 8601)
z.iso.duration(); // PnYnMnDTnHnMnS (less common)

z.ipv4();   // 192.168.0.0
z.ipv6();   // 2001:db8:85a3::8a2e:370:7334
z.cidrv4(); // 192.168.1.0/24
z.cidrv6(); // 2001:db8::/32

z.jwt();
z.jwt({ alg: "HS256" });

z.hostname();
z.emoji();
z.base64();
z.base64url();
z.ulid();
z.cuid();
z.cuid2();
z.nanoid();

// =============================================================================
// INTEGERS
// =============================================================================

z.number().lt|lte|gt|gte
z.int();
z.min(1);
z.max(10);

// =============================================================================
// ENUMS
// =============================================================================

const fistSchema = z.enum(["Salmon", "Tuna", "Trout"]);
// Enum-like object literal
// More specifically: { [key: string]: string | number }
const fishSchema = z.enum({
  Salmon: "Salmon",
  Tuna: "Tuna"
} as const);
console.log(fishSchema.enum);

// very similar to
const colorSchema = z.literal(["red", "green", "blue"]);
console.log(colorSchema.values);

// =============================================================================
// JS DATA OBJECT
// =============================================================================

z.date().safeParse(new Date()); // validate JS object Date()
z.date().safeParse("1970-01-01T00:00:00.000Z");
z.date().min(new Date("2000-01-01"), { error: "Too old!" });
z.date().max(new Date(), { error: "Too long!" });

// =============================================================================
// Refinements
// =============================================================================

// custom validation logic
const myStringSchema = z
  .string()
  .refine((val) => val.length <= 255);
// non-continuable if abort: true
const myStringSchema = z
  .string()
  .refine((val) => val.length > 255, { error: "Too short!", abort: true });
  .refine((val) => val === val.toLowerCase(), { error: "Must be lowercase!", abort: true });
const result = myStringSchema.safeParse("ABCDEF");
if (!result.success) {
  console.log(result.error.issues);
}
/* [
  { "code": "custom", "message": "Too short!" },
  { "code": "custom", "message": "Must be lowercase" }
] */

const userId = z
  .string()
  .refine(async (id) => { /* verify that ID exists in DB */ return true;})
const result = await userId.parseAsync("abc123"); // parseAsync() API!

// =============================================================================
// Transforms
// =============================================================================

const castToString = z.transform((val) => String(val)); // NOTE: it's better to use coercion
const stringToLength = z
  .string()
  .transform((val) => val.length);
const idToUser = z
  .string()
  .transform(async (id) => {
    // fetch user from DB
    return user;
  });
const user = await idToUser.parseAsync("abc123");
```

```ts
import { z } from "zod";

// Schema-level error messages
const errMsgs = {
  required_error: "This field is required",
  invalid_type_error: "Incorrect type",
} as const;
// 'as const': make property 'required_error' and 'invalid_type_error' a read-only property whose type is the literal string "This field is required" and "Incorrect type".

const spread = <T>(value: { [key: string]: T }) =>
  [...Object.keys(value)] as const as readonly [T, ...T[]];

// Base Entity
export const baseEntitySchema = z.object({
  id: z.string(errMsgs),
  // createdAt: z.number(),
  // updatedAt: z.number(),
});
export type BaseEntity = z.infer<typeof baseEntitySchema>;

export const omitBaseEntity = {
  id: true,
  createdAt:
} as const satisfies {
  [key in keyof BaseEntity]: boolean;
};
// 'satisfies': ensure the object matches the shape you expect - an object with the same key as BaseEntity whose value type is boolean
// NOTE: 'satisfies' just checks compatibility, unlike `as Type` type assertion that change the inferred type.

const FormSchema = z.object({
  name: z.string(),
  phoneNumber: z.optional(z.string()),
  keywords: z.array(z.string()).default([]),
});

// TL;DR: type created with Entity<T> have both the fields from T and the common fields from BaseEntity
export type Entity<T> = {
  [K in keyof T]: T[K];
} & BaseEntity;

// ========================================================================== //

const imageSchema = z
  .instanceof(FileList)
  .refine((files) => )


const toString = (num: unknown) => {
  const parsed = IntegerSchema.parse(num); // throw error if validation fails
  const safeParsed = IntegerSchema.safeParse(num); // NOT throw error if validation fails
  return String(num);
};

// TESTS
// define unit test case
// 1st param: a stirng, test case description
// 2nd param: callback fn
it("Should throw a runtime error when called with not a number", () => {
  // create assertion object for the value/fn. Here a fn is asserted that the fn throws an error matching the provided message
  // TIPS: when testing knowing errors will be thrown, pass a fn to expect(), not the result of the fn call, i.e. () => toString("123"), NOT toString("123")
  expect(() => toString("123")).toThrowError(
    "Expected number, received string" /* Zod message */
  );

  // add more expect...
});

it("Should return a string after calling toString()", () => {
  expect(toString(1)).toBeTypeOf("string");
});

// ========================================================================== //

const PersonResult = z.object({});

const PeopleResults = z.object({
  results: z.array(PersonResult),
});

type PeopleType = z.infer<typeof PeopleResults>;

export const fetchPersonName = async (id: string) => {
  const response = await fetch(
    // NOTE: never pass raw id like this into URL, this is just tutorial
    "https://www.totaltypescript.com/swapi/people/" + id + ".json"
  );
  const result = await response.json();
  const parsedData = PersonResult.parse(result);
  const { name } = parsedData;
  return name;
};

export const fetchStarWarsPeople = async () => {
  const response = await fetch(
    // NOTE: never pass raw id like this into URL, this is just tutorial
    "https://www.totaltypescript.com/swapi/people.json"
  );
  const result = await response.json();
  const parsedData = PeopleResults.parse(result);
  const { results } = parsedData;
  return results;
};

const logPeopleResults = (data: PeopleType) => {
  data.results.forEach((person) => {
    console.log(`Name: ${person.name}`);
  });
};

it("Should return the name", async () => {
  expect(await fetchPersonName("1")).toEqual("Luke Skywalker");
  expect(await fetchPersonName("2")).toEqual("C-3PO");
});

it("Should return the name #2", async () => {
  expect((await fetchStarWarsPeople())[0]).toEqual({
    name: "Luke Skywalker",
  });
});

// ========================================================================== //
```
