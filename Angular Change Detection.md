## Angular Change Detection

https://blog.eunsatio.io/develop/Angular%EC%9D%98-Change-Detection

**Angular 앱의 상태 변화 이벤트**

1. 이벤트 : click, submit 등
2. 비동기 요청 : XHR, fetch 등 외부 서버와 통신
3. 타이머 : setTimeout, setInterval

모두 비동기로 작동함.



위의 상태변화는 Zone이 처리한다. Angular에는 NgZone이라는 고유의 zone이 있다. NgZone의 onTurnDone 이벤트를 구독한 ApplicationRef이라는 클래스가 존재한다. ApplicationRef은 root component에서 어떤 async event/operations이 일어났을 때 ```tick()```을 실행시켜서 change detection을 한다. tick function은 ```detectChanges()```를 사용한 wrapper method다.



- ~~~javaScript
  detectchanges()
  ~~~

  호출 즉시 현재 컴포넌트부터 모든 자식컴포넌트까지 change detection을 수행

- ~~~javaScript 
  markForCheck() 
  ~~~

  change detection을 수행하진 않지만, change detection 이 필요한 부모 컴포넌트들에게 마킹을 함(checksEnabled로 view state를 변경). 다음 change detection이 일어났을 때, 마킹된 컴포넌트들까지 같이 change detection을 수행함.



Angular의 컴포넌트는 각각의 changeDetector를 가진다. CD(changeDetector)또한 컴포넌트와 같이 트리구조를 가진다.

기본적으로 Angular는 이벤트 발생 때마다 root에서 ```tick()```을 실행해서 모든 컴포넌트를 검사한다. 

Change Detection을 줄이기 위해 ```changeDetection : ChangeDetectionStrategy.OnPush```설정이 존재한다. 예를 들면

~~~javascript
@Component({
  template: `
    <h2>{{vData.name}}</h2>
    <span>{{vData.email}}</span>
  `,
  changeDetection: ChangeDetectionStrategy.OnPush
})
class VCardCmp {
  @Input() vData;
}
~~~

이 컴포넌트는 ```@Input```값이 변하지 않으면 change detection을 실행하지 않는다.

하지만 onPush전략을 유지하면서  ```@Input```프로퍼티가 없는 어떤 값의 변화가 있을 때 change detection을 발생시키고 싶을 수 있다. 이 때 ```markForCheck()```를 사용한다. 예를 들면

~~~javascript
constructor(private cd: ChangeDetectorRef) {}
 
ngOnInit() {
  this.addItemStream.subscribe(() => {
    this.counter++; // 앱 상태 변화
    this.cd.markForCheck(); // 경로 지정
  })
}
~~~

```markForCheck()```에 의해 모든 부모컴포넌트의 view state가 checksEnabled이 되었고, 이벤트가 끝나면 CD에 의해 change detection이 일어난다. change detection이 끝나면 다시 모든 컴포넌트가 onPush 상태로 돌아간다.





-----

https://blog.ninja-squad.com/2018/09/27/angular-performances-part-4/

OnPush는 기본적으로 @Input이 바뀌지 않으면 changeDetection을 하지 않지만, 몇가지 **트릭**이 존재한다.

1. observable에 async pipe 사용 (async는 Promise나 Observable에 subscribe하는데 쓰인다), DOM을 직접 변경하는 방법과 유사해보임(3번)

   ~~~javascript
   @Component({
     selector: 'ns-observable-on-push-with-async',
     template: `<img *ngIf="color | async as c"
                     [src]="'pony-' + c + '.gif'">`,
     changeDetection: ChangeDetectionStrategy.OnPush
   })
   export class PonyComponent {
     color: Observable<string>;
   
     constructor(colorService: ColorService) {
       this.color = colorService.get();
     }
   }
   ~~~

2. ChangeDetectorRef.detectChanges()를 호출. detach된 컴포넌트에 대해서도 마찬가지

   ~~~javascript
   @Component({
     selector: 'ns-clock',
     template: `
       <h2>Clock</h2>
       <p>{{ getTime() }}</p>
       <button (click)="start()">Start</button>
     `
   })
   export class ClockComponent implements OnDestroy {
     time: number;
     timeSubscription: Subscription;
   
     constructor(private ref: ChangeDetectorRef) {
       this.ref.detach();
     }
   
     start() {
       this.timeSubscription = interval(10).pipe(
         take(1001), // 0, 1, ..., 1000
         map(time => time * 10)
       ).subscribe(time => {
         this.time = time;
         // manually trigger the change detection every second
         if (this.time % 1000 === 0) {
           this.ref.detectChanges();
         }
       });
     }
   
     getTime() {
       return this.time;
     }
   
     ngOnDestroy() {
       this.timeSubscription.unsubscribe();
     }
   
   }
   ~~~

3. DOM을 직접 업데이트하는 방법. change detection과 무관하게 변화된 값을 보여줄 수 있다.

   ~~~javascript
   @Component({
     selector: 'ns-clock',
     template: `
         <h2>Clock</h2>
         <p #clock></p>
         <button (click)="start()">Start</button>
     `
   })
   export class ClockComponent implements OnDestroy {
     time: number;
     timeSubscription: Subscription;
     @ViewChild('clock') clock: ElementRef<HTMLParagraphElement>;
   
     constructor(private ref: ChangeDetectorRef) {
       this.ref.detach();
     }
   
     start() {
       this.timeSubscription = interval(10).pipe(
         take(1001), // 0, 1, ..., 1000
         map(time => time * 10)
       ).subscribe(time => {
         this.time = time;
         if (this.time % 1000 === 0) {
           this.clock.nativeElement.textContent = `${time}`;
         }
       });
     }
   
     ngOnDestroy() {
       this.timeSubscription.unsubscribe();
     }
   
   }
   ~~~

4. 코드를 zone의 밖에서 실행시키는 방법, 컴포넌트에서 특정 함수만 change detection과 무관하게 실행시키고 싶을때 유용하다.

   ~~~javascript
   constructor(private zone: NgZone) {
   }
   
   start() {
     this.zone.runOutsideAngular(() => {
       this.timeSubscription = interval(10).pipe(
         take(1001), // 0, 1, ..., 1000
         map(time => time * 10),
       ).subscribe(time => {
         this.time = time;
         if (this.time % 1000 === 0) {
           this.clock.nativeElement.textContent = `${time}`;
         }
       });
     });
   }
   ~~~

5. markForCheck를 사용하는 방법.

   ~~~~javascript
   @Component({
     selector: 'ns-pony',
     template: `<img [src]="getPonyImageUrl()">`,
     changeDetection: ChangeDetectionStrategy.OnPush
   })
   export class PonyComponent implements OnInit, OnDestroy {
   
     @Input() ponyModel: PonyModel;
     private intervalId: number;
   
     constructor(private ref: ChangeDetectorRef) {
     }
   
     ngOnInit() {
       this.intervalId = window.setInterval(() => {
         this.ponyModel.color = this.randomColor();
         this.ref.markForCheck();
       }, 1000);
     }
   
     ngOnDestroy(): void {
       window.clearInterval(this.intervalId);
     }
   }
   ~~~~

   

