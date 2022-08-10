# 아이템 20 다른 타입에는 다른 변수 사용하기
* 자바스크립트에서 변수는 다른 타입으로 재사용 가능
* 하지만 타입스크립트에서는 __두가지 오류__ 발생

```typescript
function fetchProduct(id: string) {}
function fetchProductBySerialNumber(id: number) {}
let id = '12-34-56';
fetchProduct(id);

id = 123456;
// 'number' 형식은 'string' 형식에 할당할 수 없습니다.ts(2322)
fetchProductBySerialNumber(id);
              //'string' 형식의 인수는 'number' 형식의 매개 변수에 할당될 수 없습니다.ts(2345)
```

이는 타입스크립트에서 '12-34-56'을 `string`으로 추론했기 때문.

__*변수의 값은 바뀔 수 있지만 그 타입은 보통 바뀌지 않는다.*__

```typescript
let id: string|number = "12-34-56";
fetchProduct(id);

id = 123456;  // OK
fetchProductBySerialNumber(id);  // OK
```
타입을 확장하면 에러 없이 사용 가능.  
`string|number`로 할당문에서 union type으로 범위를 좁혀 사용 가능함.

__union type은 string이나 number같은 간단한 타입에 비해 다루기가 어려움__  
*차라리 별도의 변수를 도입하는 것이 낫다.*

```typescript
const id = "12-34-56";
fetchProduct(id);

const serial = 123456;  // OK
fetchProductBySerialNumber(serial);  // OK
```

>변수를 별도로 분리하는 이유
>* 서로 관련이 없는 두 개의 값을 분리
>* 변수명을 더 구체적으로 지을 수 있음
>* 타입 추론을 향상, 타입 구문이 불필요해짐
>* 타입이 간결하게 됨
>* let 대신 const를 사용하여 코드를 간결하게 하고, 타입 체커가 타입을 추론하기 쉽게 함

타입이 바뀌는 변수는 피해야 하며, 목적이 다른 곳에는 별도의 변수명을 사용.
***
단, 가려진 변수와 혼동하지 말 것
> ```typescript
>const id = '12-34-56';
>fetchProduct(id);
>
>{
>   const id = 123456; // OK
>   fetchProductBySerialNumber(id); // OK
>}
>```
> 하지만 보통 이런 코드는 혼란을 야기하기 때문에 사용하지 않음.

---
# 요약
* 변수의 값은 바뀔 수 있지만 타입은 일반적으로 바뀌지 않음.
* 혼란을 막기 위해 타입이 다른 값을 다룰 때에는 변수를 재사용하지 않도록 한다.