# 아이템 10 객체 래퍼 타입 피하기

기존 Javascript의 기본형
> string, number, boolean, null, undefined, [symbol](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Symbol "심볼은 고유식별자(UID) 생성 시 사용. 객체의 key로 사용하면 기본적으로 노출되지 않는 특성이 있음."), [bigint](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/BigInt "BigInt는 Number 원시 값이 안정적으로 나타낼 수 있는 최대치인 2^53 - 1보다 큰 정수를 표현할 수 있는 내장 객체이며, Math객체의 메소드와 사용 불가. Number와 혼합사용 불가.")

기본형은 불변(immutable)<sup>[1](#footnote_1 "callByValue. 할당, 복제 시 원본의 값을 사용.")</sup><a name="footnote1_return"></a>하며 메소드를 가지지 않음.  
하지만 string 타입의 데이터는 메소드를 가지고 있는것처럼 보임.
```javascript
> 'primitive'.charAt(3);  //실제로 'primitive'는 string이지만 String객체로 wrapping되어 사용됨.
"m"
```
`'primitive'`의 charAt은 String객체로 wrapping 된 객체의 메소드이며 사용 후에는 wrapping 된 String 객체를 버림.  
String.prototype을 몽키패치<sup>[2](#footnote_2 "본래는 게릴라 패치. runtime 동안 사용되는 모듈이나 클래스를 변경하는 것.")</sup><a name="footnote2_return"></a> 하면 string이 String으로 wrapping 되는 내부 동작을 관찰 할 수 있음.
```javascript
const originalCharAt = String.prototype.charAt;
String.prototype.charAt = function(pos) {
  console.log(this, typeof this, pos);
  return originalCharAt.call(this, pos);
};
console.log('primitive'.charAt(3));
```
위의 코드는 `[String: 'primitive'] 'object' 3`, `m`을 차례로 출력함.  
`this`를 통해 `'primitive'`가 `[String: 'primitive']`처럼 wrapping 된 것을 확인.  

하지만 `string`기본형과 `String` 객체 래퍼가 __*항상 동일하게 동작하지는 않음*__<sup>[3](#footnote_3 "String 객체는 오직 자기 자신하고만 동일함.")</sup><a name="footnote3_return"></a>.  

또한 기본형에 다른 속성을 할당하는 경우 그 속성이 사라지게 됨<sup>[4](#footnote_4 "기본형을 래퍼타입으로 변환 후 속성 할당 후 그 래퍼가 소멸되는 것.")</sup><a name="footnote4_return"></a>.  

>### 기본형의 객체 wraper 타입
>기본형|객체 wraper
>-|-
>number|Number
>boolean|Boolean
>symbol|Symbol
>bigint|BigInt

타입스크립트는 기본형과 객체 래퍼 타입을 별도로 모델링.  
*`string`사용 시 유의해야 함.*
```typescript
function isGreeting(phrase: String) {
  return [
    'hello',
    'good day'
  ].includes(phrase);
          // ~~~~~~
          // 'String' 형식의 인수는 'string' 형식의 매개 변수에 할당될 수 없습니다.
          // 'string'은(는) 기본 개체이지만 'String'은(는) 래퍼 개체입니다.
          // 가능한 경우 'string'을(를) 사용하세요.ts(2345)
}
```
>`string` -> `String` 할당 가능⭕  
>`String` -> `string` *할당 불가능*❌

### 타입 선언은 전부 기본형 타입으로 되어있음.
래퍼 객체는 타입 구문의 첫 글자를 대문자로 표기하는 방법으로도 사용할 수 있음.
```typescript
const s: String = "primitive";
const n: Number = 12;
const b: Boolean = true;
```
__*그러나 기본형 타입을 객체 래퍼에 할당하는 구문은 오해하기 쉽고, 굳이 그렇게 할 필요가 없음(item 19 참조)*__  
`new` 없이 `BigInt`와 `Symbol`을 호출하는 경우는 기본형을 생성하기 때문에 사용해도 됨.

---
# 요약
* 객체 래퍼를 직접 사용하거나 인스턴스를 생성하는 행위 지양
* 기본형 타입 지정 시 기본형 타입만을 사용
---
<a name="footnote_1">[1](#footnote1_return)</a>| callByValue. 할당, 복제 시 원본의 값을 사용.  
<a name="footnote_2">[2](#footnote2_return)</a>| 본래는 게릴라 패치. runtime 동안 사용되는 모듈이나 클래스를 변경하는 것.  
<a name="footnote_3">[3](#footnote3_return)</a>| `String` 객체는 오직 자기 자신하고만 동일함.  
<a name="footnote_4">[4](#footnote4_return)</a>| 기본형을 래퍼타입으로 변환 후 속성 할당 후 그 래퍼가 소멸되는 것.