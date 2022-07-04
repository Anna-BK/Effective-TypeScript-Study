# 아이템 11 잉여 속성 체크의 한계 인지하기
타입스크립트는 타입이 명시된 변수에 객체 리터럴을 할당 할 때   
1. 해당 타입의 속성이 있는지
2. 그 외의 속성은 없는지

확인한다.
>```typescript
>interface Room {
>  numDoors: number;
>  ceilingHeightFt: number;
>}
>const r: Room = {
>  numDoors: 1,
>  ceilingHeightFt: 10,
>  elephant: 'present',
>// ~~~~~~~~~~~~~~~~~~
>// '{ numDoors: number; ceilingHeightFt: number; elephant: string; }' 형식은 'Room' 형식에 할당할 수 없습니다.
>// 개체 리터럴은 알려진 속성만 지정할 수 있으며 'Room' 형식에 'elephant'이(가) 없습니다.ts(2322)
>};
>```
>구조적 타입 시스템에서 발생할 수 있는 오류를 잡을 수 있도록 *잉여속성 체크* 과정 수행


하지만 임시변수를 도입하면 obj객체를 Room 타입에 할당 가능 
>```typescript
>interface Room {
>  numDoors: number;
>  ceilingHeightFt: number;
>}
>const obj = {
>  numDoors: 1,
>  ceilingHeightFt: 10,
>  elephant: 'present',
>};
>const r: Room = obj;  // OK
>```
>obj의 타입은
>`
>{ numDoors: number; ceilingHeightFt: number; elephant: string }
>`으로 추론됨.   
>obj 타입은 Room 타입의 부분집합을 포함하므로 할당 가능하며 타입체커 통과.

## 위 예제의 차이점
* 첫번째 예제는 '잉여 속성 체크' 과정 수행.
* '잉여속성 체크'는 조건이 정해져 있기 때문에 두번째 예제에서는 동작하지 않음.<sup>[1](#footnote_1 "객체 리터럴에서만 동작.")</sup><a name="footnote1_return"></a>
* __*잉여 속성 체크와 할당 가능 검사는 별도의 과정*__

*타입스크립트는 의도와 다르게 작성된 코드를 찾으려 함*
```typescript
interface Room {
  numDoors: number;
  ceilingHeightFt: number;
}
function setDarkMode() {}
interface Options {
  title: string;
  darkMode?: boolean;
}
function createWindow(options: Options) {
  if (options.darkMode) {
    setDarkMode();
  }
  // ...
}
createWindow({
  title: 'Spider Solitaire',
  darkmode: true
// ~~~~~~~~~~~~~
//'{ title: string; darkmode: boolean; }' 형식의 인수는 'Options' 형식의 매개 변수에 할당될 수 없습니다.
//개체 리터럴은 알려진 속성만 지정할 수 있지만 'Options' 형식에 'darkmode'이(가) 없습니다.
//'darkMode'을(를) 쓰려고 했습니까?ts(2345)
});
```
위 코드 실행 시 런타임에서는 오류가 발생하지 않지만, 의도한 대로 동작하지는 않음.

```typescript
interface Room {
  numDoors: number;
  ceilingHeightFt: number;
}
function setDarkMode() {}
interface Options {
  title: string;
  darkMode?: boolean;
}
const o1: Options = document;  // OK
const o2: Options = new HTMLAnchorElement;  // OK
const intermediate = { darkmode: true, title: 'Ski Free' };
const o: Options = intermediate;  // OK
const o3 = { darkmode: true, title: 'Ski Free'} as Options; // OK
```
* 변수, 타입 단언(as) 사용시에는 체크가 되지 않는 것을 확인 할 수 있음
* 잉여 속성 체크를 원하지 않을 때는 인덱스 시그니처<sup>[2](#footnote_2 "[key:string]: any; 의 형태로 타입 선언")</sup><a name="footnote2_return"></a>를 사용   
---
* 약한(weak) 타입<sup>[3](#footnote_3 "key?: string; 의 형태로 타입 선언")</sup><a name="footnote3_return"></a>에도 잉여 속성 체크와 비슷한 동작인 *공통 속성 체크*가 일어남.
```typescript
interface LineChartOptions {
  logscale?: boolean;
  invertedYAxis?: boolean;
  areaChart?: boolean;
}
const opts = { logScale: true };
const o: LineChartOptions = opts;
   // ~ '{ logScale: boolean; }' 유형에 'LineChartOptions' 유형과 공통적인 속성이 없습니다.ts(2559)
```
> ## 공통 속성 체크와 잉여 속성 체크의
>> ### 공통점   
>> 오타를 잡는 데 효과적이며 구조적으로 엄격하지 않음.
>
>> ### 차이점  
>> 공통 속성 체크: 약한 타입과 관련된 할당문마다 수행.   
>> 잉여 속성 체크: 오직 객체 리터럴에만 적용.

---
# 요약
* 객체 리터럴을 변수에 할당하거나 함수에 매개변수로 전달할 때 잉여 속성 체크가 수행.
* 잉여 속성 체크는 구조적 할당 가능성 체크와 역할이 다름.
* 잉여 속성 체크에는 한계가 있음.
---
<a name="footnote_1">[1](#footnote1_return)</a>| 객체 리터럴에서만 동작. 엄격한 리터럴 체크라고도 한다.  
<a name="footnote_2">[2](#footnote2_return)</a>| `[key:string]: any;` 의 형태로 타입 선언  
<a name="footnote_3">[3](#footnote3_return)</a>| `key?: string;` 의 형태로 타입 선언