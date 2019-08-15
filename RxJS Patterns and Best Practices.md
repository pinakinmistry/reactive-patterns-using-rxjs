# What, Why and How RxJS with Reactive Patterns and Best Practices

## What is RxJS
RxJS, Reactive Extensions for JavaScript, is a utility library for handling streams and events in reactive way. It provides elegant and powerful ways to establish continuous channels between producer and consumer to communicate events and data.

## Why is more important than what and how
Imperative approach to handle streams of events, data, changes in data, error handling and recovery in these streams is very difficult. Due to asynchronous and continuous nature of stream, imperative approach leads to chaos in the code. Thankfully, even choas have some patterns that can be tackled in effective way. RxJS provides operators to handle these patterns.

## Everything is a stream

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

Everything is a stream. This includes what you are reading, listening, thinking, doing, etc. Data, change in data, events, errors are no exceptions. Even these are streams flowing in any application that we need to observe and react to. We just `subscribe()` to these `Observable` streams and react to `next()` event to handle it.

## Problems with Imperative/non Reactive Approach for handling streams
1. Timing issue; notifying consumer before they are subscribed.
2. Sequencing issue
3. Copying data/state locally
4. Additional listeners to update local state
5. Public emitters of data
6. State ownership not defined
7. Local updates not propogated to other consumers
8. Low Data Encapsulation
9. Low cohesion
10. High coupling
11. Strange Side Effects
12. Unpredictable

## Imperative Approach Using Event Bus

> branch: custom-events-2

> Diagram of Observer and Subject

### lesson.ts
```ts
export interface Lesson {
    id:number;
    description:string;
    duration?:string;
    completed?:boolean;
}
```

### eventbus.service.ts
```ts
import * as _ from 'lodash';

export const LESSONS_LIST_AVAILABLE = 'NEW_LIST_AVAILABLE';
export const ADD_NEW_LESSON = 'ADD_NEW_LESSON';
// ... many more events like these

export interface Observer {
    notify(data:any);
}

interface Subject {
    registerObserver(eventType:string, obs:Observer);
    unregisterObserver(eventType:string, obs:Observer);
    notifyObservers(eventType:string, data:any);
}

class EventBus implements Subject {

    private observers : {[key:string]: Observer[]} = {};

    registerObserver(eventType:string, obs: Observer) {
        this.observersPerEventType(eventType).push(obs);
    }

    unregisterObserver(eventType:string, obs: Observer) {
        const newObservers = _.remove(
            this.observersPerEventType(eventType), el => el === obs );
        this.observers[eventType] = newObservers;
    }

    notifyObservers(eventType:string, data: any) {
        this.observersPerEventType(eventType)
            .forEach(obs => obs.notify(data));
    }

    private observersPerEventType(eventType:string): Observer[] {
        const observersPerType = this.observers[eventType];
        if (!observersPerType) {
            this.observers[eventType] = [];
        }
        return this.observers[eventType];
    }
}

export const globalEventBus = new EventBus();
```

### event-bus-experiments.component.ts
```ts
import {Component, OnInit} from '@angular/core';
import {globalEventBus, LESSONS_LIST_AVAILABLE, ADD_NEW_LESSON} from "./eventbus.service";
import {testLessons} from "../shared/model/test-lessons";

@Component({
    selector: 'event-bus-experiments',
    templateUrl: './event-bus-experiments.component.html',
    styleUrls: ['./event-bus-experiments.component.css']
})
export class EventBusExperimentsComponent implements OnInit {

    lessons: Lesson[] = [];

    ngOnInit() {
        console.log('Top level component broadcasted all lessons ...');
        this.lessons = testLessons.slice(0);

        globalEventBus.notifyObservers(LESSONS_LIST_AVAILABLE,
            testLessons.slice(0));

        setTimeout(() => {
            this.lessons.push({
                id: Math.random(),
                description: 'New lesson arriving from backend'
            });
            // LessonsListComponents gets notified even if we comment out below line.
            // because it is using reference to this.lessons in its notify method
            globalEventBus.notifyObservers(LESSONS_LIST_AVAILABLE, this.lessons);    
        }, 5000);
    }

    addLesson(lessonText: string) {
        globalEventBus.notifyObservers(ADD_NEW_LESSON, lessonText);
    }
}

```

### event-bus-experiments.component.html
```html
<div class="course-container">

    <h2> Event Bus Experiments</h2>

    <lessons-counter></lessons-counter>

    <lessons-list></lessons-list>

    <input #input>

    <button class="button button-highlight"
        (click)="addLesson(input.value)" >Add Lesson</button>
</div>
```

### lessons-list.component.ts
```ts
import {Component} from '@angular/core';
import * as _ from 'lodash';
import {globalEventBus, Observer, LESSONS_LIST_AVAILABLE, ADD_NEW_LESSON} from "../event-bus-experiments/eventbus.service";
import {Lesson} from "../shared/model/lesson";

@Component({
    selector: 'lessons-list',
    templateUrl: './lessons-list.component.html',
    styleUrls: ['./lessons-list.component.css']
})
export class LessonsListComponent implements Observer {

    lessons: Lesson[] = [];

    constructor() {
        console.log('lesson list component is registered as observer ..');
        globalEventBus.registerObserver(LESSONS_LIST_AVAILABLE, this);

        globalEventBus.registerObserver(ADD_NEW_LESSON, {
            notify: lessonText => {
                // What this is pointing to, LessonsListComponent or EventBusExperimentsComponent?
                this.lessons.push({
                    id: Math.random(),
                    description: lessonText
                });
            }
        } );
    }

    notify(data: Lesson[]) {
        console.log('Lessons list component received data ..');
        this.lessons = data;
    }

    toggleLessonViewed(lesson:Lesson) {
        console.log('toggling lesson ...');
        lesson.completed = !lesson.completed;
    }

    delete(deleted:Lesson) {
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
                <input [checked]="lesson.completed" type="checkbox" (click)="toggleLessonViewed(lesson)">
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
import {globalEventBus, Observer, LESSONS_LIST_AVAILABLE, ADD_NEW_LESSON} from "../event-bus-experiments/eventbus.service";
import {Lesson} from "../shared/model/lesson";

@Component({
  selector: 'lessons-counter',
  templateUrl: './lessons-counter.component.html',
  styleUrls: ['./lessons-counter.component.css']
})
export class LessonsCounterComponent implements Observer {

    lessonsCounter = 0;

    constructor() {
        console.log('lesson list component is registered as observer ..');
        globalEventBus.registerObserver(LESSONS_LIST_AVAILABLE, this);

        globalEventBus.registerObserver(ADD_NEW_LESSON, {
            notify: lessonText => this.lessonsCounter += 1
        } );
    }

    notify(data: Lesson[]) {
        console.log('counter component received data ..');
        this.lessonsCounter = data.length;
    }

}

```

### Reactive Approach
1. Single Data Owner
2. Separate register/unregister observers from emit data/notify observers
3. Notify observers should be private
4. Make data as something that observers can subscribe to to observe changes and get notified about the change, say Observable

## Observer, Observable and Subject - Nuts and Bolts of Reactive Programming
> branch: observable-pattern
```ts
// Separate ability of being notified or emitting new data (next) from subcribe/unsubscribe
export interface Observer {
    next(data: any);    
    complete();
    error();
}
```

```ts
export interface Observable {
    subscribe(obs: Observer);
    unsubscribe(obs: Observer);
}
```

```ts
// Subject is private so that only owner of data can emit new data
interface Subject extends Observer, Observable  {
}    
```

## Observable, Observer and Subject in Action

> branch: introduce-rxjs, prepare-lessons ???

### app-data.service.ts
```ts
import * as _ from 'lodash';

class SubjectImpelementation implements Subject {
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

// Initialization
const lessons: Lesson[] = [];

// Private Subject
const lessonsListSubject = new SubjectImplementation();

// Public Observable
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
    lessons = _.cloneDeep(newList);
    lessonsListSubject.next(lessons);
}

```

## Refactoring: DataStore as Service

### app-data.service.ts
```ts
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

## Refactoring Imperative Style to Reactive Style

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
        console.log('Top level component broadcasted all lessons ...');
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
export class LessonsCounterComponent implements Observer, OnInit {

    lessonsCounter = 0;

    ngOnInit() {
        console.log('lesson list component is registered as observer ..');
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

### Updating Components:

```ts
...
// Change below
// store.lessonsList$.subscribe(this);
// to
store.subscribe(this);
```

## Replacing custom implementation with RxJS (Reactive Extensions)

> branch: introduce-rxjs

### app-data.service.ts
```ts
import * as _ from 'lodash';
import {BehaviorSubject, Observable, Observer} from 'rxjs';
import {lesson} from '../shared/model/lesson';

class DataStore {
    // No need of local state as the state can be maintained in the BehaviorSubject itself as it remembers the last emitted value.
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
1. Local state
2. Nested subscriptions
3. Subscriptions
4. Long lived observables causing memory leaks


## Stateless Service
1. No local states
2. No subscriptions
3. Return observables
        return observableAPICall()
            .do(data => subject.next(data));
4.  Short lived observable by unsubscribing after first notification using first()
5. Stateless services provide observables as streams of data/events that components can subscribe too.
6. Avoid multiple calls to observable by ensuring it completes before emitting the value using publishLast().refCount()

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

    <table class="courses-list card card-strong" *ngIf="courses$ | async as courses else loadingCourses">
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

    <table class="lessons-list card card-strong" *ngIf="latestLessons$ | async as lessons else loadingLessons">
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
- Stateful with state in private BehaviorSubject

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

### loginRoute.ts
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

    <table class="lessons-list card card-strong" *ngIf="latestLessons$ | async as lessons else loadingLessons">
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
        private UserService: UserService, 
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

    @Output() subbscribe = new EventEmitter();

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

## Avoid Nested Subscription using switchMap()

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
        private UserService: UserService, 
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
- No property and event bindings in parent components

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
        <button (click)="previousLessonsPage()">Previous</button>
        <button (click)="nextLessonsPage()">Next</button>
    </div>
    <lessons-list lessons="lesson$ | async"></lessons-list>
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
    private subject = new BehaviorSubject<Lesson[]>([]);
    lessonsPage$: Observable<Lesson[]> = this.subject.asObservable();
    currentPageNumber = 1;

    constructor(private http: Http)

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
            lessons => this.subject.next(lessons)
        );
    }
 

}
```

### server.ts
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

### lessonsRoute.ts
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


## Master Detail Implementation using Observable

> branch: master-detail

```ts
export interface Lesson {
    id:number;
    description:string;
    seqNo:number;
    duration:string;
    url?:string;
    tags?:string;
    pro?:boolean;
    longDescription:string;
    courseId?:string;
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
            <button (click)="previousLessonsPage()">Previous</button>
            <button (click)="nextLessonsPage()">Next</button>
        </div>
        <lessons-list lessons="lesson$ | async"></lessons-list>
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

    @Output() selected: EventEmitter<Lesson>;

    select(lesson) {
        this.selected.next(lesson);
    }

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
            <button (click)="previousLessonsPage()">Previous</button>
            <button (click)="nextLessonsPage()">Next</button>
        </div>
        <lessons-list lessons="lesson$ | async" (selected)="selectDetails($event)"></lessons-list>
    </ng-template>
</div>
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

    selectDetails(lesson: Lesson) {
        this.details$ = this.coursesService.getLessonDetails(lesson.url);
    }

    backToMaster() {
        this.details$ = undefined;
    }
}
```


## Just Pipelines of Streams of Data and Observers Reacting to Change in Data


## Stateful Service


## Error Handling


## Pre Fetching Data


## Global Loading Indicator


## Reactive Forms as Observable


