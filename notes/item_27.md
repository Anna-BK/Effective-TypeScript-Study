# Item 27. 함수형 기법과 라이브러리로 타입 흐름 유지하기

라이브러리들에서 제공되는 기법들은 타입스크립트와 조합하여 사용하면 훨씬 효율적으로 사용할 수 있다.  
타입 정보가 그대로 유지되면서 **타입 흐름(flow)이 계속 전달**되도록 하기 때문이다.  
하지만 직접 루프에 구현하게 되면 타입 체크에 대한 관리도 지속적으로 해야 한다.  
```typescript
    // 절차형
    const csvData = '...';
    const rawRows = csvData.split('\n');
    const headers = rawRows[0].split(',');

    const rows = rawRows.slice(1).map(rowStr => {
        const row = {};
        rowStr.split(',').forEach((val, j) => {
            row[headers[j]] = val;
         // ~~~~~~~~~~~~~~~ '{}' 형식에서 'string' 형식의 매개변수가 포한됨
         //                 인덱스 시그니처를 찾을 수 없습니다.
        });

        return row;
    });
```
```typescript
    // 함수형
    const rows = rawRows.slice(1)
        .map((rowStr) => rowStr.split(",")
            .reduce((row, val, i) => ((row[headers[i]] = val), row), {})
                                   // ~~~~~~~~~~~~~~~ '{}' 형식에서 'string' 형식의 매개변수가 포한됨
                                   //                 인덱스 시그니처를 찾을 수 없습니다.
        );
```
```typescript
    // 로대시 함수
    import _ from 'lodash';
    const rows = rawRows.slice(1)
        .map(rowStr => _.sipObject(headers, rowStr.split(',')));
        // 타입이 _.Dictionary<string>[]
```
이처럼 라이브러리를 사용하면 코드를 짧게 작성할 수 있다.  
하지만 서드파티 라이브러리를 기반으로 코드를 짧게 줄이는 데 시간이 많이 걸린다면, 사용하지 않는 것이 좋다.  

#### ✔️ 그러나 같은 코드를 작성한다면 서드파티 라이브러리를 사용하는 것이 무조건 유리하다.

타입 정보를 참고하며 작업할 수 있기 때문에 서드파티 라이브러리 기반으로 바꾸는 데 시간이 단축된다.  
  
절차형 버전과 함수형 버전은 같은 오류를 발생한다.  
`{}` 타입으로 `{[column: string]: string}` 또는 `Record<string, string>`을 제공하면 오류가 해결된다.  
하지만 **로대시 버전은 별도의 수정 없이도 타입 체커를 통과**한다.  
  
`Dictionary`는 로대시의 타입 별칭이다.  
`Dictionary<string>`은 `{[key: string]: string}` 또는 `Record<string, string>`과 동일하다.  

#### ⭐ 로대시 버전은 타입 구문이 없어도 `rows`의 타입이 정확하다 ⭐

```typescript
    interface BasketballPlayer {
        name: string;
        team: string;
        salary: number;
    }
    declare const reosters: {[team: string]: BasketballPlayer[]};
```
```typescript
    let allPlayers = [];
     // ~~~~~~~~~ 'allPlayers' 변수는 형식을 확인할 수 없는 경우
     //           일부 위치에서 암시적으로 'any[]' 형식입니다.
    
    for (const players of Object.values(rosters)) {
        allPlayers = allPlayers.concat(players);
                  // ~~~~~~~~~~ 'allPlayers' 변수에는 암시적으로
                  //             'any[]' 형식이 포함됩니다.
    }
```
루프를 사용해 단순 목록을 만들려면 배열에 `concat`을 사용해야 한다.  
하지만 해당 코드는 동작이 되지만 타입 체크가 되지 않는다.  
```typescript
    let allPlayers: BasketballPlayer[] = [];

    for (const players of Object.values(rosters)) {
        allPlayers = allPlayers.concat(players);        // ok
    }
```
``` typescript
    const allPlayers = Object.values(rosters).flat();
```
이러한 오류는 `allPlayers`에 타입 구문을 추가하면 해결되며, 더 나은 해결법은 `Array.prototype.flat`을 사용하는 것이다. 
`flat` 메서드는 다차원 배열을 평탄화해주며 타입 시그니처는 `T[][] => T[]`와 같은 형태이다.  
  
```typescript
    const teamToPlayers: {[team: string]: BasketballPlayer[]} = {};
    for (const player of allPlayers) {
        const { team } = player;
        teamToPlayers[team] = teamToPlayers[team] || [];
        teamToPlayers[team].push(player);
    }

    for (const players of Object.values(teamToPlayers)) {
        players.sort((a, b) => b.salary - a.salary);
    }

    // 함수형
    const bestPaid = Object.values(teamToPlayers).map(players => players[0]);
    bestPaid.sort((playerA, playerB) => playerB.salary - playerA.salary);
    console.log(bestPaid);
```
```typescript
    // 로대시
    const bestPaid = _(allPlayers)
        .groupBy(player => player.team)
        .mapValues(players => _.maxBy(players, p => p.salary)!)
        .values()
        .sortBy(p => -p.salary)
        .value()                // 타입이 BasketballPlayer[]
```
로대시로 작성할 경우 코드 길이가 절반으로 줄었고, 보기도 깔끔하다.  
또한 로대시와 언더스코어의 개념인 *체인*을 사용함으로써 더 자연스러운 순서로 일련의 연산을 작성할 수 있다.  
  
`_(v)`는 값을 **래핑(wrap)**하고, `.value()`는 **언래핑(unwrap)**한다.  
래핑된 값은 타입을 보기 위해 체인의 각 함수 호출을 조사할 수 있으며, 결과는 항상 정확하다.  

> ### 🤔 내장된 `Array.prototype.map` 대신 `_.map`을 사용하는 이유는 무엇일까?
> 한 가지 이유는 콜백을 전달하는 대신 **속성의 이름**을 전달할 수 있기 때문이다.  
> 타입스크립트 타입 시스템이 정교하기 때문에 다양한 동작을 정확히 모델링할 수 있다.  
> 함수 내부적으로는 문자열 리터럴 타입과 인덱스 타입의 조합으로만 이루어져 있기 때문에 타입이 자연스럽게 도출된다. *(Item 14 참고)*  

라이브러리들은 함수 호출 시 매개변수 값을 건드리지 않고 매번 새로운 값을 반환함으로써, 새로운 타입을 안전하게 반환한다. *(Item 20 참고)*  
타입스크립트의 많은 부분이 자바스크립트 라이브러리의 동작을 정확히 모델링하기 위해서 개발되었다.  
그러므로 이를 잘 활용해야지 타입스크립트의 원래 목적을 달성할 수 있다.  

## 📝 요약
- 타입 흐름을 개선하고, 가독성을 높이고, 명시적인 타입 구문의 필요성을 줄이기 위해 직접 구현하기보다는 내장된 함수형 기법과 유틸리티 라이브러리를 사용하는 것이 좋다.