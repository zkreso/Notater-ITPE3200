# Reactive programming

Angular uses reactive programming heavily.

### Hot vs Cold Observable

A cold observable is if the data is produced inside the observable, while a hot observable if the data is produced outside the observable.

**Hot observable**

```
import * as Rx from "rxjs";

const random = Math.random()

const observable = Rx.Observable.create((observer) => {
    observer.next(random);
});

// subscription 1
observable.subscribe((data) => {
  console.log(data); // 0.11208711666917925 (random number)
});

// subscription 2
observable.subscribe((data) => {
   console.log(data); // 0.11208711666917925 (random number)
});
```

The random number is produced "outside" the observable. Calling next will just give us that number until it changes by some external force.

**Cold observable:**

```ts
import * as Rx from "rxjs";

const observable = Rx.Observable.create((observer) => {
    observer.next(Math.random());
});

// subscription 1
observable.subscribe((data) => {
  console.log(data); // 0.24957144215097515 (random number)
});

// subscription 2
observable.subscribe((data) => {
   console.log(data); // 0.004617340049055896 (random number)
});
```

The random number is created "inside" the observable. Calling next will create new data every time. In a cold observable, the observable calls the producer to change the data. In a hot observable, something else is responsible for when to change the data. An observable just returns the current state of the data.

It's usually fine for an observable to be either. It really depends on when you want the state of the data to change.

### RxJs operator example

```ts
productSuppliers$ = this.selectedProducts$.pipe(
  switchMap(product =>
	    forkJoin(product.supplierIds.map(supplierId =>
		    this.http.get<Supplier>(´${this.sUrl}/${supplierId}`)))
  ));
```

Every time product changes, do switchMap on inner observable. Inner observable is again the inner observable of forkJoin. forkJoin's inner observable is the stream of that particular products supplier id's (product.supplierIds) mapped on to a http request. forkJoin only works on completed observables, which http request is.

Going backwards:

We do get requests for each supplierId. forkJoin joins these as an array. there is an array for each product. switchMap turns the product id's into an observable of these arrays
