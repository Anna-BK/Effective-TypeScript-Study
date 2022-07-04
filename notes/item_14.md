# 아이템 14. 타입 연산과 제너릭 사용으로 반복 줄이기 
B 타입을 정의하기 위해서 A 타입의 property : type 을 B타입에 매핑하여 재사용하고자 할 때가 있는데, 이때 타입스크립트에서 사용할 수 있는 방법들을 알아보자.

## 맵드 타입
```typescript
type TopNavState = {
	[k in 'userId' | 'pageTitle' | 'recentFiles'] : string;
};
```
결과물

```typescript
type TopNavState = {
	userId : string;
    pageTitle : string;
    recentFiles : string;
}
```
k 는 properties들(배열)을 차례로 돌며 property의 이름을 설정하고 해당 property의 타입을 string으로 지정한다.

 
```typescript
type TopNavState = {
	[k in 'userId' | 'pageTitle' | 'recentFiles'] : State[k]
};

type State = {
	userId : string;
    pageTitle : string;
    recentFiles : string[];
    pageContents : string;
};
```
위에서 string이라는 타입 대신에 State[k]를 썼다. State[k]는 'State에서 k property의 타입'을 의미하는 것이므로 결과는 아래와 같다.
```typescript
type TopNavState = {
	userId : string;
    pageTitle : string;
    recentFiles : string[];
}
```
즉, State[k]를 이용하면 동적으로 (State에 있는) 타입을 설정할 수 있다. 

 


## 제네릭 이용하기 
용례 : 부모 A 타입의 일부인 자식 B 타입을 만들때

```typescript
type TopNavState = {
	[k in 'userId' | 'pageTitle' | 'recentFiles'] : State[k]
};
```
위에서 State의 일부인 TopNavState 타입 생성에 사용된 코드를 아래처럼 쓸 수도 있다.
```typescript
type TopNavState = Pick<State, 'userId' | 'pageTitle' | 'recentFiles'>;

type Pick <T, K> = { [k in K] : T[k] };
```

Pick은 State에서 명시된 property들('userId' | 'pageTitle' | 'recentFiles')과 이들의 타입을 이용해 새로운 타입을 생성하겠다는 뜻으로 해석될 수 있다.

 

Pick은 새로운 타입을 만드는 함수처럼 사용되는데 이를 **제네릭 타입**이라고 부른다.

 

 

 

## 인덱싱 이용하기
우리는 아래처럼 여러 종류에 해당하는 aAction 타입, bAction 타입의 상위 타입(인터페이스 개념)을 유니온 연산자를 이용해 정의하곤 한다.

```typescript
interface SaveAction {
	type : 'save',
    ...
}
interface LoadAction {
	type : 'load',
    ...
}

type Action = SaveAction | LoadAction;
```
어떤 코드에서 저 Action들의 type을 사용하는 경우가 있다면 타입을 지정해줘야할 것이다. 이때, Action 타입에 인덱싱을 적용 (Action[type])하여 'save' | 'load' 타입을 생성할 수 있다.

```typescript
const somefunc = function (type : Action[type] ){ .... }

const somefunc = function (type : 'save' | 'load' ){ .... }
```

## keyof  타입A
타입A의 모든 property들의 유니온을 반환한다.
```typescript
interface Options {
	width : number;
    height : number;
    color : string;
    label : string;
};

type OptionsUpdate = { [k in keyof Options]? : Options[k] };

type OptionsKeys = keyof Options; // "width" | "height" | "color" | "label" 

interface OptionsUpdate {
	width? : number;
    height? : number;
    color? : string;
    label? : string;
};
```
타입 A의 모든 속성을 옵션으로 가지는 새로운 타입 B를 만드는 경우가 많기 때문에, Partial이라는 제네릭 타입을 많이 사용한다.

위의 예시에서는 OptionsUpdate를 Partial\<Options\>로 정의할 수 있다.

 

## typeof 값
값으로 부터 타입을 만들어 낸다.

```typescript
const INIT_OPTIONS = {
	width : 640,
    height : 480,
    color : "#00FF00",
    label : 'VGA'
 }
 
 type Options = typeof INIT_OPTIONS;
 
 type Options {
 	width : number,
    height : number,
    color : string,
    label : string
}
```

## 제네릭 타입 : ReturnType<함수타입>
여기서 함수타입이란 ()=>void 와 같은 것을 예시로 들 수 있다.

어떤 함수의 return 값을 타입으로 만들고자 할때 사용한다.

```typescript
function getUserInfo(userId : string) {
	///
    return {
    	userId,
        name,
        age,
        height,
        weight
    };

type UserInfo = ReturnType<typeof getUserInfo>;
```
UserInfo는 { userId : string, name : string, .... } 과 같이 getUserInfo에서 반환되는 타입일 것이다. 

 

 

## 제네릭 타입 : 매개변수 제한에는 extends를 사용하자
함수에서 파라미터 제한에 타입을 사용하듯, 타입을 생성하는 함수인 제네릭 타입도 extends를 사용하여 매개변수 제한을 할 수 있다.


앞에서 나온 Pick도 사실은 아래처럼 k가 반드시 T의 property에 속해야하므로 아래처럼 정의되어야 한다.

```typescript
type Pick <T, K extends T> = {
	[k in K] : T[k]
};
```
 