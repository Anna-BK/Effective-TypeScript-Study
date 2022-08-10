# 아이템 21 타입 넓히기
런타임에 모든 변수는 유일한 값을 가짐.  
그러나 타입스크립트가 작성된 코드를 체크하는 정적 분석 시점에 변수는 '가능한'값들의 집합인 타입을 가지게 됨.  

상수를 사용해 변수를 초기화 할 때 타입을 명시하지 않으면 타입 체커는 타입을 결정해야 함.

>넓히기 widening
>---
>지정된 단일 값을 가지고 할당 가능한 값들의 집합을 유추하는 과정.  
>넓히기의 과정을 이해하면 오류의 원인을 파악하고 타입 구문을 더 효과적으로 사용할 수 있음.

```typescript
interface Vector3 { x: number; y: number; z: number; }
function getComponent(vector: Vector3, axis: 'x' | 'y' | 'z') {
  return vector[axis];
}
let x = 'x';
let vec = {x: 10, y: 20, z: 30};
getComponent(vec, x);
               // 'string' 형식의 인수는 '"x" | "y" | "z"'
               // 형식의 매개 변수에 할당될 수 없습니다.ts(2345)
```
위 코드는 런타임에 오류 없이 실행되지만, 편집기에서는 오류가 표시됨.  
`getComponent`함수는 두번째 매개변수에 `'x' | 'y' | 'z'`타입을 기대했지만 x의 타입은 넓히기를 통해 `string`으로 추론됨. `string`타입은 `'x' | 'y' | 'z'`타입에 할당이 불가능하므로 *오류* 발생.


타입 넓히기가 진행될 때 주어진 값으로 추론 가능한 타입이 여러개 -> 과정이 모호함
```typescript
const mixed = ['x', 1];
```
`mixed`의 타입이 될 수 있는 후보들
* ('x' | 1)
* ['x', 1]
* [string, number]
* readonly [string, number]
* (string|number)[]
* readonly (string|number)[]
* [any, any]
* any[]  
정보가 충분하지 않다면 mixed가 어떤 타입으로 추론되어야 하는지 알 수 없음.

위의 예제에서 타입스크립트는 다음과 같은 코드를 예상했기 때문에 x의 타입을 string으로 추론함.
```typescript
let x = 'x';
x = 'a';
x = 'Four score and seven years ago...';
```

타입스크립트는 넓히기 과정을 제어할 수 있도록 몇가지 방법을 제공
1. const를 통해 명시적 타입구문 제공
2. 타입 체커에 추가적인 문맥 제공
3. const 단언문 사용

>## 1. const
>```typescript
>const x = 'x';  // type is "x"
>let vec = {x: 10, y: 20, z: 30};
>getComponent(vec, x);  // OK
>```
> `const`를 사용하면 재할당을 할 수 없기 때문에 문자열보다 더 좁은 타입인 문자 리터럴 타입 'x'로 추론.  
>그러나 __`const`는 만능이 아님__  
>객체나 배열의 경우 여전히 문제가 있음
>

>## 2. 타입 체커에 추가적인 문맥 제공
>함수의 매개변수로 값을 전달 하는 행위 등.  
>자세한 내용은 아이템 26 참조.

>## 3. const 단언문 사용
>```typescript
>const v1 = {
>  x: 1,
>  y: 2,
>};  // Type is { x: number; y: number; }
>
>const v2 = {
>  x: 1 as const,
>  y: 2,
>};  // Type is { x: 1; y: number; }
>
>const v3 = {
>  x: 1,
>  y: 2,
>} as const;  // Type is { readonly x: 1; readonly y: 2; }
>```
>위 예제처럼 값 뒤에 `as const`를 붙이면 최대한 좁은 타입으로 추론하며, v3의 겨우 타입 넓히기가 동작하지 않았음.

넓히기로 인해 오류가 발생한다면, 명시적 타입 구문 또는 const 단언문을 추가하는 것을 고려.

---
# 요약
* 타입스크립트가 넓히기를 통해 상수의 타입을 추론하는 법을 이해해야 한다
* 동작에 영향을 줄 수 있는 방법인 const, 타입 구문 문맥, as const에 익숙해져야 함