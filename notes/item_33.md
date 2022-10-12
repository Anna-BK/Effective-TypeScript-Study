# 아이템 33 string 타입보다 더 구체적인 타입 사용하기
string 타입의 범위는 매우 넓다.

그렇기 때문에 string 타입으로 변수를 선언하려 한다면 __그보다 더 좁은 타입이 적절하지 않은지 검토해야__ 함.

---
음악 컬렉션을 만들기 위해 앨범의 타입을 정의 할 때
```typescript
interface Album {
  artist: string;
  title: string;
  releaseDate: string;  // YYYY-MM-DD
  recordingType: string;  // "live" || "studio"
}
```
* string타입이 남발되었음
* releaseDate에 다른 날짜 형식을 넣거나 recordingType에 대소문자를 잘못 지정하는 등 주석에 타입정보를 적은대로 사용하지 않을 수 있음  

또한 string타입의 범위가 넓기 때문에 제대로 된 Album 객체를 사용하더라도 매개변수 순서가 잘못된 것이 오류로 드러나지 않음  
```typescript
function recordRelease(title: string, date: string) { /* ... */ }
recordRelease(kindOfBlue.releaseDate, kindOfBlue.title);  // string이기 때문에 에러가 발생하지 않음
```
`recordRelease`함수 호출 시 매개변수의 순서가 바뀌었지만, 둘 다 문자열이기 때문에 타입 체커가 정상으로 인식함.  
이렇게 string이 남용된 코드를 __*문자열을 남발하여 선언stringly typed*__ 라 표현하기도 함.

`artist`나 `title`은 어떠한 문자열이 올 지 모르니 `string`타입이 적절함.  
그러나 `releaseDate`는 `Date`객체를 사용하여 날짜 형식으로만 제한하는 것이 좋음.  
`recordingType`필드는 `"live"`와 `"studio"` 단 두개의 값으로 유니온 타입을 정의 할 수 있음(enum 사용은 추천하지 않음 item53 참고).

```typescript
type RecordingType = 'studio' | 'live';

interface Album {
  artist: string;
  title: string;
  releaseDate: Date;
  recordingType: RecordingType;
}
```
위의 코드처럼 바꾸면 타입스크립트는 오류를 더 세밀하게 체크  
이러한 방식에는세가지 장점이 더 있음.  
### 1. 타입정보 유지
### 2. 타입을 명시적으로 정의
### 3. 객체의 속성 체크를 더욱 세밀하게 할 수 있음

>## 첫번째
>타입을 명시적으로 정의함으로써 다른 곳으로 값이 전달되어도 타입 정보가 유지됨.  
>특정 레코딩 타입의 앨범을 찾는 함수를 작성 한다면 ->
>```typescript
>function getAlbumsOfType(recordingType: string): Album[] {}
>```
>위와 같이 정의 할 수 있음
>위 코드에서는 `getAlbumsOfType`함수를 호출하는 곳에서 `recordingType`의 값이 `stirng`타입이어야 한다는 것 외에는 다른 정보가 없음.  
>주석으로 써 놓은 `"live"` || `"studio"`는 Album의 정의에 숨어 있고 함수를 사용하는 사람은 `recordingType`이 `"live"` 나 `"studio"`여야 한다는 것을 알 수 없음.

>## 두번째
>타입을 명시적으로 정의하고 해당 타입의 의미를 설명하는 주석을 붙여넣을 수 있음.
>```typescript
>/** 이 녹음은 어떤 환경에서 이루어졌는지?  */
>type RecordingType = 'live' | 'studio';
>```
>이렇게 주석을 붙이고 `getAlbumsOfType`이 받는 매개변수를 `string`에서 `RecordingType`으로 변경하면, 함수를 사용하는 곳에서 `RecordingType`의 설명을 볼 수 있음.

>## 세번째
>keyof 연산자로 더욱 세밀하게 객체의 속성 체크가 가능해짐.  
>__함수의 매개변수에 string을 잘못 사용하는 일은 흔한 일.__  
>어떠한 배열에서 한 필드의 값만 추출하는 함수를 작성한다고 할 때
>```javascript
>function pluck(record, key) {
>  return record.map(r => r[key]);
>}
>```
>위 함수의 시그니처를 다음처럼 작성 할 수 있음.
>```typescript
>function pluck(record: any[], key: string): any[] {
>  return record.map(r => r[key]);
>}
>```
>타입 체크가 되긴 하지만 any 타입이 있어서 정밀하지 못함.  
>또한 반환 값에 any를 사용하는 것은 매우 좋지 않은 설계임.  
>*타입 시그니처를 개선하는 첫 단계로 제너릭을 도입*하면
>```typescript
>function pluck<T>(record: T[], key: string): any[] {
>  return record.map(r => r[key]);
>                   //책에서는 에러가 발생한다고 하는데 에러가 발생하지 않았음
>}
>```
>타입스크립트는 key의 타입이 `string`이기 때문에 범위가 너무 넓다는 오류를 발생시킴.
>`Album`의 배열을 매개변수로 전달하면 기존의 `string`타입의 넓은 범위와는 반대로  
>`"artist"`, `"title"`, `"releaseDate"`, `"recordingType"`
>단 네개의 값만이 유효하게 됨.  
>```typescript
>// "artist" | "title" | "releaseDate" | "recordingType"
>type K = keyof Album;
>```
>그러므로 `key: string`을 `key: keyof T`로 바꾸면 됨.
>```typescript
>type RecordingType = 'studio' | 'live';
>
>interface Album {
>  artist: string;
>  title: string;
>  releaseDate: Date;
>  recordingType: RecordingType;
>}
>function pluck<T>(record: T[], key: keyof T) {
>  return record.map(r => r[key]);
>}
>```
>위 코드에서 추론된 타입은 `function pluck<T>(record: T[], key: keyof T): T[keyof T][]`이며  
>`T[keyof T]`는 T객체 내의 가능한 모든 값의 타입.  
>__*그런데 key의 값으로 하나의 문자열을 넣게 되면, 그 범위가 너무 넓어서 적절한 타입이라고 보기 어려움.*__  
>  
>예를들어
>```typescript
>const releaseDates = pluck(albums, 'releaseDate');
>```
>위 코드에서 `releaseDates`의 추론된 타입은 `const releaseDates: (string | Date)[]`지만 `Date[]`여야 함.  
>`keyof T`는 `string`에 비하면 훨씬 범위가 좁기는 하지만 그래도 여전히 넓음.
>따라서 범위를 더 좁히기 위해 두번째 제너릭 매개변수를 도입해야 함.
>```typescript
>function pluck<T, K extends keyof T>(record: T[], key: K): T[K][] {
>  return record.map(r => r[key]);
>}
>```
>매개변수 타입이 정밀해진 덕분에 Album의 키에 자동완성 기능을 제공 할 수 있게 해 줌.

* string은 any와 비슷한 문제를 가지고 있음.  
* 잘못 사용하게 되면 무효한 값을 허용하고 타입 간의 관계도 감추어 버림.  
* 타입 체커를 방해하고 실제 버그를 찾기 힘들게 함.



---
# 요약
* '문자열을 남발하여 선언된' 코드를 피하자. `string`보다 더 구체적인 타입을 사용하는게 좋음.
* 변수의 범위를 보다 정확하게 표현하고 싶다면 `string`타입보다는 문자열 리터럴 타입의 유니온을 사용.
* 객체의 속성 이름을 함수 매개변수로 받을 때는 `string`보다 `keyof T`를 사용하는게 좋다.