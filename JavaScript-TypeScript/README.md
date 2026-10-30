# JavaScript, TypeScript & RxJS Interview Questions

## 1. JavaScript Fundamentals

### Q1. JavaScript কী?
JavaScript হলো web-এর প্রধান programming language; browser এবং Node.js-এর মতো runtime-এ চলে।

### Q2. `var`, `let`, `const`-এর পার্থক্য?
- `var`: function-scoped; redeclare ও reassign করা যায়।
- `let`: block-scoped; reassign করা যায়, একই scope-এ redeclare করা যায় না।
- `const`: block-scoped; binding reassign করা যায় না। Object-এর property অবশ্য পরিবর্তনযোগ্য হতে পারে।

### Q3. `==` ও `===`?
`==` type coercion করতে পারে; `===` type coercion ছাড়া strict comparison করে। সাধারণত `===` ব্যবহার করো।

### Q4. `null` ও `undefined`?
`undefined` সাধারণত value assign না হওয়া/অনুপস্থিত value; `null` হলো intentional empty value।

### Q5. Hoisting কী?
Execution-এর আগে declaration processing-এর আচরণ। `var` declaration hoist হয়ে `undefined` হয়; `let`/`const` declaration-এর আগে temporal dead zone-এ থাকে।

### Q6. Closure কী?
Inner function outer lexical scope-এর variable access করতে পারে, outer function শেষ হওয়ার পরও।

```javascript
function createCounter() {
  let count = 0;
  return () => ++count;
}

const counter = createCounter();
console.log(counter()); // 1
```

### Q7. Promise কী?
Asynchronous operation-এর eventual success/failure represent করে। State: pending, fulfilled, rejected।

### Q8. `async/await` কী?
Promise-ভিত্তিক asynchronous code লেখার readable syntax।

```javascript
async function loadUsers() {
  const response = await fetch('/api/users');
  if (!response.ok) throw new Error('Request failed');
  return response.json();
}
```

### Q9. Event Loop কী?
JavaScript runtime-এর mechanism, যা call stack, task queue এবং microtask queue সমন্বয় করে asynchronous callback execute করে।

## 2. TypeScript

### Q10. TypeScript কী?
JavaScript-এর ওপর static type system যোগ করে। TypeScript compile/transpile হয়ে JavaScript হয়।

### Q11. `interface` ও `type`?
দুটিই type shape প্রকাশ করতে পারে। Interface declaration merging ও extension সমর্থন করে; type alias union, intersection, primitive aliasসহ আরও বিস্তৃত type expression করতে পারে।

### Q12. `any` ও `unknown`?
`any` type checking এড়িয়ে যায়। `unknown` value ব্যবহার করার আগে type narrow করতে বাধ্য করে—তাই `unknown` বেশি নিরাপদ।

### Q13. Union ও Intersection Type?

```typescript
type Id = number | string;
type Person = { name: string };
type Employee = Person & { employeeId: number };
```

Union মানে একাধিক type-এর যেকোনো একটি; intersection মানে একাধিক type-এর বৈশিষ্ট্য একত্রে।

### Q14. Generics কী?
Reusable, type-safe function/class/interface তৈরিতে ব্যবহার হয়।

```typescript
function identity<T>(value: T): T {
  return value;
}
```

### Q15. Optional chaining ও nullish coalescing?
- `?.` null/undefined হলে property access বন্ধ করে।
- `??` কেবল null/undefined হলে fallback দেয়; `||` falsy value-তেও fallback দেয়।

## 3. RxJS

### Q16. RxJS কী?
Reactive Extensions for JavaScript। Observable stream এবং operator দিয়ে asynchronous data/event পরিচালনা করে।

### Q17. Observable কী?
সময় অনুযায়ী শূন্য বা একাধিক value emit করতে পারে এমন stream। Subscribe করলে execution শুরু হতে পারে; Observable cold বা hot হতে পারে।

### Q18. Observable ও Promise-এর পার্থক্য?
- Promise একবার resolve/reject হয়।
- Observable একাধিক value emit করতে পারে।
- Observable unsubscribe করা যায়।
- Observable operator দিয়ে stream transform করা যায়।

### Q19. `subscribe()` কী?
Observable execute/consume করে next, error, complete notification গ্রহণ করে।

### Q20. `pipe()` কী?
একাধিক RxJS operator ধারাবাহিকভাবে প্রয়োগ করে।

### Q21. `map()` কী?
প্রতিটি emitted value transform করে।

### Q22. `switchMap()` কী?
নতুন source value এলে আগের inner subscription unsubscribe করে নতুন inner Observable-এ switch করে। Search autocomplete-এ কার্যকর।

### Q23. `mergeMap()` কী?
Inner Observable concurrentভাবে subscribe করে; result emission interleave হতে পারে।

### Q24. `concatMap()` কী?
Inner Observable একটির পর একটি sequentialভাবে চালায়।

### Q25. `exhaustMap()` কী?
একটি inner Observable চলার সময় নতুন source emission উপেক্ষা করে। Duplicate submit ঠেকাতে উপকারী হতে পারে।

### Q26. `debounceTime()` কী?
নির্দিষ্ট সময় নতুন emission না এলে সর্বশেষ value emit করে।

### Q27. `distinctUntilChanged()` কী?
আগের emitted value-এর সঙ্গে সমান হলে consecutive duplicate emission বাদ দেয়।

### Q28. `takeUntilDestroyed()` কী?
Angular-এর utility; associated Angular context destroy হলে Observable subscription complete করতে সাহায্য করে।

## RxJS Operator Quick Table

| Operator | ব্যবহার |
|---|---|
| `map` | Value transform |
| `filter` | Condition অনুযায়ী value রাখে |
| `tap` | Side effect / debugging |
| `switchMap` | Latest inner stream |
| `mergeMap` | Concurrent inner streams |
| `concatMap` | Sequential inner streams |
| `exhaustMap` | চলমান inner থাকলে নতুন emission ignore |
| `debounceTime` | Input pause-এর পর emit |
| `distinctUntilChanged` | Consecutive duplicate বাদ |
| `catchError` | Error handling |
| `finalize` | Complete/error/unsubscribe-এ cleanup |

## Revision Checklist

- [ ] Scope, hoisting, closure
- [ ] Promise, async/await, event loop
- [ ] `==` বনাম `===`
- [ ] TypeScript interface/type
- [ ] `any` বনাম `unknown`
- [ ] Union/intersection/generics
- [ ] Observable বনাম Promise
- [ ] switchMap/mergeMap/concatMap/exhaustMap
- [ ] debounceTime ও distinctUntilChanged
- [ ] Subscription cleanup
