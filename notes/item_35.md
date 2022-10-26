# Item 35. 데이터가 아닌, API와 명세를 보고 타입 만들기

파일 형식, API, 명세 등은 타입을 직접 작성하지 않고 **자동으로 생성**할 수 있다.  
특히, 예시 데이터가 아니라 **명세를 참고해 타입을 생성**하기 때문에, 사용자가 실수를 줄일 수 있다.  
  
```typescript
    function calculateBoundingBox(f: Feature): BoundingBox | null {
        let box: BoundingBox | null = null;

        const helper = (coords: any[]) => {
            // ...
        };

        const { geometry } = f;
        if (geometry) {
            helper(geometry.coordinates);
        }

        return box;
    }
```
`Feature` 타입은 명시적으로 정의된 적이 없다.  
`foucusOnFeature` 함수 예제를 사용하여 작성할 수 있지만 공식 *GeoJson* 명세를 사용하는 것이 효과적이다.  
```
    npm install --save-dev @types/geojson
```
```typescript
    import { Feature } from 'geojson';

    function calculateBoundingBox(f: Feature): BoundingBox | null {
        let box: BoundingBox | null = null;

        const helper = (coords: any[]) => {
            // ...
        };

        const { geometry } = f;
        if (geometry) {
            helper(geometry.coordinates);
                         // ~~~~~~~~~~~
                         // 'Geometry' 형식에 'coordinates' 속성이 없습니다.
                         // 'GeometryCollection' 형식에 'coordinates' 속성이 없습니다.
        }

        return box;
    }
```
`geometry`에 `coordinates` 속성이 있다고 가정한 것이 오류의 원인이다.  
이러한 관계는 점, 선, 다각형을 포함한 많은 도형에서는 맞는 개념이다.  
그러나, *GeoJson*은 `GeometryCollection`일 수도 있으며, `GeometryCollection`은 `coordinates` 속성이 존재하지 않는다.  
  
해당 오류는 `GeometryCollection`을 **명시적으로 차단하는 방법**으로 해결할 수 있다.  
```typescript
    const { geometry } = f;
    if (geometry) {
        if (geometry.type === 'GeometryCollection') {
            throw new Error('GeometryCollections are not supported');
        }

        helper(geometry.coordinates);       // Ok
    }
```
그러나 타입을 차단하기보다는 모든 타입을 지원하는 방법이 더 좋은 방법이므로 **조건을 분기하는 방법**을 사용하는 것이 좋다.  
```typescript
    const geometryHelper = (g: Geometry) => {
        if (geometry.type === 'GeometryCollection') {
            geometry.geometries.forEach(geometryHelper);
        } else {
            helper(geometry.coordinates);       // Ok
        }
    };

    const { geometry } = f;
    if (geometry) {
        geometryHelper(geometry);       // Ok
    }
```
### ✔️ 명세를 기반으로 타입을 작성한다면 사용 가능한 모든 값에 대해서 작동한다는 확신을 가질 수 있다.  
  
API 호출에도 비슷한 고려 사항들이 적용되며, API의 명세로부터 타입을 생성할 수 있다면 그렇게 하는 것이 좋다.  
특히 *GraphQL*처럼 자체적으로 타입이 정의된 API에서 잘 동작한다.  
  
*GraphQL* API는 타입스크립트와 비슷한 타입 시스템을 이용하여, 가능한 모든 **쿼리**와 **인터페이스**를 명세하는 스키마로 이루어진다.  
```typescript
    // GitHub GraphQL API query
    query {
        repository(owner: "Microsoft", name: "TypeScript") {
            createAt
            description
        }
    }

    // GitHub GraphQL API 결과
    {
        "data": {
            "repository": {
                "createAt": "2014-06-17T15:28:39Z",
                "description":
                    "TypeScript is a supreset of JavaScript that compiles to JavaScript."
            }
        }
    }
```
*GraphQL*의 장점은 특정 쿼리에 대한 타입스크립트 타입을 생성할 수 있다는 점이다.  
```typescript
    query getLicense($owner: String!, $name: String!) {
        repository(owner: $owner, name: $name) {
            description
            licenseInfo {
                spdxId
                name
            }
        }
    }
```
`$owner`와 `$name`은 타입이 정의된 *GraphQL* 변수이다.  
`String`은 *GraphQL*의 타입이며, 타입스크립트에서는 `string`이 된다. (Item 10 참고)  
타입스크립트에서 `string` 타입은 null이 불가능하지만 *GraphQL*의 `String` 타입에서는 가능하므로 타입 뒤의 `!`는 null이 아님을 명시한다.  
  
*GraphQL* 쿼리를 타입스크립트로 변환시키는 도구 중에는 **Apollo**가 있다.
> ### Apollo 란?
> **Apollo**는 *GraphQL*의 클라이언트 라이브러리 중 하나로 상태 관리 플랫폼이며, React, Vue, Angular를 모두 지원한다.

쿼리에서 타입을 생성하려면 *GraphQL* 스키마가 필요하며, *Apollo*는 api.github.com/graphql로부터 스키마를 얻는다.
```typescript
    export interface getLicense_repository_licenseInfo {
        __typename: "License";
        /** Short identifier specified by <https://spdx.org/licenses> */
        spdxId: string | null;
        /** The license full name specified by <https://spdx.org/licenses> */
        name: string;
    }

    export interface getLicense_repository {
        __typename: "Repository";
        /** The description of the repository. */
        description: string | null;
        /** The license associated with the repository */
        licenseInfo: getLicense_repository_licenseInfo | null;
    }

    export interface getLicense {
        /** Lookup a given repository by the owner and repository name. */
        repository: getLicense_repository | null;
    }

    export interface getLicenseVariables {
        owner: string;
        name: string;
    }
```
#### ✔️ CheckPoint
- 쿼리 매개변수(getLicenseVariables)와 응답(getLicense) 모두 인터페이스가 생성되었다.
- null 가능 여부는 스키마로부터 응답 인터페이스로 변환되었다. `repository`, `description`, `licenseInfo`, `spdxId` 속성은 null이 가능한 반면, `name`과 쿼리에 사용된 변수들은 그렇지 않다.
- 편집기에서 확인할 수 있도록 주석은 *JSDoc*으로 변환되었다. (Item 48 참고) 이 주석들은 *GraphQL* 스키마로부터 생성되었다.

이처럼 타입은 단 하나의 원천 정보인 *GraphQL* 스키마로부터 생성되기 때문에 **타입과 실제 값이 항상 일치**한다.  
만약 명세 정보나 공식 스키마가 없다면 *quicktype*과 같은 도구를 사용할 수 있다.  
하지만 그러한 경우 생성된 타입이 실제 타입과 일치하지 않을 수 있다는 점을 주의해야 한다.  
  
## 📝 요약
- 코드의 구석 구석까지 타입 안전성을 얻기 위해 API 또는 데이터 형식에 대한 타입 생성을 고려해야 한다.
- 데이터에 드러나지 않는 예외적인 경우들이 문제가 될 수 있기 때문에 데이터보다는 **명세**로부터 코드를 생성하는 것이 좋다.
