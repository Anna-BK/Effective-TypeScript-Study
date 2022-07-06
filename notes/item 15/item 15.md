# 아이템 15

## 동적인 데이터에 인덱스 시그니처 사용하기

---
### 요약
    1. 런타임 까지 객체의 속성을 알 수 없을 경우에만(ex. CSV 파일에서 로드) 인덱스 시그니처 사용
    2. 안전한 접근을 위해 인덱스 시그니처의 값 타입에 undefined 추가 권장
    3. 가능하면 인덱스 시그니처보다 정확한 타입을 사용하는 인터페이스, Record, 매핑된 타입 이용

---

### 인덱스 시그니처 설명 및 단점

자바스크립트 객체는 문자열 키를 타입의 값에 관계없이 매핑

→ 타입스크립트는 타입에 ‘인덱스 시그니처’를 명시해 유연하게 매핑을 표현할 수 있음

```jsx
const rocket = {
	name: 'Falcon 9',
	variant: 'Blcok 5',
	thrust: '7,607 kN'
};
```

```tsx
type Rocket = {[Property: string]: string};
const rocket: Rocket = {
	name: 'Falcon 9',
	variant: 'Blcok 5',
	thrust: '7,607 kN'
}
```

[Property: string]: string 이 부분이 인덱스 시그니처이고 세 가지 의미를 가짐

1. 키의 이름 : 키의 위치만 표시하는 용도(타입 체커에서는 사용하지 않음
2. 키의 타입 : string이나 number 또는 symbol의 조합이어야 하지만 보통은 string 사용
3. 값의 타입 : 어떤 것이든 될 수 있음

타입 체크가 수행 후 발견할 수 있는 인덱스 시그니처의 단점

1. 잘못된 키를 포함해 모든 키가 허용됨

    name 대신 Name으로 작성해도 유효
    
2. 특정 키가 필요하지 않음
    
    {}도 유효한 Rocket 타입
    
3. 키마다 다른 타입을 가질 수 없음
    
    thrust는 string이 아니라 number여야 할 수도 있는데 그럴 수 없게 됨
    
4. 타입스크립트의 언어 서비스가 도움이 되지 않는 경우가 존재
    
    name: 을 입력할 때 키는 무엇이든 가능하므로 자동완성 기능이 동작하지 않음
    

**※ 인덱스 시그니처는 부정확하므로 더 나은 방법을 찾아야 함※**

```tsx
interface Rocket {
	name: string;
	variant: string;
	thrust_kN: number;
}

const falconHeavy: Rocket = {
	name: 'Falcon Heavy';
	variant: 'v1';
	thrust_kN: 15_2000;
}

// 자동완성, 정의로 이동, 이름 바꾸기 등이 모두 동작
```

thrust_kN의 타입은 number이고 타입스크립트는 모든 필수 필드가 존재하는지 확인

---

### 인덱스 시그니처의 사용 : 동적 데이터 표현

예를 들어 CSV 파일로 헤더 행(row)에 열(column) 이름이 있고, 데이터 행을 열 이름으로 매핑하는 객체로 나타내고 싶은 경우

```tsx
function parseCSV(input: string) : ({columnName: string]: string}[] {
	const lines = input.split('\n');
	const [header, ... rows] = lines;
	const headerColumns = header.split(',');
	return rows.map(rowStr => {
		const row: {[columnName: string]: string} = {};
		rowStr.split(',').forEach((cell,i) => {
			row[headerColumns[i]] = cell;
		});
		return row;
	});
}
```

일반적인 상황에선 열 이름이 무엇인지 알 수 없음 ⇒ 인덱스 시그니처 사용

열 이름을 알고 있는 특정한 상황에 parseCSV 사용 ⇒ 미리 선언해둔 타입으로 단언문 사용

```tsx
interface ProductRow {
	productId: string,
	name: string,
	price: string
}

declare let csvData: string
const products = parseCSV(csvData) as unknown as ProductRow[];
```

선언해 둔 열들이 런타임에 실제로 일치한다는 보장은 없음 → 값 타입에  undefined 추가

```tsx
function safeParseCSV(
	input: string
): {[columnName; string]: string | undefined}[] {
	return parseCSV(input);
}
```

이제 모든 열의 undefined 여부를 체크하게 됨

```tsx
const rows = parseCSV(csvData);
const prices: {[product: string]: number} = {};
for (const row of rows) {
	prices[row.productId] = Number(row.price);
}

const safeRows = safeParseCSV(csvData);
for (const row of safeRows) {
	prices[row.productId] = Nubmer(row.price);
	// 'undefined' 형식을 인덱스 형식으로 사용할 수 없습니다.
}
```

연관배열의 경우, 인덱스 시그니처 대신 Map 타입 사용을 고려(프로토타입 체인 관련 문제 우회)

---

### 인덱스 시그니처의 대안

어떤 타입에 가능한 필드가 제한되어 있는 경우라면 인덱스 시그니처로 모델링하면 안 됨

예를 들어  데이터에 A, B, C, D 같은 키가 있지만 얼마나 많이 있는지 모른다면 선택적 필드 또는 유니온 타입으로 모델링하면 됨

```tsx
interface Row1 { [column: string]: number} // 너무 광범위
interface Row2 { a: number; b?: number; c?: number; d?: nubmer} // 최선
type Rows3 = 
| {a: number;}
| {a: number; b:number;}
| {a: number; b:number; c:number;}
| {a: number; b:number; c:number; d:number}; // 가장 정확하지만 사용하기에 번거로움
```

string 타입이 너무 광범위해서 인덱스 시그니처 사용하는 데 문제가 있을 때의 대안 두 가지

1. Record 사용(키 타입에 유연성을 제공하는 제너럭 타입 → string의 부분 집합 사용 가능)
    
    ```tsx
    type Vec3D = Record<'x' | 'y' | 'z', number>;
    // Type Vec3D = {
    // 	x: number;
    // 	y: number;
    // 	z: number;
    // }
    ```
    
2. 매핑된 타입을 사용 → 매핑된 타입은 키마다 별도의 타입을 사용하게 해 줌
    
    ```tsx
    type Vec3D = {[k in 'x' | 'y' | 'z']: number};
    // Type Vec3D = {
    // 	x: number;
    // 	y: number;
    // 	z: number;
    // }
    
    type Vec3D = {[k in 'a' | 'b' | 'c']: k extends 'b'? string: number};
    // Type ABC = {
    // 	a: string;
    // 	b: string;
    // 	c: string;
    // }
    ```