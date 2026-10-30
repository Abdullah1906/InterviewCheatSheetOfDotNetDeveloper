# Angular 20 Interview Questions & Answers

**Standalone | Signals | Routing | Forms | HTTP | RxJS**

## 1. Angular Fundamentals

### Q1. Angular কী?
Angular হলো TypeScript-ভিত্তিক frontend framework। Component, dependency injection, router, forms এবং HTTP client দিয়ে scalable single-page application তৈরি করা যায়।

### Q2. Component কী?
Component হলো UI-এর একটি অংশ। এতে TypeScript class, template, styles এবং metadata থাকে।

### Q3. Standalone Component কী?
Standalone component সরাসরি অন্য component/route-এ import করা যায়; শুধু component declare করার জন্য NgModule বাধ্যতামূলক নয়।

```typescript
import { Component } from '@angular/core';

@Component({
  selector: 'app-home',
  standalone: true,
  template: `<h1>স্বাগতম</h1>`
})
export class HomeComponent {}
```

### Q4. Data Binding-এর ধরন কী?
- Interpolation: `{{ title }}`
- Property binding: `[value]="name"`
- Event binding: `(click)="save()"`
- Two-way binding: `[(ngModel)]="name"`

### Q5. `@Input` ও `@Output` কী?
Parent থেকে child-এ data পাঠাতে input; child থেকে parent-এ event পাঠাতে output ব্যবহার হয়। নতুন Angular-এ `input()` ও `output()` function API-ও আছে।

## 2. Lifecycle ও Dependency Injection

### Q6. Lifecycle hooks কী?
Component-এর lifecycle-এর নির্দিষ্ট সময়ে Angular hook call করে। গুরুত্বপূর্ণ hook: `ngOnChanges`, `ngOnInit`, `ngAfterViewInit`, `ngOnDestroy`।

### Q7. Service কেন ব্যবহার হয়?
Reusable business logic, API communication এবং shared functionality component থেকে আলাদা রাখতে service ব্যবহার হয়।

```typescript
import { Injectable, inject } from '@angular/core';
import { HttpClient } from '@angular/common/http';

@Injectable({ providedIn: 'root' })
export class EmployeeService {
  private http = inject(HttpClient);

  getAll() {
    return this.http.get<Employee[]>('/api/employees');
  }
}
```

### Q8. Angular Dependency Injection কী?
Angular injector provider configuration অনুযায়ী service বা dependency তৈরি ও সরবরাহ করে।

## 3. Routing ও Application Structure

### Q9. Angular Router কী?
SPA-তে URL অনুযায়ী component/view দেখায়, সাধারণত পুরো page reload ছাড়াই।

### Q10. Lazy Loading কী?
প্রয়োজনের সময় feature/component load করা। Initial JavaScript bundle ছোট রাখতে সাহায্য করে।

```typescript
export const routes: Routes = [
  {
    path: 'employees',
    loadComponent: () =>
      import('./employees/employees.component')
        .then(m => m.EmployeesComponent)
  }
];
```

### Q11. Route Guard কী?
Navigation allow/deny করার client-side mechanism। এটি UX-এর জন্য; backend authorization-এর বিকল্প নয়।

### Q12. Interceptor কী?
HTTP request/response pipeline-এ cross-cutting behavior যোগ করে—যেমন token header, logging, error handling।

## 4. Forms

### Q13. Template-driven ও Reactive Forms-এর পার্থক্য?
Template-driven form template-এ বেশি configuration রাখে; Reactive Forms TypeScript-এ explicit form model তৈরি করে এবং complex validation/dynamic form-এ সুবিধাজনক।

```typescript
form = new FormGroup({
  name: new FormControl('', { nonNullable: true, validators: [Validators.required] }),
  email: new FormControl('', [Validators.email])
});
```

### Q14. FormGroup, FormControl, FormArray?
- FormControl: একটি field।
- FormGroup: একাধিক control-এর group।
- FormArray: dynamic control collection।

## 5. Angular Signals ও Change Detection

### Q15. Signal কী?
Signal হলো reactive state primitive। Value পড়তে function-এর মতো call করতে হয়; update করলে dependent computation/UI update হতে পারে।

```typescript
count = signal(0);
doubleCount = computed(() => this.count() * 2);

increment() {
  this.count.update(value => value + 1);
}
```

### Q16. `computed()` কী?
অন্য signal-এর ওপর নির্ভরশীল read-only derived value তৈরি করে।

### Q17. `effect()` কী?
যে signal পড়া হয়েছে তার পরিবর্তনে side effect চালায়। Derived state তৈরির জন্য সাধারণত `computed()` ভালো।

### Q18. `@if`, `@for`, `@switch` কী?
Angular-এর built-in template control flow syntax।

```html
@if (isLoggedIn()) {
  <p>স্বাগতম</p>
} @else {
  <p>লগইন করুন</p>
}

@for (employee of employees(); track employee.id) {
  <p>{{ employee.name }}</p>
}
```

### Q19. `track` কেন লাগে?
List item-এর stable identity জানিয়ে DOM update দক্ষ করে। সম্ভব হলে unique ID track করো।

### Q20. OnPush কী?
একটি change detection strategy; নির্দিষ্ট input/event/async বা reactive update-এ subtree check সীমিত করে performance উন্নত করতে পারে।

## 6. Angular 20-এ API Integration

### Q21. HttpClient কীভাবে register করবে?

```typescript
bootstrapApplication(AppComponent, {
  providers: [provideHttpClient()]
});
```

### Q22. API call কীভাবে করবে?

```typescript
@Injectable({ providedIn: 'root' })
export class EmployeeService {
  private http = inject(HttpClient);

  getEmployees() {
    return this.http.get<Employee[]>('/api/employees');
  }
}
```

### Q23. HTTP Observable কীভাবে consume করবে?
`async` pipe, `toSignal`, বা subscription ব্যবহার করা যায়। Subscription করলে lifecycle অনুযায়ী cleanup নিশ্চিত করো।

### Q24. JWT Interceptor-এর কাজ কী?
API request-এ প্রয়োজন অনুযায়ী bearer token যোগ করে। Token storage strategy নিরাপদ হতে হবে; XSS ঝুঁকি বিবেচনা করতে হবে।

### Q25. Route guard কি API secure করে?
না। Browser-এর guard bypass করা যায়। Backend-এ প্রতিটি protected endpoint-এ authentication ও authorization enforce করতে হবে।

## 7. Angular Best Practices

- Feature-based folder structure
- Standalone APIs
- Lazy loading
- Reactive Forms for complex forms
- Signals for local reactive state
- RxJS for asynchronous streams
- Typed API models
- Global error handling
- Loading/error/empty states
- Backend authorization
- Production build ও bundle analysis

## Revision Checklist

- [ ] Component, template, binding
- [ ] Standalone architecture
- [ ] Lifecycle hooks
- [ ] DI ও services
- [ ] Routing, lazy loading, guards
- [ ] Interceptors
- [ ] Reactive Forms ও validators
- [ ] Signals, computed, effect
- [ ] Change detection ও OnPush
- [ ] HttpClient ও RxJS
