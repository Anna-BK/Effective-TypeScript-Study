# Item 26. 타입 추론에 문맥이 어떻게 사용되는지 이해하기

자바스크립트에서 표현식을 상수로 분리할 수 있으며, 타입스크립트에서 리팩터링도 동작한다.  
```javascript
    // 인라인 형태
    setLanguage('JavaScript');

    // 참조 형태
    let language = 'JavaScript';
    setLanguage(language);
```
하지만 문자열 타입을 **문자열 리터럴 타입의 유니온**으로 변경하면 오류가 발생한다. *(Item 33 참고)*  
```typescript
    type Language = 'JavaScript' | 'TypeScript' | 'Python';
    function setLanguage(language: Language) { /* ... */ }

    setLanguage('JavaScript');          // ok

    let language = 'JavaScript';
    setLanguage(language);
             // ~~~~~~~~ 'string' 형식의 인수는
             //          'Language' 형식의 매개변수에 할당할 수 없습니다.
```
인라인 타입의 경우, 해당 타입이 문자열 리터럴인 *JavaScript*이므로 할당이 가능합니다.  
#### ✔️ 그러나 이 값을 변수로 분리하면 타입스크립트는 할당 시점에 타입을 추론한다.   

위의 경우 `string`으로 추론했고, `Language` 타입으로 할당이 불가능하므로 오류가 발생하였다.  
이러한 문제를 해결하는 두 가지 방법이 있다.  

### 1️⃣ 타입 선언에서 가능한 값 제한하기
```typescript
    let language: Language = 'JavaScript';
    setLanguage(language);                      // ok
```
타입 선언에서 `language`의 가능한 값을 제한할 경우, 오타가 있을 경우 오류를 표시해주는 장점도 가진다.  

### 2️⃣ 변수를 상수로 선언하기
```typescript
    const language = 'JavaScript';
    setLanguage(language);                      // ok
```
`const`를 사용했기 때문에 타입 체커에게 `language`는 변경할 수 없다고 알려준다.  
때문에 타입스크립트는 `language`를 문자열 리터럴로 추론할 수 있다.  
*JavaScript*는 `language`에 할당할 수 있으므로 타입 체크를 통과하며, `language`를 재할당한다면 타입 선언이 필요하다. *(Item 21 참고)*  
  
사용되는 문맥으로부터 값을 분리하게 되면 추후 근본적인 문제를 발생시킬 수 있다.  
이러한 오류가 발생하는 몇 가지 경우와, 해결 방안은 다음과 같다.  

 ## ⚠️ 튜플 사용 시 주의점
 ```typescript
    // 매개변수는 (latitude, longitude) 쌓입니다.
    function panTo(where: [number, number]) { /* ... */ }

    panTo([10, 20]);            // ok

    const loc = [10, 20];
    panTo(loc);
       // ~~~~ 'number[]' 형식의 인수는
       //      `[number, number]' 형식의 매개변수에 할당될 수 없습니다.
 ```
`loc` 변수의 경우, 타입스크립트가 `number[]`로 추론한다.  
즉, 길이를 알 수 없는 숫자의 배열이므로 튜플 타입에 할당할 수 없다.  
   
`any`를 사용하는 대신 `const`를 사용할 수 있지만, 이미 `loc`는 상수이므로 **타입 선언**을 제공해야 한다.  
```typescript
    const loc: [number, number] = [10, 20];
    panTo(loc);         // ok
```
또다른 방법은 **삼수 문맥**을 제공하는 것이다.  
`const`는 단지 값이 가리키는 참조가 변하지 않는 *얕은(shallow) 상수*인 반면, `as const`는 *내부까지(deeply) 상수*라는 것을 타입스크립트에게 알려준다. 하지만 타입 단언은 너무 과하게 정확하기 때문에, `loc`의 매개변수가 `readonly`이라고 추론한다.  
따라서 `panTo` 함수에 `readonly` 구문을 추가하는 것이 최선의 해결책이다.  
```typescript
    function panTo(where: readonly [number, number]) { /* ... */ }

    const loc = [10, 20] as const;
    panTo(loc);
```
타입 시그니처를 수정할 수 없다면 **타입 구문을 사용해야 한다**.  
타입 구문은 타입 정의에 실수가 있을 때 오류가 타입 정의가 아니라 호출되는 곳에서 발생하는 단점을 가진다.  
```typescript
    const loc = [10, 20, 30] as const;
    panTo(loc);
       // ~~~ 'readonly [10, 20, 30]' 형식의 인수는 'readonly [number, number]' 형식의 매개변수에 할당될 수 없습니다.
       //     'length' 속성의 형식이 호환되지 않습니다.
       //     '3' 형식은 '2' 형식에 할당할 수 없습니다.
```

 ## ⚠️ 객체 사용 시 주의점
 ```typescript
    type Language = 'JavaScript'  | 'TypeScript' | 'Python';
    interface GovernedLanguage {
        language: Language;
        organization: string;
    }

    function complain(language: GovernedLanguage) { /* ... */ }

    complain({ language: 'TypeScript', organization: 'Microsoft' });        // ok

    const ts = {
        language: 'TypeScript',
        organization: 'Microsoft',
    };
    complain(ts);
          // ~~ '{ language: string; organization: string }' 형식의 인수는
          //    'GovernedLanguage' 형식의 매개변수에 할당될 수 없습니다.
          //    'language' 속성의 형식이 호환되지 않습니다.
          //    'string' 형식은 'Language' 형식에 할당할 수 없습니다.
 ```
`ts` 객체에서 `language`의 타입은 `string`으로 추론된다.  
위 문제는 **타입 선언을 추가**하거나 **상수 단언**을 사용해 해결할 수 있다. *(Item 9 참고)*  

## ⚠️ 콜백 사용 시 주의점
콜백을 다른 함수로 전달할 경우, 타입스크립트는 콜백의 매개변수 타입을 추론하기 위해 문맥을 사용한다.
```typescript
    function callWithRandomNumbers(fn: (n1: number, n2: number) => void) {
        fn(Math.random(), Math.random());
    }

    callWithRandomNumbers((a, b) => {
        a;  // 타입이 number
        b;  // 타입이 number
        console.log(a + b);
    });
```
`callWithRandomNumbers`의 타입 선언으로 인하여 `a`와 `b`의 타입이 `number`로 추론된다.  
콜백을 상수로 뽑아내면 문맥이 소실되기 때문에 *noImplicitAny* 오츄가 발생하게 된다.  
```typescript
    const fn = (a: number, b: number) => {
        console.log(a + b);
    }
    callWithRandomNumbers(fn);
```
이러한 문제는 매개변수에 타입 구문을 추가해서 해결할 수 있다.
또는 가능한 경우 전체 함수 표현식에 타입 선언을 적용하는 것이다. *(Item 12 참고)*

## 📝 요약
- 타입 추론에서 문맥이 어떻게 쓰이는지 주의해서 살펴봐야 한다.
- 변수를 뽑아서 별도로 선언할 경우 오류가 발생한다면 타입 선언을 추가해야 한다.
- 변수가 정말로 상수라면 상수 단언(as const)을 사용해야 한다. 그러나 상수 단언을 사용하면 정의한 곳이 아니라 사용한 곳에서 오류가 발생하므로 주의해야 한다.