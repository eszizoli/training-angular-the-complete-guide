# Angular - The Complete Guide Training

This repo is created for own use based on [Udemy Angular - The Complete Guide (2024 Edition)](https://www.udemy.com/course/the-complete-guide-to-angular-2).

Angular is a frontend JavaScript framework, which helps to building interactive, modern web user interfaces. It's also a collection of tools and features, CLI, debugging tools, IDE plugins, etc.
It can simplify the process of building complex, interactive web user interfaces.
(Declarative coding, separation of concerns via components, OOP concepts and principles.)

- [Angular Components](docs/components.md)

## Creating an Angular Project

Angular development requires to install **Node.js**: [Download Node.js®](https://nodejs.org/en/download/prebuilt-installer)

Install Angular CLI tool: `npm install -g @angular/cli`

Create a new Angular project:

- `ng new first-angular-app`
- Stylesheet format: **CSS**
- Enable Server-Side Rendering (SSR): **No**

Recommended development editor: [Visual Studio Code editor](https://code.visualstudio.com/)  
Recommended extensions for VSCode: [Angular Essentials](https://marketplace.visualstudio.com/items?itemName=johnpapa.angular-essentials)  
Recommended extensions for debugging: [Angular DevTools](https://chromewebstore.google.com/detail/angular-devtools)

Running the code:

1. Install dependencies: `npm install`
2. Run the development server: `npm start` or `ng serve`

## Using Angular CLI

|Description                 |Script                   |Comment                                               |
|----------------------------|-------------------------|------------------------------------------------------|
|Create a new project        |`ng new <app-name>`      |A new folder will be created for project.             |
|Generate a new component    |`ng g c <component-name>`|To skip *.spec file generation use option --skip-tests|

## Working with Angular

Angular uses component base separation. How to connect components together:
![How to connect components together](images/understanding-components.png)

**Standalone Components** are the recommended way of building components.  
In this way all required imports added in the component type script file:

```ts
@Component({
  standalone: true,
  imports: [CardComponent, DatePipe],
  ...
})
```

**Module-based Components** are the previous concept to work with components.  
Angular Modules make components available to each other.  
It is required to use a separate `app.module.ts` file to define imports.

### Data binding and change detection

#### Change detection mechanism by Zone.js

```ts
// create a public property
selectedUser = USERS[index];

// use getter to create a property with calculated data
get imagePath() {
  return 'assets/users/' + this.selectedUser.avatar;
}

// handler user event in code
onSelectUser() {
  console.log('Clicked');
}
```

Data binding in template:

```html
<div>
  <button (click)="onSelectUser()" > <!-- Event binding -->
    <img [src]="imagePath" [alt]="selectedUser.name" /> <!-- Property binding -->
    <span>{{ selectedUser.name }}</span> <!-- Binding with string interpolation -->
  </button>
</div>
```

Under the hood, Angular uses zone.js by default for change detection, error handling, async tracking.  
Zone.js notifies Angular about user events, expired timers, etc.  
When a new event occurs, Angular checks for changes for all components within a zone in the order of hierarchy levels, from the root app component to the last component.  
![Change detection](images/change-detection.png)

A zone related to a component hierarchy which connected to a page.

#### Managing state and changing data with Signals

> Signals has supported since **Angular v16**.

There is another option for updating state. Using **Signals** to notify Angular about value changes and required UI updates.  
A signal is an object that stores a value (any type of value, including nested objects).

```ts
// import
import { signal } from '@angular/core';

// create a new signal object
selectedUser = signal(USERS[index]);

// set/change value
selectedUser.set(USERS[2]);

// get value
const name = selectedUser().name;

// computed value based on a signal
const imagePath = computed(() => 'assets/users/' + this.selectedUser().avatar);
```

Data Binding in template:

```html
<div>
  <button (click)="onSelectUser()" > <!-- Event binding -->
    <img [src]="imagePath()" [alt]="selectedUser().name" /> <!-- Property binding -->
    <span>{{ selectedUser().name }}</span> <!-- Binding with string interpolation -->
  </button>
</div>
```

Angular manages subscriptions to the signal to get notified about changes. When a change occurs, Angular is then **able to update the part of the UI** that needs updating.  
![Signals - Trackable data container](images/signals-data-container.png)

### Component Inputs

Using property decorator to create an input to get data from parent component.

```ts
// create property with Input decorator
// using required option if it is mandatory to use in the parent template
@Input({ required: true }) name!: string;
```

Passing data with property binding from a parent component:

```html
<app-user [name]="users[0].name" />
```

#### Signal Inputs

Using Signals instead of input decorator:

```ts
// input data container via Signals
name = input.required<string>();
propertyWithDefaultValue = input('defaultValue');
optionalProperty = input<number>();
```

### Component Outputs (events)

Using property decorator to create an output to notify a parent component about changes.

```ts
// create event with Output decorator
@Output() select = new EventEmitter();
@Output() select = new EventEmitter<string>(); // it should recommended to define type of event argument

// emit event
onSelectUser() {
  this.select.emit(this.id);
}
```

```html
<button (click)="onSelectUser()"> <!-- binding event handler -->
```

Add event listener in parent component:

```ts
// create handler to process event emitted by a child component
onSelectUser(id: string) {
    console.log('Selected user with id ' + id);
  }
```

```html
<!-- binding select event of a child component -->
<app-user ... (select)="onSelectUser($event)" /> <!-- using a special $event object to pass data -->
```

#### output function

There is another option for define an output event with output function.

```ts
// create event with Output decorator
select = output<string>();

// emit event
onSelectUser() { this.select.emit(this.id); }
```

It is not a Signal! The same event will be created as with Output decorator, but with this solution the Output decorator can be omitted.  
It is recommended to use this then input signal was used.

### Outputting Content

Using **@for** to outputting list content (in HTML template):

```ts
@for (user of users; track user.id) {
  <li>
    <app-user [user]="user" (select)="onSelectUser($event)" />
  </li>
}
```

Using **@if** conditional content (in HTML template):

```ts
@if (selectedUser) {
  <app-tasks [name]="selectedUser.name" />
} @else {
  <p id="fallback">Select a user to see their tasks!</p>
}
```

### Separate data models

It is recommended to use a separate model files for each components according to the command pattern.

- create a separate model file: `<component-name>.model.ts`
- add `export` key word for each type and interfaces in the model file
- import model file with `type` keyword in components where the model is needed

![Separate data model](images/separate-data-model.png)

### CSS styling with Class Bindings

Using CSS Class bindings to dynamically change CSS styling based on data state.

```ts
// create a new boolean property to control CSS styling in typescript file
@Input({ required: true }) selected!: boolean;

// add CSS styling to CSS file
button:active,
.active { ... }

// add class binding to template file
<button [class.active]="selected" ... />

// pass state via property binding from template file of a parent component
<app-user [selected]="user.id === selectedUserId" ... />
```

### Using Directives & Two-Way-Binding

Two-Way-Binding with Angular Forms:

```ts
// import and register FormsModule in typescript file:
import { FormsModule } from '@angular/forms';
@Component({ imports: [FormsModule], ... })

// create properties for form values
enteredData = '';
...

// create submit event handler
onSubmit() {
  console.log('Submitted data: ' + this.enteredData)
}

// create Angular form in template file
<form (ngSubmit)="onSubmit()"> <!-- bind submit event -->
  <input type="text" id="data" name="data" [(ngModel)]="enteredData" /> <!-- bind form property with ngModel -->
  ...
</form>
```

#### Two-Way-Binding with Signals

To use Signals for two-way-binding, just use a signal object instead of normal property. All other parts are the same as above.

```ts
// create a signal object for store form value
enteredData = signal('');
```

### Content Projection with ng-content

When a component template is used with-in another component as a wrapper, the Angular does not keep the wrapped markup content by default. It will be overwritten by the markup defined used component template.  
To merge both markup the Angular content projection should be used:

```html
// create a shared component template
<div>
  <ng-content />
</div>

// use shared component in a child template
<app-shared-component>
  <button>Click</button>
  ...
</app-shared-component>
```

### Transforming Template Data with Pipes

With Pipes, the displayed data can be transformed, with which we can format the dates or numbers, or possibly apply a language translation.  
An example of formatting a date using a built-in date formatter solution:

```ts
// import and register
import { DatePipe } from '@angular/common';
@Component({ imports: [DatePipe], ... })

// use pipe in template file
<time>{{ task.dueDate | date: 'fullDate' }}</time>
```

### Services and Dependency Injection

It is recommended to use a separate service file to manage data. This solution can help to reduce the input and output parameters.

Recommended service filename convention: `<component-name>.service.ts`, e.g.: `tasks.service.ts`

```ts
// import
import { Injectable } from "@angular/core";

// add Injectable decorator
@Injectable({ providedIn: 'root' })
// create service class with methods
export class TasksService {
  getUserTasks(...) { ... }
  addTask(...) { ... }
  ...
}
```

Using service in component typescript file:

```ts
// import
import { inject, ... } from '@angular/core';

export class NewTaskComponent {
  ...

  // option a: adding dependency via constructor
  constructor(private tasksService: TasksService) {}

  // option b: using injection function
  private tasksService = inject(TasksService);

  // using service
  onSubmit() {
    this.tasksService.addTask(...);
  }
}
```

### Using local storage for data storage

```ts
// get data from local storage
const data = localStorage.getItem('tasks');

// parse data
this.tasks = JSON.parse(data);

// save data, it will be overwrite all data!
localStorage.setItem('tasks', JSON.stringify(this.tasks));
```
