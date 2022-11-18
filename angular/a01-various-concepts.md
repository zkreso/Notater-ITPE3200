# Various Angular concepts

## Modules and Components

An application in Angular is built out of reusable **components** (like any other frontend JS framework). Each component consists of 4 files:
- **component-name.component.ts**, which contains the component class and related functionality
- **component-name.compnent.spec.ts**, which is used for writing units tests for that component
- **component-name.component.html**, which contains the html of the component
- **component-name.component.css**, which contains any component-specific css styles

Some of these are obviously optional and can be ignored or deleted.

A component must belong to a **module** in Angular. That is done by "declaring" the component (using the declare keyword) in the module. (A module can also contain some other things in addition to components). I think a component can only belong to one module. Modules are kind of similar to namespace in Java/C#. In any case, to use a component in a different module you need to import the module it belongs to and also make sure to use the exports keyword inside on the component.

Each Angular applicaton must have at least one module and one component, which is the root module and root component (usually named app). There is no maximum number, and it's up to the designer to decide how many modules and components they want. You could make the whole application in the root component. For larger applications, it's usually helpful to seperate different parts in to different components and modules. Some approaches that can be used:

- Have one module with all components
    - This can work for small applications or when trying out and learning about Angular. It's not at all scaleable, it's hard to test, and it loads everything immediately, so it's not recommended for real applications.
- Have one module for each component
    - This leads to a lot of modules. There isn't really any disadvantage except that you create a lot of boilerplate: Every component you want to use needs to be imported seperately. You also end up with a bunch of module files.
- Have one module per view
    - This is probably what is most commonly used. You group components related to a view in its own module. If there are some components that need to be shared across views, they are usually placed in a shared module.
    - The [official documentation](https://angular.io/guide/module-types) has a good guideline for this pattern.

PS: In Angular 14 (which is the current version) it is possible to create standalone components (components that don't belong to any module). It is fully functioning, but is a "developer preview" feature, meaning that the syntax might change in the future. See the documentation for how to make components standalone.

### Short description of the properties of a module:

- **Declarations**: These are all the components that belong to the module. "Declarables"
- **Providers**: All the injectable objects that should be available to the components of the module.
- **Exports**: Any declared components you want to be usable by other modules which import this module.
- **Imports**: Used for importing all the exported components of a different module.

*Resources:*

[Bundling Angular Modules](https://christiankohler.net/bundling-angular-modules)

[Angular Modules Best Practices 2021](https://christiankohler.net/angular-modules-best-practices-2021)

[Angular documentation: ngModule](https://angular.io/api/core/NgModule)

## Services / Dependency Injection

Common use cases for services is to create a logger or API request service.

Services by convention are named **service-name.service.ts**. Service classes don't need any decorator, unless they themselves have a dependency, in which case they need to be decorated with @Injectable (which is imported from angular/core). @Injectable simply tells Angular that this class can be injected with a dependency. @Component implements @Injectable so it's not necessary to decorate components with both @Injectable and @Component.

To use the service anywhere else in the program, it must be provided to it in some way. There are three ways to do this:

- Register it in the providers field of a module. This way, all components of this module will have access to it.
- Register it in the providers field of a component. This way, only that component will have access to it.
- Specify in the service itself with the `providedIn:` property. `'root'` is the most commonly used value here, and the one that Angular will generate if you use the CLI tool to create a new service.  `'root`' will make the service available anywhere in the application. It will only create one instance of it, and in case it isn't used in the application, the compiler will recognize that and not load it at all.

*Resources:*

[Introduction to Angular Services](https://angular.io/guide/architecture-services)

## CLI

Angular CLI is an optional tool that allows you to create components etc. using the command line. Even though it's optional, it is used in 99% of Angular projects. It is developed by the Angular developers. 

**Creating a new component**

```sh
ng generate component component-name

# alternatively, shorthand:
ng g c component-name
```

This will create the .ts, .html and .css files for a component, with all the boilerplate, as well as register it (declare it) in the parent module. There will be a constructor and onInit method in the component by default. This can be removed if it's not used.

If there are multiple modules that can become parents, you might get an error saying it doesn't know which one it should belong to. You can use `--module=module-name` to specify which one.

**Creating a new service**

```sh
ng generate service service-name

# alternatively, shorthand:
ng g s service-name
```

**Creating a new app**

This isn't that relevant since we are creating the app through Visual Studio. But this is what you would type if you just wanted to create an angular app outside of any IDE.

```sh
ng new app-name
```

### Forms in Angular

Angular has two different modules to use forms: Reactive forms and Template-driven forms.

# Error handling

## Application errors

Whenever the app throws an **unhandled** exception, Angular intercepts that exception and invokes the `handleError(error)` method of the `ErrorHandler` class (which is part of the `@angular/core` module). The default behavior of the `handleError` method is to write the error to the browser console.

If we want to change the default behavior, we must create a service that implements the `ErrorHandler` class and overwrite the `handleError(error)` method. For example:

```ts
@Injectable()
export class GlobalErrorHandlerService implements ErrorHandler {
 
  constructor() { 
  }
 
  handleError(error) {
     console.error('An error occurred:', error.message);
     console.error(error);
     alert(error);
  }
  
}
```

The class must be provided with a special token, `ErrorHandler`:

```ts
  providers: [
    { provide: ErrorHandler, useClass: GlobalErrorHandlerService },
  ]
```

### Using other services in custom error handler class

Providers won't be available to `ErrorHandler` as it is created before them. To be able to use other services in our class, we need to use the `Injector` instance directly. A common use for this is if we want to redirect the user to the login page if there is an unauthorized error. Example:

```ts
import { ErrorHandler, Injectable, Injector} from '@angular/core';
import { Router } from '@angular/router';
 
@Injectable()
export class GlobalErrorHandlerService implements ErrorHandler {
 
    constructor(private injector: Injector) {
    }
 
    handleError(error) {
 
        let router = this.injector.get(Router);
        console.log('URL: ' + router.url);
        console.error('An error occurred:', error.message);
       
       alert(error);
   }
}
```
Here we had to:
- Import Injector from `@angular/core`
- Import the service we wanted to inject.
- In the constructor, inject the injector (not the service)
- To access the service, we can use `this.injector.get(_service_)`

## Http errors

The subscribe method has three callback arguments: `success`, error and `completed`.

### Handling http error in component:

```ts
public getRepos() {
    this.loading = true;
    this.errorMessage = "";
    this.githubService.getReposCatchError(this.userName)
      .subscribe(
        (response) => {                           //Next callback
          console.log('response received')
          this.repos = response;
        },
        (error) => {                              //Error callback
          console.error('error caught in component')
          this.errorMessage = error;
          this.loading = false;
    
          //throw error;   //You can also throw the error to a global error handler
        }
      )
}
```

### Handling http error in service:

```ts
getRepos(userName: string): Observable<repos[]> {
return this.http.get<repos[]>(this.baseURL + 'usersY/' + userName + '/repos')
    .pipe(
    catchError((err) => {
        console.log('error caught in service')
        console.error(err);

        //Handle the error here

        return throwError(err);    //Rethrow it back to component
    })
    )
}
```

Note the difference between `throw new Error(error)` and `return throwError(error)` !

You can retry with retry. If you want to skip the error, you can output an empty observable instead of an error.


#### Resources

https://www.youtube.com/watch?v=kb9CBd2c4uA

https://angular.io/tutorial/toh-pt6#error-handling

https://iamturns.com/continue-rxjs-streams-when-errors-occur/

https://stackoverflow.com/a/58661330

https://www.tektutorialshub.com/angular/angular-http-error-handling/
