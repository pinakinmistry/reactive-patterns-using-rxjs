# Why, What and How RxJS with Reactive Patterns and Best Practices

Modern Apps are becoming more and more engaging in terms of dynamic data/content and increased interactivity (user or system events).

User events like infinite scrolls, switching views, post/swipe/like/share/comments, etc. and system events like real time updates, buffering, auto play/pause, online/offline switchovers, notifications, background processing, etc. make app go from one state to next state on an event just like a state machine.

> TODO: State machine diagram

These modern use cases require a modern approach of reactive programming in which we react to data and events by treating them as continuous observable streams.

**We just `subscribe()` to these `Observable` streams and react to `next()` event to handle the change.**

> TODO: Subscribe using bell icon image
> We will see what is `subscribe()`, `Observable` and `next()` soon but before that, what is a stream?

## Everything is a Stream

> TODO: Everything is a stream picture

Everything is a Stream! This includes what you are reading, listening, thinking, understanding, doing, etc. right now and in future.

It is ongoing, untimely, short/long lived, endless flow which can be interrupted/terminated. So keep breathing.

**Data, change in data, events, errors are streams too.**

These streams are flowing in any application that we can observe and react to. There are producers where data/events originate and one or more consumers of that data/event that observe them and react to next incoming change.

Let's see an example in action:

> Copy below code in browser's devtools console and move your cursor over the page to see the logs as shown in following image

```ts
document.addEventListener('mouseenter', function () {
    console.log('Mouseenter Event');
});
document.addEventListener('mouseleave', function () {
    console.log('Mouseleave Event');
});
document.addEventListener('click', function () {
    console.log('Click Event');
});
document.addEventListener('mousemove', function () {
    console.log('Mousemove Event');
});
```

> TODO: Image of above console
> These are endless streams until you close the browser window or `unsubscribe()` to these events using `removeEventListener()`

## Imperative vs Declarative Approach

Imperative approach is step by step sequence of instructions like an algorithm. It is focused more on `how to do` instead of `what to do` making code more verbose and difficult to read and understand.

Declarative approach focuses more on what needs to be done. It hides the implementation details with help of high level directives about what to do and delegating the how part to library or framework that does the heavy lifting of `how to do` well. This makes application code more consise and precise and easy to reason about.

> AngularJS, Underscore, LoDash, D3 are some examples of such frameworks and libraries.

## Reactive Approach

Reactive approach follows decalarive approach wherein we react to an event by providing what needs to be done when that event occurs. And then it keeps repeating the same whenever that event occurs making it reactive to change.

> React, Angular, VueJS are few examples of such frameworks and libraries.

## What is RxJS

RxJS, Reactive Extensions for JavaScript, is a utility library for handling streams and events in reactive way.

It provides elegant and powerful ways to establish continuous channels between producers and consumers to communicate data and events.

## Why is more important than what and how

**Why should we learn RxJS?**

Imperative approach to handle streams of events, data, changes in data, error handling and recovery in these streams is very difficult.

Due to asynchronous and continuous nature of stream, imperative approach leads to chaos in the code.

Thankfully, even chaos has some patterns that can be tackled in effective way.

RxJS provides operators to handle these patterns.

There are good reasons why RxJS is the defacto library in Angular framework. In fact, it is baked into many core components like `Form`, `Router`, `Route`, `Params`, `Http`, `HttpClient` etc. in Angular.

RxJS has a dedicated conference called RxJS Live. Follow @RxJSLive on Twitter.

> Look Ma what I found, RxJS!

## Drawbacks of Imperative/non Reactive Approach for handling streams

Imperative programming is difficult due to below drawbacks:

- Sharing data/state and its ownership
- Copying data/state locally
- Local updates not propogated to other consumers
- Additional listeners to update local state
- State ownership not defined
- Global event bus
- Public emitters of data
- Timing issue; notifying consumers before they are subscribed
- Sequencing issue
- Low Data Encapsulation
- Low cohesion
- High coupling
- Strange Side Effects
- Spaghetti code
- Unpredictable

## Lessons ToDo App Using Imperative Approach

Let's implement Lessons ToDo App using imperative approach with a global event bus that will provide a communication channel between producer (Subject) of data and consumers (Observers) of that data.

### Producer

- EventBusExperimentsComponent

### Consumers

- LessonsListComponent
- LessonsCounterComponent

> branch: custom-events-2
> TODO: Image of UI

### event-bus-experiments.component.html

```html
<div class="course-container">

    <h2> Event Bus Experiments</h2>

    <lessons-counter></lessons-counter>

    <lessons-list></lessons-list>

    <input #input />

    <button class="button button-highlight"
        (click)="addLesson(input.value)">Add Lesson</button>
</div>
```

### Data Model: lesson.ts

```ts
export interface Lesson {
    id:number;
    description:string;
    duration?:string;
    completed?:boolean;
}
```

> Diagram of Observer and Subject

`EventBus` service implements `Subject` interface for building a communication channel of custom events for consumers of data.

### Service: eventbus.service.ts

```ts
import * as _ from 'lodash';

// Custom Events
export const LESSONS_LIST_AVAILABLE = 'LESSONS_LIST_AVAILABLE';
export const ADD_NEW_LESSON = 'ADD_NEW_LESSON';
// ... there can be many more custom events like these

export interface Observer {
    // Get notified with new data
    notify(data:any);
}

interface Subject {
    // Accepts an eventType and observer that needs to be notified on that event
    subscribeObserver(eventType:string, obs:Observer);

    // Accepts an observer that no longer needs to be notified on given eventType
    unsubscribeObserver(eventType:string, obs:Observer);

    // Notifies all observers of given eventType with given data
    notifyObservers(eventType:string, data:any);
}

class EventBus implements Subject {

    // Mapping of event type and list of observers subscribed to it
    private observers : {[key:string]: Observer[]} = {};

    // Maintains list of all observers of given eventType
    subscribeObserver(eventType:string, obs: Observer) {
        this.observersPerEventType(eventType).push(obs);
    }

    // Removes given observer from the list of given eventType
    unsubscribeObserver(eventType:string, obs: Observer) {
        const newObservers = _.remove(
            this.observersPerEventType(eventType), el => el === obs );
        this.observers[eventType] = newObservers;
    }

    // Notifies all observers of given eventType with new data
    notifyObservers(eventType:string, data: any) {
        this.observersPerEventType(eventType)
            .forEach(obs => obs.notify(data));
    }

    // Returns list of observers for given eventType
    private observersPerEventType(eventType:string): Observer[] {
        const observersPerType = this.observers[eventType];
        if (!observersPerType) {
            this.observers[eventType] = [];
        }
        return this.observers[eventType];
    }
}

// Make Event Bus globally available to other parts of application
export const globalEventBus = new EventBus();
```

## Problems in Individual Components

- Local state
- Local state initialization
- Local state updates/mutations
- Notifying updates to other components
- Sharing state by reference
- Updates shared without notification
- No clear data ownership
- Low cohesion
- High coupling

### event-bus-experiments.component.ts

```ts
import {Component, OnInit} from '@angular/core';
import {globalEventBus, LESSONS_LIST_AVAILABLE, ADD_NEW_LESSON} from "./eventbus.service";
// Mock lessons as if sent by backend
import {testLessons} from "../shared/model/test-lessons";

@Component({
    selector: 'event-bus-experiments',
    templateUrl: './event-bus-experiments.component.html',
    styleUrls: ['./event-bus-experiments.component.css']
})
export class EventBusExperimentsComponent implements OnInit {

    // Local copy of lessons (local state)
    lessons: Lesson[] = [];

    ngOnInit() {
        // Initialize state
        this.lessons = testLessons;

        // Notify all subscribed observers about lessons just initialized
        // Note that testLessons is notified to all observers by reference instead of by value
        // This causes tight coupling and side effects
        globalEventBus.notifyObservers(LESSONS_LIST_AVAILABLE,
            testLessons);

        setTimeout(() => {
            // Mock new lesson sent by backend
            // Local state updated and notified to all subscribed observers
            this.lessons.push({
                id: Math.random(),
                description: 'New lesson arriving from backend'
            });
            // LessonsListComponents and LessonsCounterComponent gets notified
            // even if we comment out below line because it is using reference
            // to EventBusExperimentsComponent's lessons in its notify method
            globalEventBus.notifyObservers(LESSONS_LIST_AVAILABLE, this.lessons);
        }, 5000);
    }

    addLesson(lessonText: string) {
        // Notifying new data to all subscribed observers
        globalEventBus.notifyObservers(ADD_NEW_LESSON, lessonText);
    }
}
```

### lessons-list.component.ts

```ts
import {Component} from '@angular/core';
import * as _ from 'lodash';
import {
    globalEventBus, Observer, LESSONS_LIST_AVAILABLE, ADD_NEW_LESSON
} from "../event-bus-experiments/eventbus.service";
import {Lesson} from "../shared/model/lesson";

@Component({
    selector: 'lessons-list',
    templateUrl: './lessons-list.component.html',
    styleUrls: ['./lessons-list.component.css']
})
export class LessonsListComponent implements Observer {

    // Local state
    lessons: Lesson[] = [];

    constructor() {
        // Subscribe to initial list of lessons
        globalEventBus.subscribeObserver(LESSONS_LIST_AVAILABLE, this);

        // Subscribe to new lesson being added in future to update the local lessons list
        globalEventBus.subscribeObserver(ADD_NEW_LESSON, {
            notify: lessonText => {
                // this.lessons here is pointing to EventBusExperimentsComponent's lessons (local state)
                // as it is shared by reference
                this.lessons.push({
                    id: Math.random(),
                    description: lessonText
                });
            }
        } );
    }

    notify(data: Lesson[]) {
        // Get notified once Lessons are available
        // Note that data is reference to testLessons from EventBusExperimentsComponent
        this.lessons = data;
    }

    toggleLessonViewed(lesson:Lesson) {
        // Local updates not notified to subscribed observers
        lesson.completed = !lesson.completed;
    }

    delete(deleted:Lesson) {
        // Local updates not notified to subscribed observers
        _.remove(this.lessons,
            lessons => lesson.id === deleted.id);
    }
}
```

### lessons-list.component.html

```ts
<table class="table lessons-list card card-strong">
    <tbody>
        <tr *ngFor="let lesson of lessons">
            <td class="viewed">
                <input [checked]="lesson.completed" type="checkbox"
                    (click)="toggleLessonViewed(lesson)">
            </td>
            <td class="lesson-title" [ngClass]="{'lesson-viewed':lesson.completed}">
                {{ lesson.description }}
            </td>
            <td class="duration">
                <i class="md-icon duration-icon">access_time</i>
                <span>{{lesson.duration}}</span>
            </td>
            <td>
                <button class="button button-highlight"(click)="delete(lesson)">X</button>
            </td>
        </tr>
    </tbody>
</table>
```

### lessons-counter.component.ts

```ts
import { Component, OnInit } from '@angular/core';
import {globalEventBus, Observer, LESSONS_LIST_AVAILABLE, ADD_NEW_LESSON}
    from "../event-bus-experiments/eventbus.service";
import {Lesson} from "../shared/model/lesson";

@Component({
  selector: 'lessons-counter',
  templateUrl: './lessons-counter.component.html',
  styleUrls: ['./lessons-counter.component.css']
})
export class LessonsCounterComponent implements Observer {

    // Local state
    lessonsCounter = 0;

    constructor() {
        // Subscribe to initial list of lessons
        globalEventBus.subscribeObserver(LESSONS_LIST_AVAILABLE, this);

        // Subscribe to new lesson being added in future to update the local lessons list
        globalEventBus.subscribeObserver(ADD_NEW_LESSON, {
            notify: lessonText => this.lessonsCounter += 1
        } );
    }

    notify(data: Lesson[]) {
        // Update the counter based on new state
        this.lessonsCounter = data.length;
    }
}
```

> **Even LessonsListComponents is mutating the shared state causing mixed data ownership, tight coupling, spaghetti code and hard to predict side effects**

## Reactive Approach Using Streams

Updating state and notifying observers should be private to the data owner.

This can be achieved by separating the ability of emitting new data from subcribe/unsubscribe.

1. `Stream`line data ownership using `Subject` and `Observable` as 2 different interfaces
2. `Subject` is private interface for producer to emit data/notify observers
3. `Observable` is public interface for observers to subscribe/unsubscribe

> **Make data as something that observers can subscribe to, to observe changes and get notified about it**

## Observer, Observable and Subject - Nuts and Bolts of Reactive Programming

> branch: observable-pattern

**Observer:** Consumer of data/event emitted by Subject using `next()`

> `next` here is analogous to next function in NodeJS for what to do `next` on an event

```ts
// Consumer
export interface Observer {
    next(data: any);
    complete();
    error();
}
```

**Observable:** A stream that `Observer` can observe using `subscribe` and later `unsubscribe` once done.

```ts
// Public stream
export interface Observable {
    subscribe(obs: Observer);
    unsubscribe(obs: Observer);
}
```

```ts
// Subject is a producer of data/event.
// It is private so that only owner of data can emit new data and notify observers
// It extends both Observer and Observable so that it can produce/emit new data or event privately using `next()`
// and provides a public interface to `Observers` to subscribe and unsubsribe.
interface Subject extends Observer, Observable  {
}
```

## Observable, Observer and Subject in Action

Let's implement `Subject` interface to maintain list of observers with `subscribe()` and `unsubscribe()` methods to add or remove observers in this list and `next()` method to notify these observers about next data.

> branch: introduce-rxjs, prepare-lessons ???

### app-data.service.ts

```ts
import * as _ from 'lodash';

class SubjectImplementation implements Subject {
    private observers: Observer[] = [];

    subscribe(obs: Observer) {
        this.observers.push(obs);
    }

    unsubscribe(obs: Observer) {
        _remove(this.observers, el => el === obs);
    }

    next(data: any) {
        this.observers.forEach(obs => obs.next(data));
    }
}

// Initialization of lessons private to this service
const lessons: Lesson[] = [];

// lessonsListSubject is private in this service
// so that no other component can emit data using next()
const lessonsListSubject = new SubjectImplementation();

// Public Observable
// Note how public subscribe and unsubscribe of Observable is
// calling private subscribe and unsubscribe of Subject
// This is how we hide the data ownership in Subject using Observable as its public interface
export const lessonsList$: Observable = {
    subscribe: obs => {
        lessonsListSubject.subscribe(obs);
        // Notify late subscribers with current data
        obs.next(lessons);
    },
    unsubscribe: obs => lessonsListSubject.subscribe(obs),
};

// Populate and Notify
export function initializeLessonsList(newList: Lesson[]) {
    // Deep cloning newList to avoid unintentional sharing of list with component
    lessons = _.cloneDeep(newList);
    // Notify all observers about new lessons list
    lessonsListSubject.next(lessons);
}
```

## Refactoring: DataStore as Service

Note that `lessons`, `lessonsListSubject`, `lessonsList$` and `initializeLessonsList` are closely related and should be combined together under a class say `DataStore`. In fact, we can add all the other methods like `addLesson`, `toggleLesson` and `deleteLesson` in this class.

### app-data.service.ts

```ts
class SubjectImplementation implements Subject {
    // ...
}

class DataStore {
    private lessons: Lesson[] = [];
    private lessonsListSubject = new SubjectImplementation();

    public lessonsList$: Observable = {
        subscribe: obs => {
            this.lessonsListSubject.subscribe(obs);
            obs.next(this.lessons);
        },
        unsubscribe: obs => this.lessonsListSubject.subscribe(obs),
    };

    // Broadcast every mutation
    public initializeLessonsList(newList: Lesson[]) {
        this.lessons = _.cloneDeep(newList);
        this.broadcast();
    }

    public addLesson(newLesson: Lesson) {
        this.lessons.push(_.cloneDeep(newLesson));
        this.broadcast();
    }

    public toggleLesson(toggled: Lesson) {
        const lesson = _.find(this.lessons, lesson => lesson.id === toggled.id);
        lesson.completed != lesson.completed;
        this.broadcast();
    }

    public deleteLesson(deleted: Lesson) {
        _.remove(this.lessons, lesson => lesson.id === deleted.id);
        this.broadcast();
    }

    // Common broadcast method
    private broadcast() {
        this.lessonsListSubject.next(this.lessons);
    }
}

export const store = new DataStore();
```

## Refactoring Imperative Style to Reactive Style using DataStore service

Let's replace global event bus with store in our components now.

### event-bus-experiments.component.ts

```ts
import {Component, OnInit} from '@angular/core';
import {testLessons} from "../shared/model/test-lessons";
import {store} from "./app-data.service";

@Component({
    selector: 'event-bus-experiments',
    templateUrl: './event-bus-experiments.component.html',
    styleUrls: ['./event-bus-experiments.component.css']
})
export class EventBusExperimentsComponent implements OnInit {

    ngOnInit() {
        // Notify all observers about Lessons
        store.initializeLessonsList(testLessons);

        setTimeout(() => {
            const newLesson = {
                id: Math.random(),
                description: 'New lesson arriving from backend'
            };
            store.addLesson(newLesson);
        }, 5000);
    }

    addLesson(lessonText: string) {
        store.addLesson(newLesson);
    }
}

```

### lessons-counter.component.ts

```ts
import { Component, OnInit } from '@angular/core';
import {Lesson} from "../shared/model/lesson";
import {store, Observer} from "../event-bus-experiments/app-data.service";

@Component({
  selector: 'lessons-counter',
  templateUrl: './lessons-counter.component.html',
  styleUrls: ['./lessons-counter.component.css']
})
// Implements Observer to get notified about next data
export class LessonsCounterComponent implements Observer, OnInit {

    lessonsCounter = 0;

    ngOnInit() {
        console.log('lesson list component is subscribeed as observer ..');
        store.lessonsList$.subscribe(this);
    }

    next(data: Lesson[]) {
        console.log('counter component received data ..');
        this.lessonsCounter = data.length;
    }

}

```

### lessons-list.component.ts

```ts
import {Component} from '@angular/core';
import * as _ from 'lodash';
import {Lesson} from "../shared/model/lesson";
import {store, Observer} from "../event-bus-experiments/app-data.service";

@Component({
    selector: 'lessons-list',
    templateUrl: './lessons-list.component.html',
    styleUrls: ['./lessons-list.component.css']
})
// Implements Observer to get notified about next data
export class LessonsListComponent implements Observer, OnInit {

    lessons: Lesson[] = [];

    ngOnInit() {
        store.lessonsList$.subscribe(this);
    }

    next(data: Lesson[]) {
        this.lessons = data;
    }

    toggleLessonViewed(lesson:Lesson) {
        store.toggleLesson(lesson);
    }

    delete(lesson:Lesson) {
        store.deleteLesson(lesson);
    }
}
```

## Store as Observable

We can convert store itself to an observable by implementing Observable interface. This way, we can avoid public property lessonsList$ and have the interface methods implemented directly at class level.

### app-data.service.ts

```ts
class DataStore implements Observable {
    private lessons: Lesson[] = [];
    private lessonsListSubject = new SubjectImplementation();

    subscribe(obs: Observer) {
        this.lessonsListSubject.subscribe(obs);
        obs.next(this.lessons);
    }

    unsubscribe(obs: Observer) {
        this.lessonsListSubject.subscribe(obs);
    }

    public initializeLessonsList(newList: Lesson[]) {
        this.lessons = _.cloneDeep(newList);
        this.broadcast();
    }

    public addLesson(newLesson: Lesson) {
        this.lessons.push(_.cloneDeep(newLesson));
        this.broadcast();
    }

    public toggleLesson(toggled: Lesson) {
        const lesson = _.find(this.lessons, lesson => lesson.id === toggled.id);
        lesson.completed != lesson.completed;
        this.broadcast();
    }

    deleteLesson(deleted: Lesson) {
        _.remove(this.lessons, lesson => lesson.id === deleted.id);
        this.broadcast();
    }

    private broadcast() {
        this.lessonsListSubject.next(this.lessons);
    }
}

export const store = new DataStore();
```

### Updating Components

```ts
...
// Change below
store.lessonsList$.subscribe(this);
// to
store.subscribe(this);
```

## Replacing custom implementation with RxJS (Reactive Extensions)

Let's now replace custom implementation of `Subject`, `Observable` and `Observer` by using these from RxJS.

Note that we are using `BehaviorSubject` instead of `Subject` from RxJS. `BehaviorSubject` takes initial state as input and remembers last emitted state. With this feature, we don't need local state. We can get the latest state from `BehaviorSubject` using `getValue()` method. Another important benefit is that `Observers` subscribing late will still get notified with latest state even without call to `next()`.

> branch: introduce-rxjs

### app-data.service.ts

```ts
import * as _ from 'lodash';
import {BehaviorSubject, Observable, Observer} from 'rxjs';
import {lesson} from '../shared/model/lesson';

class DataStore {
    // No need of local state as the state can be maintained in the
    // BehaviorSubject itself as it remembers the last emitted value.
    // private lessons: Lesson[] = [];
    private lessonsListSubject = new BehaviorSubject([]);

    public lessonsList$: Observable<Lesson[]> = lessonsListSubject.asObservable();

    public initializeLessonsList(newList: Lesson[]) {
        this.lessons = _.cloneDeep(newList);
        this.broadcast();
    }

    public addLesson(newLesson: Lesson) {
        const lessons = this.cloneLessons();
        lessons.push(_.cloneDeep(newLesson));
        this.lessonsListSubject.next(lessons);
    }

    public toggleLesson(toggled: Lesson) {
        const lessons = this.cloneLessons();
        const lesson = _.find(lessons, lesson => lesson.id === toggled.id);
        lesson.completed != lesson.completed;
        this.lessonsListSubject.next(lessons);
    }

    deleteLesson(deleted: Lesson) {
        const lessons = this.cloneLessons();
        _.remove(lessons, lesson => lesson.id === deleted.id);
        this.lessonsListSubject.next(lessons);
    }

    private cloneLessons() {
        return _.cloneDeep(this.LessonsListSubject.getValue());
    }
}

export const store = new DataStore();
```

### Updating Components

```ts
import {Observer} from 'rxjs';

class LessonsListComponent implements Observer<Lesson[]>, OnInit {
    ...
    next(data: Lesson[]) {
        ...
    }

    complete() { ... }

    error(err: any) { ... }
}
```

## More Reactive Patterns

> branch: stateless-services

### Problems with Services

1. Maintaining local state
2. Nested subscriptions due to nested async event handling/API calls
3. Maintaining subscriptions and later unsubscribe them
4. Long lived observables causing memory leaks

## Stateless Service

1. No local states
2. No subscriptions
3. Return observables directly instead of subscribing

    ```ts
        return observableAPICall()
            .do(data => subject.next(data));
    ```

4. Short lived observable by unsubscribing after first notification using `first()`
5. Stateless services provide observables as streams of data/events that components can subscribe too
6. Avoid multiple calls to observable by ensuring it completes before emitting the value using `publishLast().refCount()`

### courses.service.ts

```ts
import {Injectable} from '@angular/core';
import {AngularFirebaseDatabase} from 'angularfire2';
import {Observable} from 'rxjs';

import {Course} from '../shared/model/course';
import {Lesson} from '../shared/model/lesson';

@Injectable()
export class CoursesService {
    // No Local State
    // No Local Observable

    constructor(private db: AngularFirebaseDatabase) {}

    getAllCourses(): Observable<Course[]> {
        return this.db.list('courses')
            // Unsubscribe after first notification using first()
            .first()
            .do(console.log);
    }

    getLatestLessons(): Observable<Lesson[]> {
        return this.db.list('lessons', {
            query: {
                orderByKey: true,
                limitToLast: 10
            }
        })
        // Unsubscribe after first notification using first()
        .first()
        .do(console.log);
    }

    findCourseByUrl(courseUrl: string): Observable<Course> {
        return this.db.list('courses', {
            query: {
                orderByChild: 'url',
                equalTo: courseUrl
            }
        })
        .first()
        .map(data => data[0]);
    }

    findLessonsForCourse(courseId: string): Observable<Lesson[]> {
        this.db.list('lessons', {
            query: {
                orderByChild: 'courseId',
                equalTo: courseId
            }
        })
        .first();
    }
}
```

## Stateless Components

1. No local state
2. Injects stateless services to make observables available to view template
3. Auto subscribe to these observables using async pipes
4. Use async as syntax to avoid multiple pipes and subcriptions
5. Stateless components are plugging streams of data with view using async pipes

### home.component.ts

```ts
export class HomeComponent implements OnInit {
    // No Local States
    // Only Local Observables
    courses$: Observable<Course[]>;
    latestLessons$: Observable<Lesson[]>;

    constructor(private coursesService: CoursesService) {}

    ngOnInit() {
        this.courses$ = this.coursesService.getAllCourses();
        this.latestLessons$ = this.coursesService.getLatestLessons();
    }
}
```

### home.component.html

```html
<div class="screen-container">
    <h2>Stateless Component using Stateless Service</h2>

    <table class="courses-list card card-strong"
        *ngIf="courses$ | async as courses else loadingCourses">
        <tr class="course-summary" *ngFor="let course of courses">
            <td>
                {{course.description}}
            </td>
            <td>
                <button class="button button-primary" [routerLink]="['/course', course.url]">
                    View
                </button>
            </td>
        </tr>
    </table>

    <ng-template #loadingCourses>
        <div>Loading...</div>
    </ng-template>

    <h2>Latest Lessons</h2>

    <table class="lessons-list card card-strong"
        *ngIf="latestLessons$ | async as lessons else loadingLessons">
        <tr class="course-summary" *ngFor="let course of lessons">
            <td>
                {{course.description}}
            </td>
            <td>
                <button class="button button-primary" [routerLink]="['/course', course.url]">
                    View
                </button>
            </td>
        </tr>
    </table>

    <ng-template #loadinglessons>
        <div>Loading...</div>
    </ng-template>
</div>
```

## Observable Service

Sometime we do need state in service in form of local `Observable` that other components can subscribe to. For example logged in user needs to be maintained as a global state for entire application.

> TODO: UI screenshot
> branch: observable-data-service

### user.ts

```ts
export interface User {
    firstName: string;
    lastName?: string;
}
```

### user.service.ts

```ts
export const UNKNOWN_USER: User = {
    firstName = 'Unknown',
};

export class UserService {
    // Private Subject
    private subject = new BehaviorSubject(UNKNOWN_USER);
    // Public Observable
    user$: Observable<User> = this.subject.asObservable();

    constructor(private http: Http) {}

    login(email: string, password: string): Observable<User> {
        const headers = new Headers();
        headers.append('Content-Type', 'application/json');
        return this.http.post('/api/login', {
            email,
            password,
        })
        .map(res => res.json())
        .do(user => this.subject.next(user))
        // Avoid multiple calls by ensure observable completes first before emitting the value
        .publishLast().refCount();
    }
}
```

### loginRoute.ts (Server side)

```ts
import {User} from '../app/shared/model/user';

const auth = {
    'pp@pp.com': 'test123',
};

const users: {[key: string]: User} = {
    'pp@pp.com': { firstName: 'Pinakin' },
};

export function loginRoute(req, res) {
    const payload = req.body;

    if(auth[payload.email] && auth[payload.email] === payload.password) {
        res.status(200).json(users[payload.email]);
    } else {
        res.sendStatus(500);
    }
}
```

### top-menu.component.ts

```ts
export class TopMenuComponent implements OnInit {

    isUserLoggedIn$: Observable<boolean>;

    constructor(private userService: UserService) {}

    ngOnInit() {
        this.isUserLoggedIn$ = this.userService.user$
            .map(user => user !== UNKNOWN_USER);
    }
}
```

### top-menu.component.html

```html
<header class="l-header">
    <ul class="top-menu disable-link-styles">
        <li>
            <a routerLink="home" routerLinkActive="menu-active">Home</a>
        </li>
        <li *ngIf="!(isUserLoggedIn$c | async)">
            <a routerLink="login" routerLinkActive="menu-active">Home</a>
        </li>
        <li *ngIf="(isUserLoggedIn$c | async)">
            <a (click)="logout()">Logout</a>
        </li>
    </ul>
</header>
```

### course-detail.component.html

```html
<div class="screen-container">

    <course-detail-header
        [course]="course"
        [lessons]="lessons"
        [firstName]="(userService.$user | async).firstName"
        (subscribe)="onSubscribe($event)"
    ></course-detail-header>

    <table class="lessons-list card card-strong"
        *ngIf="latestLessons$ | async as lessons else loadingLessons">
        <tr class="lesson-summary" *ngFor="let lesson of lessons">
            <td>
                {{lesson.description}}
            </td>
            <td>
                <i class="md-icon duration-icon">access_time</i>
                <span>{{lesson.duration}}</span>
            </td>
        </tr>
    </table>

    <ng-template #loadinglessons>
        <div>Loading...</div>
    </ng-template>
</div>
```

### course-detail.component.ts

```ts
export class CourseDetailComponent implements OnInit {
    //Still Local State
    course: Course;
    lessons: Lesson[];

    constructor(
        private route: ActivatedRoute,
        private coursesService: CoursesService,
        private newsLetterService: NewsLetterService,
        private userService: UserService,
    ) {}

    ngOnInit() {
        // Nested Subscriptions
        route.params
            .subscribe(params => {
                this.coursesService.findCourseByUrl(params['id'])
                    .subscribe(data => {
                        this.course = data;
                        this.courseService.findLessonsForCourse(this.course.id)
                            .subscribe(data => this.lessons = data);
                    });
            });
    }

    onSubscribe(email: string) {
        this.newsLetterService.subscribeToNewsLetter(email)
            .subscribe(
                () => alert('Subscription successful'),
                console.error
            );
    }

}
```

### course-detail-header.component.ts

```ts
export class CourseDetailHeaderComponent {

    @Input() course: Course;
    @Input() lessons: Lesson[];
    @Input() firstName: string;

    @Output() subscribe = new EventEmitter();

    onSubscribe(email: string) {
        this.subscribe.emit(email);
    }
}
```

### course-detail-header.component.html

```html
<h2>{{course.description}}</h2>
<h5>Total lessons: {{lessons.length}}</h5>

<newsletter [firstName]="firstName" (subscribe)="onSubscribe($event)"></newsletter>
```

### login.component.ts

```ts
export class LoginComponent implements OnInit {

    constructor(private userService: UserService, private router: Router) {}

    login(email: string, password: string) {
        this.userService.login(email, password)
            .subscribe(
                () => this.router.navigateByUrl('/home'),
                console.error
            );
    }
}
```

## Avoid Nested Subscription using `switchMap()`

Nested subscriptions arise when we have sequence of async event handling like in case of API calls

> branch: bubble-events

### course-detail.component.ts

```ts
export class CourseDetailComponent implements OnInit {
    //Local Observable
    course$: Observable<Course>;
    lessons$: Observable<Lesson[]>;

    constructor(
        private route: ActivatedRoute,
        private coursesService: CoursesService,
        private newsLetterService: NewsLetterService,
        private userService: UserService,
    ) {}

    ngOnInit() {
        // Avoid Nested Subscriptions using switchMap
        this.course$ = this.route.params
            .switchMap(params => this.coursesService.findCourseByUrl(params['id']))
            .first()
            .publishLast().refCount();

        this.lessons$ = this.course
            .switchMap(course => this.coursesService.findLessonsForCourse(course.id))
            .first()
            .publishLast().refCount();
    }

    onSubscribe(email: string) {
        this.newsLetterService.subscribeToNewsLetter(email)
            .subscribe(
                () => alert('Subscription successful'),
                console.error
            );
    }

}
```

## Avoid Prop Drilling (nested property and event bindings) using smart component

- Handler for custom event `subscribe` is passed from CourseDetailsComponent to NewsletterComponent thru HeaderComponent
- Avoid nested event handlers by converting NewsletterComponent to a smart component by injecting UserService and NewsletterService in it directly
- No property and event bindings in parent and intermediate components

### Newsletter.component.ts

```ts
export class NewsletterComponent implements OnInit {
    user$: Observable<User>;

    constructor(
        private userService: UserService,
        private newsletterService: NewsletterService,
    ) {}

    ngOnInit() {
        this.user$ = this.userService.user$;
    }

    subscribeToNewsletter(emailField) {
        this.newsletterService.subscribeToNewsletter(emailField.value)
            .subscribe(
                () => {
                    emailField.value = '';
                    alert('Subscription successful');
                },
                console.error
            );
    }
}
```

## Another Example: Paginated Table

Let's implement `lessons$` observable as a paginated table.

> TODO: UI
> branch: lessons-pager

### all-lessons.component.html

```html
<div class="screen-container">
    <course [id]="1"></course>
    <course [id]="2"></course>
</div>
```

### course.component.html

```html
<div class="course-md">
    <h2>{{(course$ | async as course).description}}</h2>

    <div class="lessons-nav">
        <button (click)="previousLessons()">Previous</button>
        <button (click)="nextLessons()">Next</button>
    </div>
    <lessons-list lessons="lessons$ | async"></lessons-list>
</div>
```

### course.component.ts

```ts
export class CourseComponent extends OnInit {

    @Input() id: number;

    course$: Observable<Course>;
    lessons$: Observable<Lesson[]>;

    constructor(
        private coursesService: CoursesService,
        private lessonsService: LessonsService,
    ) {}

    ngOnInit() {
        this.course$ = this.coursesService.getCourseById(this.id);
        this.lessons$ = this.lessonsService.lessonsPage$;
        this.lessonsService.loadFirstPage(this.id);
    }

    previousLessons() {
        this.lessonsService.previous();
    }

    nextLessons() {
        this.lessonsService.next();
    }

}
```

### courses.service.ts

```ts
@Injectable()
export class CoursesService {

    constructor(private http: Http) {}

    getCourseById(courseId: number): Observable<Course> {
        return this.http.get(`/api/courses/${courseId}`)
            .map(res => res.json());
    }

    getLessonDetails(lessonId: string): Observable<Lesson> {
        return this.http.get(`/api/lessons/${lessonId}`)
            .map(res => res.json());
    }
}
```

### lessons.service.ts

```ts
export class LessonsService {

    private courseId: number;
    // Private Subject
    private subject = new BehaviorSubject<Lesson[]>([]);
    // Public Observable
    lessonsPage$: Observable<Lesson[]> = this.subject.asObservable();
    currentPageNumber = 1;

    constructor(private http: Http) {}

    loadFirstPage(courseId: number) {
        this.courseId = courseId;
        this.currentPageNumber = 1;
        this.loadPage(this.currentPageNumber);
    }

    previous() {
        if (this.currentPageNumber - 1 >= 1) {
            this.currentPageNumber -= 1;
            this.loadPage(this.currentPageNumber);
        }
    }

    next() {
        this.currentPageNumber += 1;
        this.loadPage(this.currentPageNumber);
    }

    loadPage(pageNumber: number) {
        this.http.get('/api/lessons', {
            params: {
                courseId: this.courseId,
                pageNumber: 1,
                pageSize: LessonsService.PAGE_SIZE,
            }
        })
        .map(res => res.json().payload)
        .subscribe(
            lessons => this.subject.next(lessons),
            // Avoid error handling at subject level as it will stop emitting new values once in error
            // error => this.subject.error(error)
        );
    }
}
```

### server.ts (Server side)

```ts
const bodyParser = require('body-parser');
const app: Application = express();

app.use(bodyParser.json());

app.route('/api/newsletter').post(newsletterRoute);
app.route('/api/login').post(loginRoute);

app.route('/api/courses/:id').get(courseRoute);
app.route('/api/lessons').get(lessonsRoute);
app.route('/api/lessons/:id').get(lessonDetailRoute);

app.listen(8090, () => {
    console.log('server running at port 8090');
})
```

### lessonsRoute.ts (Server side)

```ts
export function lessonsRoute(req, res) {
    const courseId = parseInt(req.query['courseId']);
    const pageNumber = parseInt(req.query['pageNumber']);
    const pageSize = parseInt(req.query['pageSize']);
    const lessons = dbData[courseId].lessons;
    const start = (pageNumber - 1) * pageSize;
    const end = start + pageSize;
    const lessonsPage = _.slice(lessons, start, end);

    res.status(200).json({
        payload: lessonsPage.map(({
            url,
            description,
            duration,
        }, index) => ({
            url,
            description,
            duration,
            seqNo: index,
        }));
    });
}
```

## Master Details Implementation using Observable

Let's implement `details$` observable to get details of a lesson. `lessons$` observable is the master list providing `click` handler to show details of lesson being clicked.

> TODO: UI
> branch: master-detail

```ts
export interface Lesson {
    id:number;
    courseId?:string;
    seqNo:number;
    duration:string;
    url?:string;
    tags?:string;
    pro?:boolean;
    description:string;
    longDescription:string;
    videoUrl?:string;
}
```

### course.component.html

```html
<div class="course-md">
    <h2>{{(course$ | async as course).description}}</h2>

    <div *ngIf="details$ | async as lessonDetails else masterTmpl">
        <button (click)="backToMaster()"></button>
        <lesson-details [lesson]="lessonDetails"></lesson-details>
    </div>

    <ng-template #masterTmpl>
        <div class="lessons-nav">
            <button (click)="previousLessons()">Previous</button>
            <button (click)="nextLessons()">Next</button>
        </div>
        <lessons-list lessons="lessons$ | async" (selected)="selectDetails($event)"></lessons-list>
    </ng-template>
</div>
```

### lesson-details.component.html

```html
<h3>{{lesson.description}}</h3>
<h5>{{lesson.duration}}</h5>

<iframe *ngIf="lesson.videoUrl" [src]="lesson.videoUrl | safeUrl"></iframe>

<h5>Description</h5>
<p>{{lesson.longDescription}}</p>
```

### safeUrl.pipe.ts

```ts
@Pipe({
    name: 'safeUrl'
})
export class SafeUrlPipe extends PipeTransform {

    constructor(private sanitizer: DomSanitizer) {}

    transform(url: string) {
        return this.sanitizer.bypassSecurityTrustResourceUrl(url);
    }
}
```

### lessons-list.component.html

```html
<table class="table lessons-list card card-strong"
    *ngIf="lessons else loadingLessons">
    <tbody>
        <tr *ngFor="let lesson of lessons" (click)="select(lesson)">
            <td class="lesson-title">
                {{ lesson.description }}
            </td>
            <td class="duration">
                <i class="md-icon duration-icon">access_time</i>
                <span>{{lesson.duration}}</span>
            </td>
        </tr>
    </tbody>
</table>

<ng-template #loadingLessons>
    <div>Loading...</div>
</ng-template>
```

### lessons-list.component.ts

```ts
export class LessonsListComponent {
    @Input() lessons: Lesson[];

    // selected is the custom event emitted to notify parent CourseComponent to about selected lesson
    @Output() selected: EventEmitter<Lesson>;

    select(lesson) {
        this.selected.next(lesson);
    }
}
```

### course.component.ts

```ts
export class CourseComponent extends OnInit {

    @Input() id: number;

    course$: Observable<Course>;
    lessons$: Observable<Lesson[]>;
    details$: Observable<Lesson>;

    constructor(
        private coursesService: CoursesService,
        private lessonsService: LessonsService,
    ) {}

    ngOnInit() {
        this.course$ = this.coursesService.getCourseById(this.id);
        this.lessons$ = this.lessonsService.lessonsPage$;
        this.lessonsService.loadFirstPage(this.id);
    }

    // selected lesson is passed to this method by LessonList using $event in html template like this
    // (selected)="selectDetails($event)"
    selectDetails(lesson: Lesson) {
        this.details$ = this.coursesService.getLessonDetails(lesson.url);
    }

    backToMaster() {
        this.details$ = undefined;
    }
}
```

## Error Handling

Handling error is as important as handling success because of the real world situations like server error or unavailable, network issue, going offline, etc. At times, error handling is not given equal importance due to ignorance to various error scenarios that can occur in real world apps.

- Avoid `subject.error()` as it cannot emit values once in error
- Avoid `subscribe()`ing at service level
- Instead return `Observable<any>` using `do(data => subject.next(data))` from service
- And `subscribe()` for data and error handling at component level

> TODO: UI
> branch: error-handling

### lessons.service.ts

```ts
export class LessonsService {

    private courseId: number;
    private subject = new BehaviorSubject<Lesson[]>([]);
    lessonsPage$: Observable<Lesson[]> = this.subject.asObservable();
    currentPageNumber = 1;

    constructor(private http: Http) {}

    loadFirstPage(courseId: number): Observable<any> {
        this.courseId = courseId;
        this.currentPageNumber = 1;
        return this.loadPage(this.currentPageNumber);
    }

    previous(): Observable<any> {
        if (this.currentPageNumber - 1 >= 1) {
            this.currentPageNumber -= 1;
            return this.loadPage(this.currentPageNumber);
        }
    }

    next(): Observable<any> {
        this.currentPageNumber += 1;
        return this.loadPage(this.currentPageNumber);
    }

    loadPage(pageNumber: number): Observable<any> {
        return this.http.get('/api/lessons', {
            params: {
                courseId: this.courseId,
                pageNumber: 1,
                pageSize: LessonsService.PAGE_SIZE,
            }
        })
        .map(res => res.json().payload)
        .do(lessons => this.subject.next(lessons))
        .publishLast().refCount();
    }
}
```

### course.component.ts

```ts
export class CourseComponent extends OnInit {

    @Input() id: number;

    course$: Observable<Course>;
    lessons$: Observable<Lesson[]>;
    details$: Observable<Lesson>;

    constructor(
        private coursesService: CoursesService,
        private lessonsService: LessonsService,
        private messagesService: MessagesService,
    ) {}

    ngOnInit() {
        this.course$ = this.coursesService.getCourseById(this.id);
        this.lessons$ = this.lessonsService.lessonsPage$;
        this.lessonsService.loadFirstPage(this.id)
            .subscribe(
                () => {},
                err => messagesService.error('Could not load first page')
            );
    }

    previousLessons() {
        this.lessonsService.previous().subscribe(
            () => {},
            err => messagesService.error('Could not load previous lessons')
        );
    }

    nextLessons() {
        this.lessonsService.next().subscribe(
            () => {},
            err => messagesService.error('Could not load next lessons')
        );
    }

    selectDetails(lesson: Lesson) {
        this.details$ = this.coursesService.getLessonDetails(lesson.url);
    }

    backToMaster() {
        this.details$ = undefined;
    }
}
```

### messages.services.ts

```ts
export class MessagesService {

    private errorsSubject = new BehaviorSubject<string[]>([]);
    public errors$ = errorsSubject.asObservable();

    constructor() {}

    error(...errors: string[]) {
        this.errorsSubject.next(errors);
    }
}
```

### messages.component.ts

```ts
export class MessagesComponent implements OnInit {

    errors$: Observable<string[]> = Observable.of([]);

    constructor(messagesService: MessagesService) {}

    ngOnInit() {
        this.errors$ = this.messagesService.$errors;
    }

    close() {
        this.messagesService.error();
    }
}
```

### messages.component.html

```html
<div clsas="messages-frame" *ngIf="(messages$ | async as messages).length > 0">
    <div class="messages messages-error">
        <i class="md-icon close-icon" (click)="close()">close</i>
        <div *ngFor="let message of messages">{{message}}</div>
    </div>
</div>
```

## Router Data Prefetching

> TODO: UI

We can pre fetch data before entering a route using `resolve` property in route configuration. `resolve` takes an object with key as the data property name and value as a resolver service that implements `Resolve` interface. The prefetched data then becomes available as a property on `route.data` observable.

> branch: loading-indicator

### router.config.ts

```ts
export const routerConfig = [
    // ...
    {
        path: '/courses/:id',
        component: CourseDetailComponent,
        resolve: {
            detail: CourseDetailResolver
        }
    }
    // ...
];
```

### course-detail.resolver.ts

```ts
@Injectable()
// CourseDetailResolver resolves to <[Course, Lesson[]]>, a tuple of type array
// with required course at [0] and its lessons at [1]
export class CourseDetailResolver implements Resolve<[Course, Lesson[]]> {

    constructor(
        private coursesService: CoursesService
    ) {}

    resolve(
        route: ActivatedRouteSnapshot,
        state: RouterStateSnapshot
    ): Observable<[Course, Lesson[]]> {
        return this.coursesService.findCourseByUrl(route.params['id'])
            // switchMap takes a function as 2nd argument that can
            // return the resolved course and lessons
            .switchMap(course => this.coursesService.findLessonsForCourse(course.id),
                (course, lessons) => [course, lessons]);
    }
}
```

### course-detail.component.ts

```ts
export class CourseDetailComponent implements OnInit {
    //Local Observable
    course$: Observable<Course>;
    lessons$: Observable<Lesson[]>;

    constructor(
        private route: ActivatedRoute,
    ) {}

    ngOnInit() {
        // courses and lessons are resolved and available in this.route.data observable
        this.course$ = this.route.data.map(data => data['detail'][0]);
        this.lessons$ = this.route.data.map(data => data['detail'][1]);
    }
}
```

## Global Loading Indicator

Global loading indicator can be implemented by using `router.events` observable. For a route with data prefetching, `RoutesRecognized` and `NavigationStart` can be used to observe whether route and its required data is still loading.

> TODO: UI

### loading.compoent.html

```html
<div class="loading-indicator" *ngIf="($loading | async)">
    <img src="/images/loading.gif" />
</div>
```

### loading.component.ts

```ts
export class LoadingComponent implements OnInit {

    loading$: Observable<boolean>;

    constructor(
        private router: Router
    ) {}

    ngOnInit() {
        // A route with data pre fetching can use RoutesRecognized and NavigationStart
        // to observe whether route and its required data is still loading
        this.loading$ = this.router.events.map(event => event instanceof RoutesRecognized ||
            event instanceof NavigationStart);
    }
}
```

## Pre Save a Form Draft

Data entered by user can be persisted as draft without an explicit save draft button. This can be very well done in reactive way using form as an observable. `form.valueChanges()` returns an observable which we can subscribe to to save valid drafts.

> branch: form-draft-save

### create-lesson.component.ts

```ts
export class CreateLessonComponent extends OnInit {

    form: FormGroup;

    private static readonly DRAFT_COOKIE = 'lesson-draft';

    constructor(
        private fb: FormBuilder,
    ) {
        this.form = this.fb.group({
            description: ['', Validators.required],
            url: ['', Validators.required],
            longDescription: [''],
        });
    }

    ngOnInit() {
        const draft = Cookies.get(CreateLessonComponent.DRAFT_COOKIE);
        if(draft) {
            this.form.setValue(JSON.parse(draft));
        }

        this.form.valueChanges
            .filter(() => this.form.valid)
            .do(validDraft => Cookies.set(
                CreateLessonComponent.DRAFT_COOKIE, JSON.stringify(validDraft)
            ))
            .subscribe();
    }
}
```

### create-lesson.component.html

```html
<div class="screen-container">
    <h2>Create New Lesson</h2>

    <form [formGroup]="form" autocomplete="false" class="lesson-form">
        <fieldset>
            <legend>Lesson</legend>
            <div class="form-field">
                <label>Description</label>
                <input name="title" formControlName="description" />
            </div>
            <div class="form-field">
                <label>Lesson Url</label>
                <input name="title" formControlName="url" />
            </div>
            <div class="form-field">
                <label>Long Description</label>
                <input name="title" formControlName="longDescription" />
            </div>
        </fieldset>

        <div class="form-buttons">
            <button class="button button-primary">Save New Lesson</button>
        </div>
    </form>
</div>
```

## Always `unsubsribe()` once done

## Just Pipelines of Streams of Data and Observers Reacting to Change in Data
